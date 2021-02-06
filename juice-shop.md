# OWASP Juice Shop "Blind" Writeup

## Introduction

This writeup covers my "blind" foray into the OWASP Juice Shop intentionally-vulnerable web application. I will be using this as a bit of a learning exercise, and I'll basically be documenting my approach, things that worked, things that didn't work, and so on.

As a companion text, I will have at my side the Web Application Hacker's Handbook, 2nd Ed.

## Reconnaisance

Good recce is always important, so I start by booting up Burp and setting up Firefox to proxy requests through to Burp. Burp will then passively build up a site map as I browse. Meanwhile, I am also looking for interesting tidbits: usernames, names of employees, naming conventions, etc.

Here are the things that pop out at me as I browsed:

- There is a review from a user with email `admin@juice-sh.op`. Presumably an administrator.
- Another review, from `bender@juice-sh.op`.
- `jim@juice-sh.op`
- `mc.safesearch@juice-sh.op`
- `bjoern@owasp.org`
- `morty@juice-sh.op`
- Customer feedback form has an `Author` field, which is disabled - could probably intercept and edit this
- Also has a Captcha; would be worth checking the implementation of this.
- Complaints form has a disabled author field too
- Also allows uploading of a file, will need to test the restrictions and validation on this.
- There's 2FA apparently
- There's an option to request a data export. Wonder if we can request someone else's data?

That should do for now.

Next, we'll do a quick wordlist search for hidden content. We'll use `dirb` for this, and one of the `SecLists` Discovery wordlists.
