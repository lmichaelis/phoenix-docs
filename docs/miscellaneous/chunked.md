# The chunked binary format
Multiple different file formats use a special chunk based format for identifying separate parts of binary files.
This documentation calls this format *Chunked Binary*.

!!! important
    *Chunked Binary* files are binary files which are always encoded with the little-endian byte order. The number 
    `0xCAFEBABE` will be represented as `BE BA FE CA` when viewing the file in a hex editor.

With this format, a binary file is split up into sections (called chunks) which each contain a separate kind of data.
For a mesh, for example, there is a section containing general information, one containing the vertex position data and
one with polygon data.

The structure of this format can be roughly summed up as follows.

```c title="Chunked binary structure"
#pragma pack(push, 1)

typedef struct chunked_binary_chunk {
    uint16_t chunk_id; // (1)
    uint32_t chunk_size; // (2)
    uint8_t data[]; // (3)
} chunked_binary_chunk_t;

struct chunked_binary {
    chunked_binary_chunk_t chunks[]; // (4)
};

#pragma push(pop)
```

1. The chunk ID depends on the format of file being parsed. It is noted alongside the documentation of the format.
2. The number of bytes in the data section of the chunk.
3. An array of length `chunk_size` containing the raw data of the chunk. The format of this data is specific to the 
   format of the file being parsed. The documentation of that format describes the actual content of these sections.
4. The entire input is filled with chunks. There are no excess bytes at the end.
