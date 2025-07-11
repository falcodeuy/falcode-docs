---
layout: default
title: Git Crypt
nav_order: 2
parent: Organization
permalink: /organization/git-crypt
---

## 🔐 git-crypt Setup Guide

### 📦 One-time repository setup (by project maintainer)

#### 1. Install `git-crypt`

**macOS:**

```bash
brew install git-crypt
```

**Ubuntu/Debian:**

```bash
sudo apt install git-crypt
```

---

#### 2. Initialize `git-crypt` in the repository

```bash
cd your-project/
git-crypt init
```

---

#### 3. Configure encrypted files

Create or edit a `.gitattributes` file in the root of your repository:

```bash
.production.env filter=git-crypt diff=git-crypt
```

You can add more files or directories, for example:

```bash
infra/secrets/*.env filter=git-crypt diff=git-crypt
```

---

#### 4. Commit and push

```bash
git add .gitattributes
git commit -m "Initialize git-crypt and configure encrypted files"
git push
```

Now the repository is encrypted, but no one (not even you) can decrypt it until a GPG key is added.

---

### Add a user and their GPG key

#### 1. Generate a GPG key (if you don’t have one)

```bash
gpg --full-generate-key
```

Choose:

* Key type: RSA and RSA
* Key size: 4096
* Expiration: 0 (never)
* Provide name/email and passphrase

---

#### 2. Get your GPG key ID

```bash
gpg --list-secret-keys --keyid-format LONG
```

Example output:

```
sec   rsa4096/ABCD1234EF567890 2025-06-23 [SC]
      ABCD1234EF567890ABCDEF1234567890ABCD1234
```

Use the **full fingerprint** (the long one on the second line).

---

#### 3. Add the user to the repository

Share your public key to someone with access

```bash
gpg --export --armor "someone@something.com" > public_key.asc
```

---

#### 4. Add your user

Request someone with access to add your user with the public key

```bash
gpg --import public_key.asc
git-crypt add-gpg-user --trusted someone@something.com
git add .git-crypt/keys
git commit -m "Added user to git-crypt"
git push
```

---

#### 5. To unlock the repository (after clone or pull)

```bash
git-crypt unlock
```

Now you can see and use `.production.env` or other encrypted files.

#### (Optional) Write the private to file

If you need the private key for Github actions run:

```bash
gpg --armor --export-secret-keys D439AF0F1C6343D0DF266B0E78D063180593A3BB > gpg-private-key.asc
```

Where the ID is obtained with:
```bash
gpg --list-secret-keys --keyid-format LONG
```

Copy the `gpg-private-key.asc` content into secret for Github.

