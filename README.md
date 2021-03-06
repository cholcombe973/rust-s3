[![](https://camo.githubusercontent.com/2fee3780a8605b6fc92a43dab8c7b759a274a6cf/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f72757374632d737461626c652d627269676874677265656e2e737667)](https://www.rust-lang.org/downloads.html)
[![](https://travis-ci.org/durch/rust-s3.svg?branch=master)](https://travis-ci.org/durch/rust-s3)
[![](http://meritbadge.herokuapp.com/rust-s3)](https://crates.io/crates/rust-s3)
[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/durch/rust-s3/blob/master/LICENSE.md)
[![Join the chat at https://gitter.im/rust-s3/Lobby](https://badges.gitter.im/durch/rust-s3.svg)](https://gitter.im/durch/rust-s3?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)
## rust-s3 - [Documentation](https://durch.github.io/rust-s3/)

Tiny Rust library for working with Amazon S3

*Increasingly more loosly based on [crates.io](https://github.com/rust-lang/crates.io/tree/master/src/s3) implementation.*

### Intro
Very modest interface towards Amazon S3.
Supports `put`, `get` and `list`, with `delete` on the roadmap and will be done eventually,
probably around the time I discover I need it in some other project :).

### What is cool

The main (and probably only) cool feature is that `put` commands return a presigned link to the file you uploaded.
This means you can upload to s3, and give the link to select people without having to worry about publicly accessible files on S3.

### Configuration

Getter and setter functions exist for all `Link` params... You don't really have to touch anything there, maybe `amz-expire`,
it is configured for one week which is the maximum Amazon allows ATM.

### Usage

*In your Cargo.toml*

```
[dependencies]
rust-s3 = '0.2.3'
```

#### Example

```
extern crate s3;
use s3::{Bucket, put_s3, get_s3, list_s3};

const AWS_ACCESS: &'static str = "access_key";
const AWS_SECRET: &'static str = "secret_key";

fn main () {
  // Bucket instance
  let bucket = Bucket::new(S3_BUCKET.to_string(),
                              None,
                              AWS_ACCESS.to_string(),
                              AWS_SECRET.to_string(),
                              None);

  // Put
  let put_me = "I want to go to S3".to_string();
  let url = put_s3(&bucket,
                  &"/",
                  &put_me.as_bytes());
  println!("{}", url);

  // List
  let bytes = list_s3(&bucket,
                      &"/",
                      &"/",
                      &"/");
  let string = String::from_utf8_lossy(&bytes);
  println!("{}", string);

  // Get
  let path = &"test_file";
  let mut buffer = match File::create(path) {
            Ok(x) => x,
            Err(e) => panic!("{:?}, {}", e, path)
          };
  let bytes = get_s3(&bucket, Some(&path));
  match buffer.write(&bytes) {
    Ok(_) => {} // info!("Written {} bytes from {}", x, path),
    Err(e) => panic!("{:?}", e)
  }
}
  ```

