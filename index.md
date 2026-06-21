## 1. 简介

**Raylib** 是一个简单且易用的 C/C++ 库，用于学习游戏编程和开发多媒体应用程序。它高度封装了底层图形、音频和输入 API，提供了极其简洁的函数接口。

**核心特点：**
- **极简 API**：函数命名直观，无需复杂的面向对象层级。
- **无外部依赖**：核心库不依赖第三方库（底层依赖 OpenGL/DirectX 等系统 API）。
- **跨平台**：支持 Windows, Linux, macOS, Android, Web (HTML5) 等。
- **C 风格接口**：在 C++ 中可直接使用，也可配合 `raylib-cpp` 等 C++ 封装库使用。

---

## 2. 核心架构与游戏循环

Raylib 遵循经典的游戏开发模式：**初始化 -> 游戏循环 -> 清理**。所有的渲染和逻辑更新都必须在 `BeginDrawing()` 和 `EndDrawing()` 之间进行。

```cpp
#include "raylib.h"

int main() {
    // 1. 初始化
    InitWindow(800, 450, "Raylib C++ 基础架构");
    SetTargetFPS(60); // 设置目标帧率

    // 2. 游戏主循环
    while (!WindowShouldClose()) // 检测是否点击了关闭按钮或按下 ESC
    {
        // 更新逻辑 (Update)
        // ...

        // 绘制画面 (Draw)
        BeginDrawing();
            ClearBackground(RAYWHITE); // 清屏
            DrawText("Hello, Raylib!", 190, 200, 20, LIGHTGRAY);
        EndDrawing();
    }

    // 3. 清理资源
    CloseWindow();
    return 0;
}
```

---

## 3. 核心模块 (Core)

### 3.1 窗口管理
控制应用程序窗口的创建、状态和属性。

| 函数 | 说明 |
|---|---|
| `InitWindow(width, height, title)` | 初始化窗口和 OpenGL 上下文 |
| `CloseWindow()` | 关闭窗体并释放上下文 |
| `WindowShouldClose()` | 检测用户是否请求关闭窗口 |
| `SetWindowSize(width, height)` | 动态调整窗口大小 |
| `SetWindowState(flags)` | 设置窗口状态（如 `FLAG_FULLSCREEN_MODE`） |
| `IsWindowFullscreen()` | 检查是否处于全屏模式 |

### 3.2 输入处理
Raylib 的输入分为**状态检测**（持续为真）和**事件检测**（仅在触发帧为真）。

#### 键盘输入
```cpp
// 状态检测（按住时持续返回 true）
if (IsKeyDown(KEY_RIGHT)) player.x += 5;

// 事件检测（仅在按下的那一帧返回 true）
if (IsKeyPressed(KEY_SPACE)) Jump();
```

#### 鼠标输入
```cpp
Vector2 mousePos = GetMousePosition(); // 获取鼠标坐标
if (IsMouseButtonPressed(MOUSE_LEFT_BUTTON)) { /* 左键点击 */ }
float wheelMove = GetMouseWheelMove(); // 获取滚轮滚动值
```

#### 手柄输入
```cpp
if (IsGamepadAvailable(0)) {
    float axisX = GetGamepadAxisMovement(0, GAMEPAD_AXIS_LEFT_X);
    if (IsGamepadButtonPressed(0, GAMEPAD_BUTTON_RIGHT_FACE_DOWN)) { /* A键 */ }
}
```

### 3.3 时间与帧率
| 函数 | 说明 |
|---|---|
| `SetTargetFPS(fps)` | 限制最大帧率（内部使用 sleep） |
| `GetFPS()` | 获取当前实际帧率 |
| `GetFrameTime()` | 获取上一帧所花费的时间（秒），用于帧率无关的移动 |
| `GetTime()` | 获取自 `InitWindow` 以来的总时间（秒） |

---

## 4. 2D 渲染模块

### 4.1 基本形状 (Shapes)
用于绘制简单的几何图形，无需加载外部资源。

```cpp
DrawLine(startX, startY, endX, endY, color);
DrawCircle(centerX, centerY, radius, color);
DrawRectangle(posX, posY, width, height, color);
DrawTriangle(v1, v2, v3, color); // v1, v2, v3 为 Vector2

// 绘制带圆角的矩形
DrawRectangleRounded(rec, roundness, segments, color);
```

### 4.2 纹理与图像 (Textures)
处理精灵图、背景图等。

```cpp
// 加载与卸载
Texture2D tex = LoadTexture("sprite.png");
UnloadTexture(tex);

// 绘制
DrawTexture(tex, posX, posY, tint); 
DrawTextureEx(tex, position, rotation, scale, tint);

// 高级绘制（用于精灵图裁剪、翻转、缩放）
Rectangle sourceRec = { 0, 0, 32, 32 }; // 源图裁剪区域
Rectangle destRec = { 100, 100, 64, 64 }; // 目标绘制区域
Vector2 origin = { 32, 32 }; // 旋转/缩放中心点
DrawTexturePro(tex, sourceRec, destRec, origin, 45.0f, WHITE);
```

### 4.3 文本与字体 (Text)
```cpp
// 默认字体绘制
DrawText("Score: 100", 10, 10, 20, BLACK);

// 自定义字体
Font font = LoadFont("custom_font.ttf");
Vector2 pos = { 10, 50 };
DrawTextEx(font, "Custom Text", pos, 30.0f, 2.0f, DARKBLUE);
UnloadFont(font);

// 测量文本尺寸
int textWidth = MeasureText("Hello", 20);
```

---

## 5. 3D 渲染模块

### 5.1 3D 相机 (Camera)
3D 场景必须通过相机来观察。

```cpp
Camera3D camera = { 0 };
camera.position = (Vector3){ 10.0f, 10.0f, 10.0f };
camera.target = (Vector3){ 0.0f, 0.0f, 0.0f };
camera.up = (Vector3){ 0.0f, 1.0f, 0.0f };
camera.fovy = 45.0f;
camera.projection = CAMERA_PERSPECTIVE;

// 在游戏循环中：
BeginMode3D(camera);
    DrawCube((Vector3){0, 0, 0}, 2.0f, 2.0f, 2.0f, RED);
    DrawGrid(10, 1.0f); // 绘制参考网格
EndMode3D();

// 自动更新相机（支持多种模式：自由、轨道、第一人称等）
UpdateCamera(&camera, CAMERA_FREE); 
```

### 5.2 模型与网格 (Models)
加载和渲染 3D 资产（支持 `.obj`, `.gltf`, `.glb` 等）。

```cpp
Model model = LoadModel("cube.glb");
// 设置材质纹理（可选）
// model.materials[0].maps[MATERIAL_MAP_DIFFUSE].texture = LoadTexture("diffuse.png");

BeginMode3D(camera);
    DrawModel(model, (Vector3){0, 0, 0}, 1.0f, WHITE);
EndMode3D();

UnloadModel(model);
```

---

## 6. 音频模块 (Audio)

在使用任何音频功能前，必须初始化音频设备。

```cpp
InitAudioDevice();

// 1. 声音 (Sounds) - 适合短促的音效，完全加载到内存
Sound fx = LoadSound("jump.wav");
SetSoundVolume(fx, 0.8f); // 设置音量 0.0 - 1.0
PlaySound(fx);
UnloadSound(fx);

// 2. 音乐流 (Music) - 适合背景音乐，流式播放，不占用大量内存
Music bgm = LoadMusicStream("bgm.mp3");
PlayMusicStream(bgm);

// 在游戏循环中必须调用此函数以更新音乐流
UpdateMusicStream(bgm); 

UnloadMusicStream(bgm);
CloseAudioDevice();
```

---

## 7. 数学与碰撞检测

### 7.1 数学向量
Raylib 提供了丰富的 `Vector2`, `Vector3`, `Vector4` 和 `Matrix` 运算函数。

```cpp
Vector2 a = { 10, 20 };
Vector2 b = { 5, 5 };

Vector2 sum = Vector2Add(a, b);
float length = Vector2Length(a);
Vector2 normalized = Vector2Normalize(a);
float dot = Vector2DotProduct(a, b);
```

### 7.2 碰撞检测
内置了高效的 2D 和 3D 碰撞检测函数。

#### 2D 碰撞
```cpp
Rectangle rec1 = { 0, 0, 50, 50 };
Rectangle rec2 = { 40, 0, 50, 50 };
Circle c1 = { .center = {100, 100}, .radius = 20 };

if (CheckCollisionRecs(rec1, rec2)) { /* 矩形与矩形 */ }
if (CheckCollisionCircleRec(c1.center, c1.radius, rec1)) { /* 圆与矩形 */ }

// 获取碰撞重叠区域（用于物理反弹）
Rectangle collisionRec = GetCollisionRec(rec1, rec2);
```

#### 3D 碰撞与射线检测
```cpp
BoundingBox box = { .min = {-1,-1,-1}, .max = {1,1,1} };
Ray ray = { .position = {0,0,0}, .direction = {0,0,-1} };

RayCollision hit = GetRayCollisionBox(ray, box);
if (hit.hit) {
    Vector3 hitPoint = hit.point; // 碰撞点
}
```

---

## 8. 高级功能：着色器 (Shaders)

Raylib 允许你使用自定义的 GLSL/HLSL 着色器来实现高级视觉效果。

```cpp
Shader shader = LoadShader("vertex.vs", "fragment.fs");

// 获取 uniform 变量位置
int glowLoc = GetShaderLocation(shader, "glowIntensity");
float glowValue = 1.5f;
SetShaderValue(shader, glowLoc, &glowValue, SHADER_UNIFORM_FLOAT);

BeginShaderMode(shader);
    DrawTexture(tex, 0, 0, WHITE); // 该纹理将使用自定义着色器绘制
EndShaderMode();

UnloadShader(shader);
```

---

## 9. 完整实战示例：2D 移动与收集

以下是一个包含移动、碰撞、音效和计分系统的完整 2D 游戏框架：

```cpp
#include "raylib.h"

int main() {
    // 初始化
    const int screenWidth = 800;
    const int screenHeight = 450;
    InitWindow(screenWidth, screenHeight, "Raylib 2D Demo");
    InitAudioDevice();

    // 资源加载
    Texture2D playerTex = LoadTexture("player.png");
    Texture2D coinTex = LoadTexture("coin.png");
    Sound coinSfx = LoadSound("coin.wav");
    
    // 游戏状态
    Vector2 playerPos = { screenWidth / 2.0f, screenHeight / 2.0f };
    Vector2 coinPos = { 600, 200 };
    float playerSpeed = 300.0f; // 像素/秒
    int score = 0;

    SetTargetFPS(60);

    // 主循环
    while (!WindowShouldClose()) {
        float dt = GetFrameTime();

        // 1. 更新逻辑
        if (IsKeyDown(KEY_RIGHT)) playerPos.x += playerSpeed * dt;
        if (IsKeyDown(KEY_LEFT))  playerPos.x -= playerSpeed * dt;
        if (IsKeyDown(KEY_UP))    playerPos.y -= playerSpeed * dt;
        if (IsKeyDown(KEY_DOWN))  playerPos.y += playerSpeed * dt;

        // 碰撞检测 (假设图像大小为 32x32)
        Rectangle playerRec = { playerPos.x, playerPos.y, 32, 32 };
        Rectangle coinRec = { coinPos.x, coinPos.y, 32, 32 };

        if (CheckCollisionRecs(playerRec, coinRec)) {
            score++;
            PlaySound(coinSfx);
            // 重置金币位置
            coinPos.x = GetRandomValue(50, screenWidth - 50);
            coinPos.y = GetRandomValue(50, screenHeight - 50);
        }

        // 2. 绘制画面
        BeginDrawing();
            ClearBackground(SKYBLUE);

            DrawTexture(playerTex, playerPos.x, playerPos.y, WHITE);
            DrawTexture(coinTex, coinPos.x, coinPos.y, WHITE);

            DrawText(TextFormat("Score: %i", score), 10, 10, 30, BLACK);
            DrawFPS(10, 50); // 绘制帧率
        EndDrawing();
    }

    // 清理
    UnloadTexture(playerTex);
    UnloadTexture(coinTex);
    UnloadSound(coinSfx);
    CloseAudioDevice();
    CloseWindow();

    return 0;
}
```

---

## 10. 最佳实践与 C++ 开发建议

1. **资源生命周期管理**
   - Raylib 是 C 风格 API，**不会自动释放内存**。
   - 遵循 `Load...` 必须对应 `Unload...` 的原则。
   - **切忌**在游戏主循环（`while` 循环）内部加载资源，这会导致严重的卡顿和内存泄漏。

2. **使用 C++ 封装 (可选)**
   - 虽然 Raylib 是 C 库，但在 C++ 中，你可以使用社区维护的 `raylib-cpp` 头文件库。它利用 RAII 机制自动管理资源，并提供面向对象的接口（如 `raylib::Window`, `raylib::Texture2D`）。

3. **帧率无关的移动**
   - 永远使用 `GetFrameTime()` (即 `dt`) 乘以速度来更新位置，确保游戏在不同帧率的设备上表现一致。
   - `position.x += speed * GetFrameTime();`

4. **调试与日志**
   - 使用 `SetTraceLogLevel(LOG_DEBUG)` 开启详细日志。
   - 使用 `TraceLog(LOG_WARNING, "Custom warning: %s", msg)` 输出自定义日志到控制台。

5. **屏幕适配与坐标系**
   - Raylib 的 2D 坐标系原点在**左上角**，X 向右，Y 向下。
   - 使用 `GetScreenWidth()` 和 `GetScreenHeight()` 动态获取窗口尺寸，以支持窗口大小调整。

6. **代码组织**
   - 建议将游戏状态封装在 `struct` 或 `class` 中（如 `GameState`），将 `Update` 和 `Draw` 逻辑分离，保持主循环的整洁。
