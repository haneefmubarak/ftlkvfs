FKX (ftlkvfs)
=============

FKX is (to be) a filesystem that allows combining a small-object
blob-store with an underlying filesystem optimized for larger
files to yield improved throughput, latency, and fragmentation.
To aid in accomplishing this, FKX uses a three-tier approach to
classify files into sets of small, medium, and large files.

File Classes
------------

Small files are stored directly in the small-object blob-store
and are rewritten entirely upon any new writes. This improves
latency and fragmentation, admittedly with a small throughput
cost. These characteristics make modern KV (key-value) stores an
attractive choice for the small-object blob-store, as they
usually provide near-zero latency and high throughput for writes
alongside low amortized latency and medium throughput for reads,
all while maintaining virtually no fragmentation and saving space
by not requiring individual blocks for small values.

Medium files are split into vblocks (virtual blocks; nonidentical
to physical disk blocks), which are then stored in a CAM (content
addressed map) indexed by the hash of the vblock. When new writes
are issued to a file, new vblocks are written and the references
to altered sections of the file are updated. As a single vblock
may thus be used within multiple files, they are reference counted
and reaped eagerly. This approach has the advantage of maintaining
low latency and high throughput for writes at the slight cost
of read latency and throughput alongside some fragmentation. On
the other hand, space is also minimized due to the variable size
of vblocks and the implicit aligned-range deduplication performed
by the CAM.

Large files are stored directly on the underlying filesystem,
simply passing operations to individual files through with minimal
alteration. These files are stored in a HAMT indexed by inode using
a relatively shallow folder hierarchy. As this is the only method
of storage that interacts directly with the filesystem, this allows
for the selection of a filesystem optimized for large file storage
and operations, which is well represented in modern enterprise
file systems.
