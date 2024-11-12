See 'Releases' section for downloads.

# Docker Support

We provide a Docker container with the Valida LLVM and Rust toolchains already installed.
Docker is the only supported method of running on platforms other than x86 Linux.

```bash
# Download the container
docker pull ghcr.io/lita-xyz/llvm-valida-releases/valida-build-container:v0.5.0-alpha

cd your-valida-project

# Enter the container:
docker run --platform linux/amd64 --entrypoint=/bin/bash -it --rm -v $(realpath .):/src ghcr.io/lita-xyz/llvm-valida-releases/valida-build-container:v0.5.0-alpha

# You are now in a shell with the valida rust toolchain installed!
```

# Usage instructions

## Entering the Valida shell

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
$ git clone https://github.com/lita-xyz/fibonacci.git
```

2. `cd` into the project template:

```bash
$ cd fibonacci
```

3. Enter the Valida shell:

```bash
$ valida-shell
```

4. Build the project:

```
valida> cargo +valida build
```

5. Run the code (taking input from `stdin`):

```
valida> valida run --fast target/delendum-unknown-baremetal-gnu/debug/fibonacci log
```

6. Prove the execution (taking input from `stdin`):

```
valida> valida prove target/delendum-unknown-baremetal-gnu/debug/fibonacci proof
```

7. Verify the proof:

```
valida> valida verify target/delendum-unknown-baremetal-gnu/debug/fibonacci proof
```

## Writing Rust programs to run on Valida

The Valida Rust compiler can currently compile in `no_std` mode, I.E. it cannot yet provide access to the `std` library, but can compile Rust programs which only use functionality contained within `core`, and which are annotated as `#![no_std]`.

We do not (yet) support a main function signature that takes any arguments, so it's not possible to follow the normal method of specifying a main function in a `#![no_std]` program. The following is a demonstration of a simple program that shows how the main function must be declared instead:

```Rust
#![no_main]

valida_rs::entrypoint!main(main);

#[no_mangle]
fn main() {
   ...
}
```

For a starting point to build a project using the Rust Valida toolchain, please take a look at
[the template project](https://github.com/lita-xyz/fibonacci). You can clone this repo and use
it as a starting point for your project.

The template project depends on [the valida-rs crate](https://github.com/lita-xyz/valida-rs). This contains a macro for generating an entry point, and some custom versions of standard library functions.

For projects with dependencies on `io` or `rand`, make sure your `main` and `Cargo.toml` include the code in this template. Also, make sure you have the same `.cargo/config.toml` in your project. If you want to build the project not targeting Valida, remove the `[build]` section in `.cargo/config.toml` and `cargo` will build the project targeting the host machine, unless otherwise specified.

We edited some functions to make them compatible with the Valida VM. When using these, the default Rust functions won't work. We call the Valida version with the `entrypoint::` prefix.

* `io`: Valida only supports standard `io` to the extent of `stdin` and `stdout`. To use `println` in Valida, one needs to call `entrypoint::io::println` as in `my-project`. A better `io` library will be added later.
* `rand`: to ensure the VM can prove the calculation of a given random number, we use our own function to generate a random byte with a specific seed.

These implementations are in `valida-rs/src/io.rs` and `valida-rs/src/rand.rs`. 

## Compiling and running C programs

To enter the Valida shell, run:

```bash
valida-shell
```

See [the lita-xyz/valida-c-examples repo on Github](https://github.com/lita-xyz/valida-c-examples) for some examples of C programs which can be compiled and run on Valida. Here is an example C program from this repo, called `cat.c`:

```c
const unsigned EOF = 0xFFFFFFFF;

int main() {
    unsigned c = 0;
    while (1) {
        c = __builtin_delendum_read_advice();
        if (c == EOF) {
            break;
        } else {
            __builtin_delendum_write(c);
        }
    }
}
```

To compile, for example, the `cat.c` example, from within the Valida shell:

```bash
clang -target delendum ./cat.c -o cat
valida run cat log
```

Once running, the cat example will wait for input. After you are done providing input, press `ctrl+D`. The program should echo back what you wrote, writing its output to log.

Compiling and running the other examples follows the same procedure, substituting `$NAME` for the name of the example:

```bash
clang -target delendum ./examples/${NAME}.c -o ${NAME}
valida run ${NAME} log
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
 * Code which assumes that memory is initialized to zero might not work properly. This includes `realloc` in the C standard library. This will be resolved by updating the VM to treat uninitialized memory as having a value of zero.
 * The compiler might emit incorrect code for some 64-bit arithmetic operations. This will be resolved by adding appropriate tests and fixing any issues that come up.
