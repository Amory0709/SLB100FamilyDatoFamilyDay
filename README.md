# SLB 100 · Family Day

互动 3D 展示页 — SLB 100 Family Day 用。

## 文件

| 文件 | 用途 |
|---|---|
| `game.html` | 主页面（Three.js / WebGL，importmap 拉 0.160 from CDN） |
| `piattaforma_white.glb` | 主模型（25 MB） |
| `piattaforma_white.glb.zip` | 同上的压缩版本（4.5 MB，便于网页分发） |
| `glb-editor.html` | 离线颜色编辑器（GLB → 选中部件 → 改色 → 导出） |
| `palette.html` | 童趣马卡龙配色探索页（6 套 accent，可复制 CSS 变量） |
| `preview*.png` | Blender 渲染预览 |

## 本地预览

```bash
python3 -m http.server 8000
# 浏览器打开 http://127.0.0.1:8000/game.html
# 配色探索页 http://127.0.0.1:8000/palette.html
```

## GitHub Pages

| 页面 | URL |
|---|---|
| 主展示 | https://amory0709.github.io/SLB100FamilyDatoFamilyDay/ |
| 配色探索 | https://amory0709.github.io/SLB100FamilyDatoFamilyDay/palette.html |

## 模型出处

`piattaforma_white.glb` 是经 Blender 处理的"全白 + 蓝色细节"版本，对应
[SLB 100 Family Day] 视觉风格（modello bianco / 建筑白模风）。

## 当前状态

Step 1：背景渐变 + 软影棚拍打光 + 占位几何体（已对齐参考图视觉风格）。
下一步：把 `piattaforma_white.glb` 加载进 `game.html`。