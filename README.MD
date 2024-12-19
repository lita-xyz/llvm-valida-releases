See 'Releases' section for downloads.

# Docker-Based Installation

We provide a Docker container with the Valida LLVM and Rust toolchains already installed.
Docker is the only supported method of running on platforms other than x86 Linux.

### x86_64-based platforms

To install and use the toolchain via Docker on a 64-bit computer with an Intel-compatible chipset (x86_64), such as Intel- or AMD-based computers:

```bash
# Download the container
docker pull ghcr.io/lita-xyz/llvm-valida-releases/valida-build-container:v0.7.0-alpha-amd64

# cd your-valida-project

# Enter the container:
docker run --platform linux/amd64 -it --rm -v $(realpath .):/src ghcr.io/lita-xyz/llvm-valida-releases/valida-build-container:v0.7.0-alpha-amd64

# You are now in a shell with the valida rust toolchain installed!
```

### ARM64-based platforms

To install and use the toolchain via Docker on a 64-bit computer with an ARM64-compatible chipset (ARM64), such as Apple silicon-based computers:

```bash
# Download the container
docker pull ghcr.io/lita-xyz/llvm-valida-releases/valida-build-container:v0.7.0-alpha

# cd your-valida-project

# Enter the container:
docker run --platform linux/amd64 -it --rm -v $(realpath .):/src ghcr.io/lita-xyz/llvm-valida-releases/valida-build-container:v0.7.0-alpha

# You are now in a shell with the valida rust toolchain installed!
```

# Non-Docker-Based Installation (x86_64 Linux only)

For instructions for installation on certain x86_64 Linux platforms, such as Ubuntu 24.04 LTS or later, see [the release notes for the latest release](https://github.com/lita-xyz/valida-releases/releases/tag/v0.7.0-alpha).

# Usage instructions

## Entering the Valida shell (non-Docker only)

**This step applies only if you are using a non-Docker installation of the toolchain.**

Upon having installed the toolchain, the Valida shell should be on your `PATH`, and if you run `which valida-shell`, you should see:

```bash
$ which valida-shell
/usr/local/bin/valida-shell
```

If the result is something else, then either the installation did not complete successfully, or you had another `valida-shell` executable somewhere on your `PATH`.

If you run `valida-shell`, then you should see a shell prompt that reads `valida> `. You should then have on your `PATH` all of the executables from the Valida toolchain needed to follow the instructions below.

## Compiling and running Rust programs

For examples of how to build a Rust program which compiles and runs on Valida, see [lita-xyz/rust-examples on GitHub](https://github.com/lita-xyz/rust-examples). You can use any of these examples as a starting point for developing your own programs using the Valida toolchain. Here are steps for doing so:

1. Clone the project template:

```bash
git clone https://github.com/lita-xyz/fibonacci.git
```

2. `cd` into the project template:

```bash
cd fibonacci
```

3. Enter the Valida shell (skip this step if you are using the Docker toolchain):

```bash
valida-shell
```

4. Build the project:

```
cargo +valida build
```

5. Run the code (taking input from `stdin`):

```
valida run --fast target/valida-unknown-baremetal-gnu/debug/fibonacci log
```

6. Prove the execution (taking input from `stdin`):

```
valida prove target/valida-unknown-baremetal-gnu/debug/fibonacci proof
```

7. Verify the proof:

```
valida verify target/valida-unknown-baremetal-gnu/debug/fibonacci proof --claimed-output log
```

## Writing Rust programs to run on Valida

The Valida Rust compiler can currently compile in `no_std` mode, I.E. it cannot yet provide access to the `std` library, but can compile Rust programs which only use functionality contained within `core`, and which are annotated as `#![no_std]`.

We do not (yet) support a main function signature that takes any arguments, so it's not possible to follow the normal method of specifying a main function in a `#![no_std]` program. The following is a demonstration of a simple program that shows how the main function must be declared instead:

```Rust
#![no_main]

#[no_mangle]
fn main() {
   ...
}
```

For a starting point to build a project using the Rust Valida toolchain, please take a look at
[the template project](https://github.com/lita-xyz/fibonacci). You can clone this repo and use
it as a starting point for your project.

The template project depends on [the valida-rs crate](https://github.com/lita-xyz/valida-rs). This contains a macro for generating an entry point, and some custom versions of standard library functions.

### More examples in Rust

If you have installed the toolchain, see `/valida-toolchain/examples/rust` for some more examples of Rust Valida projects.

## Compiling and running C programs

To enter the Valida shell, run:

```bash
valida-shell
```

**You can skip the above step if you are using the Docker toolchain.**

If you have installed the toolchain, then see `/valida-toolchain/examples/c` for some examples of C programs which can be compiled and run on Valida. Here is an example C program from this repo, called `cat.c`:

```c
#include <stdio.h>

const unsigned EOF = 0xFFFFFFFF;

int main() {
    unsigned c = 0;
    while (1) {
        c = getchar();
        if (c == EOF) {
            break;
        } else {
            putchar(c);
        }
    }
}
```

To compile, for example, the `cat.c` example, from within the Valida shell:

```bash
clang -target valida ./cat.c -o cat
valida run --fast cat log
```

Once running, the cat example will wait for input. After you are done providing input, press `ctrl+D`. The program should echo back what you wrote, writing its output to log.

Compiling and running the other examples follows the same procedure, substituting `$NAME` for the name of the example:

```bash
clang -target valida ./examples/${NAME}.c -o ${NAME}
valida run --fast ${NAME} log
```

Some other examples that are provided in the `valida-c-examples` repo:

 * `reverse.c` will output its reversed input.
 * `checksum.c` will output a checksum, i.e., a sum of the characters, of its input.
 * `merkle-path.c` will verify an opening proof for a SHA256 binary Merkle tree
    * For an example proof you can use as input, see `examples/example-merkle-proof`
 * `sha256.c` will output a SHA-256 hash of the first 256 bytes of its input.
 * `sha256_32byte_in.c` will output the SHA-256 hash of a constant array of 32 bytes. This is used as a benchmark.

# Reporting issues

If you have any issues to report, please report them at [the llvm-valida-releases issue tracker](https://github.com/lita-xyz/llvm-valida-releases/issues). Please include the following elements in your bug report: what release version you encountered the bug on, steps to reproduce, expected behavior, and actual behavior.

# Known issues

 * The prover is unsound, which means that verifying a proof does not provide completely convincing evidence that the statement being proven is true. This will be resolved once some missing constraints are added.
 * There are some issues with standard I/O functions in Rust when it comes to the behavior of interactive programs. Sometimes, code behaves differently in Valida vs native code, such as sometimes needing ctrl+D to be pressed twice instead of once to signal end of input.
