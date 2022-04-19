These charts describe the format of geographic data that peermaps provides for
displaying maps. POINT, LINE, and AREA features start with a value (e.g. [0x01])
that indicates what the feature is. These feature types map directly to
primitives in OpenGL (e.g., GL_POINTS) with minimal linear processing.

All of the below are little endian.

VARINT provides a compact encoding for whole numbers. For integers less than 128
the VARINT encoding is a single byte. Learn more at https://www.npmjs.com/package/varint.

# FEATURE: POINT      

| Type [Value]  | Description                 |
|---------------|-----------------------------|
| U8 [0x01]     | feature type                |
| VARINT        | type                        |
| VARINT        | id                          |
| F32           | position lon                |
| F32           | position lat                |
| LABEL[...]    | labels                      |


# FEATURE: LINE

| Type [Value]     | Description              |
|------------------|--------------------------|
| U8 [0x02]        | feature type             |
| VARINT           | type                     |
| VARINT           | id                       |
| VARINT           | p_count (# of positions) |
| POSITION[p_count]| positions                |
| LABEL[...]       | labels                   |


# FEATURE: AREA

| Type [Value]     | Description              |
|------------------|--------------------------|
| U8 [0x03]        | feature type             |
| VARINT           | type                     |
| VARINT           | id                       |
| VARINT           | p_count (# of positions) |
| POSITION[p_count]| positions                |
| VARINT           | c_count (# of cells)     |
| CELL[c_count]    | cells                    |
| LABEL[...]       | labels                   |

Areas represent polygon shapes.

Edges can be inferred from the cells array for the purposes of drawing borders
around each area by counting the frequency of each normalized edge pair
(`[i,j]`, `[j,k]`, `[k,i]` with for example the lower coordinate always in the
first position).

For areas, all edges with a count `!= 1` are presumed to be on the surface of
the polygon, either on the exterior surface or along the interior surfaces for
holes. If you need to include interior points for mesh refinement or have
explicit control over which edges are along the polygon surface (for example,
when clipping large geometries), use `AREA_WITH_EDGES` below.

# FEATURE: AREA_WITH_EDGES

| Type [Value]          | Description                  |
|-----------------------|------------------------------|
| U8 [0x04]             | feature type                 |
| VARINT                | type                         |
| VARINT                | id                           |
| VARINT                | p_count (# of positions)     |
| POSITION[p_count]     | positions                    |
| VARINT                | c_count (# of cells)         |
| CELL[c_count]         | cells                        |
| VARINT                | e_count (# of edge_indexes)  |
| EDGE_INDEX[e_count]   | edge_indexes                 |
| LABEL[...]            | labels                       |

This variation of areas gives you explicit control over edges.

There are two ways of specifying edges: as explicit pairs of indexes into the
positions array and as "windowed" indexes to save space for the common case
where contiguous positions are linked together to form edges. See the EDGE_INDEX
section below for more details.

# POSITION

| Type [Value] | Description                  |
|--------------|------------------------------|
|  F32         | longitude                    |
|  F32         | latitude                     |


# CELL

| Type [Value] | Description                  |
|--------------|------------------------------|
|  VARINT      | i element index              |
|  VARINT      | j element index              |
|  VARINT      | k element index              |

i, j, and k are indexes into the position array.

# EDGE_INDEX

Edge indexes connect verticies from the positions array along the surface rings
of an area for both exterior and interior surfaces.

| Type [Value] | Description                     |
|--------------|---------------------------------|
|  VARINT      | i: see below                    |

If `i == 0`, this position is an "edge break" (see below).

If `i%2 == 0`, `floor((i-1)/2)` is an index into the positions array.

If `i%2 == 1`, a range of ascending consecutive edge indexes is described from
the previous edge index up to the inclusive edge index of `floor((i-1)/2)`.

For example, the following edge index sequence values:

```
[8,6,16,102,115,20,32]
```

get interpreted and expanded to:

```
[3,2,7,50,51,52,53,54,55,56,9,15]
```

An "edge break" splits up runs of edges, like picking up a pen between strokes.
These breaks are represented by a value of `0`.

For example, this sequence:

```
[8,18,6,0,62,69,41,0,6,12,24,31]
```

gets interpreted and expanded into three runs of edges:

```
[
  [3,8,2],
  [30,31,32,33,34,40],
  [2,5,11,12,13,14]
]
```

# LABEL

| Type [Value] | Description                  |
|--------------|------------------------------|
|  VARINT      | length                       |
|  [U8]        | data                         |

for a single item, there may be multiple labels. each label has a length and a
data field. these are written serially and the list of labels ends with a label
of length zero.

labels can be written in multiple languages. for each of the label tags there
should be a 'name' tag and possibly a number of additional tags. in the schema,
the separator between the tag key and tag value is an equals sign (`=`). if the
tag is just `name`, the key is a 0-length string. otherwise, the `name:` at the
beginning is removed. there may be multiple colons in the label key after
`name:`. these should be preserved. for example, the key `name:left:nl` becomes
`left:nl`.

in addition to `name`, you might encounter the tags `alt_name` and `old_name`.
with these cases, the `_name` is removed for brevity. `alt_name` becomes `alt`
and `old_name` becomes `old`. these may also have colons after the tag name (eg:
`alt_name:en`). `alt_name:en` becomes `alt:en` and `old_name:nl` becomes
`old:nl`.

please see examples below for how each of the options should look in the schema:

for this tag data:

```json
{
  "name": "Aoraki / Mount Cook",
  "name:en": "Mount Cook",
  "name:mi": "Aoraki"
}
```

the packed label data would be:

```
\x14=Aoraki / Mount Cook
\x0den=Mount Cook
\x09mi=Aoraki
\x00
```

for this tag data:

```json
{
  "name": "Toshkent",
  "name:kaa": "Tashkent",
  "name:en": "Tashkent",
  "alt_name:uz": "Тoшкент"
}
```

the packed label data would be:

```
\x0c=Toshkent
\x0fkaa=Tashkent
\x0een=Tashkent
\x11alt:uz=Тoшкент
\x00
```

please also see https://wiki.openstreetmap.org/wiki/Key:name and https://wiki.openstreetmap.org/wiki/Multilingual_names

