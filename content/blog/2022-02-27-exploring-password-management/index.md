+++
title = "Exploring the World of Password Managers"
[taxonomies]
tags = [ "privacy", "security", "passwords" ]
+++

Passwords are miserable to deal with. Bad passwords are both terrifyingly
tempting and incredibly common. Good passwords and hygiene all but require
complicated management schemes like password managers and a fair bit of
diligence. At the same time, passwords are used so frequently that it's most
practical to have access to all of your passwords on every device.

On the other hand, password breaches are so commonplace that just about anyone
who's used the internet in the past 20 years has probably had one compromised.
While efforts to supplement and displace passwords as the standard form of
authentication are slowly gaining traction, it's inevitable that passwords are
going to remain a core part of digital security for the foreseeable future.

Fortunately, many of the pitfalls of passwords can be mitigated through
two-factor authentication, or `2FA`. Even so, it is frighteningly common for
companies to rely on `SMS` for `2FA`, or otherwise allow `2FA` to be bypassed
through social engineering, access to a phone number, or an email.

In this post, I'll be talking about the password managers I've used and
recommend. Then, I'll outline the properties a good password manager should
have. Finally, I'll show you the solution I've settled on for my own needs.

As a small disclaimer, I'm not a cryptographer. My formal training essentially
boils down to a few first-year intro to CS classes in I took this year in
college. However, I have spent the last few years of my life researching
privacy and digital security. So, while I'll be talking a lot about different
password managers, remember that the best password manager for you is the one
that you fully understand.

## You Should Just Use a Password Manager

For most threat models, this is a solved problem. Problems that are as important
yet difficult as password management have many pre-existing solutions for all
sorts of devices, security levels, and preferences.

My go-to recommendation for people is [Bitwarden](https://bitwarden.com/).
Bitwarden is an Open Source password manager with secure cloud sync, convenient
and appealing clients for nearly every platform, and a very generous free tier.
They have a track record of good performance in [third-party security
audits](https://bitwarden.com/blog/third-party-security-audit/), [a browser
extension](https://addons.mozilla.org/en-US/firefox/addon/bitwarden-password-manager/),
and [zero-knowledge cloud
sync](https://bitwarden.com/help/what-encryption-is-used/) with standard,
reliable cryptography.

If you're more the self-hosting type, the [official Bitwarden
server](https://github.com/bitwarden/server) is fully Open Source and available
as a Docker image. The popular and lightweight
[vaultwarden](https://github.com/dani-garcia/vaultwarden) is an alternative
server implementation (written in Rust!) that's compatible with the regular
Bitwarden client applications, making deployment for individuals, families, and
small teams really easy.

Don't like the cloud? [KeePassXC](https://keepassxc.org/) is a Free Software
application that provides a local password database with various additional
encryption options such as keyfiles and a hardware token Challenge-Response.

While I don't necessarily recommend relying on them, there are even more
options if, for some reason, the above options don't work for you.
[1Password](https://1password.com/) provides a pretty seamless experience,
especially within the Apple ecosystem. Your [browser probably has one built
in](https://www.mozilla.org/en-US/firefox/features/password-manager/), and
[maybe even your Operating System,
too](https://support.apple.com/en-us/HT211145). These options can be especially
useful if the user is only going to use a password manager if it's less
friction than whatever they were doing before. Any password manager is going to
be way better than trying to remember your passwords or, worse, re-use them.

## What I Use

If I were a sensible person, I would still be using Bitwarden. However, I just
seem to really love making my computer harder to use for the fun of it. Once I
switched to [QubesOS](https://www.qubes-os.org/) full time, I wanted to keep all
of my passwords in an offline virtual machine, which requires an offline
password manager. At this point, I switched to KeePassXC. It works very well,
and is probably the most sensible solution for most QubesOS users or those who
otherwise prefer an entirely local solution.

I stuck with KeePassXC for the entire time that I used QubesOS... until today.
In an effort to eliminate as many GUI applications as possible from my machine,
I've been occasionally thinking about other offline options that worked from the
terminal.

It wasn't until I happened to read a few blog posts by [Filippo
Valsorda](https://filippo.io/) that the idea for a real alternative began to
form. Filippo, ["a cryptography and software engineer... in charge of
cryptography and security on the Go team at Google,"](https://filippo.io/) is
the author of [`age`](https://github.com/FiloSottile/age), a file encryption
tool.

What does this have to do with password managers? At the core of every password
manager (or, at least, any password manager that's worthwhile) is some
cryptographic algorithm to take your passwords and obfuscate them so that only
you can access them. For example, Bitwarden encrypts your passwords with
[`AES-CBC`, using `PBKDF2` to derive the encryption key from your master
password](https://bitwarden.com/help/what-encryption-is-used/). You can see this
in action with Bitwarden's [interactive crypto
page](https://bitwarden.com/crypto), which I think is pretty cool.

[`pass`](https://www.passwordstore.org/) is another password manager that
describes itself as ["the standard unix password
manager"](https://www.passwordstore.org/). `pass` is actually a very clever
script run through the command line that stores passwords at `~/.password-store`
as a tree of normal directories and files.

For example, from the [`pass`](https://www.passwordstore.org/) website, it could
look something like this:

```
Password Store
├── Business
│   ├── some-silly-business-site.com
│   └── another-business-site.net
├── Email
│   ├── donenfeld.com
│   └── zx2c4.com
└── France
    ├── bank
    ├── freebox
    └── mobilephone
```

Each of those files contains [`gpg`](https://www.gnupg.org/)-encrypted password
for the website. They're accessed by using the `pass` command to decrypt the
files with a `gpg` private key. However, I strongly dislike the idea of relying
on `gpg` to protect my passwords. While I do have a PGP key, I avoid using it
whenever there's a more suitable alternative available. [Many cryptographers
have written about the issues with
PGP](https://latacora.micro.blog/2019/07/16/the-pgp-problem.html), but the
biggest concern for me is the number of foot-guns with the PGP protocol and
`gpg` tool. Even in a standard configuration like `pass`, where I'm offloading
calling `gpg` to the script, the system is just too complicated for me to be
confident in my understanding of it.

By contrast, `age` touts its opinionated nature as a feature. The lack of
configuration, secure-by-design specification, and simplicity make it, in my
opinion, the perfect encryption tool to be used with `pass`.

## Properties of a Good Password Manager

### Confidentiality

The most obviously-essential property of a password manager is to make sure that
nobody else can read your passwords. There are several types of adversary your
password manager needs to protect against, but it primarily boils down to
protecting your passwords even if someone gains access to your encrypted
password database.

You should be able to post your encrypted database on your blog or send it to
Mark Zuckerberg himself without a concern in the world. In other words, the
security properties of your database should be completely upheld by
cryptography, and not rely on more error-prone protections like storing it
offline or keeping it in a private cloud account.

### Reliability

Have you ever been unable to login to an account when you really, really needed
to? Yeah, I have. Wasn't a fun time. Passwords are so important that even the
possibility of downtime is a serious concern. Even if you can eventually recover
the data, being unable to decrypt your password database at a vital moment can
is stressful, humiliating, and potentially disastrous.

A good password manager works when you don't have internet, when your laptop
runs out of battery, when you're traveling, and when all of those are happening
at once but you have a flight to catch in an hour. You should be following (at
bare minimum) a standard [3-2-1 backup
strategy](https://www.backblaze.com/blog/the-3-2-1-backup-strategy/) -- and your
password manager should make that as easy as possible.

### Usability

Cryptography that's hard to use is worse than no cryptography at all; just ask
PGP.

If reaching for your password manager isn't the lowest-friction way of signing
up for an account, you're going to mess up and re-use a password, or make one up
and forget it. Convenience is really key here. You can have the most secure
password manager in the world, but if you can't access it on your phone, or
another computer, you'll quickly email or text a password to yourself to log
into that account you really, really need right now.

On the other hand, it also has to be extra resistant to misuse. All it takes is
saving a password unencrypted once, or accidentally typing your Master Password
into an innocuous pop-up you didn't notice, to completely compromise yourself.

### Resiliency

You aren't the only one who needs access to your accounts. Should something
happen to you, or some other unforeseen situation comes up, there needs to be a
"break glass in case of emergency" -- without compromising on the password
manager's fundamental security properties. With password managers, this often
comes in the form of some adaptation of "Emergency Access." Essentially, the
idea is to provide a trusted party or parties with enough information to,
individually or collectively, gain access to some or all of your encrypted data.

Bitwarden has the concept of "Trusted Emergency Contacts" in their ["Emergency
Access"](https://bitwarden.com/help/emergency-access/) system. Essentially, it
works by providing Bitwarden with the secret key material necessary to decrypt
your vault encrypted to the trusted contact's public key.  That contact can then
send Bitwarden a "request for emergency access," after which, following a
amount of time you specify, they are sent the encrypted secret key material.
They than have access to view your entire vault.

1Password, on the other hand, lets you handle distribution of your secret key
material. They provide something they call an ["Emergency
Kit"](https://support.1password.com/emergency-kit/), which essentially is a PDF
containing your email, secret-key material, and a space to write out your Master
Password, along with instructions to begin the recovery. Unlike Bitwarden, who
acts as an escrow for your encrypted recovery data, 1Password lets you manually
distribute your recovery material. Notably, the trusted party does not need to
have a 1Password account, as the recovery material is a PDF that can be printed.
Like Bitwarden, however, once the trusted party receives the Emergency Kit, they
can then recover all of the material stored in your account.

An escrowed solution like Bitwarden is not ideal, as they potentially have the
power to withhold your secret key material (although I strongly doubt they would
ever do so unless somehow legally compelled to). The one benefit this has is
that you retain the power to prevent the secret key material from being sent to
them in the event that they try to abuse it. 1Password's PDF with a hand-written
master password seems like a better solution in terms of resiliency, but is
theoretically more susceptible to abuse. Ultimately, which of these two
solutions is better comes down to how much you trust your cloud provider
compared to how much you trust your emergency contacts.

One third way to solve this is to require multiple trusted parties to
collectively agree to access your password manager. Changing the number and
trustworthiness of those parties can help you adjust that ratio of "resiliency
against loss" to "abuse by trusted parties."

## Perfect Password Management

## Potential Improvements

### Better Secret Sharing
