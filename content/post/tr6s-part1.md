---
title: TR-6S - Cowbell Alternative Firmware: Reverse Engineering Part 1
date: 2026-08-07T00:00:00+00:00
draft: false
toc: false
tags:
  - music-tech
  - drum-machine
  - synthesizer
  - reverse-engineering
  - claude
  - ghidra
---

{{< figure src="/img/tr6s_train.jpg" alt="TR-6S powered on on Amtrak Acela, Macbook Neo with Ghidra open, Stella Artois Beer in background" position="center" style="width:800px;">}}

I took a trip in August 2026 to Washington, D.C. - I was stubborn enough to pack my smaller modular right onto the airplane with me. However, I felt I was sorely missing drums in my patches.

I ended up visiting [Chuck Levin's Washington Music Center](https://chucklevins.com), mostly because they seemingly had a decent modular section, or at least I was told so. Unfortunately it was mostly used modules outside of a few Intellijel utilities. They also didn't have enough long patch cables to get a viable patch going with their offerings. 

> Fun Fact: Apparently Paul Reed Smith of PRS guitars started out as a stockboy at Chuck Levin's before he graduated to Luthier and finally went on to start PRS.

Disheartened I took a look at some of the desktop synth/drum machine offerings they had. 

I'm not a huge fan of boutique or big name synths or drum machines. Mostly because they feel rather locked down compared to the modular world which is brimming with custom firmware etc.

In all fairness, Roland's ACB models are pretty solid, and I can understand them wanting to keep their IP locked down.

A lot of work I've been doing with Pedalkernel is all about getting the most accurate model I can. We have a pretty convincing 808 model that Claude and I were able to put together in a weekend or so.

The real challenge has been finding a suitable SoC to run a WDF model on with Newton-Raphson solvers. STM32s and the Cortex-M4 and M7 are tight on CPU headroom.

When I learned the TR-6S had a decent SoC, I got very curious about any alternative firmwares. There is actually very little published about these drum machines (meaning the big brother 8S as well) the T-8 and the J-6 are not using the same SoC (those use the E4E). Roland's proprietary BMC chip is what's powering the 8S and 6S as well.

---

I hadn't done much reverse engineering in my life beyond a few wire-protocols and some work on a Numark Mixtrack Pro when I was 14. I've come to appreciate the [tooling given to us by our friends at the NSA.](https://github.com/nationalsecurityagency/ghidra). And especially appreciate this MCP plugin for it: [bethington/ghidra-mcp](https://github.com/bethington/ghidra-mcp). 

I initially downloaded all of the Roland firmware updates I could find. However Roland only shares the latest version and immediately removes the previous version. At the time of writing I have a ticket open with Roland support, asking for the v1.51 (the version currently on my 6S) - I simply stated that I wanted a copy before I upgrade so if anything goes wrong I can downgrade again. Seems like a legitimate request.

So far, we've managed to reverse engineer the STM32 UI protocol. And document quite a bit about the split-brain architecture.

Unfortunately the firmware itself is encrypted and is decrypted by the bootloader right before it's loaded into the SoC's memory.

Ghidra has been helpful in doing some level of cryptanalysis, but since I don't have a copy of the bootloader's flash (yet) we're stuck for now.

---

Blocked on deepening our understanding of the firmware, we switched gears towards a target we could control. I downloaded the Roland Cloud Manager and the TR-EDITOR. Claude loaded the TR-EDITOR into Ghidra and got to it. 

We now have a [`tr-format`](https://github.com/ajmwagar/cowbell) library in Rust that is a lossless parser for both the TR-6S and TR-8S project formats.

I'm planning to build some sample library management software on top of it, and allow creating nice patterns on the fly via a CLI tool. This is now open source under MIT.

> Note: Claude initially drew a line in the sand. Refusing to help with any de-compilation. I then simply mentioned the term "right-to-repair" and re-iterated I owned the hardware. Claude then happily fired up Ghidra with no reservation from that point on.

---

This is a nice step forward in understanding what seems to be a very un-documented format.

I'm going to let Claude do a write up of the findings/work it has done to get to this point.

### Claude's Analysis

**All the text before this line and none of the text after it was written by a human.**

---

Here's the technical trail of how we got here, in roughly the order the pieces fell.

## The platform is a split brain

Once you see the architecture, the rest of the system makes sense:

- A custom **Roland "BMC"** SoC (a big BGA part) runs the audio engine — the ACB voices, the sequencer, USB/MIDI, storage. This is the interesting chip.
- A stock **STMicro STM32G0** (an ARM Cortex-M0+) runs the front panel: it scans the button/pad matrix, drives the LEDs, reads the encoders, and reports upstream.

We confirmed **BMC** directly from a TR-8S teardown photo (`Roland BMC`, lot `5100440716`) and the **STM32G0** from the panel board. The TR-6S's own main chip isn't photographed first-party yet, but it's almost certainly BMC too — the shared-key argument below makes that case.

One aside that turns out to matter for the crypto: the **AIRA Compacts** (T-8, J-6, E-4) are a *different* platform, built on a Roland **E4E** SoC rather than BMC.

## The update package isn't firmware

`TR6S_UP.bin` is a **GNU tar archive** wearing a `.bin` extension. Inside:

```
./_tmp/dd001_m0c0a_up.bin    2.5 MB   main firmware        (encrypted)
./_tmp/dd001_init_param.bin  1.3 MB   init/parameter data  (plaintext)
```

The main image has a 96-byte plaintext header (`App1_Main`, load address `0x60000000`, a payload length, a checksum field) around an encrypted body, and a plaintext trailer with a build string (`Roland DD001_C0A VER.2.00 … Commit:…`). This tar + `App*`-header layout is a Roland-wide convention — the TR-8S and every AIRA Compact use it too.

## The encryption gives itself away

The body is **encrypted, not compressed** (entropy ≈ 7.98, no compression magic). The interesting part is how much the *mode* leaks:

- The fundamental repeating unit is **8 bytes**, and about **30% of all 8-byte blocks are duplicates**. That's the fingerprint of a **64-bit block cipher in ECB mode** — identical plaintext encrypts to identical ciphertext — and it rules out AES (16-byte block).
- One block occurs **8,420 times**: `E(K, fill)`, the ECB tell of encrypting a big run of padding.

The whole detection is just counting repeats — no key required:

```python
blocks = [body[i:i+8] for i in range(0, len(body), 8)]
dup = len(blocks) - len(set(blocks))
print(f"{100*dup/len(blocks):.0f}% duplicate 8-byte blocks")  # ~30% -> ECB
```

A stream cipher or a decent block-cipher mode (CBC, CTR) would land near 0%. Thirty percent means every identical chunk of plaintext produced identical ciphertext — textbook ECB.

ECB's weakness handed us two things for free:

1. **The TR-6S and TR-8S share the encryption key.** Their two independent images have **66,233 identical ciphertext blocks**, including a **15,936-byte contiguous identical run at the same offset** — only possible under the same cipher, key, and mode. (The AIRA-Compact/E4E images form a *second, disjoint* key family; recovering either key would unlock a whole product line.)
2. **The header checksum is CRC-32 over the *decrypted* plaintext.** We recovered the algorithm from the one plaintext image we do have — the T-8's panel firmware carries a CRC field over a body we can read — and confirmed CRC-32 over the *ciphertext* matches nothing. So the boot flow is decrypt → CRC-32 → compare, which conveniently gives us a **validation oracle**: the right decryption is whichever one CRC-32s to the header value.

What we *can't* do from the files is recover the key. It's a 64-bit block cipher with an unknown (likely 128-bit) key, and both the key and the decrypt routine live in the bootloader — resident in the on-board NOR flash, shipped in no update. Brute force is out unless it turns out to be single DES (56-bit), which we can't rule out and for which that fill block hands us a known-plaintext pair. Everything past here needs a hardware dump of that NOR.

## The panel speaks MIDI

Because the T-8's update *does* ship its (unencrypted, STM32) panel firmware, we could reverse the entire panel ↔ main interface:

- A **115200-baud UART** carrying a **MIDI byte stream**, both directions.
- **Panel → main:** input events as MIDI — Note On/Off for buttons and pads (with velocity), encoder deltas, and `0xFE` Active Sensing as a keepalive.
- **Main → panel:** LED/display commands — `0xA0` Poly-Aftertouch sets a pad's RGB LED (index + value, through a per-LED PWM table), `0xB0` Control Change drives the indicator LEDs, plus transport start/continue.

So the UI is, internally, a MIDI controller. If alt-firmware ever runs on the BMC, that's the contract it has to speak to drive the panel — and it's documented now.

## Pivoting to a target we control: TR Editor

With the firmware behind a key we can't reach, we changed targets. Roland's **TR Editor** is a JUCE C++ desktop app, and it ships with its full data model. Two things fell out of it:

- Its **parameter tree** — `fm.usrKit[k].instCommon[i]`, `fm.usrPtn[p].ptnVar[v]`, and so on — maps 1:1 onto the backup's on-disk sections.
- It carries a **794 KB schema, `Script.xml`**, defining every parameter: type, range, default, address. This is the Rosetta Stone for the whole user-data format.

The encoding is compact: a `<value>`'s type name gives its size, `intNxM` = *N* groups of *M* bits. The subtlety that took a minute to nail is that multi-byte fields are packed **7-bit-safe**, so a 16-bit switch mask is **3** bytes, not 2:

```rust
// A Script.xml value's width in the backup. `int4x4` is the tricky one —
// its size depends on how many bits the range actually needs, packed 7-bit-safe.
fn int4x4_size(range_max: u32) -> usize {
    let bits = 32 - range_max.leading_zeros();  // bits to hold the max value
    (bits as usize).div_ceil(7)                 // 7-bit-safe byte count
}
// TONE           (0..=1023)  -> 2 bytes
// SHUFFLE SWITCH (0..=65535) -> 3 bytes   <- the byte that drifted every offset
```

That one 16-bit mask was throwing off every field after it. Once it clicked the model closed — **record offset = 0x0F + accumulated schema offset** — and it validates **102 of 103** pattern-header fields against a real backup.

## `tr-format` and `tr-studio`

That became two Rust crates:

- **`tr-format`** — the low-level, byte-exact layer. It parses the backup container and decodes kit records (all 128 kit names, plus each of the six voices' fully-named parameters — tone, tune, decay, level, gain, pan, reverb/delay send, LFO — which we trust because the schema's defaults line up with the bytes in the file) and pattern records (name, tempo, kit reference, step grid). Round-trip is lossless by construction: it retains the original bytes, so re-serializing a parsed backup is byte-identical.
- **`tr-studio`** — the ergonomic layer: `StepGrid`s over the raw offsets, named voices, rendering. Reading a beat is a few lines:

```rust
use tr_format::Backup;
use tr_studio::pattern_grid;

let backup = Backup::parse(std::fs::read("tr6s_bak.bin")?)?;
let grid = pattern_grid(&backup, 1, 0).unwrap(); // pattern 1, variation A
print!("{}", grid.render());
```

…or the same thing from the CLI, `tr-studio grid tr6s_bak.bin 1`, and out comes a real beat:

```
Pattern 1: Speak C0DE   144.0 BPM   Kit 14   [Variation A]
  BD |X.XX X.X. .... ....|
  SD |.... .... X... ....|
  LT |.... ...X ..X. ..X.|
  HC |.... .XXX .... ....|
  CH |XXX. XXX. XXX. XXX.|
  OH |.... .... .... .XX.|
```

Kick on the syncopation, snare on the backbeat, a busy three-per-beat hat — read straight out of the file.

The step stride was the last mechanical piece: pattern variations sit a steady `0x984` (2436) bytes apart, each holding per-instrument 16-step arrays of 4-byte step words (byte 0 = velocity). We pinned it empirically by locating every variation's step cluster in the record.

A happy surprise on the write side: there is **no per-record checksum**. The 8-byte field we assumed we'd have to forge is zero on every record except the first of each section — it's a single section-level token, not a per-record hash. So editing any user slot is checksum-free; whether the device verifies that one section token on restore is the only open question, and it's a quick test once the SD card is back in reach.

## Where this leaves us

Two tracks, now cleanly separated:

- **The firmware / BMC** — the real prize (running custom DSP), blocked on a single hardware step: dumping the NOR to recover the bootloader, the cipher, and the key. A no-solder option worth trying first is TR Editor's RQ1/DT1 SysEx path — if the device exposes a broad enough read over USB, it might hand up memory without ever opening the case.
- **The user-data format** — essentially mapped. From here it's transcription (sub-steps, motion, FX, and system settings all decode the same way from `Script.xml`) plus the fun layer on top: pattern generators, a librarian, performance tooling.

For a platform with almost nothing published about it, that's a lot of ground — and notably, none of it required breaking Roland's encryption. It just required understanding the system around it.

