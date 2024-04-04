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
Locate the master key ID by running `gpg --list-secret-keys --keyid-format LONG`.
```bash
gpg --list-secret-keys --keyid-format LONG
```
```plaintext
sec   rsa4096/9CB37AAD 2024-04-04 [SC]
      561DE61782D69B519C1512CB33A39CA69CB37AAD
uid         [ultimate] Example (For test only) <example@lazylabs.cc>
ssb   rsa4096/D269A838 2024-04-04 [E]
```
The master key ID is `9CB37AAD`.

## Create Signing Subkey
Create a signing subkey by running `gpg --expert --edit-key <master-key-id>`.

The `--expert` flag is used to enable expert mode, which allows some advanced operations. (Including signing on expired keys!)

```bash
gpg --expert --edit-key 9CB37AAD
```

### Add Key
Add a new signing subkey by running `addkey` command.

```plaintext
gpg> addkey
```


# Debug and Troubleshooting