
# C++ Raylib 2D 图形渲染完全指南

## 1. 简介与核心概念

Raylib 的核心设计哲学是 **“简单、易用、无外部依赖”**。在 2D 渲染方面，它封装了 OpenGL（底层），为开发者提供了极其直观的 API。

### 1.1 核心生命周期
Raylib 的所有渲染操作都必须在主循环的 `BeginDrawing()` 和 `EndDrawing()` 之间进行。

```cpp
#include "raylib.h"

int main() {
    // 1. 初始化
    InitWindow(800, 600, "Raylib 2D Demo");
    SetTargetFPS(60);

    // 2. 主循环
    while (!WindowShouldClose()) {
        // 2.1 更新逻辑 (Update)
        
        // 2.2 开始绘制
        BeginDrawing();
        ClearBackground(RAYWHITE); // 清屏

        // --- 在这里进行 2D 渲染 ---

        // 2.3 结束绘制
        EndDrawing();
    }

    // 3. 清理与关闭
    CloseWindow();
    return 0;
}
```

---

## 2. 基础 2D 图形绘制

Raylib 提供了丰富的基本形状绘制函数，适合用于原型开发、UI 绘制或简单的粒子效果。

### 2.1 颜色系统
Raylib 预定义了许多常用颜色（如 `RED`, `BLUE`, `RAYWHITE`），你也可以自定义：
```cpp
Color myColor = { 255, 100, 50, 255 }; // R, G, B, A (0-255)
Color semiTransparent = Fade(RED, 0.5f); // 使用 Fade 设置透明度
```

### 2.2 绘制基本形状
```cpp
// 线条
DrawLine(10, 10, 200, 200, BLACK);
DrawLineBezier(startPos, endPos, 5.0f, RED); // 贝塞尔曲线

// 矩形
DrawRectangle(50, 50, 100, 80, BLUE);
DrawRectangleLines(50, 50, 100, 80, BLACK); // 仅边框
DrawRectangleRounded({50, 50, 100, 80}, 0.3f, 8, GREEN); // 圆角矩形

// 圆形与椭圆
DrawCircle(400, 300, 50, ORANGE);
DrawCircleLines(400, 300, 50, BLACK);
DrawEllipse(400, 300, 80, 40, PURPLE); // 椭圆

// 多边形
DrawPoly({400, 300}, 6, 50, 0, YELLOW); // 中心点, 边数, 半径, 旋转角度, 颜色
```

---

## 3. 文本与字体渲染

### 3.1 默认字体
使用系统默认字体进行快速文本绘制：
```cpp
DrawText("Hello Raylib!", 10, 10, 20, DARKGRAY);
// 参数：文本, X, Y, 字体大小, 颜色

// 获取文本宽度，用于居中对齐
int textWidth = MeasureText("Hello", 20);
DrawText("Hello", (800 - textWidth) / 2, 10, 20, BLACK);
```

### 3.2 自定义字体 (TTF/OTF)
```cpp
// 加载字体
Font myFont = LoadFont("resources/my_font.ttf");

// 绘制自定义字体
Vector2 pos = { 100.0f, 100.0f };
DrawTextEx(myFont, "Custom Font Text", pos, 30.0f, 2.0f, DARKBLUE); 
// 参数：字体, 文本, 位置, 字体大小, 字间距, 颜色

// 卸载字体
UnloadFont(myFont);
```

---

## 4. 图像与纹理 (核心)

在 Raylib 中，**`Image`** 是 CPU 内存中的图像数据，**`Texture2D`** 是上传到 GPU 显存的纹理。渲染时**必须使用 `Texture2D`**。

### 4.1 加载与绘制基础纹理
```cpp
Texture2D texture = LoadTexture("resources/player.png");

// 基础绘制
DrawTexture(texture, 100, 100, WHITE);

// 使用 Vector2 绘制
DrawTextureV(texture, { 200.0f, 200.0f }, WHITE);

// 带旋转和缩放的绘制
// 参数：纹理, 位置, 旋转角度(度), 缩放比例, 颜色
DrawTextureEx(texture, { 300.0f, 300.0f }, 45.0f, 2.0f, WHITE); 

UnloadTexture(texture);
```

### 4.2 精灵图与图集 (Sprite Sheets)
在实际游戏中，我们通常将多张图合并为一张大图（图集）。Raylib 提供了强大的裁剪和映射功能。

#### `DrawTextureRec` (基础裁剪)
从纹理中截取一个矩形区域绘制到屏幕。
```cpp
Rectangle sourceRec = { 0.0f, 0.0f, 32.0f, 32.0f }; // x, y, width, height (在纹理上的坐标)
DrawTextureRec(texture, sourceRec, { 100.0f, 100.0f }, WHITE);
```

#### `DrawTexturePro` (终极武器，最常用)
提供完全的变换控制：源矩形、目标矩形、旋转原点。
```cpp
Rectangle source = { 0, 0, 32, 32 };           // 纹理上的源区域
Rectangle dest = { 400, 300, 64, 64 };         // 屏幕上的目标区域 (可缩放)
Vector2 origin = { 32, 32 };                   // 旋转和缩放的中心点 (相对于目标矩形左上角)
float rotation = 45.0f;                        // 旋转角度

DrawTexturePro(texture, source, dest, origin, rotation, WHITE);
```
*💡 **提示**：将 `origin` 设置为 `dest` 宽高的一半，可以实现围绕图像中心旋转。*

---

## 5. 2D 摄像机系统 (Camera2D)

当游戏世界大于屏幕时，我们需要 `Camera2D` 来处理世界坐标到屏幕坐标的转换。

### 5.1 摄像机设置与使用
```cpp
Camera2D camera = { 0 };
camera.target = { 400.0f, 300.0f }; // 摄像机聚焦的世界坐标点
camera.offset = { 400.0f, 300.0f }; // 目标点在屏幕上的位置 (通常设为屏幕中心)
camera.rotation = 0.0f;             // 摄像机旋转角度
camera.zoom = 1.0f;                 // 缩放级别 (1.0f 为原大)

// 在主循环中：
BeginDrawing();
ClearBackground(RAYWHITE);

BeginMode2D(camera); // 进入 2D 摄像机模式

    // 在这里绘制的所有内容，都会应用摄像机的平移、旋转和缩放
    DrawRectangle(0, 0, 1000, 1000, LIGHTGRAY); // 绘制一个大地
    DrawTexture(playerTex, playerPos.x, playerPos.y, WHITE);

EndMode2D(); // 退出 2D 摄像机模式

// 退出摄像机模式后，绘制的内容将使用屏幕绝对坐标（适合绘制 UI）
DrawText("UI Text", 10, 10, 20, BLACK); 

EndDrawing();
```

---

## 6. 离屏渲染 (RenderTexture2D)

有时我们需要将场景渲染到一张纹理上（例如：小地图、后处理效果、动态光影）。

```cpp
// 1. 加载渲染纹理 (注意：Raylib 纹理坐标系 Y 轴向下，但 OpenGL 向上，
// 所以 RenderTexture 的 Y 轴是翻转的，需要在绘制时处理)
RenderTexture2D target = LoadRenderTexture(256, 256);

BeginDrawing();
ClearBackground(RAYWHITE);

// 2. 开始渲染到纹理
BeginTextureMode(target);
    ClearBackground(BLUE);
    DrawCircle(128, 128, 50, RED);
EndTextureMode();

// 3. 将渲染好的纹理绘制到屏幕上
// ⚠️ 注意：由于 Y 轴翻转，目标高度(dest.height)需要设置为负数！
Rectangle destRect = { 100.0f, 100.0f, 256.0f, -256.0f }; 
DrawTexturePro(target.texture, 
               { 0, 0, (float)target.texture.width, (float)target.texture.height }, 
               destRect, 
               { 0, 0 }, 0.0f, WHITE);

EndDrawing();

UnloadRenderTexture(target);
```

---

## 7. 进阶渲染技术

### 7.1 混合模式 (Blend Modes)
控制像素如何与背景混合，常用于粒子系统、发光效果。
```cpp
BeginBlendMode(BLEND_ADDITIVE); // 叠加混合 (常用于发光/火焰)
    DrawTexture(fireParticle, x, y, WHITE);
EndBlendMode();

// 其他模式：BLEND_ALPHA (默认), BLEND_MULTIPLIED, BLEND_ADD_COLORS 等
```

### 7.2 基础着色器 (Shaders)
Raylib 支持 GLSL 着色器（底层会自动转换为对应平台的着色器语言）。
```cpp
Shader shader = LoadShader(0, "resources/grayscale.fs"); // 0表示使用默认顶点着色器

BeginShaderMode(shader);
    DrawTexture(myTexture, 100, 100, WHITE); // 该纹理将应用灰度效果
EndShaderMode();

UnloadShader(shader);
```

---

## 8. 完整实战示例：带摄像机和精灵动画的 2D 场景

以下是一个综合示例，展示了纹理加载、摄像机跟随、精灵图裁剪和 UI 渲染。

```cpp
#include "raylib.h"
#include <math.h>

int main() {
    const int screenWidth = 800;
    const int screenHeight = 450;

    InitWindow(screenWidth, screenHeight, "Raylib 2D 综合示例 - eoolife");

    // 定义一个虚拟的精灵图 (实际项目中应使用 LoadTexture)
    // 这里为了代码可直接运行，使用程序生成的纹理
    Image imPlayer = GenImageColor(64, 64, RED);
    ImageDrawRectangle(&imPlayer, 16, 16, 32, 32, YELLOW); // 画个简单的脸
    Texture2D playerTex = LoadTextureFromImage(imPlayer);
    UnloadImage(imPlayer);

    // 玩家状态
    Vector2 playerPos = { 400.0f, 300.0f };
    float playerSpeed = 200.0f;
    float playerRotation = 0.0f;

    // 摄像机设置
    Camera2D camera = { 0 };
    camera.target = playerPos;
    camera.offset = { screenWidth / 2.0f, screenHeight / 2.0f };
    camera.rotation = 0.0f;
    camera.zoom = 1.0f;

    SetTargetFPS(60);

    while (!WindowShouldClose()) {
        // 1. 更新逻辑
        float dt = GetFrameTime();

        // 键盘移动
        if (IsKeyDown(KEY_RIGHT)) playerPos.x += playerSpeed * dt;
        if (IsKeyDown(KEY_LEFT))  playerPos.x -= playerSpeed * dt;
        if (IsKeyDown(KEY_DOWN))  playerPos.y += playerSpeed * dt;
        if (IsKeyDown(KEY_UP))    playerPos.y -= playerSpeed * dt;

        // 鼠标滚轮缩放
        camera.zoom += ((float)GetMouseWheelMove() * 0.05f);
        if (camera.zoom > 3.0f) camera.zoom = 3.0f;
        else if (camera.zoom < 0.5f) camera.zoom = 0.5f;

        // 更新摄像机目标
        camera.target = playerPos;
        
        // 玩家自转
        playerRotation += 100.0f * dt;

        // 2. 渲染
        BeginDrawing();
        ClearBackground(RAYWHITE);

        // --- 世界坐标渲染 ---
        BeginMode2D(camera);

            // 绘制网格背景
            for (int i = -10; i < 10; i++) {
                for (int j = -10; j < 10; j++) {
                    Color gridColor = ((i + j) % 2 == 0) ? LIGHTGRAY : WHITE;
                    DrawRectangle(i * 100, j * 100, 100, 100, gridColor);
                }
            }

            // 绘制玩家 (使用 DrawTexturePro 实现中心旋转)
            Rectangle source = { 0, 0, (float)playerTex.width, (float)playerTex.height };
            Rectangle dest = { playerPos.x, playerPos.y, 64.0f, 64.0f };
            Vector2 origin = { 32.0f, 32.0f }; // 中心点
            
            DrawTexturePro(playerTex, source, dest, origin, playerRotation, WHITE);

        EndMode2D();

        // --- 屏幕坐标渲染 (UI) ---
        DrawRectangle(10, 10, 250, 110, Fade(SKYBLUE, 0.5f));
        DrawRectangleLines(10, 10, 250, 110, BLUE);
        DrawText("2D 摄像机示例", 20, 20, 20, DARKBLUE);
        DrawText(TextFormat("Player Pos: %.0f, %.0f", playerPos.x, playerPos.y), 20, 50, 16, DARKGRAY);
        DrawText(TextFormat("Camera Zoom: %.2f", camera.zoom), 20, 75, 16, DARKGRAY);
        DrawText("Use WASD to move, Mouse Wheel to zoom", 20, 100, 14, DARKGRAY);

        EndDrawing();
    }

    // 3. 清理
    UnloadTexture(playerTex);
    CloseWindow();

    return 0;
}
```

---

## 9. 性能优化与最佳实践

1. **纹理批处理 (Texture Batching)**：
   * Raylib 默认开启了纹理批处理（默认批次大小为 8192 个图元）。
   * **优化点**：频繁切换不同的 `Texture2D` 会打断批处理（Draw Call 增加）。尽量将多张小图合并为一张**大图集 (Texture Atlas)**，使用 `DrawTextureRec` / `DrawTexturePro` 来绘制。
2. **内存管理**：
   * 遵循“谁加载，谁卸载”原则。`LoadTexture` 必须对应 `UnloadTexture`，防止显存泄漏。
   * 如果需要在运行时频繁更新纹理数据，不要反复 Load/Unload，而是使用 `UpdateTexture(texture, pixels)`。
3. **图像格式**：
   * 优先使用 `.png`（支持透明通道）。Raylib 也支持 `.bmp`, `.tga`, `.jpg`, `.gif`, `.psd`, `.hdr`, `.pic`, `.pnm`。
4. **帧率控制**：
   * 始终调用 `SetTargetFPS(60)`（或你想要的帧率），这会让 `GetFrameTime()` 返回准确的时间差，确保游戏在不同性能的设备上速度一致。

---

## 10. 总结与学习资源

Raylib 的 2D 渲染 API 设计得非常符合直觉，掌握了 `Texture2D`、`DrawTexturePro` 和 `Camera2D` 这“三驾马车”，你就足以开发出绝大多数 2D 游戏和可视化应用。

**推荐资源：**
* **官方文档**：[raylib.com](https://www.raylib.com/) (最权威的 API 参考)
* **官方示例 (Cheat Sheets)**：[raylib.com/examples](https://www.raylib.com/examples.html) (包含大量可直接运行的 2D/3D 示例代码)
* **Raylib Discord**：社区非常活跃，遇到问题可以快速得到解答。

希望这份文档能帮助你（eoolife）在 Raylib 的 2D 渲染世界里畅游！如果有任何具体 API 的疑问，随时欢迎探讨。祝编码愉快！
