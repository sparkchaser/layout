layout
========
A utility for laying out binary files, with an emphasis on using
simple, human-readable specification files.

## History ##
When working with embedded systems, I frequently found myself
needing to store several different binary data blobs (code, data,
etc) into a single flash memory device.  These blobs were created
with separate processes, and there was no easy way to combine them
into a single, flashable image.  I typically used shell scripts with
a bunch of `dd` calls, but the resulting script was not easy to read
and very difficult for other utilities to extract information from.

This utility allows the user to create a binary file that is composed
of a number of smaller files.  The layout of the output file is
controlled by a specification file using the format outlined below.

## Usage ##
```shell
layout [options] inputfile
    -v, --verbose                    Generate lots of debug output
    -h, --help                       Show this message
```

## Specification File Format ##
Specification files are plain-text format (technically, treated as Ruby
code), and can use any name or file extension.  Blank lines are ignored.
Anything from a '#' symbol to the end of the line is considered a comment.
Each specification file has two main parts:  an "output" section and a
"layout" section.

Example specification files are available in the 'examples' folder.

### "output" Section ###
This section describes the overall properties of the output file.  
Example:
```Ruby
output  :name => "myfile.bin",
        :size => 2 * MiB,
        :fill => 0xff
```
 * `name` (mandatory) - name for the output file
 * `size` (mandatory) - size of the output file
 * `fill` (optional) - the output file will be pre-filled with this byte value (defaults to `0x00` if unspecified)

### "layout" Section ###
This section describes how the output file should be assembled.  
Overall format:
```Ruby
layout {
  # layout directives go here
}
```
Directives are processed in the order that they appear.  The following directives
can be used in this section.

##### `component` #####
Insert the contents of a file into the output file at a specified offset.  
Usage:
```
component filename, offset, max_file_size [, options]
```
If the file is larger than the listed file size, an error will be generated.

`options` is specified as a hash.  Currently-supported options:
 * `:checksum` - If true, the last byte of this file will be replaced by a one-byte,
                 sum-to-zero checksum of this section's contents (file contents and
                 any trailing padding).  Equivalent to using a separate `checksum`
                 directive with the same offset and size.  Defaults to 'false'.

Any unrecognized options are ignored.

##### `reserved` #####
Indicates a reserved or unused section in the output file.  Nothing is output for
this directive, the rest of the line is effectively ignored.  `reserved` lines are
generally used to explicitly document unused space.

##### `setbyte` #####
Set a particular byte to a specific value.  
Usage:
```Ruby
setbyte offset, byte_value
```

##### `checksum` #####
Calculate a single-byte, sum-to-zero checksum over a range of bytes.  
Usage:
```Ruby
checksum offset, num_bytes
```
This directive will change the byte at offset `offset + num_bytes - 1` such that
calculating a byte-sized sum of bytes `offset` through `offset + num_bytes - 1`
will evaluate to zero.

## Units ##
Offsets and sizes are specified in bytes.  The following constants are available
to help represent larger quantities:

| Constant  | Value               |
| --------- | ------------------- |
| B         | 1                   |
| KB        | 1000                |
| MB        | 1000 * 1000         |
| GB        | 1000 * 1000 * 1000  |
| KiB       | 1024                |
| MiB       | 1024 * 1024         |
| GiB       | 1024 * 1024 * 1024  |

**Note:** Use these constants like `2 * KB`, not `2 KB`.

## To-Do List ##
 * Add support for a `checksum` option on `reserved` sections
 * Make the initial output file generation code more efficient for larger files
 * Add new option for file/section IDs and table-of-contents generation

## License ##
See the `LICENSE` file in the root directory for license information.
