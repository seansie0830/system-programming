# Ch1 Quiz
## 4 Architecture-Centric Optimization Challenges

### Challenge 1: PDF-Style Tail-Indexed Binary Storage ($O(1)$ XRef Reader)

* **Architectural Link**: Disk Seeking vs. In-Memory Index Caching, Spatial Locality.
* **Concept**: PDF engines achieve instant random access to any page or object by placing the cross-reference table (XRef) at the very end of the file (`startxref`). Instead of parsing sequentially from byte 0, the reader jumps to the file tail to load offsets, then jumps directly to target objects.
* **Tasks**:
1. **Packer**: Write a program that writes variable-length records sequentially into `data.bin`. At the end of the file, append a Table of Contents (TOC) array storing each record's `offset` and `length`. Store the 4-byte TOC start offset at the absolute end of the file.
2. **Reader**: Given a record ID, locate the TOC start offset using `lseek(fd, -4, SEEK_END)`, jump to the target record's index entry, and fetch the record data.


* **Optimization Comparison**:
* *Naive Approach*: Issue two `lseek` calls per query (one to read the TOC entry from disk, one to fetch the record data).
* *Optimized Approach*: Preload the TOC into a heap array at initialization, reducing query I/O to a single `lseek` and `read`.



---

### Challenge 2: System Call Penalty & Buffer Sizing (Context Switch Cost)

* **Architectural Link**: User/Kernel mode transitions (Trap / Privilege level switch Ring 3 $\to$ Ring 0), CPU cache pollution.
* **Concept**: Every `read()` or `write()` triggers a system call requiring register preservation and kernel trap handling. This experiment demonstrates massive performance gaps using an identical $10\text{ MB}$ workload without heavy resource usage.
* **Tasks**:
1. Implement a file copy and byte-frequency counter using standard read loops.
2. Benchmark execution time across varying buffer sizes:
* $1\text{ Byte}$ (Byte-by-byte I/O — worst-case syscall overhead)
* $64\text{ Bytes}$ (L1 Cache line matching)
* $4096\text{ Bytes}$ ($4\text{ KB}$ — Standard Virtual Memory Page boundary)
* $65536\text{ Bytes}$ ($64\text{ KB}$)




* **Optimization Insights**:
* Observe the steep performance jump between $1\text{ Byte}$ and $4\text{ KB}$, followed by diminishing returns beyond $4\text{ KB}$.
* Demonstrate alignment with OS Page Cache allocation boundaries.



---

### Challenge 3: In-Place Updates vs. Append-Only Write-Ahead Logging (WAL)

* **Architectural Link**: Sequential I/O throughput vs. Random access penalties, Page Cache dirty-page management.
* **Concept**: Sequential writes leveraging `O_APPEND` optimize throughput by maximizing write buffering and minimizing disk arm/block lookup latency compared to random in-place updates.
* **Tasks**:
1. Design a fixed-size Key-Value structure file (e.g., 128 bytes per entry).
2. **Mode A (In-Place Overwrite)**: Compute `offset = ID * 128` and execute `lseek(fd, offset, SEEK_SET)` to update entries in-place.
3. **Mode B (Append-Only WAL)**: Always append new updates to the end of the file using `O_APPEND`, maintaining an in-memory hash table of `ID -> Latest Offset`.


* **Optimization Insights**:
* Measure write latency under high random mutation loads.
* Explain why production databases (such as SQLite WAL, LevelDB, and LSM-Trees) default to sequential append architectures over in-place seeking.



---

### Challenge 4: Sparse File Allocation and Hole Punching

* **Architectural Link**: File System Inode Extent Trees, Lazy Block Allocation.
* **Concept**: When `lseek` advances beyond the current file end without writing intermediate bytes, the OS avoids allocating physical disk blocks for the gap, storing only metadata updates.
* **Tasks**:
1. Create a sparse file showing an apparent logical size of $1\text{ GB}$ while occupying only $4\text{ KB}$ of physical disk space (write 1 byte at index 0, `lseek` forward by $1\text{ GB}$, and write 1 final byte).

### Challenge 5: Atomic File Updates & Durability (The Power Loss Simulator)

* **Architectural Link**: Volatile Page Cache vs. Non-Volatile Storage (Disk Controller Write Barrier), Metadata Journaling, Atomicity via Inode Swapping.
* **Concept**: `write()` only copies data to the kernel's OS Page Cache; it does not guarantee persistence. If power cuts or the process crashes mid-write, data is corrupted (torn write). To guarantee "All-or-Nothing" atomicity, databases write to a temporary file, flush to hardware, and replace the target atomically.
* **Tasks**:
1. **Direct Overwrite (Unsafe)**: Implement a saver that uses `open("data.txt", O_WRONLY | O_TRUNC)` and writes data in chunks.
2. **Atomic Writer (Safe)**:
* Write to a unique temporary file: `open("data.txt.tmp", O_WRONLY | O_CREAT | O_TRUNC, 0644)`.
* Flush both data and metadata from kernel page cache to physical disk using `fsync(fd)`.
* `close(fd)` and swap filenames atomically using `rename("data.txt.tmp", "data.txt")`.
* *(Bonus)* Call `fsync()` on the parent directory file descriptor to persist the directory entry update.


3. **Crash Test**: Insert random `raise(SIGKILL)` or `_exit(1)` statements midway through both implementations to verify that the atomic version leaves either the old intact file or the fully written new file, but never a corrupted/truncated state.


* **Optimization Trade-off**:
* Measure the high latency penalty of forcing physical disk syncs (`fsync`) versus raw buffered memory writes (`write` only).
---
