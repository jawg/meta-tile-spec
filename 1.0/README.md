# Meta Tile Specification

## 1. Introduction

This specification lays out a format to efficiently store or transport a set of tiles called meta-tile.

## 2. Definition

A meta-tile of dimension **n** is an n x n two-dimensional array of consecutive tiles, n being a power of two.  
Example: a meta-tile of dimension **8** is an array of **8x8** consecutive tiles.  

The term meta-tile can be used in different contexts with different meanings:
 * In rendering, a meta-tile is used in the process of rendering a set of raster tiles as one big tile, which will be later split into multiple tiles. It is done so to improve queries efficiency and global rendering quality for labels.
 * In storage, a meta-tile is a way to store together a set of tiles to **optimize disk space** (inodes and default block sizes are costly). In ext4 filesystems, default inode size is 128 bytes, default  block size is 4Kb. The average size of a tile is 633 bytes. Thus, storing 64 "average" tiles would take up 64 * 4.128 = 264 Kb. With meta-tiles, we can greatly reduce the overhead in block size typically. 

[http://wiki.openstreetmap.org/wiki/Meta_tiles](Reference to Meta-Tiles on the Wiki)
 
This specification will use the term meta-tile as a way to improve storage and transport efficiency, and lower storage cost overhead.

## 3. Wording

`metaTileSize: dimension of a meta-tile in a power of two`  
`offset: offset in bytes from the beginning of a data-buffer`

## 4. Format

The buffer format for the specification is the concatenation of 4 kinds of information:  

Magic Number  | MetaTileSize (n) | Offsets         | Tile binary Data
:------------ | :------------    | :------------   | :------------   
4 bytes       | 1 byte           | n x n x 4 bytes | unknown         

any MMT-compliant file starts by the magic number coded on 4 bytes, then holds the metaTileSize n on one byte, the offsets on n x n x 4 bytes, and the concatenation of tile binary data.

### 4.1. Magic number

The magic number to define a meta-tile is the byte value of letters `META` in ASCII: `0x4D455441`.

### 4.2. MetaTileSize

The dimension of a meta-tile is a power of two and can be any of these: {1, 2, 4, 8, 16, 32, 64, 128}.
The metaTileSize should be coded in a binary form: {0x01, 0x02, 0x04, 0x08, 0x10, 0x20, 0x40, 0x80}.

### 4.3. Offsets

Let n be the `metaTileSize` value.
There is a sequence of n^2 (or n-square) offset values pointing to the beginning of data for each tile.
Each offset is coded on 4-bytes

### 4.4. Data

The rest of data is a concatenation of the n x n tiles in binary.
The following order is expected:  
On a meta-tile of dimension 4, the tiles should start from the upper-left corner, then going by increasing column value until next row.  
Example: 
```
  0 |  1 |  2 |  3  
  4 |  5 |  6 |  7  
  8 |  9 | 10 | 11  
 12 | 13 | 14 | 15  
```

Sample implementation in javascript-like code:  
```javascript
var buffers = []; 
for (var row = 0; row < numRows; row++) {
  for (var col = 0; col < numColumns; col++) {
    buffers[col * (row + 1)] = metaTile[row][col];
  }
}
```

## 5. Extension

If being used in storage or API endpoints, the file extension should be **.mmtX**, X being the dimension of the meta-tile.  
Example: for a 8x8 meta-tile, file extension should be **.mmt8**

N.B.: This specification is format-agnostic and can store any kind of "tile". Raster, Vector, UTF-Grid, ...  
To differentiate them from each other, the specification requires the extension to be prefixed from the tiles format extension.  

Example: for a set of 8x8 png tiles, the meta-tile file extension would be **.png.mmt8**

## 6. MIME Types

In case of a meta-tile transport, the Content-Type should be set to `application/octet-stream`.

