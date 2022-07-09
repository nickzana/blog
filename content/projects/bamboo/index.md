+++
title = "Bamboo Media Server"
weight = 3
[taxonomies]
tags = []
+++

Bamboo is a self-hosted personal media server written in Rust that I've been
working on for the past two years with my friend
[Ersei](https://ersei.saggis.com). It aims to replace existing media servers
like Jellyfin and Plex by decoupling the front-end, API, and content as much as
possible, allowing for richer client applications and a wider range of content
and content types.

Most of what makes Bamboo's architecture different is its ability to generalize
over types and sources of content while providing common primitives for richer
metadata.

For example, Jellyfin provides a few distinct types of media that it can serve:
Movies, Music, Shows, Books, Photos, and Music Videos. Jellyfin's API is deeply
tied to its web front-end, so the content you can host on Jellyfin is limited to
what the Jellyfin web client is capable of displaying. This works okay for
classic media server content like Movies and TV, but even something slightly
beyond its expectations, like a YouTube video or Twitch stream, must be made to
conform to the formats that Jellyfin expects.

Bamboo solves this problem by defining generic types that are common to all
media, such as the Title, a UUID, and a list of URLs it can be accessed at, and
allowing specific types of media to expand upon with additional data and fields.
This way, clients can be as specific or general as they desire based on their
required functionality. A media search application may not need to care about
anything beyond the title, while a Podcast application should only accept media
that is of the Podcast data type.

Of course, being written in Rust, Bamboo utilizes Rust's type system to define
strict API specifications. Invariants, optional or mandatory fields, and data
types are explicitly encoded using `struct`s, `enum`s, and `serde` for
serialization and deserialization, removing any ambiguity in the specification.
This is important for an application as dynamic as Bamboo, as the vast range of
content that it's capable of serving is intentionally vague and thus incorrect
assumptions by the server or its clients can make for inconsistent, incorrect,
or buggy code.

The public repository for the project can be found at
[git.nickzana.dev/bamboo/bamboo](https://git.nickzana.dev/bamboo/bamboo).
