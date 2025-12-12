大家想用的话，新建一个仓库，复制html文件里的源码，放在index.html里面然后建一个photos文件夹放上你想要展示的图片就好啦。
结构命名和我的仓库保持一致，如何部署仓库，生成一个所有人都能访问的网页可以参考
[https://copicode.com/templates/git/github-pages-deploy-guide-zh.php](url)
提示词可以参考
这是一个基于我们整个开发过程优化的、最终版本的 **Prompt（提示词）**。你可以将这段提示词保存下来，它完美涵盖了从“视觉还原”到“自动加载图片”再到“手势挥动切换”的所有核心需求。

---


**角色设定**：
你是一位世界级的 Creative Technologist，专注于 WebGL (Three.js)、GLSL 高级着色与 MediaPipe 计算机视觉交互。

**任务目标**：
编写一个**单文件 (Single-File) HTML 项目**，文件名为 `index.html`。该项目是一个送给朋友的互动圣诞礼物，需部署在服务器上，**无需用户手动上传图片**，而是自动加载服务器文件夹中的资源。

**核心技术栈 (Strict)**：
* **Import Map**:
    * `three`: `https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js`
    * `three/addons/`: `https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/`
    * `@mediapipe/tasks-vision`: `https://cdn.jsdelivr.net/npm/@mediapipe/tasks-vision@0.10.3/+esm`
* **Code Style**: ES6+ Modules, Class-based architecture, Async/Await.

---

### 📝 详细技术需求

#### 1. UI 设计 (Visual Identity & Gift UI)
* **字体**: Google Fonts 'Cinzel' (标题) & 'Times New Roman'。
* **配色**: 背景纯黑 (#050505)，主色香槟金 (#d4af37)，副色奶油白 (#fceea7)。
* **DOM 结构**:
    * `#loader`: 黑色遮罩，金色 Spinner，文字 "PREPARING SURPRISE..."。当 40 张图片预加载完成或超时后淡出。
    * `#status-bar`: 右上角显示计数器 "MEMORIES: X / 40" (当前索引/总数)。
    * `#gesture-hint`: 屏幕中央巨大的半透明文字 "SWIPE TO BROWSE"，仅在 FOCUS 模式下显示。
    * **移除**: **严禁**包含 `<input type="file">` 或上传按钮。
* **标题**: 顶部居中 `<h1>` "Merry Christmas" (带辉光和渐变)。

#### 2. Three.js 场景与后期 (High-End Rendering)
* **渲染**: `WebGLRenderer` (antialias: true), `ReinhardToneMapping` (Exp: 2.0), `ShadowMap` enabled.
* **环境**: `RoomEnvironment` + `PMREMGenerator` 生成高反射金属质感。
* **后期处理 (Bloom)**: 使用 `EffectComposer` + `UnrealBloomPass` (Strength: 0.35, Radius: 0.5, Threshold: 0.8) 营造梦幻氛围。
* **灯光**: 环境光 (0.5) + 顶部聚光灯 (Gold, Intensity 1500, CastShadow)。

#### 3. 粒子系统与内容生成 (Visual Restoration Protocol)
* **核心约束**: **必须保持高密度的视觉丰富度，严禁为了性能过度简化。**
    * **主体粒子 (Main)**: **1500 个**。
    * **氛围尘埃 (Dust)**: **2500 个** (使用 PointsMaterial, AdditiveBlending)。
* **几何体多样性**:
    * `BoxGeometry` (金色/深绿色 StandardMaterial)。
    * `SphereGeometry` (金色 Standard / **红色 PhysicalMaterial**)。
        * *注*: 红色材质必须开启 `clearcoat: 1.0` 和 `roughness: 0.2` 以模拟光滑的圣诞球。
    * **Candy Cane (糖果手杖)**: 使用 `CatmullRomCurve3` + `TubeGeometry`。
        * *纹理*: 必须程序化生成 (Canvas 2D API) 绘制白底红斜纹。
* **图片加载逻辑 (Auto-Load System)**:
    * **数据源**: 自动循环加载路径 `photos/1.jpg` 至 `photos/40.jpg`。
    * **数量**: 固定为 40 张。
    * **处理**: 加载纹理后，将其应用到 `BoxGeometry` (3x3x0.1) 的正面，边缘保持金色。

#### 4. 布局算法与状态机 (Smart Layout Engine)
* **Mode 1: TREE (圣诞树模式)**
    * **分层逻辑**:
        * **照片 (Photos)**: 必须排列在螺旋圆锥体的**最外层表面** (Radius offset +4)，形成一条金色影像带。
        * **装饰物 (Particles)**: 填充在树的**内部体积**中，确保树看起来丰满、不空心。
    * 所有非照片粒子需根据自身速度缓慢自转。
* **Mode 2: FOCUS (聚焦/画廊模式)**
    * **当前照片**: 移动到相机正前方 (0, 2, 40) 并放大 (Scale 4.5)。
    * **背景墙**: 其他照片根据索引差值 (offset)，排列在当前照片背后的两侧，形成画廊背景墙效果。
    * **粒子**: 随机散开至远处背景。
* **Mode 3: SCATTER (散落模式)**
    * 所有物体炸开并在空中漂浮旋转。

#### 5. MediaPipe 交互与手势控制 (Gesture Control)
* **手掌映射**: 映射手掌中心 (Landmark 9) 到场景容器 (`mainGroup`) 的 `rotation.x/y`，实现视差跟随。
* **基础手势**:
    * **Pinch (捏合)** -> 进入 **FOCUS** 模式。
    * **Fist (握拳)** -> 进入 **TREE** 模式。
    * **Open Hand (张开)** -> 进入 **SCATTER** 模式。
* **高级交互: 挥手切换 (Swipe Navigation)**
    * 仅在 **FOCUS** 模式且手掌 **张开 (Open)** 时触发。
    * **算法**: 监测手掌中心 X 轴的移动速度 (Velocity)。
    * **逻辑**:
        * 快速向左挥 -> `prevPhoto()` (索引 -1)。
        * 快速向右挥 -> `nextPhoto()` (索引 +1)。
        * 设置 1000ms 冷却时间 (Debounce) 防止误触。
    * **反馈**: 切换时右上角计数器高亮闪烁。

---
**输出要求**：请生成包含所有上述逻辑的完整 HTML/JS 代码，确保代码健壮性，直接复制即可部署。
