# Database File

- During transaction
    - stores additional information in a second file called rollback journal or WAL file(WAL mode)

## Hot Journals

When crash occurs, SQLite make use of rollback journal or WAL file

- when used for recovery, they are called hot journal or hot WAL file

## Pages

- Main db file consists or N >= 1 pages
    - size: power of 2 between 512 ~ 65536
    - all pages in a single db file has same size
    - numbered from 1
- Page types
    - b-tree page
        - table b-tree interior page
        - table b-tree leaf page
        - index b-tree interior page
        - index b-tree leaf page
    - freelist page
        - freelist trunk page
        - freelist leaf page
    - payload overflow page
    - pointer map page
    - lock-byte page

## The Database Header

The first 100 bytes of db file contains file header(refer to official docs for detail)

### Magic header string

16 bytes which corresponds to "SQLite format 3"

### Page size

- 2 bytes value beginning at offset 16
- if you want 65536 page size, the way to interpret page size differs (TODO: limbo)

### File format version

Intended to allow for enhancements of the file format in future versions of SQLite

### Reserved bytes per page

Extra bytes used by extensions

- e.g. encryption extensions to store a nonce and/or cryptographic checksum associated with each page
- usually 0
- usable size
    - a page size specified by the 2 byte integer at offset 16 - reserved space size
    - not allowed to be less than 480
        - e.g. if the page size is 512, then the reserved space size cannot exceed 32

### Payload fractions

Not used

### File change counter

Incremented whenever the db file is unlocked after being modified

- Multiple processed can detect concurrent changes

In WAL mode, changes are detected using the wal-index and so the change counter is not needed

### In-header database size

Size of the database file in pages.

- Old versions ignored this header
- New versions fallback to real file size when the header is not avail

How is this header considered valid?

- if it's non-zero
- 4 byte change counter(offset 24) exactly matches the 4 byte version valid for number(offset 92)

### Free page list

Unused pages in db file are stored on a frelist

- 4 byte big edian(offset 32) stores the page number of the first page or zero if the freelist is empty

### Schema cookie

Incremented whenever database schema changes

### Schema format number

Refers to the high level SQL formatting rather than the low-level b-tree formatting

### Suggested cache size

Suggested cache size in pages for the db file

### Incremental vacuum settings

Used to manage auto vacuum and incremental vacuum mode

### Text encoding

Determines the encoding used for all text strings stored in the database

### User version number

Set and queried by the user version pragma. Now used by SQLite

### Application ID

Identify the database as belonging to or associated with a particular application

### Write library version number and version-valid-for number

Stores the SQLITE version number value for the SQLite library that most recently modified the database file

- the 4 byte big edian at offset 92 is the value of the change counter when the version number was stored

### Header space reserved for expansion

All other bytes are reserved for future use

## The Lock-Byte Page

A database file larger than 1073741824 contains exactly one lock-byte page

- lock-byte page is set aside for use by the OS specific VFS implementation in implementing the db file locking
  primitives
  - lock-byte page arose from the need to support Win95 which was predominant OS 
- SQLite doesn't use this page 

## 
