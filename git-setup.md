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

# Debug and Troubleshooting