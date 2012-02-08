# Rustic

This is Rustic[1], a self-denigrating REPL[2] for the [Rust programming language](https://github.com/mozilla/rust).

## Major features

 * Can be halted via ^C

## Installation

Requires Python 3 to run. Yes, Rust only requires Python 2.6, but Python's `subprocess` API changed substantially between 2.6 and 2.7, and as long as we're introducing version incompatibilites we may as well go for the gusto.

(Why not implement it in Rust, you ask? Because as a tool for learning the language, it's not very useful for a REPL to have constant breakage due to the rapid pace of language development and unanticipatable syntax changes. (Also, I needed a REPL to learn Rust in the first place.))

## Usage

Please don't. Failing that, it's quite simple: once in the loop, enter as many newline-separated commands as you like. Entering a blank line will cause all the preceding lines to be evaluated and their output printed. If there are no errors in a batch of evaluated lines, those lines are remembered for subsequent evaluation passes. For example:

    Enter:
    let a = 'hello';
    
    .rustic.scratch.rs:3:10: 3:10 error: unterminated character constant
    .rustic.scratch.rs:3 let a = 'hello';
                                   ^
    
    Enter:
    ?a
    
    .rustic.scratch.rs:3:11: 3:12 error: unresolved name: a
    .rustic.scratch.rs:3 log(error, a);
                                    ^
    error: aborting due to previous errors
    
    Enter:
    let a = "hello";
    
    Enter:
    ?a
    
    rust: "hello"

As you can see, you can use `?expr` to insert a logging statement for `expr`. 

What's really happening here is that Rustic is just remembering all the commands you've entered, and recompiling the lot of it with every evaluation pass. This means that if you evaluate commands with side effects, these will be repeated at each evaluation:

    Enter:
    log(error, "side effects!");
    
    rust: "side effects!"
    Enter:
    let b = "I sure hope there are no side effects lurking about";
    
    rust: "side effects!"

Note that using the `?expr` syntax does *not* cause the logging statement to be remembered. You can clear Rustic's remembered commands by sending an EOF character (^D on Unix).

I haven't really tried to push the limits of what it can do, however, because it's just calling `rustc` with every pass, anything that would compile normally ought to be fine:

    Enter:
    fn fac(n: int) -> int {        
        let result = 1, i = 1;
        while i <= n {
            result *= i;
            i += 1;
        }
        ret result;
    }

    Enter:
    ?fac(5)
    
    rust: 120

[1] RUST Interactive, Cruddily

[2] Really Egregiously Poorly impLemented read/eval/print loop
