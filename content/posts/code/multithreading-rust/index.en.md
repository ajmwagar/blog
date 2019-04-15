---
title: Fearless Multi-threading & Parallelism with Rust
author: Avery Wagar
date: 2019-04-15
draft: false
toc: false
tags:
  - Rust
  - Programming
  - Multi-threading
  - Concurrency
---

Multi-threading, Concurrency, and Parallelism are hard and can be scary. With problems like data-races and worrying about thread safety, it can be easy to stay away from them altogether. Luckily for us, Rust has our backs when it comes to thread safety and all the other perils that come our way when building Multi-threaded code. In-fact Rust's compiler will not let us __compile__ code that has a potential data-race in it. Therefore we are free to write all of our multi-threaded code without fear.

## Multi-threading

There are multiple ways that Rust can help us prevent data-race and enforce thread safety. 

### Arc & Mutex

One way that Rust provides helps prevents against data-races is to use `Arcs` and `Mutexes` when sharing data across threads. For example, here we have an integer that we want multiple threads to mutate. We can protect the data and other threads by using a Mutex which ensures our data is not poisoned, more about that [here](https://doc.rust-lang.org/std/sync/struct.Mutex.html#poisoning). 

```rust
// Import the required types
use std::sync::{Arc, Mutex};
use std::thread;

const N: usize = 10;

fn main() {
  // Here we're using an Arc to share memory among threads, 
  // and the data inside the Arc is protected with a mutex.
  let safe_data = Arc::new(Mutex::new(0));

  println!("Data: {:?}", safe_data); // Mutex: { data: 0 }

  // Create an array of thread handles 
  let handles = (0..N)
    .into_iter()
    .map(|_| {
      let data = Arc::clone(&safe_data);
      thread::spawn(move || {
        // Unlock the mutex so the thread may mutate the data
        // We unwrap() the return value to assert that we are not       
        // expecting threads to ever fail while holding the lock.
        let mut data = data.lock().unwrap();

        // Mutate the data
        *data += 1;

        // the lock is unlocked here when `data` goes out of scope.
      })
    })
    .collect::<Vec<thread::JoinHandle<_>>>();  

  // Wait for threads to finish mutating safe_data
  for thread in handles {
    thread.join().unwrap();
  }

  // Print out mutated data
  println!("Data: {:?}", safe_data); // Mutex: { data: 10 } or N
}
```

In the example above we safely mutate an integer across threads, this can be applied with larger data types, however, for event-based interaction across threads I recommend using an `MPSC` (Multi-Producer, Single Consumer) channel.

### MPSC Channels

`std::sync::mpsc` channels are a quick and safe way to share _events_ across threads.

For example, we can use the `mpsc` channels in a small banking program:

```rust
use std::sync::mpsc::channel;
use std::thread;

/// Transaction enum
enum Transaction {
    Widthdrawl(f64),
    Deposit(f64)
}

fn main() {
    // set the number of customers
    let n_customers = 10;

    // Create a "customer" and a "banker"
    let (customers, banker) = channel();

    let handles = (0..n_customers).into_iter().map(|i| {

        // Create another "customer"
        let customer = customers.clone();

        // Create the customer thread
        let handle = thread::spawn(move || {
            // Define Transaction
            let trans_type = match i % 2 {
                0 => Transaction::Deposit(i as f64 * 10.0),
                _ => Transaction::Widthdrawl(i as f64 * 5.0)
            };

            // Send the Transaction
            customer.send(trans_type).unwrap();
        });

        handle

    }).collect::<Vec<thread::JoinHandle<_>>>();

    // Wait for threads to finish
    for handle in handles {
        handle.join().unwrap()
    }

    // Create a bank thread
    let bank = thread::spawn(move || {
        // Create a value
        let mut bank: f64 = 10000.0;

        // Perform the transactions in order
        banker.into_iter().for_each(|i| {
            match i {
                // Subtract for Widthdrawls 
                Transaction::Widthdrawl(amount) => {
                    bank = bank - amount;
                },
                // Add for deposits
                Transaction::Deposit(amount) => {
                    bank = bank + amount;
                },
            }

            println!("Bank value: {}", bank);
        });
    });

    // Let the bank finish
    bank.join().unwrap();
}
```

The output of the above program would look something like this: 
```
Bank value: 10000
Bank value: 9995
Bank value: 10015
Bank value: 10000
Bank value: 9975
Bank value: 10035
Bank value: 10075
Bank value: 10040
Bank value: 10120
Bank value: 10075
```

As you can see `mpsc` channels are a powerful tool. Whether is sending events from a game loop to different threads, or high-frequency transactions, channels are a powerful tool when it comes to dealing with events and threads.


## Parallelism with Rayon

Now on to __Parallelism__. [Rayon](https://github.com/rayon-rs/rayon) is a Rust Library or "crate" that makes this dead simple.

Just add this to your `Cargo.toml`:

```toml
[dependencies]
rayon = "1.0"
```

And add this to your `src/main.rs`: 

```rust
use rayon::prelude::*;
```

Rayon comes with a strong parallel iterator API. That means that most of the time all you need to do to achieve __Parallelism__ is replacing your `.iter()` calls with `.par_iter()` and your `.into_iter()` calls with `.into_par_iter()`.

For example here is a Factorial function that is __not__ executing in parallel:

```rust
/// Compute the Factorial using a plain iterator.
fn factorial(n: u32) -> BigUint {
  (1..n + 1)
    .into_iter()
    .map(BigUint::from)
    .fold(BigUint::one(), Mul::mul)
}
```

And this is a Factorial function that __is__ executing in parallel:

```rust
/// Compute the Factorial using a parallel iterator.
fn factorial(n: u32) -> BigUint {
  (1..n + 1)
    .into_par_iter() // This line is different
    .map(BigUint::from)
    .reduce_with(Mul::mul).unwrap() // And this line is different
}
```

We only need to change __2 lines__ to change our function from sequential to parallel. There are plenty more examples of using Rayon in the `/rayon-demo` folder in their [git repository](https://github.com/rayon-rs/rayon/tree/master/rayon-demo).

## Conclusion

Rust, if applied in the right way, makes Multi-threading and Parallelism a breeze. Whether you use `Arcs` and `Mutexes` or `MPSC` channels, hopefully, you can proceed without fear when you have the urge to add multiple threads or parallel code into your next Rust project.
