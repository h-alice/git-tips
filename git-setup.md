# Setup GPG
## Install GPG
### MacOS
1. Download and install from [GPG Suite](https://gpgtools.org).
2. Open terminal and run `gpg --version` to verify installation.
3. Run `gpg --full-generate-key` to generate a new key.
    ```bash
    gpg --full-generate-key
    ```
    Select (1) RSA and RSA (default).
    
    Enter the key size. 4096 is recommended.

    Enter the key expiration. If don't want the key to expire, press Enter.

    Enter user information, Includes name, email address, and comment (optional).

    Enter a passphrase to protect the key, it can be skipped but its highly recommended to set a passphrase.
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
gpg --export-secret-subkeys -armor EE833747
```

We may be asked for the passphrase to unlock the key. Then the subkey will be exported in ASCII format.

Transfer the private key to the target machine. *Keep it safe*!



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