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
| U8            | l_count (# of labels)       |
| LABEL[l_count]| labels                      |


# FEATURE: LINE

| Type [Value]     | Description              |
|------------------|--------------------------|
| U8 [0x02]        | feature type             |
| VARINT           | type                     |
| VARINT           | id                       |
| VARINT           | p_count (# of positions) |
| POSITION[p_count]| positions                |
| U8               | l_count (# of labels)    |
| LABEL[l_count]   | labels                   |


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
| VARINT           | l_count (# of labels)    |
| LABEL[l_count]   | labels                   |


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

