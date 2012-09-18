# Rustic 0.1

This is Rustic,<sup>[1]</sup> the worse-is-better REPL<sup>[2]</sup> for the [Rust programming language](https://github.com/mozilla/rust).

(To be more precise, Rustic is more of an incremental source file compositor and validator (ISFCV) than a REPL, but it's more-or-less identical in terms of use.)

Although I haven't had a chance to test it on platforms other than Linux, Rustic should Just Work (it's so dead simple, it would be hard not to). Please report any bugs encountered to the [issue tracker](https://github.com/bstrie/rust-rustic/issues) on Github.

## Major Features

 * Persistent evaluation history (with optional transience)
 * Readline support
 * Upload code (with evaluated output and errors) to Gist
 * Low-friction logging
 * Works with any and all builds of Rust (now 100% more true!)

## Platforms

Tested on Linux. Assumed to work on Mac and Windows.

## Installation

The `rustic` file itself is found in the `build` directory in the Git repository (the `.rc` and `.rs` files in the upper directory are just there to appease Cargo). If you install it via Cargo (`cargo install rustic`), Rustic will be in `.cargo/bin`.

Rustic requires Python 3 to run. Maybe I'll rewrite it in Rust once the language stabilizes a bit.

Also, naturally, Rustic requires a functioning Rust compiler, reachable via the command line.

In addition, Rustic is installed alongside [colorama](http://pypi.python.org/pypi/colorama), a Python library for platform-independent coloration of terminal output. However, Rustic doesn't require those files to operate--you can delete them, or simply move Rustic somewhere else, and the only effect will be the absence of colored output.

## Usage

At the `[Input]` prompt, enter as many newline-separated commands as you like. On a readline-enabled platform, use the arrow keys to cycle through previously entered commands (among other readline niceties, such as ^R for searching your history). Entering a blank line will cause all the preceding lines to be evaluated and their output printed. If there are no errors in a batch of evaluated lines, those lines are remembered for subsequent evaluation passes. For example:

    [Input]
    let foo = 'hello';
    
    [Compiler error]
    .rustic.scratch.rs:3:16: 3:16 error: unterminated character constant
    .rustic.scratch.rs:3     let foo = 'hello';
                                         ^
    
    [Input]
    let foo = "hello";
    
    [Input]
    ?foo
    
    [Output]
    rust: "hello"

As you can see, you can use `?expr` to insert a logging statement for `expr`.

What's really happening here is that Rustic is just remembering all the commands you've ever entered, recompiling the lot of it with every evaluation pass, and running the program anew. This means that if you enter commands with side effects, these will recur with each evaluation:

    [Input]
    log(error, "I'll be back!");
    ?"I won't!"
    
    [Output]
    rust: "I'll be back!"
    rust: "I won't!"
    
    [Input]
    let foo = "I sure hope there are no side effects lurking about";
    
    [Output]
    rust: "I'll be back!"

Note that, for the sake of convenience, using the `?expr` syntax does *not* cause the logging statement to recur in future evaluations. You can clear Rustic's evaluation list by typing an `EOF` character at the input prompt (`^D` on Unix) or by using the magic word `%clear`.

If you want to prevent some commands from being saved for future evaluation passes, enter `?` on its own line to order Rustic to discard all subsequent commands in that batch after the evaluation:

    [Input]
    let a = 2;
    ?
    let b = 3;
    ?a+b
    
    [Output]
    rust: 5
    
    [Input]
    ?a+b
    
    [Compiler error]
    .rustic.scratch.rs:5:17: 5:18 error: unresolved name: b
    .rustic.scratch.rs:5     log(error, a+b);
                                          ^

Use either `^C` or `%exit` to quit Rustic. You will be presented with the option to delete your scratch files; if you'd like to manually inspect your most recent `.rs` file and compiled executable, enter `n` to preserve them. They will be saved as hidden files in the directory in which Rustic was invoked.

I haven't really tried to push the limits of what this (admittedly lame) approach to a REPL can do. However, because it's just delegating all the heavy lifting to `rustc`, any program that compiles normally ought to work as expected:

    [Input]
    fn fac(n: int) -> int {
        let result = 1, i = 1;
        while i <= n {
            result *= i;
            i += 1;
        }
        ret result;
    }
    
    [Input]
    ?fac(5)
    ?fac(10)
    
    [Output]
    rust: 120
    rust: 3628800

Use `%help` to display a list of magic words, with brief descriptions:

    [Input]
    %help
    
    [Magic]
    Commands:
      %clear to reset the evaluation environment (equivalent to ^D)
      %exit to leave the interpreter (equivalent to ^C)
      %scratch to print your most recent scratch file
      %compmsg to enable/disable compiler messages (disabled by default)
      %putgist to upload your most recent scratch file and output to Gist

Perhaps the most thrilling of these is `%putgist`, which will automatically upload both your most recently evaluated scratch file and its output to Gist:

    [Input]
    import math;
    let x = 0x42;
    let y = 0b10100100;
    ?math::min(x, y)
    
    [Output]
    rust: 66
    
    [Input]
    %scratch
    
    [Magic]
    use std;
    import math;
    fn main() {
        let x = 0x42;
        let y = 0b10100100;
        // Transient:
        log(error, math::min(x, y));
    }

    [Input]
    %putgist
    
    [Magic]
    Scratch file pasted to https://gist.github.com/1786708

[1] **Rust** **i**nteractive, **c**ruddily

[2] **R**ather **E**gregiously **P**rimitive imp**L**ementation of a read/eval/print loop
