# The FNT file format
Font (or *FNT*) files contain metadata of fonts for use with the *ZenGin*. *FNT* files only contain font metadata which
points into a TGA file containing the actual font glyphs.

!!! important
    *FNT* files are binary files which are always encoded with the little-endian byte order. The number `0xCAFEBABE`
    will be represented as `BE BA FE CA` when viewing the file in a hex editor.


Font files are structured like this:

```c title="Font file structure"
#pragma pack(push, 1)

typedef struct vec2 {
    float u;
    float v;
} vec2_t;

struct fnt_file {
    char version[2]; // (1)
    char* font_image; // (2)
    uint32_t glyph_height; // (3)
    uint32_t glyph_count; // (4)
    
    uint8_t glyph_widths[]; // (5)
    vec2_t glyph_top_left_uvs[]; // (6)
    vec2_t glyph_bottom_right_uvs[]; // (7)
};

#pragma pack(pop)
```

1. A fixed string denoting the version of the font file. It's always `"1\n"`
2. A newline (`"\n"`) terminated string containing the name of the font image file.
3. The height of each glyph in pixels.
4. The number of glyphs stored in this file.
5. An array of length `glyph_count` containing the width of each glyph.
6. An array of length `glyph_count` containing the top left UV-coordinate of each glyph.
7. An array of length `glyph_count` containing the bottom right UV-coordinate of each glyph.

!!! note
    To get the actual pixel coordinate in the glyph image for any given UV-coordinate, multiply the `x` UV-coordinate
    by the width of the image and the `y` UV-coordinate by the height.

!!! warning
    Some UV-coordinates are negative. These should be ignored since they don't have a glyph image associated with them.

!!! info "File Location"
    Glyph image files can normally be found in `Textures.vdf` inside the `_WORK/DATA/TEXTURES/FONTS/NOMIP/` (*TGA* files) and
    `_WORK/DATA/TEXTURES/_COMPILED/` (*TEX* files) folders.

    The font (*FNT*) files can be found in `Textures.vdf` inside the `_WORK/DATA/TEXTURES/_COMPILED/` folder.
