# File Types and Attributes
- Any file can be thought of as byte streams, but it is usually the case that files contain structured data not intended to be processed as such byte streams
## Ordinary Files
- A text file is a byte stream rendered as ASCII characters and typically broken into lines when processed
- An archive is a file consisting of many files, acting as an alternating sequence of headers describing a file and actual data blocks consisting of the file contents
- A load module consists of different parts of a program (code, data, etc.) and therefore follows a similar approach to alternating between headers and actual data
### Data Types and Associated Applications
- A file can only be interpreted by a program that understands the meaning behind the underlying encoding of the data
    - One approach is to require the *user* to specify the correct command to process the data (i.e. opening any file in a text editor)
    - Another approach is to consulst a registry that associates a program with a file type - this is usually determined based on the suffix of the file (.c, .txt, etc.), though a magic number at the beginning of a file can also be used to identify its type
### File Structure and Operations
- Even basic files types may have complex underlying structure, often requiring a higher-level access pattern (beyond just the basic `read` and `write`) system calls - consider databases for example, which may expect their key-value stores to be accessed using specialized `get`, `put`, and `delete` operations
### Other Types of Files
- Directories are considered files but they contain associations between names and data
    - To maintain directory structure, they are typically accessed with special operations such as `mkdir`, `rmdir`, etc.
- Interprocess communication ports, such as pipes, are not files themselves but are accessed as such - as with the `read` and `write` system calls
- Similarly, I/O devices themselves are not files but are typically able to be accessed as such
    - Some devices are able to fit easily into a byte stream model of `read` and `write` calls (keyboards, disks, etc.) but others may not (i.e. GPUs)
## File Attributes
- Aside from the standard metadata, files may also have extended attributes which may be important to file processing
    - i.e. Type of encryption/compression algorithm used on it (if encrypted/compressed)
    - i.e. Certificate if the file has been signed
    - i.e. Correct checksum