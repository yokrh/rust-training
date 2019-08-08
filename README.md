# Rust

## Install

https://www.rust-lang.org/tools/install
https://www.rust-lang.org/learn/get-started
https://doc.rust-lang.org/1.30.0/book/first-edition/crates-and-modules.html
https://wasm-dev-book.netlify.com/hello-wasm.html

```sh
#install rustup, rustc, Cargo(package manager), etc.
curl https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env

#verify
rustup --version
rustc --version
cargo --version

# update both Rust and Cargo
rustup update
```

## Hello world

```sh
# create work dir
mkdir rust/ && cd $_

# create a new rust project
cargo new hello-rust  # or 'mkdir hello-rust && cd &_ && cargo init'
cd hello-rust/
ls -la
#git commit -m "initial commit" --allow-empty

# see the manifest
cat Cargo.toml

# check the initial Hello world source code
cat src/main.rs

# build and run
cargo run
```


## dependency

```sh
# create source code using ferris_says crate(=Rust library)
cat <<EOF > src/main.rs
// fn main() {
//    println!("Hello, world!");
//}

use ferris_says::say;
use std::io:{stdout, BufWriter};

fn main() {
  let stdout = stdout();
  let out = b"Hello fellow Rustaceans!";
  let width = 24;

  let mut writer = BufWriter::new(stdout.lock());
  say(out, width, &mut writer).unwrap());
}
EOF

# try to build. it will fail
cargo build

# add dependency
echo 'ferris-says = "0.1"' >> Cargo.toml
cat Cargo.toml

# install crate and build. it will succeed
cargo build

# build and run
cargo run
```


## Use opencv-rust

https://github.com/twistedfall/opencv-rust

### Prerequisite

```sh
# opencv
brew install opencv  #v4.x
#brew install opencv@3  #v3.x

# pkg-config
brew install pkg-config
ls /usr/local/lib/pkg-config/opencv*
cp /usr/local/lib/pkg-config/opencv4.pc /usr/local/lib/pkg-config/opencv.pc
```

### Try

```sh
# create a project
cargo new opencv
cd opencv/

# prepare YOUR image file
cp ../messi5.jpg .
```

`Cargo.toml`
```toml
...
[dependencies]
# opencv = "0.18"
opencv = {versoin = "0.18", default-features = false, features = ["opencv-41"]}
```

`main.rs`
```rs
extern crate opencv;
use opencv::imgcodecs;
use opencv::highgui;

fn main() {
  println!("Hello opencv-rust");

  let image = imgcodecs::imread("messi5.jpg", 0).unwrap();
  highgui::named_window("hello opencv", 0).unwrap();
  highgui::imshow("hello opencv", &image).unwrap();
  highgui::wait_key(10000).unwrap();
}
```

```sh
# build and run
cargo run
```
yeah!!


## Use wasm

## Prerequisite

https://rustwasm.github.io/docs/book/game-of-life/setup.html
https://github.com/rustwasm/wasm-pack
https://github.com/ashleygwilliams/cargo-generate

```sh
# wasm-pack
curl https://rustwasm.github.io/wasm-pack/installer/init.sh -sSf | sh
wasm-pack --version  # verify

# cargo-generate
cargo install cargo-generate
cargo generate --help  # verify

# npm
npm -v
```

### Hello world

https://rustwasm.github.io/docs/book/game-of-life/hello-world.html
https://github.com/ruswasm/create/wasm-app

```sh
# create a project from template, insted of running 'cargo new wasm-rust'
cargo generate --git https://github.com/rustwasm/wasm-pack-template --name wasm-rust
cd wasm-rust/
ls -la  # verify

# see the manifest
cat Cargo.toml

# check the initial Hello world source code
cat src/lib.rs

# build
wasm-pack build
ls pkg/  # verify

# create wasm executable envirionment with dev server
npm init wasm-app www -y  #if it didn't create www/ directory, reopen the terminal and re-try the command
cd www/
npm install
npm audit fix
```

edit package.json to add dependency for the wasm
`www/package.json`
```json
{
  // ...
  },
  "dependencies": {
    "wasm-rust": "file:../pkg"
  }
}
```

`www/index.js`
```js
// import * as wasm from "hello-wasm-pack";
import * as wasm from "wasm-rust";
wasm.greet();
```

```sh
# solve the added dependency
npm install

# run dev server. It has hot module reload.
npm run start
```
Access to http://localhost:8080/.
yeah!!

### Try console.log

https://rustwasm.github.io/docs/book/reference/crates.html#error-reporting-and-logging
https://crates.io/crates/console_log

`wasm-rust/Cargo.toml`
```toml
...
[dependencies]
wasm-bindgen = "0.2"
log = "0.4"
console_log = "0.1.2"
...
```

`wasm-rust/src/lib.rs`
```rs
mod utils;

use wasm_bindgen::prelude::*;
use log::Level;
use log::debug;
use log::info;

// When the `wee_alloc` feature is enabled, use `wee_alloc` as the global allocator.
#[cfg(fature = "wee_alloc")]
#[global_allocator]
static ALLOC* wee_alloc**WeeAlloc = wee_alloc::WeeAlloc::INIT;

#[wasm_bindgen]
extern {
  fn alert(s* &str);
}

#[wasm_bindgen]
pub fn debug() {
  console_log::int_with_level(Level::Debug);
  debug("It works! -- debug");
  info("It works! -- info");
}
```

`wasm-rust/www/index.js`
```js
// import * as wasm from "hello-wasm-pack";
import * as wasm from "wasm-rust";

wasm.greet();
wasm.debug();
```

```sh
# update wasm
cd /path/to/wasm-rust/
wasm-pack build

# if you didn't keep the dev server running
cd www/
npm run start

# stop the dev server by ctrl+c
```
Access to http://localhost:8080/. See the devtool console.
yeah!!


## Try opencv

As a result it failed.
Could not use wasm-bindgen crate with opencv crate.


`wasm-rust/Cargo.toml`
```toml
...
[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
wasm-bindgen = "0.2"
log = "0.4"
console_log = "0.1.2"
opencv = {version = "0.18", default-features = false, features = ["opencv-41]}

[dev-dependencies]
wasm-bindgen-test = "0.2"

[profile.release]
# Tell `rustc` to optimize for small cose size.
opt-level = "s"

# [build]
# target = "wasm32-unknown-emscripten"
```

`wasm-rust/src/lib.rs`
```rs
...

use opencv::imgcodecs;
use opencv::highgui;
#[wasm_bindgen]
fn opencv() {
  info!("OpenCV");
  let image = imgcodecs::imread("messi5.jpg", 0).unwrap();
  highgui::named_window("hello opencv!", 0).unwrap();
  highgui::imshow("hello opencv!", &image).unwrap();
  highgui::wait_key(10000).unwrap();
}
```

`wasm-rust/www/index.js`
```js
...
wasm.opencv();
```

```sh
wasm-pack build  # Error
cargo build  # no Error, but no wasm
```
->
research...
->
cannot build opencv with the target `wasm32-unknown-unknown`.
`cargo build` can build, but doesn't produce .wasm file.
`cargo build --target wasm32-unknown-unknown` cannot build.
->
Tried to use Emscripten
https://kripken.github.io/blog/binaryen/2018/04/18/rust-emscripten.html
also causes same problem.

Simlar issue
https://github.com/rust-lang/cargo/issues/7004


Memo
```sh
rust up show  # show installed toolchains, targets, and active toolchain

git clone https://github.com/emscripten-core/emsdk.git
cd emsdk/
./emsdk install latest
./emsdk activate latest
source ./emsdk_env.sh --build=Release
cd ../
mv src/main.rs src/lib.re
export PKG_CONFIF_DIR=/usr/local/lib/pkgconfig
export PKG_CONFIG_SYSROOT_DIR=/usr/local/lib/pkgconfig  # because of cross-compile to wasm
cargo build --target=wasm32-unknown-emscripten  # failed
```
->
give up!!!
