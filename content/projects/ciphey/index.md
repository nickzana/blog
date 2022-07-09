+++
title = "ciphey"
weight = 1
[taxonomies]
tags = []
+++

Simply put, `ciphey` is a password and secret manager that is like
[`pass`](https://passwordstore.org) if it used
[`age`](https://age-encryption.org) instead of PGP. More than anything, it is
an experiment to determine how cryptography can be combined in a minimalist
manner to protect passwords in a way that accounts for the most realistic
threats to their confidentiality, reliability, usability, and resiliency. It
takes many of the database and key management ideas from
[1Password](https://1password.com) and [Bitwarden](https://bitwarden.com) and
combines them with the Unix-like philosophy of `pass`.

To be clear, `ciphey` is still in development. [The remaining work is outlined
below](#remaining-work). While I will be using it as my actual password manager,
I strongly recommend that you only experiment with it and not rely on it for any
actual passwords.

*NOTE: This page is a work in progress.*

## The Basics

My goal is for `ciphey` to be simple enough that someone with a bit of knowledge
about cryptography who wants to understand how their passwords are protected can
do so after a short explanation. This sections aims to be that explanation.

### Entries

Everything you store in `ciphey` goes into an entry. Generally, you'll have one
`entry` per password. However, passwords come in many different forms: as a
result, entries are plaintext, which means any file can be used as an entry.

This doesn't mean there is no structure to entries. For example, take a look at
the entry for a hypothetical [`xkcd.com`](https://xkcd.com) account below:

```
correct horse battery staple
name: xkcd.com
tag: comics
username: Tr0ub4dor&3
url: https://xkcd.com/936

To anyone who understands information theory and security and is in an
infuriating argument with someone who does not (possibly involving mixed case),
I sincerely apologize.
```

The first line of every entry is interpreted as the entry's "secret."  In this
case, the secret is `correct horse battery staple`, the password for the
account. This is usually a password or private key, but can be any value that
you might want to access by default.  If this doesn't work for you for some
reason, no worries; just leave the line blank.

The secret is followed by any number of optional field-value pairs. Every
field-value pair lives on its own line and can contain any information you want,
so long as it follows the format `FIELD: VALUE`. Fields can be repeated and in
any order. The only limitation is that your `FIELD` cannot contain a `: ` (`:`
followed by a space), but the `VALUE` can.

There are no `FIELD`s with any special meaning (yet), but keeping them
consistent will help you find entries more quickly. I strongly recommend giving
each entry a `name` field, as this is what `ciphey` uses to search for entries
by default.

Finally, the contents of the file from the first line that doesn't match the
pattern `FIELD: VALUE` down to the end of the file is left uninterpreted,
meaning that you can save any text that you'd like there. This is called the
"note." In the example above, the "note" is the caption text from
[xkcd.com/936](https://xkcd.com/936). Separate the notes and the preceding
section with a newline.

### The Entries

Your `ciphey` database is a folder of files that can live on any filesystem.
This folder is completely encrypted, and thus can safely be backed up, shared,
or otherwise accessed without any unauthorized party gaining access to the
contents of the file.

```
ciphey/
└── entries/
    ├── age1sdnql3ksstj7krh7azddygh860f6uttus5t3e9j52t4k5z34kutqyzgr4e.age
    ├── age1ryutmsg486w4y06kq7w0rqm2s08r3sgzlwcl9p9p00jducedevvqkgrvyw.age
    ├── age1mqr5htmadgzlmcdjks79d6ucfe5k46ruavnaxm08m64lpl6h5udqhkwwvp.age
    └── age1r908d20yj3ntp9q8g0448kwflfgf3au2qmhkkx7zp8xnesfdjfhqpd3r2d.age
```

Each file in the "entries" folder contains a single "entry." The name of each
file is the `age` public key the entry is encrypted to. Every entry into your
database is encrypted to a unique public key. Without that public key's
corresponding private key, the contents of the entry cannot be accessed.

However, keep in mind that some metadata does leak. Namely, any data associated
with the filesystem (date created, date last modified, owner/group, etc.) is NOT
protected by `ciphey` by default. It's still a good idea to keep your database
away from any curious eyes when it's avoidable.

### Your Keys

Every time you unlock your `ciphey` database, you must have two things:

1. Your `Primary Passphrase` (something you know)
2. A `Device Key` (something you have)

Every "device" you have (i.e. your desktop, laptop, and phone) has its own
`Device Key`. This key is generated and stored as a normal file when you first
set up `ciphey` on the device, and never leaves it.

Each `Device Key` is encrypted with your `Primary Passphrase`. Without your
`Primary Passphrase`, the `Device Key` cannot be used.

Alternatively, you can use a `Device Key` stored on a hardware token, such as a
Yubikey or OnlyKey. This has the added benefit that the key material never
leaves the hardware token, protecting against attacks that can steal key
material from your computer's local disk. In order to decrypt your entries, an
attacker would have to physically have access your hardware token, in addition
to learning your `Primary Passphrase`.

### Entry Keys

Every entry in your database is encrypted to a specific `Entry Key`. These entry
keys are stored next to the `database` directory in the `keys` directory. Every
entry gets its own key so that access can be cryptographically restricted on an
entry-by-entry basis.

The keys stored on the filesystem are encrypted to every `Device Key` that
you've given access to the entry.

### Disaster Recovery

When you set up your `ciphey` database for the first time, you'll be prompted to
create a `Recovery Key`, which is an `age` key encrypted with your `Recovery
Passphrase` (By default, your `Primary Passphrase`, but this can optionally be
changed). This key should be written down and stored in a safe place, such as in
a safe. This way, if you lose all of your devices, you can still regain access
to your passwords.

You can also give a copy of this key and your `Recovery Passphrase` to anybody
else who should have access to your accounts in case of an emergency. This way,
you can be sure that you're unlikely to ever lose access to your `ciphey` keys.

### Summary

Every entry is encrypted with its own unique `Entry Key`.

`Entry Key`s are encrypted to each `Device Key` that is granted access to the
entry. They can be encrypted to other people's `Device Key`s to share an entry
with them.

`Device Key`s are encrypted with your `Primary Passphrase`. They can either live
on your filesystem or on a hardware token, like a Yubikey.

When you first set up `ciphey`, you also create a `Recovery Key`, which every
`Entry Key` is encrypted to by default. This `Recovery Key` is encrypted with
your `Recovery Passphrase`, which are to be stored offline in a safe place
and/or given to trusted individuals so they can be used in case of an emergency.

### Usage

## Implementation

### Entry Parsing

The entry format was designed to be easy to parse for both humans and programs.
The following Rust snippet is a (simplified) example of parsing an Entry from a
String `s`:

```rust
// The entry's secret
let mut secret: String;
// A vector of key/value pairs
let mut fields: (String, String) = Vec::new();
// The entry's notes
let mut notes: Option<String> = None;

// An iterator over the decrypted lines of the entry file
let mut lines = s.lines();

// The first line contains the secret
secret = lines.next();

while let Some(line) = lines.by_ref().next() {
	if let Some((key, value)) = line.split_once(": ") {
		// Add the key and value to fields
		fields.push((key, value));
	} else {
		// Take the rest of the lines and put them into notes
		notes = Some(
			std::iter::once(line)
				.chain(lines.by_ref())
				.collect::<Vec<String>>()
				.join("\n"),
		);
	}
}
```

This particular implementation has a lot of room for improvement. The biggest
issue adding complexity is that Rust's `take_while` iterator method is
consuming, not peekable, meaning that ownership of the lines it checks is passed
to its closure. Thus, the following snippet, which is much easier to
read, would not work:

```rust
// An iterator over the decrypted lines of the entry file
let mut lines = s.lines();

// The first line contains the secret
secret = lines.next();

// Take lines while they can be parsed as key/value pairs split by ": "
fields = lines.take_while(line.split_once(": ")).map(...);
// first line failing take_while is incorrectly dropped here

// Join the remaining lines as notes
notes = lines.collect::<Vec<String>>().join("\n");
```

This is because even though the first line that fails the conditional is
consumed by `take_while`, it would neither be added to the fields (as it cannot
be parsed as one) nor collected into the vector over the lines of notes, instead
being dropped, causing data loss.

There are a few Rust crates that provide a peekable `take_while`, but they're
too big to justify as adding as dependencies for just this feature. It's
possible that a simpler approach is possible using an iterative method, but I
have not been able to come up with one yet.

## Remaining Work

### Password History

At the moment, there is no built-in support for keeping track of entry changes
over time. `pass` uses git to keep track of changes at the filesystem level,
while password managers like Bitwarden have built-in support for storing the
history of passwords over time.

Currently, there is nothing stopping someone from manually checking entries into
git or another backup solution. However, one way to provide better integration
into `ciphey` would be to allow for `pre-hooks` and `post-hooks`.  These hooks
are scripts that are run before or after specific actions are taken by `ciphey`.
This would allow, for example, automatically committing the changes to a git
repository and pushing those changes to a remote server after every change made
by `ciphey`.

### File storage

For the moment, entries must be plaintext files. If you want to encrypt an
arbitrary file with `ciphey`, you have to manually create an encryption key,
store that key as an entry in `ciphey`, and then manually encrypt said file with
the newly-created key.

It would be beneficial to devise a way to treat encrypted files just like any
entry so that the key material for other files can be automatically managed in
the same way.

### Backup

While `ciphey` makes sure that you don't lose your secret key material through
the use of `Recovery Key`s, `ciphey` does not have any built-in backup system
for the encrypted `ciphey` database. Of course, you can still back up `ciphey`
like any other directory (which is a benefit of it living as just a plain
directory), but it may be useful to consider ways that `ciphey` can better
integrate or accomodate backups.
