# lockbox

## Step-by-Step CTF Writeup

### 1. Attack Surface and Static Analysis

I started by running static analysis on the binary with tools like `strings` and a hex editor. That exposed the raw text sections and hardcoded assembly-style fragments inside the program.

Right next to the normal `--unlock <code>` flow, I found an undocumented hidden argument: `--emergency`. That immediately told me I did not need to brute-force or break a 64-character HMAC key. The real goal was to recover the fallback payload instead.

### 2. Identifying Stack Allocation Behavior

The binary dump contained a mixed block of text and stack-offset-looking tokens:

```text
OS=g PTE1 H=00@ }LM33HD5H D$0H _A0Z3Y_3H D$8H 1G0E_mCmH D$@H 3{YXCFNJH D$PBH D$PH
```

Tokens like `D$0H`, `D$8H`, and `D$@H` line up with x86-64 stack offsets, the same sort of layout you would expect from instructions like `mov [rsp+0x0], ...`. That meant the compiler split the flag into separate 8-byte chunks and stored them sequentially on the stack.

### 3. Inverting the Obfuscation Pipeline

The developer used three layers of obfuscation: splitting, reversing, and ROT13. To get the flag, I inverted that pipeline step by step.

#### Phase A: Isolate the Memory Chunks

I extracted the exact 8-byte text fragments associated with the stack memory layout:

- Chunk 1: `3{YXCFNJ`
- Chunk 2: `1G0E_mCm`
- Chunk 3: `_A0Z3Y_3`
- Chunk 4: `}LM33HD5`

#### Phase B: Perform In-Place Chunk Reversal

Reversing the entire string globally breaks the format, so I reversed each chunk individually to match how the stack data was written:

- `3{YXCFNJ` -> `JNFCXY{3`
- `1G0E_mCm` -> `mCm_E0G1`
- `_A0Z3Y_3` -> `3_Y3Z0A_`
- `}LM33HD5` -> `5DH33ML}`

After concatenating those reversed chunks in stack order, the reconstructed string was:

```text
JNFCXY{3mCm_E0G13_Y3Z0A_5DH33ML}
```

#### Phase C: Apply ROT13 Decryption

The final layer was ROT13. Applying ROT13 to the correctly reconstructed string gives the decoded result:

- `JNFCXY{3` -> `OWASPKL{3`
- `mCm_E0G1` -> `zPz_R0T1`
- `3_Y3Z0A_` -> `3_L3M0N_`
- `5DH33ML}` -> `5QU33ZY}`

### 4. Final Flag Extraction

Putting the decrypted chunks together yields the final flag:

`OWASPKL{3zPz_R0T13_L3M0N_5QU33ZY}`

## Summary

The key insight was that the binary was not hiding the flag with strong cryptography alone. The real trick was understanding how the compiler arranged the bytes in memory. Once I treated the data as separate stack chunks, reversed each chunk in place, and applied ROT13, the flag came out cleanly.
