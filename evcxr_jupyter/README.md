# EvCxR Jupyter Kernel

[![Latest Version](https://img.shields.io/crates/v/evcxr_jupyter.svg)](https://crates.io/crates/evcxr_jupyter)

A [Jupyter](https://jupyter.org/) Kernel for the Rust programming language.

## Installation

### Linux (Debian/Ubuntu)

```sh
sudo apt install libzmq3-dev jupyter-notebook
cargo install evcxr_jupyter
evcxr_jupyter --install
```

### Mac OS X

install jupyter or jupyterlab (eg. via anaconda)

```sh
brew install zeromq pkg-config
cargo install evcxr_jupyter
evcxr_jupyter --install
```

### Windows

Note that Evcxr on Windows appears to be substantially slower than on other
platforms. We're not yet sure why.

* Install jupyter or jupyterlab (eg. via anaconda)
* Install ZMQ
  * Manually compiling zeromq 4.3.2 (using cmake + Visual Studio) has been
    reported to work.
  * [These
    instructions](https://github.com/erickt/rust-zmq/issues/208#issuecomment-380353800)
    might or might not help.

```sh
cargo install evcxr_jupyter
evcxr_jupyter --install
```

## Usage notes

* To see what variables you've got defined, type ":vars".
* Don't ask Jupyter to "interrupt kernel", it won't work. Rust threads can't be
  interrupted.
* If your code panics, all variables will be lost. You can optionally run
  `:preserve_vars_on_panic 1` to turn on preservation of variables. However note
  that this will slow down compilation. Also, only variables that either are not
  referenced by the code being run, or are Copy will be preserved.
* If your code segfaults (e.g. due to buggy unsafe code), aborts, exits etc, the
  process in which the code runs will be restarted. All variables will be lost.

## Custom output

The last expression in a cell gets printed. By default, we'll use the debug
formatter to emit plain text. If you'd like, you can provide a function to show
your type (or someone else's type) as HTML (or an image). To do this, the type
needs to implement a method called ```evcxr_display``` which should then print
one or more mime-typed blocks to stdout. Each block starts with a line
containing BEGIN\_EVCXR\_OUTPUT followed by the mime type, then a newline, the
content then ends with a line containing EVCXR\_END\_CONTENT.

For example, the following shows how you might provide a custom display function for a
type Matrix. You can copy this code into a Jupyter notebook cell to try it out.

```rust
use std::fmt::Debug;
pub struct Matrix<T> {pub values: Vec<T>, pub row_size: usize}
impl<T: Debug> Matrix<T> {
    pub fn evcxr_display(&self) {
        let mut html = String::new();
        html.push_str("<table>");
        for r in 0..(self.values.len() / self.row_size) {
            html.push_str("<tr>");
            for c in 0..self.row_size {
                html.push_str("<td>");
                html.push_str(&format!("{:?}", self.values[r * self.row_size + c]));
                html.push_str("</td>");
            }
            html.push_str("</tr>");
        }
        html.push_str("</table>");
        println!("EVCXR_BEGIN_CONTENT text/html\n{}\nEVCXR_END_CONTENT", html);
    }
}
let m = Matrix {values: vec![1,2,3,4,5,6,7,8,9], row_size: 3};
m
```

It's probably a good idea to either print the whole block at once, or to lock
stdout then print the block. This should ensure that nothing else prints to
stdout at the same time (at least no other Rust code).

If the content is binary (e.g. mime type "image/png") then it should be base64
encoded.

## Startup options
If you always want particular options set, you can add these to init.evcxr
which, if present will be run on startup. Sample locations:
* Linux: ~/.config/evcxr/init.evcxr
* Mac: /Users/Alice/Library/Preferences/evcxr/init.evcxr
* Windows: C:\Users\Alice\AppData\Roaming\evcxr\init.evcxr

For example, if you want to always turn on optimization (will make stuff slower
to compile), you might put the following in init.evcxr:
```
:opt 2
```

## sccache integration

sccache caches compilation outputs. If you frequently use the same crates, this
can speed things up quite a bit.

* `cargo install sccache`
* Add `:sccache 1` to your init.evcxr (see Startup options above).
* See [sccache](https://github.com/mozilla/sccache) for more details about
  sccache.

## Uninstall

```sh
evcxr_jupyter --uninstall
cargo uninstall evcxr_jupyter
```
