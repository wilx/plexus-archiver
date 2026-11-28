# Extraction paths and symbolic links

Extractors resolve existing path components before checking that output stays
inside the destination directory. Symbolic links are followed before processing
subsequent `..` components, including when the output file does not exist yet.
The same rules resolve the configured destination directory and classify it as
a file or directory.

Existing parent directories are cached within one extraction. Cached entries
are reused only after a fresh check of their filesystem identity without
following symbolic links. Missing paths and symbolic links are not cached, and
the cache is cleared after success or failure. Resolution stops after 64
symbolic-link expansions, rejecting cyclic or excessively long paths.

## Archive formats and compatibility

TAR, including its compressed variants, and ZIP-based formats such as JAR,
WAR, and EAR use the shared entry extractor.

- Intermediate symbolic links are followed before
  subsequent parent components are processed. Containment is checked even
  when the destination leaf does not exist. The overwrite check and the write
  both use the resolved destination.
- Regular-file and directory entries reject a symbolic link at the final
  destination component, including links to a contained target and dangling
  links. If a directory entry is selected for writing, a collision with an
  existing regular file fails. Failures while resolving or creating a parent
  directory are reported.
- Symbolic links are created using the target stored in the archive. The
  extractor skips permission changes for link entries and attempts to set the
  modification time on the link itself. Link timestamps are best effort and
  depend on filesystem support. The target's permissions and timestamps are
  preserved.

Standalone GZIP, BZIP2, Snappy, XZ, and Zstandard extractors use the shared
destination validation and may report invalid destination paths during
validation. Each uses a format-specific stream copy to its configured output
file. The per-entry rules above apply to the archive readers.
