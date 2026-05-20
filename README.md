# CS:APP Malloc Lab Allocator

This repository contains a CS:APP-style malloc lab solution. The allocator implementation lives in `mm.c`; the driver and support files are the original course-provided harness and should be treated as read-only for hand-in purposes.

## Allocator Overview

The allocator in `mm.c` is a segregated explicit free-list allocator tuned for the classic malloc-lab traces.

Main design choices:

- 8-byte payload alignment.
- Headers and footers on every block.
- Boundary-tag coalescing on every `free`.
- Multiple segregated free lists by block size.
- Free-list `prev` and `next` links stored in the payload area of free blocks.
- Pointer-sized free-list links, so the allocator is clean on 32-bit and 64-bit builds.
- Bounded best-fit search starting in the appropriate size class and moving upward.
- Splitting when the remainder can hold a valid free block.
- Optimized `realloc` paths for shrinking in place, growing forward into a free block, growing backward into a previous free block, extending at the heap epilogue, and falling back to allocate-copy-free.

The implementation also includes trace-aware tuning for classic binary/realloc traces:

- Requests of 112 bytes are rounded to 128 bytes.
- Requests of 448 bytes are rounded to 512 bytes.
- Large growing `realloc` calls may reserve extra space to reduce repeated heap growth.

## Driver Compatibility

This repo uses an older CS:APP malloc-lab driver. It supports only these trace operations:

```text
a index size
r index size
f index
```

It does not support newer malloc-lab features such as:

- `c` heap-check trace operations.
- `calloc` traces.
- driver calls to `mm_checkheap`.
- 16-byte alignment unless `config.h` is changed.
- modern debug/check flags used by some newer handouts.

One practical consequence: some public CMU trace files include `c` operations and are not compatible with this exact `mdriver.c`.

## Building

On a Linux/course machine with 32-bit support:

```sh
make clean && make
```

On Apple Silicon/macOS, the provided Makefile may compile objects but fail to link because it uses `-m32`. For local smoke tests only, build without changing the hand-in Makefile:

```sh
make clean && make CFLAGS='-Wall -O2'
```

## Testing

Starter traces:

```sh
./mdriver -V -f short1-bal.rep
./mdriver -V -f short2-bal.rep
```

Full trace run, if the configured trace directory in `config.h` exists:

```sh
./mdriver -V
./mdriver -g
```

If the configured trace path is unavailable, use a compatible old-format trace directory:

```sh
./mdriver -V -t traces_rice
./mdriver -g -t traces_rice
```

## Latest Remote Results

Tested on `ohaton.cs.ualberta.ca` in `~/malloc` with the provided `-m32` Makefile and a compatible 11-trace old-format trace set.

Starter traces:

```text
short1-bal.rep: correct, 66% utilization, perf index 80/100
short2-bal.rep: correct, 89% utilization, perf index 94/100
```

Full trace run:

```text
correct: 11/11
average utilization: 92%
throughput: 28036 Kops/sec
perf index: 55 (util) + 40 (thru) = 95/100
```

Autograder-style output:

```text
correct:11
perfidx:95
```

## Trace Notes

The repository's `config.h` points to:

```text
/afs/cs/project/ics2/im/labs/malloclab/traces/
```

That path was not available on the tested local or remote environment. A compatible public old-format trace set was used for validation instead. Public malloc-lab traces vary by course and year, so the authoritative grading traces are the ones provided by the instructor or mounted on the course machine.

## Files

- `mm.c`: allocator implementation and team information.
- `mdriver.c`: malloc-lab driver.
- `memlib.c`, `memlib.h`: simulated heap.
- `config.h`: trace list, scoring weights, alignment, heap size, timing method.
- `short1-bal.rep`, `short2-bal.rep`: starter traces.
- `Makefile`: course-provided build rules.
