# Introduction

This guide is for setting up GPG for signing commits in Git.

### Why Should I Sign My Commits?

Signing commits with GPG enhances the security, integrity, and trustworthiness of software development workflows. 

By providing a means to verify commit authenticity and protect against tampering, GPG signatures play a crucial role in ensuring the reliability and accountability of code repositories.

For more information, check out the "Non-repudiation" in cybersecurity. It's a fun topic!

# Setup GPG

## Install GPG

### MacOS

1. Download and install from [GPG Suite](https://gpgtools.org).

2. Open terminal and run `gpg --version` to verify installation.

### Windows

1. Download and install from [Gpg4win](https://gpg4win.org).

2. Open command prompt and run `gpg --version` to verify installation.

### Linux

Linus is the easiest one to install GPG. Since it's already installed in most distributions.

- Open terminal and run `gpg --version` to verify installation.

# Generate Master Key
Run `gpg --full-generate-key` to generate a new key.

```bash
gpg --full-generate-key
```

- Select (1) RSA and RSA (default).

- Enter the key size. 4096 is recommended.

- Enter the key expiration. If don't want the key to expire, press Enter.

- Enter user information, Includes name, email address, and comment (optional).

- Enter a passphrase to protect the key, it can be skipped but its highly recommended to set a passphrase.

Now the key will be generated. We can list the keys by running `gpg --list-secret-keys --keyid-format SHORT`.

```bash
gpg --list-secret-keys --keyid-format SHORT
```

```plaintext
sec   rsa4096/9CB37AAD 2024-04-04 [SC]
      561DE61782D69B519C1512CB33A39CA69CB37AAD
uid         [ultimate] Example (For test only) <example@lazylabs.cc>
ssb   rsa4096/D269A838 2024-04-04 [E]
```

In most case, the **master key** already has the capabilities to sign and encrypt. We can just export the public key and add it to the git service provider.

But for best practices, we'll create a **signing subkey** for commit signing.

If the private of **master key** is lost, the whole key pair should be revoked. But if the private of **signing subkey** is lost, only the subkey needs to be revoked.

You can also skip to the [Export Key](#export-key) section if you don't want to create a signing subkey.

# Create Signing Key

## Select Master Key

Locate the master key ID by running `gpg --list-secret-keys --keyid-format SHORT`.

```bash
gpg --list-secret-keys --keyid-format SHORT
```

```plaintext
sec   rsa4096/9CB37AAD 2024-04-04 [SC]
      561DE61782D69B519C1512CB33A39CA69CB37AAD
uid         [ultimate] Example (For test only) <example@lazylabs.cc>
ssb   rsa4096/D269A838 2024-04-04 [E]
```

The master key ID is `9CB37AAD`.

In the following process, we'll use the master key as example.
## Create Signing Subkey
Create a signing subkey by running `gpg --expert --edit-key <master-key-id>`.

The `--expert` flag is used to enable expert mode, which allows some advanced operations. (Including signing on expired keys!)

```bash
gpg --expert --edit-key 9CB37AAD
```

## Add Key
Add a new signing subkey by running `addkey` command.

```plaintext
gpg> addkey
```

```plaintext
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
  (14) Existing key from card
```

We choose (8) here to choose our own capabilities.

```plaintext
Possible actions for a RSA key: Sign Encrypt Authenticate 
Current allowed actions: Sign Encrypt 

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished
```

We can toggle the capabilities by pressing the corresponding key.

For commit signing, we must make sure the `Sign` capability is enabled.

After setting the capabilities, determine the key size.

```plaintext
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 
```

Choose whatever key size we want. Now we'll be asked for the expiration date.

Whether it's not recommanded to set an expiration date for the **master key**, it's recommanded to set an expiration date for the **signing subkey**.

After setting the expiration date, we'll be asked for the passphrase.

And finally, the subkey will be created. Use `save` to save the changes and exit.

# Export Key

## Gather Information and Locate Key ID

Locate the key ID by running `gpg --list-secret-keys --keyid-format SHORT`.

```bash
gpg --list-secret-keys --keyid-format SHORT
```

```plaintext
sec   rsa4096/9CB37AAD 2024-04-04 [SC]
      561DE61782D69B519C1512CB33A39CA69CB37AAD
uid         [ultimate] Example (For test only) <example@lazylabs.cc>
ssb   rsa4096/D269A838 2024-04-04 [E]
ssb   rsa3072/EE833747 2024-04-04 [SE]
```

The key ID of the subkey we just created is `EE833747`.

## Public Key
Export the public key by running `gpg --armor --export <master-key-id>`.
    
    ```bash
    gpg --armor --export 9CB37AAD
    ```

We'll get the public key in ASCII format. And add to the git service provider.

The UI may be differ from one service provider to another, but the process is the nearly the same.

## Transfer Private Sub-Key
Export the private key by running `gpg --export-secret-subkeys -armor <sub-key-id>`.
    
```bash
gpg --export-secret-subkeys -armor EE833747!
```
Notice the `!` at the end of the key ID. It's used to indicate the key is a subkey.

We may be asked for the passphrase to unlock the key. Then the subkey will be exported in ASCII format.

Transfer the private key to the target machine. *Keep it safe*!

# Git Settings
Finally we get to the git settings. That should be the easiest part.

## Git Configuration
First, the most important part is check if our e-mail address matches the e-mail address in the GPG key.

Or otherwise, the commit will be marked as `Unverified`.

```bash
git config --get --global user.email 
```

Set the email address if it's not set.

```bash
git config --global user.email "<email>"
```

If the e-mail address is correct, we can move on to the next step.

## Add GPG Path to Git
Add the GPG path to git by running `git config --global gpg.program gpg`.

It may be different depending on the environment. So the following command may be altered to match our requirements.

```bash
git config --global gpg.program $(which gpg)
```

The `$(which gpg)` command is used to get the path of the GPG executable.

The following is an example for Windows, which I used in my environment.

```cmd
git config --global gpg.program "C:\Program Files (x86)\GnuPG\bin\gpg.exe"
```

## Add Our GPG Key to Git
Add the GPG key to git by running `git config --global user.signingkey <key_id>`.

```bash
git config --global user.signingkey EE833747
```

## Enable Commit Signing
Enable commit signing by running `git config --global commit.gpgsign true`.

```bash
git config --global commit.gpgsign true
```

Now we can try to commit and see if it's signed.

# Best Practices
- **Master Key**: 
    Keep the master key offline and secure. For example, store it on a USB drive and keep it in a safe place.

    For daily use, we'll use the subkey.



# Debug and Troubleshooting

### My commit is marked as `Unverified`!

- Make sure the email address in the commit matches the email address in the GPG key.

- We can check the committer email by running `git log --pretty=format:"%h %ae"`.

- Also check the git config by running `git config --get user.email`.
    
- And, most importantly, make sure the GPG key is added to the git service provider.

### I can't even commit!!!

- We can debug the git process by running `GIT_TRACE=1 git commit`.

    Linux-based systems:
    ```bash
    GIT_TRACE=1 git commit
    ```

    Windows:
    ```cmd
    set GIT_TRACE=1
    git commit
    ```

    Maybe it's not related to GPG at all. 
    
    We'll discuss GPG-related issues in the following section.

### It shows 'No secret key' error.

- Make sure the proper key is selscted.

- If it still shows the same error, check the key capabilities by running `gpg --edit-key <key-id>`.

- The key should include the `S` capability, indicating it's a signing key.

### Debug the GPG process by test signing.

- We can debug the GPG process by running `gpg -bsau <key-id>`.

    - The `-b` flag is used to create a detached signature.

    - The `-s` flag is used to sign the data.

    - The `-a` flag is used to create an ASCII-armored output.

    - The `-u` flag is used to specify the key ID, followed by the key ID we want to use.

    ```bash
    gpg -bsau EE833747
    ```

- Then we can enter whatever text we want to sign. Then use `Ctrl+D` to finish the input.

- If everything is correct, we'll get the signature in ASCII format.

# References
[\[1\] GitHub Docs](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key)
