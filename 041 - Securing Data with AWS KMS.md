# Securing Data with AWS KMS

---

<aside>

**Purpose:** Create a customer managed KMS key, then encrypt and decrypt a sensitive file using the AWS CLI to verify data can be protected and recovered.

</aside>

---

### Procedure

1. Create a customer managed KMS key (Console):
    1. Sign in to the AWS Management Console and open AWS Key Management Service (KMS).
    2. In the navigation pane, choose Customer managed keys.
    3. Choose Create key.
    4. For Key type, choose Symmetric.
    5. For Key usage, keep Encrypt and decrypt.
    6. Choose Next.
    7. Type an alias for the KMS key (example: `alias/devops-KMS-Key`).
    8. Review the settings and choose Finish.
2. Encrypt the sensitive file (CLI):
    1. Encrypt and Base64 decode the output into a binary file:

```bash
aws kms encrypt \
	--key-id alias/devops-KMS-Key \
	--plaintext fileb:///root/SensitiveData.txt \
	--output text \
	--query CiphertextBlob | base64 --decode > /root/EncryptedData.bin
```

1. Confirm the encrypted output file exists: `/root/EncryptedData.bin`.
2. Decrypt the encrypted file (CLI):
    1. Decrypt and Base64 decode the plaintext into a new file:

```bash
aws kms decrypt \
	--ciphertext-blob fileb:///root/EncryptedData.bin \
	--output text \
	--query Plaintext | base64 --decode > /root/DecryptedData.txt
```

1. Verify decrypted data matches original:
    1. Compare the original and decrypted files:

```bash
diff /root/SensitiveData.txt /root/DecryptedData.txt
```

### Quick check

- **Expected result:** `diff` returns no output, confirming encryption and decryption succeeded and the files are identical.