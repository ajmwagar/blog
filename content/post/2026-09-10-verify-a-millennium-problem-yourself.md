---
title: "Verify a Millennium Prize Problem Yourself (in an Afternoon)"
author: Avery Wagar
date: 2026-09-10T00:00:00-07:00
draft: true
description: "How to independently verify the 2026 Navier-Stokes and Euler Lean proofs yourself: elan, lake, a cloud box, and an afternoon. No math PhD required — that's the whole point."
keywords: ["Lean 4", "elan", "lake", "formal verification", "Navier-Stokes", "Millennium Prize", "tutorial", "self-verify"]
categories: ["Formal Methods", "Tutorial"]
tags: ["lean", "verification", "millennium-prize", "tutorial", "rustup-for-math"]
toc: true
---

![Terminal running lake build with green checkmarks](/img/ns-verify/terminal-lake-build.png)

Two weeks ago, "I independently verified the solution to a Millennium Prize
Problem" was a sentence only a handful of people on Earth could say. This week
anyone with a cloud account and an afternoon can say it, and it's worth saying
precisely because it means nothing about *us* and everything about the math.

Here's the full walkthrough. I ran this on a cloud VM; my main workstation is
a micro VM with 2 GB of RAM that would OOM before Mathlib finished unpacking,
so if I can do this, your laptop-adjacent server can.

## What you're actually verifying

Quick orientation, because the vocabulary is doing a lot of work:

- **Lean 4** — a programming language where you write proofs instead of
  (well, alongside) programs. Its *kernel* is a tiny checker that accepts
  nothing but airtight derivations from a fixed set of axioms.
- **Mathlib** — Lean's community math library. ~1.5M lines of already-proved
  mathematics: calculus, measure theory, functional analysis. Your proofs can
  cite it the way Rust code cites std.
- **`lake`** — Lean's build tool (think `cargo`). `lake build` compiles and
  *kernel-checks* every file.
- **`elan`** — Lean's toolchain manager (think `rustup`). The repo's
  `lean-toolchain` file pins the exact compiler version, and elan fetches it.

The two repos we're checking:

- [openai/NavierStokesAndEuler](https://github.com/openai/NavierStokesAndEuler) —
  the Clay Millennium alternatives (C)+(D) for Navier–Stokes, plus unforced
  3D Euler blowup. ~2,500 Lean files.
- [tristanbuckmaster/fluid_lean](https://github.com/tristanbuckmaster/fluid_lean) —
  forced blowup for 3D Euler, Boussinesq, IPM. ~3,700 Lean files, some of them
  machine-generated interval-arithmetic certificates tens of megabytes wide.

## Materials

- A Linux box with **4+ cores, 32 GB RAM** (64 if you're nervous), **~40 GB disk**.
  A spot instance costs a few dollars for the whole afternoon. GitHub's free
  CI runners (4 vCPU/16 GB) also work for the OpenAI repo — that's exactly how
  [our verification repo](https://github.com/FuturePresentLabs/ns-verify) runs it.
- 10 minutes of setup, 1–7 hours of waiting (cache-dependent), zero math.

## Step 1 — Install elan

```sh
curl -sSfL https://elan.lean-lang.org/elan-init.sh | sh -s -- -y
export PATH="$HOME/.elan/bin:$PATH"
```

That's Lean installed. It reads each project's `lean-toolchain` file and
fetches the exact compiler the authors used. If you've used `rustup`, you
already know this dance.

## Step 2 — Clone at a pinned commit

```sh
git clone https://github.com/openai/NavierStokesAndEuler
cd NavierStokesAndEuler
git checkout 8937a8f4cbc7abaab5e9e97d1cc7f5d2319d9538
```

Why pin? Because the claim you're verifying is "commit X compiles," not
"whatever the branch looks like when you read this." We verified OpenAI's repo
at `8937a8f` and Buckmaster's `fluid_lean` at `d012468` — the SHAs go in your
verdict at the end.

## Step 3 — Get the Mathlib cache (the difference between an afternoon and a day)

```sh
lake exe cache get!
```

Mathlib from source is a full day of compiling. The Mathlib team publishes
pre-built artifacts matched to exact toolchains, so if the repo's toolchain is
a released Lean version, this step downloads several GB of pre-verified oleans
in minutes. If the repo pins an RC toolchain (both of ours do, of course),
the cache lookup misses and you compile Mathlib from source — that's the
multi-hour phase, it's normal, and it's still kernel-checked at the end.

## Step 4 — Build (this is the verification)

```sh
lake build 2>&1 | tee build.log
```

Then wait. This is the entire verification. No configuration, no decisions,
no "do you trust this flag." Every proof obligation in every file either
re-derives from the kernel's axioms, or the build stops with an error and
the whole thing fails loudly.

Exit code zero means: every theorem in the repository is now proven on *your
machine*, from *your* copy of the kernel, independent of the authors, the
labs, the universities, and me.

## Step 5 — The axiom audit

A green build proves the repo's theorems follow from Lean's axioms plus
whatever the authors *declared* as axioms. So check the declarations:

```lean
import NavierStokes
#print axioms NavierStokes.Comparator.navier_stokes_breakdown_R3
#print axioms NavierStokes.Comparator.navier_stokes_breakdown_periodic
```

(`lean --run audit.lean`, or `lake env lean audit.lean`.)

You want exactly:

```
'NavierStokes.Comparator.navier_stokes_breakdown_R3' depends on axioms: [propext, Classical.choice, Quot.sound]
```

Those three are Lean's standard logical axioms — every Mathlib theorem
reports them. What you must *not* see is `sorryAx` (Lean's "this proof is
unfinished" marker) or any custom axiom. Our audits cover ten headline
theorems across both repos — four in OpenAI's (including both Clay
alternatives and the unforced Euler breakdown) and six in Buckmaster's — and
every one reports exactly the standard trio. The only `sorry`s in either
repository live in deliberate challenge files that exist to be filled in.

For Buckmaster's repo the import root differs per project
(`EulerBlowup.Num.Cert.FinalPrime` is one); the exact audit commands and full
outputs are committed at
[FuturePresentLabs/ns-verify](https://github.com/FuturePresentLabs/ns-verify).

## Step 6 — Write the verdict

Three lines, in public, with the log attached:

```
VERDICT: BUILD_OK
commit: <sha>, toolchain: <version>, jobs: N, wall time: T
axioms: [propext, Classical.choice, Quot.sound] for all headline theorems
```

If you get a failure instead — publish that too, immediately. An honest
failure report from a nobody is worth more than a green checkmark from
anyone with an incentive. That's the whole epistemology.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| OOM during build | `decide +kernel` certificate files are memory-hungry | Bigger box, or build with `-j2` |
| Cache miss, 6h build | Repo pins an RC toolchain | Let it run; it's not hung |
| One job stuck for 30+ min | You're on a multi-MB certificate file | Normal; wait |
| `lake: unknown command` | elan not on PATH in this shell | `export PATH="$HOME/.elan/bin:$PATH"` |

## Why bother

Because the interesting thing this week isn't that a machine can do
Millennium-Prize mathematics. It's that for the first time in history, the
result arrives with its own verifier attached — and the verifier doesn't
care who you are. A machine shop in Seattle and a Fields medalist who run
the same commands get the same answer, from the same kernel, and neither of
us had to trust the other.

That's new. Math used to be the discipline where trust was earned over
decades of peer review. Now it's the discipline where trust is a build log.
Run it yourself and you're part of that.

Terence Tao put the priority ordering well the same week: the competition
that matters is being *"the first to announce a new mathematical insight,"*
not the first to announce a solution. Verification doesn't compete with that
work — it protects it. When the announcement race gets cheap, the expensive
and valuable thing left is knowing *why* the proof works. That part still
belongs to the mathematicians; the rest of us can check the arithmetic.

*(If you want the pre-configured CI version instead of the DIY tour, it lives
at [FuturePresentLabs/ns-verify](https://github.com/FuturePresentLabs/ns-verify)
— badge, logs, pinned SHAs, and the honesty section included.)*
