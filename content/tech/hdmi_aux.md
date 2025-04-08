---
title: "HDMI AUX Data"
date: 2025-04-08
draft: false
---

## AVI Info Frame


## HDR Info Frame

| Name          | Offset              | Remark                                  |
| ------------- | ------------------- | --------------------------------------- |
| EOTF          | byte[0]             |                                         |
| metadata_type | byte[1]             |                                         |
| G.x           | byte[2] + byte[3]   | Little Endian 16-bit                    |
| G.y           | byte[4] + byte[5]   | Little Endian 16-bit                    |
| B.x           | byte[6] + byte[7]   | Little Endian 16-bit                    |
| B.y           | byte[8] + byte[9]   | Little Endian 16-bit                    |
| R.x           | byte[10] + byte[11] | Little Endian 16-bit                    |
| R.y           | byte[12] + byte[13] | Little Endian 16-bit                    |
| WP.x          | byte[14] + byte[15] | Little Endian 16-bit                    |
| WP.y          | byte[16] + byte[17] | Little Endian 16-bit                    |
| MaxMDL        | byte[18] + byte[19] | Little Endian 16-bit unit: cd/m^2       |
| MinMDL        | byte[20] + byte[21] | Little Endian 16-bit unit: cd/m^2/10000 |
| MaxCLL        | byte[22] + byte[23] | Little Endian 16-bit unit: cd/m^2       |
| MaxFALL       | byte[24] + byte[25] | Little Endian 16-bit unit: cd/m^2       |

