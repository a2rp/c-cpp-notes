[Back to notes index](../README.md)

| [Previous: C structures, unions, and enumerations](07-c-structures-unions-and-enumerations.md) | [Notes index](../README.md) | [Next: C preprocessing and macros](09-c-preprocessing-and-macros.md) |
| --- | --- | --- |
# 8. C files, streams, and errors

The C standard I/O library represents an open stream with FILE. Streams provide buffered input and output for files and other sources. Every operation can fail, so check return values and release the stream even when an earlier operation fails.

## Open and close files

fopen opens a file using a mode such as read, write, or append. A write mode can truncate an existing file, so use it only when replacement is intended. A null return indicates failure. Use perror or another error-reporting mechanism to provide useful diagnostics without exposing sensitive paths or data.

fclose flushes buffered output and closes the stream. It can fail, so check its result when successful persistence matters. Do not reuse a FILE pointer after it has been closed.

```c
#include <stdio.h>

static int write_status_file(const char *path)
{
    FILE *file = fopen(path, "w");
    if (file == NULL) {
        perror("fopen");
        return 1;
    }

    int failed = fprintf(file, "status=ready") < 0;
    if (fclose(file) != 0) {
        perror("fclose");
        failed = 1;
    }
    return failed;
}

int main(void)
{
    return write_status_file("status.txt");
}
```

The example writes a small text file in the current working directory. Production code should choose a controlled path, define overwrite behavior, and report whether close or flush failed.

## Read a text stream

fgetc returns either a character converted to int or EOF. The int type is required so EOF remains distinguishable from every possible unsigned char value. After a loop ends on EOF, check ferror to distinguish an input error from normal end of file.

```c
#include <stdio.h>

static int print_file(const char *path)
{
    FILE *file = fopen(path, "r");
    if (file == NULL) {
        perror("fopen");
        return 1;
    }

    int character;
    while ((character = fgetc(file)) != EOF) {
        if (putchar(character) == EOF) {
            perror("putchar");
            fclose(file);
            return 1;
        }
    }

    int failed = ferror(file) != 0;
    if (fclose(file) != 0) {
        perror("fclose");
        failed = 1;
    }
    return failed;
}
```

This example prints text bytes through the active C locale and stream mode. Applications that need a specific text encoding should define and validate that encoding explicitly.

## Text and binary modes

A text stream can have implementation-specific text transformations. A binary stream requests byte-oriented handling, but writing C objects directly still does not create a portable data format. Encode fields with documented widths, byte order, and validation rather than serializing an entire struct.

fread and fwrite report the number of complete elements transferred. Check that count against the requested count and use feof or ferror to understand a short read. A partial record may require recovery rather than being treated as valid input.

## Resource cleanup and errors

A file stream is a resource. Close it exactly once on every path after a successful fopen. When code acquires several resources, use a cleanup section or small helper functions to make releases visible. Do not discard a write error just because the next cleanup operation succeeded.

errno can provide additional system error information after some failures, but it should be read only when the called function documents that it sets errno. Error messages should explain the failed operation and enough context for recovery, without printing secrets.

Paths also require care. Avoid concatenating untrusted input into a filesystem path without validation. Use a controlled directory and understand permissions, symlink behavior, and race conditions for security-sensitive files.

## Key points

- Check fopen, read, write, flush, and close results.
- Use int to store fgetc results so EOF can be represented.
- Distinguish normal end of file from a stream error with ferror.
- Define overwrite, encoding, and binary format behavior explicitly.
- Release every successfully opened stream exactly once.

## Practice

1. Modify the write example to append instead of replace, and explain when that is appropriate.
2. Add a read error path and verify that fclose still runs after an earlier failure.
3. Explain why storing fgetc directly in char can make EOF ambiguous.
4. Design a portable file format for two integers and a string without writing the raw struct bytes.

## References

- [GNU C Library I/O on streams](https://www.gnu.org/software/libc/manual/html_node/Streams.html)
- [GNU C Library opening streams](https://www.gnu.org/software/libc/manual/html_node/Opening-Streams.html)
- [GNU C Library errors](https://www.gnu.org/software/libc/manual/html_node/Error-Reporting.html)
