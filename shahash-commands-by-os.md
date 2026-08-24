# SHA-256 File Hash Verification

Use the following commands to calculate the SHA-256 hash of a file.

## Windows

```powershell
certutil -hashfile "FILE_PATH" SHA256
```

Example:

```powershell
certutil -hashfile "message.txt" SHA256
```

---

## Linux

```bash
sha256sum "FILE_PATH"
```

Example:

```bash
sha256sum "message.txt"
```

---

## macOS

```bash
shasum -a 256 "FILE_PATH"
```

Example:

```bash
shasum -a 256 "message.txt"
```

---

## Verify the Hash

Compare the generated hash with the SHA-256 hash provided by the file publisher.

The hashes must match exactly:

```text
Expected:
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855

Calculated:
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855
```

A matching hash confirms that the downloaded file is byte-for-byte identical to the file associated with the published hash.

However, a matching hash does **not** prove that the file is trustworthy if both the file and the published hash came from the same compromised source. When available, verify a digitally signed checksum file using GPG.
