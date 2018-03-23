# Overview

Reading the book < Netwok programeting with go >. I'm tring to make the golang code to rust.

Unfortuntely found few issues.

1. Rust doesn't have lookup host function, or rephase it, it doesn't include this function in the stable release. It used to have look_addr in 1.8, but depreated, the feature only for nightly build.

2. Rust with std::time, this module only works for sec, which will get the last seconds, it's not like golang to have native the current utc time, you have to use crat time to run with it.
3. Rust ARGS with if condition likes this

```
    match args.len() {
        2 => {
            let ip  = &args[1];
            let addr = IpAddr::V4(ip.parse().unwrap());
            println!("Your input {:?}\n", addr);
            if addr.is_ipv4() {
                println!("The address is {}", ip);
            } else {
                println!("Invalid address");
                help();
                return;
            };
        },
        _ => {
           help();
        }
```
