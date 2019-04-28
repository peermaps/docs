These charts describe the format of geographic data that peermaps provides for
displaying maps. POINT, LINE, and AREA features start with a value (e.g. [0x01])
that indicates what the feature is. These feature types map directly to
primitives in OpenGL (e.g., GL_POINTS) with minimal linear processing.

All of the below are little endian.


# FEATURE: POINT      

| No. of bytes | Type [Value] | Name  |Description                  |
|--------------|--------------|-------|-----------------------------|
| 1            | U8 [0x01]    | feature type                 |
| 4            | U32          | type                         |
| 8            | U64          | id                           |
| 4            | F32          | position lon                 |
| 4            | F32          | position lat                 |


# FEATURE: LINE

| No. of bytes | Type [Value]  | Description                  |
|--------------|---------------|------------------------------|
| 1            | U8 [0x02]     | feature type                 |
| 4            | U32           | type                         |
| 8            | U64           | id                           |
| 2            | U16           | p_len (positions length)     |
| 8*p_len      | POSITION[plen]| positions                    |


# FEATURE: AREA

| No. of bytes | Type [Value]   | Description                  |
|--------------|----------------|------------------------------|
| 1            | U8 [0x03]      | feature type                 |
| 4            | U32            | type                         |
| 8            | U64            | id                           |
| 2            | U16            | p_len (positions length)     |
| 8*p_len      | POSITION[p_len]| positions                    |
| 2            | U16            | c_len (# of cells)           |
| 6*c_len      | CELL[c_len]    | cells                        |


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
