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

## The Freelist

Unused pages are stored on the freelist and are reused when additional pages are required.

Freelist is organized as a linked list of freelist trunk pages with each trunk page containing page numbers for
zero or more freelist leaf pages

- the first integer on a freelist trunk page is the page number of the next freelist trunk page or zero if this
  is the last freelist trunk page
- the second integer on a freelist trunk page is the number of leaf page pointers to follow

The number of freelist pages is stored as a 4-byte big-edian integer in the database header at an offset of 36
from the beginning of the file.
The db header also stores the page number of the first freelist trunk page as a 4 byte big edian integer at an
offset of 32 from the beginning of the file.

## B-tree Pages

- Two variants of b-trees used by SQLite
    - Table b-trees: use a 64-bit signed integer key and store all data in the leaves
    - Index b-trees: use arbitrary keys and store no data at all
- B-tree page
    - Interior
        - usually has more than 1 keys
            - if page 1(with db header) is the interior page, then the number of keys can be 1
        - large keys are split up into overflow pages so that no single key uses more than one fourth of the
          available storage space on the page and hence every internal page is able to store at least 4 keys
            - this only happens in index b-trees(table b-trees use integer keys)
            - when the size of payload for a cell exceeds a certain threshold, then only the first few bytes of
              the payload are stored on the b-tree page and the balance is stored in a linked list of content
              overflow pages
    - Leaf: keys + associated data(for table b-trees only)
- Within an interior b-tree page, each key and the pointer to its immediate left are combined into a structure
  called a
  `cell`
    - the rightmost pointer is held separately
    - a leaf b-tree page has no pointers, but still uses the cell structure
- root page
    - page without parent
    - complete b-tree is identifiable by using root page number
- B-tree page structure
  - 100 byte database file header(only in page 1)
  - 8 or 12 byte b-tre page header 
  - Unallocated space
  - Cell content area
  - Reserved region: for extensions 
