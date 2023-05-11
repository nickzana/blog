+++
title = "Password Security through Compartmentalization"
[taxonomies]
tags = [ "privacy", "security", "passwords" ]
+++

Not all passwords are created equal. You can find evidence of this in the
countless people who use "layered" password schemes. While a some people just
re-use the same password on every website, others have an intuition for the
different trust levels between websites that they use.

Nobody needs absolute protection for every single account that they sign up for.
If you make an account to sign up for a newsletter, the impact of that account
getting breached is small. On the other hand, compromised "critical" ones like
email, Google, and bank accounts can cause catastrophic (and often
unrecoverable) damage to your financial, social, and professional identity.

There aren't many proxies for this in the offline world. Abysmal "security"
measures like Social Security numbers (in the U.S.; those who live in countries
with government-issued public/private keys can smugly skip this part) are
incredibly sensitive, given that they're basically master keys to our financial
lives. Each person generally only has a single SSN, meaning that every
institution who mandates it gets the same one.

People using the "layered" strategy online recognize different sensitivity
levels of accounts. Generally, they'll have one "high security" password used
for critical accounts, a "regular" password for things like social media and
other relatively trustworthy apps, and then a "throwaway" password for junk
accounts like retailers, newsletters, and the like.

My generation distrusts online companies and have shown through the above scheme
that they can recognize these different security levels. Unfortunately, today's
biggest password managers create single, all-or-nothing security vaults. I've
yet to see any password manager explore encouraging users to compartmentalize
their vaults cryptographically.

Compartmentalization enables a wide variety of additional use-cases and
potential features. When stop making "vaults" and start distinguishing
"passwords" and "devices," features like two-factor encryption, password
sharing, and compartmentalization become a natural extension of both the
cryptography and the user's mental model.

Now, to be fair, 1Password has [built in support for sharing vault
entries](https://support.1password.com/share-items/) (sadly, I can't find too
much of the feature's clever "Psst!" branding anymore). Even if I think it's a
bit clunky, it covers a lot of compartmentalization's use cases.

In the [Access control
enforcement](https://1passwordstatic.com/files/security/1password-white-paper.pdf)
section of their security whitepaper, 1Password explains that they have several
mechanisms for protecting shared passwords: cryptographic, server-side, and
client-side protections.

*1Password warns that this section of their whitepaper is potentially
incomplete. I've done my best to verify that all of the below is accurate, but
if the information is incorrect or outdated, please [contact
me](mailto:me@nickzana.dev).*

Say Alice, a 1Password user, wants to share a password with Bob, who doesn't use
1Password. Alice can generate a link to an encrypted *copy* of the password on
1Password's servers. The password is encrypted with a newly-generated key that
is then encoded into the link Alice is sharing. Alice needs to send this link to
Bob somehow, but once he has it, he can enter it into his web browser and view
the password.

If Alice would like, she can also use email-based access control that's enforced
by the 1Password server. When Bob opens the link, 1Password will send him an
email with a one-time code that he has to enter before he can access the
password.

This is a pretty good low-friction alternative to sharing passwords over text or
email. It provides some marginal benefit in terms of server-side controls. In
the end, though, it just shifts the sharing of the cryptographic material from
the password itself to the link. Unless Alice shares the link over an end-to-end
encrypted channel with Bob, the largest benefits provided by this method are the
server-side access controls for things like email verification, time expiration,
and one-time access limits.

It also comes with the downside of being inherently web-based. Yes, asking the
person you want to share a password with to download a specific piece of
software is unreasonable. But this weakens the security of this particular
feature if you want to use it for another purpose, such as using a password on a
public or otherwise untrusted computer.

Don't get me wrong, this is a good improvement. But for the case of sharing
a limited set of passwords with someone else - yourself - it doesn't quite stack
up.

On the other end of the spectrum, 1Password has many features that focus on
enterprise users, where compartmentalization, access control, and centralized
management are major focuses. [From their
whitepaper](https://1passwordstatic.com/files/security/1password-white-paper.pdf#chapter*.16):

> Alice is running a small company and would like to share the office Wi-Fi
> password and the keypad code for the front door with the entire team. Alice
> will create the items in the Everyone vault, using the 1Password client on her
> device. These items will be encrypted with the vault key. When Alice adds Bob
> to the Everyone vault, Alice's 1Password client will encrypt a copy of the
> vault key with Bob's public key.

> Bob will be notified that he has access to a new vault, and his client will
> download the encrypted vault key, along with the encrypted vault items. Bob's
> client can decrypt the vault key using Bob's private key, giving him access to
> the items within the vault and allowing Bob to see the Wi-Fi password and the
> front door keypad code.

If a password manager could enforce per-device access cryptographically (likely
by giving each device its own keypair, to which entry and/or vault keys are
encrypted), losing your phone wouldn't mean a potential compromise of, say, your
bank password or SSN, so long as only more "trusted" devices have access to
these entries.

I could even imagine a service that allowed you to temporarily access passwords
from an untrusted device. Imagine if you had a relatively unimportant account.
Maybe it's a web-based game, or just information that's convenient to have in a
lot of places like your bookmarks. Downloading a client (or, more likely,
visiting a website, as terrible an idea as that is) and entering just a vault
password on any public computer could gain access to just these "low security"
entries in your vault. This vastly improves over the typical methods that solve
this problem, usually by either re-using passwords on these low-sensitivity
sites or logging into a password vault via the web.

Compartmentalization is one of the strongest defenses against exposing private
files to compromised environments. It can be mitigated if that environment only
has access to the minimum information that it needs. Given the potentially vast
differences in trust levels between our devices, it seems necessary that we
begin to reassess the threats we're designing our password managers around.
