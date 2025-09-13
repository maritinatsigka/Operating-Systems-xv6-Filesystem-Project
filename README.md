# xv6 Filesystem Extensions

This repository contains my implementation of the **Operating Systems (K22)** second course project (2024–2025) at the National and Kapodistrian University of Athens (NKUA).  
The project extends the **xv6 educational operating system** with new filesystem features:  
- **Large file support** through double-indirect block addressing.  
- **Symbolic links (symlinks)** with a new `symlink` system call.  

---

## 📌 Project Overview

The default xv6 filesystem supports files up to **268 blocks** (≈268 KB) due to its inode structure (12 direct and 1 single-indirect pointer).  
This project enhances xv6 by introducing:  

1. **Large Files via Double-Indirect Blocks**  
   - Added a new **double-indirect block pointer** to the inode structure.  
   - With this change, a file can now reach **65,803 blocks**:  
     - 11 direct blocks  
     - 1 single-indirect block (256 entries)  
     - 1 double-indirect block (256 × 256 entries)  
   - **Modifications include**:  
     - `fs.h`: updated constants (`NDIRECT = 11`, `NDINDIRECT = NINDIRECT * NINDIRECT`, etc.) and extended inode address array.  
     - `bmap()`: modified to resolve double-indirect addressing during read/write.  
     - `itrunc()`: updated to properly free blocks at all levels (direct, indirect, double-indirect).  

2. **Symbolic Links**  
   - Implemented a new system call:  
     ```c
     int symlink(char *target, char *path);
     ```  
   - **Changes include**:  
     - `stat.h`: added `T_SYMLINK` inode type.  
     - `fcntl.h`: added `O_NOFOLLOW` flag for `open()`.  
     - `user.h`, `usys.pl`, `syscall.h`, `syscall.c`: integrated the `symlink` syscall.  
     - `sys_symlink()`: creates a symlink inode and stores the target path.  
     - `sys_open()`: updated to resolve symlinks recursively unless `O_NOFOLLOW` is set.  
   - **Behavior**:  
     - Supports nested symlinks with a maximum resolution depth (to avoid infinite loops).  
     - Handles broken links gracefully by returning errors.  

---

## ⚙️ Features
- **Large file support** up to 65,803 blocks (≈64 MB).  
- **Double-indirect block addressing** integrated into xv6’s filesystem.  
- **Symbolic links** implemented as a new inode type.  
- `open()` transparently resolves symlinks unless `O_NOFOLLOW` is specified.  
- Error handling for broken or circular links (depth threshold).  
- Fully compatible with xv6 utilities (`bigfile`, `symlinktest`, `usertests`).  

---

## Build and run xv6
   ```bash
   make qemu
   ```

---

## Run the provided test programs
   ```bash
   bigfile        # verifies large file support
   symlinktest    # verifies symbolic links
   usertests -q   # regression tests
   ```

## 📄 Notes
- **Filesystem constants (fs.h)**:
  ```bash
   #define NDIRECT 11
   #define NINDIRECT (BSIZE / sizeof(uint))
   #define NDINDIRECT (NINDIRECT * NINDIRECT)
   #define MAXFILE (NDIRECT + NINDIRECT + NDINDIRECT)
   ```
- **Configuration**:
  - Flat directory structure is assumed.
  - One-to-one mapping between inode pointers and allocated blocks.
  - No timestamp checking for overwrites.
  - Symlinks resolution depth limit avoids infinite loops.
