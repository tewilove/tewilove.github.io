## MTK V5 DA arbitrary code execution

This is an combination of several flaws in MTK V5 DA\(aka Download Agent\).

### DA boot sequence

boot ROM -> PRELOADER -> DA\(Stage 1\) -> DA\(Stage 2\)

Note that PRELOADER is optional depends on vendor configuration.

### DA(Stage 1) memory layout

| Start      | End        | Use                     |
| ---------- | ---------- | ----------------------- |
| 0          | 0x??????   | boot ROM                |
| 0x00101000 | 0x00110000 | DA1 heap                |
| 0x00200000 | 0x002xxxxx | DA1 text/data/bss/stack |

### The bugs

1. Missing size check in DRAM_INIT command handler.

   Pseudo code of this command handler looks like:

   ```c
   ...
   usb_recv(&size, sizeof(size));
   buffer = malloc(size); // Size passed to malloc() directly.
   usb_recv(buffer, size); // What if buffer == NULL?
   ...
   ```

2. Improper page table setup.

   Maybe it happened to be a tired day designing a precise and unprivileged memory map for

   PRELOADER and DA. MTK decided to create an 1:1 Level 1 block at 0 with 1 GB size and made

   it RWX, and this worked great in the past days.

3. DA1 memory layout makes exploitation easy and stable.

   Let size = 0x00110000 in DRAM_INIT, since the DA1 heap is fairly small, malloc() will certainly

   fail and return NULL, consequent call to usb_recv() will skew up the whole heap with full user

   controlled data, essentially leading to arbitrary code execution.

### Response from MTK

```
In MTK's V5 DA, the issue already fix on 2023/11/16 (codebase in Cleanroom), and
In MTK's V6 DA, there is no related code flow.
```
