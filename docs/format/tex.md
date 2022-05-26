# The TEX file format
TEX files contain [DXT1-5](#dxt), [palette](#palette) or [raw](#raw) textures alongside [mipmaps](#mipmaps) of each
texture. Texture files are binary files which start with a header after followed by palette and image data.

!!! important
    *TEX* files are binary files which are always encoded with the little-endian byte order. The number `0xCAFEBABE`
    will be represented as `BE BA FE CA` when viewing the file in a hex editor.

The header has a structure as follows:

```c title="ZTEX header structure"
#pragma pack(push, 1)

typedef enum ztex_format {
    tex_bgra8  = (uint32_t) 0, // (1)
    tex_rgba8  = (uint32_t) 1, // (2)
    tex_abgr8  = (uint32_t) 2, // (3)
    tex_argb8  = (uint32_t) 3, // (4)
    tex_bgr8   = (uint32_t) 4, // (5)
    tex_rgb8   = (uint32_t) 5, // (6)
    tex_argb4  = (uint32_t) 6, // (7)
    tex_a1rgb5 = (uint32_t) 7, // (8)
    tex_r5g6b5 = (uint32_t) 8, // (9)
    tex_p8     = (uint32_t) 9, // (10)
    tex_dxt1   = (uint32_t) 10, // (11)
    tex_dxt2   = (uint32_t) 11, // (12)
    tex_dxt3   = (uint32_t) 12, // (13)
    tex_dxt4   = (uint32_t) 13, // (14)
    tex_dxt5   = (uint32_t) 14, // (15)
} ztex_format_t;

struct ztex_header {
    char magic[4]; // (16)
    uint32_t version; // (17)
    ztex_format_t format; // (18)
    uint32_t width; // (19)
    uint32_t height; // (20)
    uint32_t mipmap_count; // (21)
    uint32_t reference_width; // (22)
    uint32_t reference_height; // (23)
    uint32_t average_color; // (24)
};

#pragma pack(pop)
```

1. 32-bit ARGB pixel format with alpha, using 8 bits per channel
2. 32-bit ARGB pixel format with alpha, using 8 bits per channel
3. 32-bit ARGB pixel format with alpha, using 8 bits per channel
4. 32-bit ARGB pixel format with alpha, using 8 bits per channel
5. 24-bit RGB pixel format with 8 bits per channel              
6. 24-bit RGB pixel format with 8 bits per channel              
7. 16-bit ARGB pixel format with 4 bits for each channel        
8. 16-bit ARGB pixel format where 5 bits are reserved for each color and 1 bit is reserved for alpha
9. 16-bit RGB pixel format with 5 bits for red, 6 bits for green, and 5 bits for blue
10. 8-bit color indexed
11. DXT1 compression texture format
12. DXT2 compression texture format
13. DXT3 compression texture format
14. DXT4 compression texture format
15. DXT5 compression texture format
16. The ZTEX magic value. Always `"ZTEX"`.
17. The version of the ZTEX format used. Always `0`.
18. A 32-bit integer indicating the image data format used.
19. The width of the original image in pixels.
20. The height of the original image in pixels.
21. The number of mipmap of the original stored.
22. Purpose unknown. The name was adopted from [ZenLib](https://github.com/ataulien/ZenLib/blob/master/zenload/ztex.h#L57-L58).
23. Purpose unknown. The name was adopted from [ZenLib](https://github.com/ataulien/ZenLib/blob/master/zenload/ztex.h#L57-L58).
24. A 32-bit ARGB value containing the average color of the image.

!!! note
    All color values should be understood as [word-order](https://en.wikipedia.org/wiki/RGBA_color_model#Representation),
    so an `RGBA` color `#aabbccdd` would be represented as the bytes `dd cc bb aa` in little-endian byte order. An alpha
    byte of `0xff` means full opacity.

## Mipmaps
Mipmaps are stored from the smallest to the largest image. If the mipmap-image with the level `0` has a with of `w`
and a height of `h` pixels then the mipmap image with the level `n` has a width of `w >> n` and a height of `h >> n`
pixels (each mipmap-level is half the size of the previous one).

## DXT
If the texture format is `dxt1`, `dxt2`, `dxt3`, `dxt4` or `dxt5` the data stored right after the header is DXT
compressed. To figure out the size of each mipmap-level in bytes, for DXT1 compressed data calculate 
`max(1, w / 4) * max(1, h / 4) * 8`[^1] and for DXT2-DXT5 calculate `max(1, w / 4) * max(1, h / 4) * 16`[^1] where `w` and `h`
depend on the [mipmap](#mipmaps) level. In practice, only DXT1, DXT3 AND DXT5 are used.

## Palette
With the `tex_p8` format, the texture file contains a palette-list before the actual texture data. This palette follows
right after the header and contains `256` 32-bit ARGB8 colors. These are then referenced in the texture data section
right after the palette data. Each byte in the texture data represents one pixel. Its color is the color in the palette
at the index indicated by the byte's value.

## Raw
If the format is not `tex_p8` and not one of the `dxt` formats, the texture stores raw data according to the format
described above. For `tex_bgra8`, `tex_rgba8`, `tex_abgr8` and `tex_argb8` formats, each pixel is 4 bytes in size. With
formats `tex_bgr8` and `tex_rgb8`, each pixel is 3 bytes in size. `tex_argb4`, `tex_a1rgb5` and `tex_r5g6b5` contain
2-byte sized pixels.

[^1]: This calculation has been adopted from [ZenLib](https://github.com/ataulien/ZenLib/blob/master/zenload/ztex2dds.cpp#L71-L77).
