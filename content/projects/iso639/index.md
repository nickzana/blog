+++
title = "iso639_enum"
weight = 4
[taxonomies]
tags = []
+++

`iso639_enum` is a small Rust crate I wrote for my Bamboo media server project.
ISO639 is a standard that enumerates world languages and provides two and three
character codes to represent them. This is important in the context of a media
server because any media in a particular language (basically anything with
spoken or written words) will represent its language in its metadata in some
form, usually as a two or three character language code.

The documentation for the crate can be found at
[lib.rs/iso639_enum](https://lib.rs/iso639_enum).

The repository for the crate can be found at
[git.nickzana.dev/nick/rust-iso639](https://git.nickzana.dev/nick/rust-iso639).
