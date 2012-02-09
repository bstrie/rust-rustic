# Rustic

This is Rustic[1], the supremely lazy REPL[2] for the [Rust programming language](https://github.com/mozilla/rust).

## Features

 * Persistent command history
 * Readline support
 * Easy logging
 * Works with any and all builds of `rustc`

## Platforms

Tested on Linux. Should work on Mac and Windows with a few tweaks (readline support unavailable on Windows).

## Installation

The file you want to be executing is in the `build` directory in the Git repository (the `.rc` and `.rs` files are just there to appease Cargo). If you install it via Cargo (`cargo install rustic`), Rustic will be in `~/.cargo/bin`. 

Rustic requires Python 3 to run. Yes, Rust itself only requires Python 2.6, but Python's subprocess library changed substantially between 2.6 and 2.7, and as long as we're introducing version incompatibilites we may as well go for the gusto.

(Why not implement it in Rust, you ask? Because as a tool for learning the language, it's not very useful for a REPL to have constant breakage due to the rapidity of language development and inevitable syntax changes. (Also, I needed a REPL before I could even begin to learn Rust.))

Also, naturally, Rustic requires a functioning version of `rustc`.

## Usage

At the `Input:` prompt, enter as many newline-separated commands as you like. On a readline-enabled platform, use the arrow keys to cycle through previously entered commands. Entering a blank line will cause all the preceding lines to be evaluated and their output printed. If there are no errors in a batch of evaluated lines, those lines are remembered for subsequent evaluation passes. For example:

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

What's really happening here is that Rustic is just remembering all the commands you've entered, recompiling the lot of it with every evaluation pass, and running the program anew. This means that if you evaluate commands with side effects, these will recur with each evaluation:

    Input:
    log(error, "side effects!");
    
    Output:
    rust: "side effects!"
    
    Input:
    let b = "I sure hope there are no side effects lurking about";
    
    Output:
    rust: "side effects!"

Note that using the `?expr` syntax does *not* cause the logging statement to recur in future evaluations. You can clear Rustic's remembered commands by typing an `EOF` character at the input prompt (`^D` on Unix).

If you don't want your commands to be saved for future evaluation passes, enter `?` on its own line to order Rustic to discard all subsequent commands in that batch after the initial evaluation:

    Input:
    let a = 2;
    ?
    let b = 3;
    ?a+b
    
    Output:
    rust: 5
    
    Input:
    ?a+b
    
    Compiler error:
    .rustic.scratch.rs:4:13: 4:14 error: unresolved name: b
    .rustic.scratch.rs:4 log(error, a+b);
                                      ^

Use `^C` to quit Rustic. You will be presented with the option to delete your scratch files; if you'd like to manually inspect your most recent `.rs` file and compiled executable, enter `n` to preserve them. They will be saved as hidden files in the directory in which Rustic was invoked.

I haven't really tried to push the limits of what this (admittedly lame) approach to a REPL can do. However, because it's just delegating all the heavy lifting to `rustc`, any program that compiles normally ought to work as expected:

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
