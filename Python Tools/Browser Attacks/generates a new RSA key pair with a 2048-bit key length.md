The script you've provided generates a new RSA key pair with a 2048-bit key length, then exports the public and private keys in PEM format and prints them.

```python
from Crypto.PublicKey import RSA

new_key = RSA.generate(2048)

public_key = new_key.public_key().exportKey("PEM")
private_key = new_key.exportKey("PEM")

print(f"Public Key: {public_key}")
print()
print(f"Private Key: {private_key}")
```


### Explanation:

1. **`from Crypto.PublicKey import RSA`**: 
   - This imports the RSA module from the `pycryptodome` library, which is used for generating RSA keys, encrypting, and decrypting data.

2. **`new_key = RSA.generate(2048)`**: 
   - This generates a new RSA key pair with a key size of 2048 bits. The key pair will consist of a public key and a private key, which are mathematically related.

3. **`public_key = new_key.public_key().exportKey("PEM")`**:
   - This extracts the public key from the generated RSA key pair and exports it in PEM format. PEM (Privacy-Enhanced Mail) is a base64-encoded format commonly used for storing cryptographic keys and certificates.

4. **`private_key = new_key.exportKey("PEM")`**:
   - This exports the private key in PEM format. It's important to store the private key securely because it can be used to decrypt data encrypted with the public key.

5. **`print(f"Public Key: {public_key}")`**:
   - This prints the generated public key to the console in PEM format.

6. **`print(f"Private Key: {private_key}")`**:
   - This prints the generated private key to the console in PEM format.

### Example Output:

```text
Public Key: -----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA... (truncated)

Private Key: -----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA... (truncated)
```

### How to Run:
1. Make sure you have `pycryptodome` installed:
   ```bash
   pip install pycryptodome
   ```
2. Save the code to a `.py` file (e.g., `generate_rsa_keys.py`).
3. Run the file:
   ```bash
   python generate_rsa_keys.py
   ```

### Security Considerations:
- **Private Key**: The private key must be kept secret. If someone gains access to it, they can decrypt any data encrypted with the corresponding public key.
- **Public Key**: The public key can be shared freely and used by others to encrypt data that only the private key can decrypt.
