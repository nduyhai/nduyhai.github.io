+++
date = '2025-05-02T21:02:52+07:00'
draft = false
title = '🔐 Secure Password Handling in Go with PassForge: Modern & Legacy Support'
+++

Managing passwords securely is non-negotiable—but what happens when you're modernizing an old system where passwords are hashed with outdated algorithms like `descrypt`?

With [PassForge](https://github.com/nduyhai/passforge), you don’t have to choose between security and compatibility. Its **Delegate Encoder** feature allows you to **seamlessly support legacy formats** while enforcing modern standards for new passwords.

---

## 🌟 Why PassForge?

PassForge is a Go library that abstracts password encoding behind a flexible interface. It supports:

* ✅ `bcrypt`, `argon2`, `scrypt`, `pbkdf2` for modern encoding
* ⚠️ Legacy support for `descrypt`, `ldap`, and others via delegation
* 🔁 A **delegate encoder** to route password verification/encoding based on prefix

---

## 🔎 Inspired by Spring Security’s `DelegatingPasswordEncoder`

The **Delegate Encoder** pattern in PassForge is heavily inspired by **Spring Security**’s `DelegatingPasswordEncoder` introduced in Spring Security 5.

In Spring Security:

```java
PasswordEncoder encoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
```

This produces hashes like:

```
{bcrypt}$2a$10$8Jd...
{pbkdf2}AAA...
{noop}plainTextPassword
```

At runtime:

* Spring **parses the prefix** (`{bcrypt}`, `{pbkdf2}`, etc.).
* Uses the matching encoder for **verification**.
* Always encodes new passwords with a **default algorithm** (usually bcrypt or Argon2).

This approach allows **backward compatibility**, **progressive migration**, and **future flexibility**.

**PassForge adopts this proven design in the Go ecosystem.**

---

## 🧩 The Delegate Encoder: Bridge Between Past and Future

Imagine this:
Your database has thousands of users with passwords stored using legacy `descrypt`. You want new users to use `bcrypt`, but you can't force everyone to reset their password on day one.

### ✅ Enter `DelegateEncoder`

PassForge provides a way to **map encoders by prefix** like `{bcrypt}`, `{descrypt}`, `{argon2}`, etc.

### 🔧 Sample Use Case

```go
encoders := map[string]passforge.PasswordEncoder{
    "bcrypt":  passforge.NewBcryptEncoder(),
    "descrypt": passforge.NewDescryptEncoder(), // deprecated, legacy
}

delegate := passforge.NewDelegatingPasswordEncoder("bcrypt", encoders)

// Encode a new password (uses default: bcrypt)
hash, _ := delegate.Encode("newSecurePassword")
// hash will look like: {bcrypt}...

// Verify legacy password (uses descrypt if prefix is {descrypt})
delegate.Verify("legacyPass", "{descrypt}abc123...")  // true or false
```

This gives you:

* Backward compatibility for old users
* Forward migration path without downtime
* Transparent encoding/verification via a unified interface

---

## 🔐 Security Recommendation

While it's okay to **support verification for deprecated encoders**, you should **never use them for new password storage**. PassForge helps enforce this by:

* Allowing you to define the default encoder (e.g. `bcrypt`)
* Encapsulating legacy encoders only under explicit prefixes

You can even build a **"rehash on login"** strategy:
If a user logs in with a legacy-encoded password, verify it with `{descrypt}`, but on success, **re-encode with `{bcrypt}`** and update the DB.

---

## 📦 Installation

```bash
go get github.com/nduyhai/passforge
```

---

## 🧠 Final Thoughts

Legacy systems shouldn’t be a roadblock to modern security. PassForge's **delegate encoder** design, inspired by **Spring Security’s DelegatingPasswordEncoder**, lets you gradually sunset weak algorithms while adopting strong, future-proof hashing strategies.

If you’re dealing with authentication migrations or dual encoder support, PassForge might just be your best friend.
