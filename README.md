# pitchscope_player_heapoverflow1
# pitchscope_player_heapoverflow1
PoC and artifacts for heap buffer overflow vulnerability #1 in PitchScope Player.

## Vulnerability Summary
- **Vulnerability Type**: Heap Buffer Overflow (Out-of-bounds Write)
- **CWE**: CWE-787
- **Affected Component**: `BitSourceStatic::Copy_From_Transform_Samples`
- **Crash Location**: `vulnb_target.cpp` line 97
- **Allocation Site**: `BitSourceStatic::Create_Secondary_Buffer` at `vulnb_target.cpp` line 41
- **Crash Behavior**: 1-byte out-of-bounds write immediately after the end of a 200-byte heap-allocated secondary buffer.

The `Create_Secondary_Buffer` function allocates a 200-byte heap buffer. The `Copy_From_Transform_Samples` function copies transformed audio samples into this buffer without proper bounds checking. Crafted input causes a 1-byte write past the buffer boundary, triggering heap memory corruption.

## Reproduction
### Build
Compile with AddressSanitizer for crash detection:
```bash
clang++ -g -fsanitize=address vulnb_target.cpp -o poc_vulnb

Run PoC
./poc_vulnb

Expected ASan Crash

=================================================================
==8020==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x611000000108 at pc 0x5d5372a751a4 bp 0x7ffd710cb530 sp 0x7ffd710cb528
WRITE of size 1 at 0x611000000108 thread T0
0x611000000108 is located 0 bytes to the right of 200-byte region [0x611000000040,0x611000000108)

Full crash log is stored in asan_crash.log.

Root Cause
Missing bounds check before writing bytes into the destination heap buffer. The copy routine does not validate remaining available space and can write beyond the allocated buffer size.
Suggested Fix
Add bounds checking inside Copy_From_Transform_Samples:
Track total allocated buffer length and remaining writable bytes.
Stop copy operation before writing past the buffer boundary.
Validate input sample count against buffer capacity before starting sample copy.
Artifacts
vulnb_target.cpp: Vulnerable source code
asan_crash.log: Raw ASan crash report
ghsa.md: GitHub Security Advisory text
cve_description.txt: CNA submission description
Credits
Vulnerability discovered and reported by Lidi Jie

