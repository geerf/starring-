
# StarringFX

**A GPU-accelerated "star ring" particle effect for HarmonyOS. Native C++ (EGL / OpenGL ES 3) rendering through XComponent, running 3840 ring particles + 192 dust entirely on the vertex shader at 60 fps.**

一个适用于 HarmonyOS 的 GPU「星环」粒子特效。通过 XComponent 使用原生 C++（EGL / OpenGL ES 3）渲染，3840 个环粒子 + 192 个尘埃全部由顶点着色器解析计算，稳定 60 fps。

---

## Preview / 效果预览
IMG_859.gif

The ring slowly rotates and breathes; particles twinkle, drift with jitter, and periodically "eject" small groups that fade out and refill. An entrance burst animates the ring from a bright center point on surface creation.

星环缓慢自转并整体呼吸；粒子闪烁、抖动漂移，并周期性向外「喷射」小股粒子后渐隐回填。surface 创建时粒子从中心亮点迸发入场。

## Features / 特性

- **Fully GPU-driven**: every particle's motion (rotation, jitter, twinkle, breathing, entrance burst, ejects, depth brightening) is an analytic function of time `t` computed in the vertex shader. The CPU only uploads a handful of uniforms per frame.
- **Self-driven render thread**: a dedicated thread paced to 60 fps, the UI thread never participates in rendering.
- **Thread-safe live parameters**: center ratio, base width, density, pause/resume are all atomics — safe to call at any time, effective on the next frame.
- **Two modes**:
  - *Fullscreen* (default): ring center at `(0.52, 0.44)` of the surface with a vignette pass.
  - *Embedded*: you specify the center ratio and a base width in vp, and the ring scales to fit (radius = `boxSide / 2 − 90` vp).
- **PC resize fallback**: on some PCs the XComponent `OnSurfaceChanged` callback never fires; `notifyStarRingSurfaceSize()` lets ArkTS push the viewport size explicitly.
- Works on phone / tablet / 2-in-1.

- **全 GPU 驱动**：每个粒子的运动（自转、抖动、闪烁、呼吸、入场迸发、喷射、深度增亮）都是顶点着色器内关于时间 `t` 的解析函数，CPU 每帧只上传少量 uniform。
- **渲染线程自驱**：独立线程按 60 fps 节流渲染，UI 线程零参与。
- **线程安全实时参数**：环心比例、基准宽、密度、暂停/恢复均为原子变量，任意线程随时调用，下一帧生效。
- **两种模式**：
  - *独立全屏模式*（默认）：环心位于 surface 的 `(0.52, 0.44)`，带 vignette 暗角。
  - *应用内嵌模式*：指定环心比例与基准宽（vp），星环按比例缩放（半径 = `盒宽 / 2 − 90` vp）。
- **PC 端拉伸兜底**：部分 PC 上 XComponent 的 `OnSurfaceChanged` 回调不触发；提供 `notifyStarRingSurfaceSize()` 由 ArkTS 侧主动推送视口尺寸。
- 支持手机 / 平板 / 2-in-1。

## Requirements / 环境要求

- HarmonyOS NEXT (API 22+)
- DevEco Studio 5.x+ (or the `hvigor` / `devecocli` command line)
- No signing configuration is included — the project builds an unsigned HAP. Add your own signing config in `build-profile.json5` if you want to run it on a real device via DevEco Studio's auto-sign, or run directly on the emulator.

- HarmonyOS NEXT（API 22 及以上）
- DevEco Studio 5.x 及以上（或 `hvigor` / `devecocli` 命令行）
- 项目不含签名配置，构建产物为未签名 HAP。如需真机运行，请在 `build-profile.json5` 中添加你自己的签名配置（DevEco Studio 自动签名即可），或使用模拟器运行。

## Quick Start / 快速开始

```bash
# command line build (unsigned HAP) — requires the HarmonyOS command line tools
devecocli build
# or simply open the project in DevEco Studio and press Build / Run
```

The output HAP is at `entry/build/default/outputs/default/entry-default-unsigned.hap`.

The demo page (`entry/src/main/ets/pages/Index.ets`) renders the ring fullscreen with two buttons:

- **Pause / Resume** — demonstrates `starRingPause()` / `starRingResume()`
- **Fullscreen / Embedded** — toggles between fullscreen mode and embedded mode via `setStarRingCenter()`

演示页（`entry/src/main/ets/pages/Index.ets`）全屏渲染星环，提供两个按钮：

- **Pause / Resume** —— 演示 `starRingPause()` / `starRingResume()`
- **Fullscreen / Embedded** —— 通过 `setStarRingCenter()` 在全屏与内嵌模式间切换

## API / 接口

All functions are exported from `libstarringfx.so`:

| Function | Description |
|---|---|
| `registerStarRingContext(ctx: object): boolean` | Call in `XComponent.onLoad(ctx)` to register surface lifecycle callbacks. |
| `setStarRingDensity(densityPixels: number): void` | Pass `display.getDefaultDisplaySync().densityPixels`; affects sprite sizes / jitter. |
| `setStarRingCenter(cxRatio, cyRatio, baseWidthVp): void` | Embedded mode. `baseWidthVp > 0`: ring center ratio + base width in vp; `baseWidthVp = 0`: restore fullscreen mode (default center `0.52 / 0.44`). |
| `notifyStarRingSurfaceSize(wPx, hPx): void` | Explicitly push the surface viewport size in px (PC fallback where `OnSurfaceChanged` never fires). Idempotent with the automatic callback. |
| `starRingPause(): void` | Pause rendering (render thread sleeps, surface kept). |
| `starRingResume(): void` | Resume rendering. |

所有接口由 `libstarringfx.so` 导出：

| 接口 | 说明 |
|---|---|
| `registerStarRingContext(ctx: object): boolean` | 在 `XComponent.onLoad(ctx)` 中调用，注册 surface 生命周期回调。 |
| `setStarRingDensity(densityPixels: number): void` | 传 `display.getDefaultDisplaySync().densityPixels`；影响精灵尺寸 / 抖动幅度。 |
| `setStarRingCenter(cxRatio, cyRatio, baseWidthVp): void` | 内嵌模式。`baseWidthVp > 0`：环心比例 + 基准宽（vp）；`baseWidthVp = 0`：恢复独立全屏模式（默认环心 `0.52 / 0.44`）。 |
| `notifyStarRingSurfaceSize(wPx, hPx): void` | 主动推送 surface 视口尺寸（px），用于 `OnSurfaceChanged` 不触发的 PC 端；与自动回调幂等。 |
| `starRingPause(): void` | 暂停渲染（渲染线程休眠，surface 保留）。 |
| `starRingResume(): void` | 恢复渲染。 |

### Embedded mode geometry / 内嵌模式几何

```
baseW   = min(winW, winH * 1080 / 1920)                     // ring base width (vp)
boxSide = min(2 * (0.47 * baseW + 90), max(winW, winH))     // XComponent square size (vp)
ring radius = boxSide / 2 − 90                              // (vp, dust margin)
```

Use a square XComponent of `boxSide`, call `setStarRingCenter(0.5, 0.5, baseW)`, and the ring will fit exactly inside the box with a 90 vp dust margin.

使用边长 `boxSide` 的正方形 XComponent，调用 `setStarRingCenter(0.5, 0.5, baseW)`，星环将精确填满方盒并在四周留出 90 vp 的尘埃边距。

## Integration / 集成到你的项目

1. Copy `entry/src/main/cpp/` (`star_ring_renderer.h/.cpp` + the star-ring part of `napi_init.cpp`) into your module and merge the CMake target:

   ```cmake
   add_library(yourlib SHARED napi_init.cpp star_ring_renderer.cpp)
   target_link_libraries(yourlib PUBLIC
       libace_napi.z.so libace_ndk.z.so libhilog_ndk.z.so libEGL.so libGLESv3.so)
   ```

2. Declare the NAPI surface in a `.d.ts` next to your module (see `entry/src/main/ets/libstarringfx.so/index.d.ts`).

3. Wire up the XComponent (ArkTS):

   ```ts
   import nativeCap from 'libstarringfx.so';
   import display from '@ohos.display';

   XComponent({ id: 'starring', type: XComponentType.SURFACE, libraryname: 'starringfx' })
     .onLoad((ctx?: object) => {
       if (ctx) {
         nativeCap.registerStarRingContext(ctx);
       }
       nativeCap.setStarRingDensity(display.getDefaultDisplaySync().densityPixels);
       // embedded mode (optional):
       nativeCap.setStarRingCenter(0.5, 0.5, baseW);
     })
     .width(boxSide).height(boxSide)
   ```

4. On window resize during embedded mode, re-measure and push:

   ```ts
   nativeCap.setStarRingCenter(0.5, 0.5, newBaseW);
   nativeCap.notifyStarRingSurfaceSize(boxSide * densityPx, boxSide * densityPx);
   ```

1. 将 `entry/src/main/cpp/`（`star_ring_renderer.h/.cpp` 与 `napi_init.cpp` 中的星环桥接部分）复制进你的模块，并合并 CMake 目标：

   ```cmake
   add_library(yourlib SHARED napi_init.cpp star_ring_renderer.cpp)
   target_link_libraries(yourlib PUBLIC
       libace_napi.z.so libace_ndk.z.so libhilog_ndk.z.so libEGL.so libGLESv3.so)
   ```

2. 在模块旁声明 NAPI 接口的 `.d.ts`（参考 `entry/src/main/ets/libstarringfx.so/index.d.ts`）。

3. 接线 XComponent（ArkTS）：

   ```ts
   import nativeCap from 'libstarringfx.so';
   import display from '@ohos.display';

   XComponent({ id: 'starring', type: XComponentType.SURFACE, libraryname: 'starringfx' })
     .onLoad((ctx?: object) => {
       if (ctx) {
         nativeCap.registerStarRingContext(ctx);
       }
       nativeCap.setStarRingDensity(display.getDefaultDisplaySync().densityPixels);
       // 内嵌模式（可选）：
       nativeCap.setStarRingCenter(0.5, 0.5, baseW);
     })
     .width(boxSide).height(boxSide)
   ```

4. 内嵌模式下窗口尺寸变化时，重新测量并推送：

   ```ts
   nativeCap.setStarRingCenter(0.5, 0.5, newBaseW);
   nativeCap.notifyStarRingSurfaceSize(boxSide * densityPx, boxSide * densityPx);
   ```

## Technical Notes / 技术要点

- **Analytic particles**: each particle stores only `(theta0, omega, rho, h)` + look/jitter/twinkle attributes in static VBOs. The vertex shader reconstructs its position at time `t` — no per-frame CPU simulation, no buffer re-upload.
- **Ejects**: up to 32 concurrent "eject" slots are uploaded as uniforms (`uEjectA/uEjectB`); ejected particles interpolate toward an outward direction, fade, then refill from their orbit.
- **Rendering thread**: creates its own EGL context/surface bound to the OHNativeWindow from the XComponent callback, renders at ~60 fps, and joins on surface destroy. All cross-thread state is `std::atomic` or mutex-guarded.
- **Surface size bootstrap**: `OnSurfaceCreated` actively queries the component size (some devices never fire `OnSurfaceChanged`), so the first frame is never black.

- **解析式粒子**：每个粒子只在静态 VBO 中存 `(theta0, omega, rho, h)` 与外观/抖动/闪烁属性，位置由顶点着色器在时间 `t` 重建——无需逐帧 CPU 模拟，无需重传缓冲。
- **喷射**：最多 32 个并发「喷射」槽以 uniform（`uEjectA/uEjectB`）上传；被喷射粒子沿外向插值位移、渐隐，随后回到原轨道回填。
- **渲染线程**：基于 XComponent 回调传入的 OHNativeWindow 自建 EGL context/surface，按 ~60 fps 渲染，surface 销毁时收线程；所有跨线程状态为 `std::atomic` 或互斥锁保护。
- **surface 尺寸自举**：`OnSurfaceCreated` 主动查询组件尺寸（部分设备 `OnSurfaceChanged` 永不触发），保证首帧不黑屏。

## Project Structure / 项目结构

```
entry/src/main/
├── cpp/
│   ├── CMakeLists.txt          # builds libstarringfx.so
│   ├── napi_init.cpp           # NAPI bridge: XComponent callbacks + exported functions
│   ├── star_ring_renderer.h    # renderer interface
│   └── star_ring_renderer.cpp  # EGL/GLES3 particle renderer (vertex-shader driven)
├── ets/
│   ├── libstarringfx.so/index.d.ts   # NAPI type declarations
│   └── pages/Index.ets         # demo page
└── module.json5
```
