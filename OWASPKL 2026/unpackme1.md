# unpackme1 - Medium

This challenge was still a packed binary, but it added a simple anti-unpacking trick. The goal was the same as before: identify the packer, repair the binary if needed, unpack it, and extract the hidden flag from the unpacked file.

## Finding the Corrupted Signatures

I started by locating the packer signatures in the file:

```bash
strings -a -t x unpackme1 | grep -E "UPX!|VQY!"
```

That showed the normal UPX header at one offset and two altered signature blocks elsewhere. The corrupted bytes had been changed from `UPX!` to `VQY!`, which breaks the unpacker until they are repaired.

## Manual Byte Patching

To fix the binary safely, I opened it in a hex editor and jumped to the corrupted offsets one by one.

At each damaged location, the bytes `56 51 59 21` needed to be restored to `55 50 58 21`, which converts `VQY!` back to `UPX!`.

After patching both corrupted blocks, I saved the file and exited the editor.

## Unpacking

Once the signatures were restored, UPX could decompress the binary normally:

```bash
upx -d unpackme1
```

## Extracting the Flag

The unpacked binary contained the flag as a plaintext string, so the last step was to search for the challenge format:

```bash
strings unpackme1 | grep -E "OWASPKL\{.*\}"
```

That returned the final flag:

OWASPKL{Unpackm3_4mat3ur0923257}
