# spz-anime-text2scene-bench

3D 场景文生世界基准数据集。10 个预选场景，6 个已完成全格式覆盖（SPZ v3/v4 + PLY + GLB + 360°全景 + JSON 元数据），4 个待补完。

## 生成流程

```
文字提示词
  → [腾讯混元3D](https://3d.hunyuan.tencent.com/sceneTo3D) → splats（.spz v3 + .ply） + 360°.png
  → [SPZ 官网](https://nianticlabs.github.io/spz/)  → .spz (v4) + JSON 元数据
  → [spz2glb](https://spz-ecosystem.github.io/spz2glb/) 打包 → .glb (KHR_gaussian_splatting_compression_spz_2 无损打包)
```

## 每个场景的完整数据清单

以 `Winter Forest Cabin` 为例（9 文件，~102 MB）：

```
Winter Forest Cabin/
├── assets/
│   ├── prompt.txt                         输入提示词 (267 B)
│   ├── Winter Forest Cabin_360.png        全景渲染图 (22.19 MB)
│   ├── Winter Forest Cabin_splats-info.json    ply (splat) 元数据 (406 B)
│   ├── Winter Forest Cabin_v3-info.json    spz v3 元数据 (528 B)
│   ├── Winter Forest Cabin_v4-info.json    spz v4 元数据 (529 B)
├── raw/
│   ├── Winter Forest Cabin_v3.spz         腾讯混元3D 直接导出 spz v3 (12.70 MB)
│   └── Winter Forest Cabin_v4.spz         SPZ 官网 ply → spz v4 (12.54 MB)
├── ply/
│   └── Winter Forest Cabin_splat.ply      腾讯混元3D 直接导出 ply (41.97 MB)
└── glb/
    └── Winter Forest Cabin_v3.glb         spz2glb 打包 v3 spz → glb (12.70 MB)
```

**特殊说明**：
- `*.spz` v3、`*.ply` 和 360° 全景图均由 [腾讯混元3D](https://3d.hunyuan.tencent.com/sceneTo3D) 世界模型文生世界模式直接导出
- `*_v4.spz` 由 SPZ 官网 (`nianticlabs.github.io/spz`) 读取 ply 后转换，文件前缀改为 `_v4` 便于区分版本
- ply 文件未改后缀，仅前缀统一用 `_splat` 命名（与混元3D 导出一致）
- `_splats-info.json` = splats (.ply) 的元数据，`_v3-info.json` = splats (spz v3) 元数据，`_v4-info.json` = splats (spz v4) 元数据
- GLB 由 spz2glb 打包 v3 spz → glb（无损，GLB 文件大小 ≈ SPZ 文件大小）

## 场景列表与文件清单

### 已完成（6/10）

| 场景 | 文件数 | 总大小 | SPZ v3 | SPZ v4 | PLY | GLB | 360° | JSON |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| classroom_anim | 9 | 68.39 MB | 8.66 MB | 8.47 MB | 29.23 MB | 8.66 MB | 13.37 MB | 3 份 |
| cyberpunk_city | 9 | 91.75 MB | 11.96 MB | 11.79 MB | 39.65 MB | 11.96 MB | 16.39 MB | 3 份 |
| steampunk_workshop | 9 | 107.08 MB | 13.36 MB | 13.08 MB | 44.47 MB | 13.36 MB | 22.81 MB | 3 份 |
| Winter Forest Cabin | 9 | 102.09 MB | 12.70 MB | 12.54 MB | 41.97 MB | 12.70 MB | 22.19 MB | 3 份 |
| chinese_garden | 9 | 106.39 MB | 14.07 MB | 13.73 MB | 47.28 MB | 14.08 MB | 17.24 MB | 3 份 |
| gothic_cathedral | 9 | 110.99 MB | 13.50 MB | 13.20 MB | 45.31 MB | 13.50 MB | 25.48 MB | 3 份 |
| **合计** | **54** | **586.69 MB** | — | — | — | — | — | — |

### 待补完（4/10，仅 prompt）

| 场景 | 状态 |
|:---|:---|
| medieval_castle | prompt.txt 已有 (205 B) |
| space_station | prompt.txt 已有 (235 B) |
| tropical_beach | prompt.txt 已有 (217 B) |
| underwater_reef | prompt.txt 已有 (193 B) |

## spz2glb SPZ→GLB 转换性能（稳态）`2026-06-25`

> 数据采集方法：spz2glb 直接转换 SPZ → GLB，每场景连续运行 N 次，
> **丢弃首轮冷启动数据**（OS page cache + 二进制加载），取后续稳态中位数。
> 验证（L1/L2/L3）仅在 CLI 端执行，不计入转换时间。

| 模型 | 耗时(ms) | SPZ 大小 | 峰值内存 | 分配 | 失败 |
|:---|:---:|:---|:---:|:---:|:---:|
| classroom_anime_v3 | 87 | 8.66 MB | 17.32 MB | 4 | 0 |
| cyberpunk_city_v3 | 96 | 11.96 MB | 23.92 MB | 4 | 0 |
| steampunk_workshop_v3 | 111 | 13.36 MB | 26.72 MB | 4 | 0 |
| Winter Forest Cabin_v3 | 117 | 12.70 MB | 25.39 MB | 4 | 0 |
| chinese_garden_v3 | 135 | 14.07 MB | 28.15 MB | 4 | 0 |
| gothic_cathedral_v3 | 138 | 13.50 MB | 27.01 MB | 4 | 0 |
| **平均值** | **114** | **12.38 MB** | **24.08 MB** | 4 | 0 |

**吞吐量**: ~100 MB/s（线性关系，R² > 0.95）。内存 = 2× 文件大小。

**排除项**：首轮 steampunk_workshop 267ms（JIT + 磁盘冷缓存）已丢弃。验证三层不在转换管线热路径上。SPZ→GLB 文件增量可忽略（~1MB）。

## Disclaimer

**This is a personal independent development project.**

- This project is developed independently by the author in their personal capacity
- This project is **not affiliated** with any university, institution, or employer
- This is **not a work-for-hire** or institutional teaching achievement
- The views and opinions expressed in this project are solely those of the author
- CC BY 4.0 applies — see [License](#license) for details

## License

**Dataset**: CC BY 4.0 — Junhan Pu, 2026
**Code** (if any): MIT License — see [LICENSE](LICENSE) for details
