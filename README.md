# Rustic

This is Rustic[1], an incredibly lazy REPL[2] for the [Rust programming language](https://github.com/mozilla/rust).

## Features

 * Persistent command history
 * Readline support
 * Easy logging
 * Works with any and all builds of `rustc`

## Platforms

Tested on Linux. Should work on Mac and Windows with a few tweaks (readline support unavailable on Windows).

## Installation

Requires Python 3 to run. Yes, Rust only requires Python 2.6, but Python's subprocess API changed substantially between 2.6 and 2.7, and as long as we're introducing version incompatibilites we may as well go for the gusto.

(Why not implement it in Rust, you ask? Because as a tool for learning the language, it's not very useful for a REPL to have constant breakage due to the rapidity of language development and inevitable syntax changes. (Also, I needed a REPL before I could even begin learning Rust.))

## Usage

At the `Input:` prompt, enter as many newline-separated commands as you like. Entering a blank line will cause all the preceding lines to be evaluated and their output printed. If there are no errors in a batch of evaluated lines, those lines are remembered for subsequent evaluation passes. For example:

    Input:
    let foo = 'hello';
    
    Compiler error:
    .rustic.scratch.rs:5:12: 5:12 error: unterminated character constant
    .rustic.scratch.rs:5 let foo = 'hello';
                                     ^
    
    Input:
    let foo = "hello";
    
    Input:
    ?foo
    
    Output:
    rust: "hello"

As you can see, you can use `?expr` to insert a logging statement for `expr`. 

What's really happening here is that Rustic is just remembering all the commands you've entered, recompiling the lot of it with every evaluation pass, and running the program anew. This means that if you evaluate commands with visible side effects, these will be repeated at each evaluation:

    Input:
    log(error, "side effects!");
    
    Output:
    rust: "side effects!"
    
    Input:
    let b = "I sure hope there are no side effects lurking about";
    
    Output:
    rust: "side effects!"

Note that using the `?expr` syntax does *not* cause the logging statement to be remembered. You can clear Rustic's remembered commands by sending an `EOF` character (`^D` on Unix).

If you don't want your commands to be saved for future evaluation passes, enter `?` on its own line to order Rustic to discard all subsequent commands in that batch:

    Input:
    let a = 2;
    ?
    let b = 3;
    ?a+b
    
    Output:
    rust: 5
    
    Input:
    ?b
    
    Compiler error:
    .rustic.scratch.rs:4:11: 4:12 error: unresolved name: b
    .rustic.scratch.rs:4 log(error, b);
                                    ^

I haven't really tried to push the limits of what this approach can do. However, because it's just calling `rustc` with every pass, anything that would compile normally ought to be fine:

    Input:
    fn fac(n: int) -> int {
        let result = 1, i = 1;
        while i <= n {
            result *= i;
            i += 1;
        }
        ret result;
    }
    
    Input:
    ?fac(5)
    ?fac(10)
    
    Output:
    rust: 120
    rust: 3628800

[1] RUST Interactive, Cruddily

[2] Really Egregiously Poorly impLemented read/eval/print loop
