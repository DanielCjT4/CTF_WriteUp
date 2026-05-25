# unpackme0 - Easy

The first step was identifying the packer. This binary was packed with UPX, so the fix was straightforward:

```bash
upx -d <file>
```

After unpacking the binary, I calculated the MD5 hash of the unpacked file:

```bash
certutil -hashfile unpackme0 md5
```

The result was:

```text
1cc6a3b62cac36ab18e0c4685a7f4bdf
```

Flag: OWASPKL{1cc6a3b62cac36ab18e0c4685a7f4bdf}
