These charts describe the format of geographic data that peermaps provides for
displaying maps. POINT, LINE, and AREA features start with a value (e.g. [0x01])
that indicates what the feature is. These feature types map directly to
primitives in OpenGL (e.g., GL_POINTS) with minimal linear processing.

All of the below are little endian.


# FEATURE: POINT      

| No. of bytes | Type [Value]  | Description                 |
|--------------|---------------|-----------------------------|
| 1            | U8 [0x01]     | feature type                |
| 4            | U32           | type                        |
| 8            | U64           | id                          |
| 4            | F32           | position lon                |
| 4            | F32           | position lat                |
| 1            | U8            | l_count (# of labels)       |
| -            | LABEL[l_count]| labels                      |


# FEATURE: LINE

| No. of bytes | Type [Value]     | Description              |
|--------------|------------------|--------------------------|
| 1            | U8 [0x02]        | feature type             |
| 4            | U32              | type                     |
| 8            | U64              | id                       |
| 2            | U16              | p_count (# of positions) |
| 8*p_count    | POSITION[p_count]| positions                |
| 1            | U8               | l_count (# of labels)    |
| -            | LABEL[l_count]   | labels                   |


# FEATURE: AREA

| No. of bytes | Type [Value]     | Description              |
|--------------|------------------|--------------------------|
| 1            | U8 [0x03]        | feature type             |
| 4            | U32              | type                     |
| 8            | U64              | id                       |
| 2            | U16              | p_count (# of positions) |
| 8*p_count    | POSITION[p_count]| positions                |
| 2            | U16              | c_count (# of cells)     |
| 6*c_count    | CELL[c_count]    | cells                    |
| 1            | U8               | l_count (# of labels)    |
| -            | LABEL[l_count]   | labels                   |


# POSITION

| No. of bytes | Type [Value] | Description                  |
|--------------|--------------|------------------------------|
| 4            |  F32         | longitude                    |
| 4            |  F32         | latitude                     |


# CELL

| No. of bytes | Type [Value] | Description                  |
|--------------|--------------|------------------------------|
| 2            |  U16         | i element index              |
| 2            |  U16         | j element index              |
| 2            |  U16         | k element index              |

i, j, and k are indexes into the position array.

# LABEL

| No. of bytes | Type [Value] | Description                  |
|--------------|--------------|------------------------------|
| 2            |  U16         | length                       |
| length       |  [U8]        | data                         |

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
\x14\x00=Aoraki / Mount Cook
\x0d\x00en=Mount Cook
\x09\x00mi=Aoraki
\x00\x00
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
\x0c\x00=Toshkent
\x0f\x00kaa=Tashkent
\x0e\x00en=Tashkent
\x11\x00alt:uz=Тoшкент
\x00\x00
```

please also see https://wiki.openstreetmap.org/wiki/Key:name and https://wiki.openstreetmap.org/wiki/Multilingual_names

