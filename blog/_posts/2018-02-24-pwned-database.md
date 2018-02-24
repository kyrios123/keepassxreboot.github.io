---
layout: blog
title:  "The Pwned buzz and why you really don't need this database"
date:   2018-02-24 16:08 +0100
category: "Other"
permalink: /:path/
author: Janek Bevendorff
excerpt_separator: <!--more-->
---

If you watched your social media channels over the last few days, you probably
haven't missed the media buzz about the launch of the new
[Pwned V2 Passwords](https://www.troyhunt.com/ive-just-launched-pwned-passwords-version-2/)
database, not least because of our popular competitor 1Password integrating it as
a service for you, so you can check if your beloved passwords have been compromised.

The database contains the SHA-1 hashes of half a billion leaked passwords. So
if you want a new and secure password, you can now easily check it against a
fairly large collection of known bad and compromised passwords. Neat, right?

Well, let me explain why you really don't need this database and why you shouldn't
even try using it.

<!--more-->

##### Trivial defeat

For testing purposes, I downloaded the [database dump](https://haveibeenpwned.com/Passwords),
which has a compressed size of about 8GB and decompresses to 30GB of plain text.
Each line in the file contains the SHA-1 hash of a password and usage counts.

A quick first test of the top [10 most-popular passwords](https://en.wikipedia.org/wiki/List_of_the_most_common_passwords)
of 2017 gave no surprising results. All 10 were found fairly easily towards the top
of the file. The first fun fact: adding whitespace to the end of the passwords tends
to defeat the search. " 12345678" can be found in line 4778908, but "12345678 "
does not exist.

So here we directly have our problem: despite the (seemingly) huge mass of half a billion
passwords, it's surprisingly trivial to create password variants that are not in
the database. Neither of "password ", "pass>word", or "pass}word" could be found and
they are all within edit distance of 1 to the original "password".
Permutations are also reasonably effective. "dRowssap" is "password" spelled backwards
with a capitalized R and it cannot be found.

##### The math

The math behind this is rather simple. Assume, we have a password with only lowercase
and uppercase letters and numbers. This gives us an alphabet size of *62* (*26 + 26 + 10*).
With an 8-character password, we end up with *62^8 = 218,340,105,584,896* possible
permutations (with repetition) of these 8 characters, which is already *a lot*
more than the *501,636,842* passwords contained in the database. So with any
random 8-character combination of letters and numbers we have a chance of roughly
*~2.3 × 10^-6* of finding it in the database.

##### Password managers to the rescue

With chances of finding trivial 8-character passwords in the database already being
so low, here comes the paradox part: How does a password manager, whose one and only
purpose is to securely store arbitrarily many and complex passwords, so you as a
human being don't have have to remember them, find it useful to check your password
against half a billion of those passwords?

There are only two answers for this:

1. You don't use your password manager to generate secure passwords (start doing
    that *now*!)
2. You are using services that got compromised and didn't hash their passwords properly
    (stop using them!)

The second point probably needs a little explanation. When you create an account at an
online service, your password isn't stored in plain text. Instead, a hash function
is run on it (hopefully one that is slow) to create a fixed-length representation
of your password. This representation is always the same for the same input password.
To make it impossible to pre-calculate a large number of hashes and compare them,
a so-called salt is added to the password before hashing, which is stored in plaintext
next to the hash. So next time, you log in, the service takes your password, appends the
salt, runs it through the hash function and checks if the output matches the one stored
in the user database.

So what happens if the user database is leaked? Well, don't worry too much, the attacker
can only get the salted hash and not your actual password. Unless..., the service didn't
do their homework and stored your password in plain text nonetheless, in which case it
will eventually end up in one of those Pwned databases.

##### Complex passwords and brute force

Now let's assume an attacker got hold of your password's hash. They can still try to
guess the correct password by brute force. The amount of time needed depends largely
on two factors:

- the number of possible password candidates
- speed memory requirements of the password hashing scheme

The number of possible password candidates is controlled by the complexity of your
password. The other factor is probably out of your reach when you are using an online
service. It determines how many guesses an attacker can make per seconds. You want this
to be slow. This is why we use Argon2 in KeePassXC 2.3 (or the older
AES-KDF for KDBX 3.1 in KeePassXC 2.2).

A plain SHA-1 hash is fast and therefore not a good password hashing algorithm
(and it also hasn't aged well [for other reasons](https://www.wikiwand.com/en/SHA-1#/Attacks)).

[Benchmarks](https://gist.github.com/epixoip/a83d38f412b4737e99bbef804a270c40) suggest
that an expensive and beefy modern GPU cluster can perform about **8.5 billion** SHA-1
hashes per second (!). Such a cluster could generate all hashes of the whole Pwned
database in 0.6 seconds.

If we take our original 8-character password, it could generate all possible
passwords in a little more than seven hours. And this is the worst-case
estimate. On average, an attacker will find the correct password after 50% of the
time, which is 3.5 hours. So take this into account the next
time you think about sending your SHA-1 password hash to some online password
database (even if it's just the first 5 characters of the hash).

So what if we just increase our password size? We use a damn password manager, so
why not just dial it up to 30 characters? This will create *62^30 ≈ 5.9 × 10^53*
possible candidates. Testing all these at 8.5 billion hashes per second would take
a rough *2.2  × 10^36* years. That's about *159,594,201,897,441,234,244,091,519*
times the age of the universe (take half of that to get from worst case to average,
I'm being generous). I guess you can see that the chances of one of those passwords
being in the Pwned database are very, veeeery low and that's only with letters and
numbers. If we take the full ASCII set (excluding the NULL character), we have
*127^30* possible candidates, which is not quite the number of atoms in the universe,
but at least Archimedes' estimate for how many grains of sand would fill it.