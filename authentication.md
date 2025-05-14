---
theme: seriph
background: /authentication/background.jpg
class: 'text-center'
highlighter: shiki
lineNumbers: false
canvasWidth: 680

---

## Designing a secure authentication package
_(and the dangers of "there is ~~an app~~ a package for that")_

Mattis Turin-Zelenko, chax.at

---

### Outline

* Designing an authentication package
  * Design Goals
  * Lessons learned
* The TOTP "conspiracy"
  * Security in Open Source

---

### Motivation
* Multiple projects all requiring authentication
* Code changes slightly per project, based on requirements
  * OpenID SSO via Google, Facebook, Microsoft, ...
  * SAML SSO
  * Two-factor authentication
* Let's standardize everything into a single package!

---

### Goal 1 - Secure by default

<center><img src="/authentication/UnsecuredMongodb.png" width="450" /></center>

* Package is secure by default
* Throw error if secrets are not provided or insecure
* Provide options to make it less secure (e.g. for test/dev environments)

---

### Goal 2 - Hard to use wrong

* Use scary names for scary things
* Provide documentation for _why_ they are scary
* e.g. `UnsafePbkdf2HashingAlgorithm`, `@UnsafeDisableCsrfCheckForNonGetMethods('We can disable the CSRF check here, because...')`
* Code reviews immediately show potential problems
* Forbid unsafe settings when `NODE_ENV=production`

---

### Goal 3 - Quick and easy to use

* Internal package allows us to tailor it to our common project architecture
* Package provides a NestJS module that can be imported and configured in <1h
* Extensive documentation - package will be used in _every_ project!
* Proper releases, changelog, ...

---

### Development Strategy
* Pair Programming
* Research existing open source packages...
* ...use them "under the hood" if feasible
* Use generic solutions if possible (e.g. one OpenID connector instead of Google/Microsoft/...)
  * ...if they are spec compliant
  * (looking at you, LinkedIn!)


---
layout: center
---

## The TOTP "conspiracy"

<center><img src="/authentication/ConspiracyMeme.jpg" width="350" /></center>

---

### Background
* Frequent advice: Use "battle-tested" package
  * many common bugs already fixed
  * many people verified the code
  * ...but did they really?
* We wanted to use a totp package...
* ...but then we checked the code

---

### Speakeasy

<center><img src="/authentication/Speakeasy.png" width="550" /></center>

---

### Can you spot the bug?

<center><img src="/authentication/SpeakeasyBug.png" width="450" /></center>

---

### There are actually 3 bugs!

<center><img src="/authentication/SpeakeasyBugHighlight.png" width="450" /></center>

---

### otplib

<center><img src="/authentication/Otplib.png" width="500" /></center>

---

### TOTP Algorithm Requirements
* RFC 4226
> R6 - The algorithm MUST use a strong shared secret. The length of the shared secret MUST be at least 128 bits. This document RECOMMENDs a shared secret length of 160 bits.

* 16 bytes minimum, 20 bytes recommended
* Secrets are base32 encoded (8 characters = 5 byte)
* Minimum: ~25 characters, recommended: 32 characters

---

### otplib - PR#182

<center><img src="/authentication/OtplibPr.png" width="200" /></center>

* A small change in those >30k changed lines...

<center><img src="/authentication/OtplibPrChange.png" width="550" /></center>

* ...no mention in the changelog...
* ...and no response from the maintainer since May 2022

---

### passport-totp

<center><img src="/authentication/PassportTotp.png" width="550" /></center>

---

### passport-totp "complete, working example"

* Can you spot the bugs? (Usage: `utils.randomKey(10);`)
<center><img src="/authentication/PassportTotpExample.png" width="450" /></center>

---

### passport-totp "complete, working example"

* There are a few...

<center><img src="/authentication/PassportTotpExampleBugs.png" width="450" /></center>

---

### Debunking the "conspiracy" - why did this happen?
* Initial Google Authenticator libpam implementation used random 80 bit secrets
  * 80 bits in base32 result in 16 characters
  * Saving 16 characters in UTF-8 takes 16 bytes = 128 bit
  * ...but that's not the required 128 bits of randomness!
* Other packages copied this implementation (not reading RFC 4226)
* Google Authenticator fixed this in 2016

---

### What can we do to prevent this?
* Verify dependencies yourself (especially if security-critical)
  * feasability with 1205 dependencies?
  * reduce dependencies?
* Use (paid) services (auth0, ...) from specialized companies
  * Vendor lock-in?
  * closed source?
* Other ideas?

--- 

### How many projects look like this?

<center><img src="/authentication/Dependency.png" width="200" /></center>

---

### Summary
* Re-usable packages are great, especially for security-critical code
  * even if "just" for specific internal use cases
  * Design goal 1: Secure by default
  * Design goal 2: Hard to use wrong
  * Design goal 3: Quick & easy to use
* TOTP "conspiracy"
  * "Battle tested" does not equal bug-free
  * Verify dependencies, especially if security-critical
