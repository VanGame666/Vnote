# LVGL问题复盘

---

## 问题 1：按住时可以拖动（滚动干扰）

### 问题现象
按住长按控件时，手指拖动会导致屏幕滚动，不是真正的"独占交互"。

### 根本原因
LVGL 的**滚动链机制**和**手势冒泡机制**：

| 机制 | 说明 |
|-----|-----|
| `LV_OBJ_FLAG_SCROLL_CHAIN` | 子控件按住时，如果子控件不能滚动，会把滚动事件传递给父容器 |
| `LV_OBJ_FLAG_GESTURE_BUBBLE` | 子控件的手势事件会冒泡给父容器处理 |
| 事件冒泡 | `LV_EVENT_PRESSED` 等事件没有被阻止，父容器可以收到 |

### 解决方案：5 层防护
```c
// 第 1 层：控件本身不可滚动
lv_obj_clear_flag(obj, LV_OBJ_FLAG_SCROLLABLE);

// 第 2 层：禁止聚焦时滚动
lv_obj_clear_flag(obj, LV_OBJ_FLAG_SCROLL_ON_FOCUS);

// 第 3 层：禁止滚动链（最关键！防止父容器滚动）
lv_obj_clear_flag(obj, LV_OBJ_FLAG_SCROLL_CHAIN);

// 第 4 层：禁止手势冒泡
lv_obj_clear_flag(obj, LV_OBJ_FLAG_GESTURE_BUBBLE);

// 第 5 层：事件层级阻止冒泡
if(code == LV_EVENT_PRESSED || code == LV_EVENT_PRESSING || ...) {
    lv_event_stop_bubbling(e);
}
```

---

## 问题 2：长按达到时间后崩溃（事件系统问题）

### 问题现象
按住长按控件，达到设定时间后程序崩溃。

### 排查过程 & 失败尝试

#### 尝试 1：在定时器中直接 `lv_event_send`
```c
static void timer_check(lv_timer_t * t) {
    ...
    lv_event_send(obj, LV_EVENT_READY, NULL);  // ← 崩溃
}
```

**崩溃原因：** 类的 `event_handler` 中有这段代码：
```c
case LV_EVENT_READY:
    pgif->triggered = false;  // ← 这里把触发标记重置了！
    break;
```

**执行顺序问题：**
```
lv_event_send(LV_EVENT_READY)
  ├─ 类的 event_handler 先被调用 ← triggered 被重置为 false
  └─ 用户的回调后被调用
```

定时器还没删除，下一次回调又进来，**无限循环**最终崩溃。

---

#### 尝试 2：删除类中那个 `LV_EVENT_READY` case
```c
// 删除了类中的 LV_EVENT_READY case，但还是崩溃
```

**崩溃原因：** 定时器回调是在特殊上下文执行的，**直接发送事件有重入问题**（LVGL 内部 bug 或复杂交互）。

---

#### 尝试 3：用 `lv_async_call` 推迟到下一个主循环
```c
lv_async_call(async_send_event, obj);  // ← 推迟发送，还是崩溃
```

**崩溃原因：** 事件系统本身在某些情况下还是有问题（具体原因不明）。

---

### 最终解决方案：直接函数指针回调

```c
// 头文件中定义直接回调
typedef struct {
    ...
    void (*callback)(lv_obj_t *, void *);  // 直接回调，不经过事件系统
    void * user_data;
} lv_press_gif_t;

// 定时器中直接调用
if(pgif->callback) {
    pgif->callback(obj, pgif->user_data);  // ✅ 简单、安全、可靠
}
```

**为什么这个方案最好？**
1. ✅ 完全绕过 LVGL 事件系统的复杂性
2. ✅ 没有重入问题
3. ✅ 代码简单，容易调试
4. ✅ 性能更好（少了一层事件分发）

---

## 总结教训

| 教训 | 说明 |
|-----|-----|
| 滚动链是很多"意外滚动"的根源 | 做自定义控件一定要清除 `LV_OBJ_FLAG_SCROLL_CHAIN` |
| 定时器中别直接发事件 | 特殊上下文 + 事件系统 = 不可预测的崩溃 |
| 最简单的方案往往最可靠 | 直接函数指针回调不"高级"，但没有副作用 |
| 类的事件处理要小心 | 别在类的事件处理中修改会影响外部逻辑的状态（如 triggered） |



# 为什么之前会模糊和位置错乱？

**根本原因：Stride（行跨度）对齐**

LVGL 为了性能，每行的字节数会对齐到 4、8、16 等边界，不是简单的 `宽度 × 每像素字节数`。

---

## 问题1：像素位置计算错误

**错误代码：**
```c
int argb_idx = (y * 200 + x) * 4;  // 假设每行=200像素×4字节
```

**实际情况：**
- 宽度=200像素，ARGB8888每像素4字节
- 200×4=800字节，但 LVGL 会对齐到**1024字节**（比如 64 对齐）
- 所以每行实际是 **1024 字节**，不是 800 字节
- 用 `y * 800` 找像素，相当于每行都偏移了 224 字节，所以文字**整体歪了、错位了**

**正确做法：**
```c
int stride = argb_buf.header.stride;  // 获取实际行字节数（比如 1024）
int idx = y * stride + x * 4;         // 正确计算像素位置
```

---

## 问题2：缓冲区不够大

**错误：** `uint8_t argb_data[200 * 50 * 4] = 40,000 字节`

**实际需要：** `1024 字节/行 × 50 行 = 51,200 字节`

缓冲区只有 40,000，装不下 51,200，导致**越界写内存**，文字被截断或只显示底部一条线。

---

## 问题3：手动指定 stride

**错误：** `lv_draw_buf_init(..., 200, ...)` 手动传 stride=200

**正确：** 传 `0` 让 LVGL 自动计算对齐后的 stride

```c
lv_draw_buf_init(&argb_buf, 300, 100, LV_COLOR_FORMAT_ARGB8888, 
                 0,  // stride=0 = 自动计算
                 argb_data, sizeof(argb_data));
```

---

## 总结

| 问题 | 现象 | 原因 |
|------|------|------|
| Stride 计算错 | 文字整体歪、错位、单个字符清晰但位置不对 | 没考虑行对齐 |
| 缓冲区太小 | 只显示底部一条横线、大部分不显示 | 越界写内存 |
| 手动传 stride | 对齐方式和预期不一致 | LVGL 内部会重新对齐 |

**最终解决方案核心就是一句话：**
```c
int stride = buf.header.stride;  // 用实际的 stride！
```


# LVGL 资源生命周期管理完整笔记

## 1. 核心的父-子对象树结构

LVGL 中的 UI 控件（Widgets）通过父-子关系构成一棵对象树。

- **唯一父对象**：除特殊对象（如屏幕）外，每个 UI 控件都有且仅有一个父对象。
- **自动递归删除**：使用 `lv_obj_del()` 或 `lv_obj_delete()` 删除一个对象时，其**所有子对象都会被自动、递归地删除**。
- **生命周期绑定**：子对象的生命周期严格绑定在父对象上。父对象移动/删除，子对象随之移动/删除。

**实用函数**：
- 删除父容器及其所有子控件：`lv_obj_del(parent)`
- 仅删除父容器的所有子控件，保留父容器：`lv_obj_clean(parent)`

---

## 2. UI 体系中的特殊对象（不依附于普通父容器）

| 对象类型 | 是否依附父对象 | 说明 |
|---------|--------------|------|
| **屏幕 (Screen)** | 否 | 所有 UI 对象树的根节点，没有父对象。删除屏幕会递归删除其上的所有 UI 控件。 |
| **图层 (Layer)** | 否 | 每个显示器自带四个独立图层（如 `lv_layer_top()`），可用于全局弹框等，独立于活动屏幕。 |

---

## 3. 独立于对象树的系统资源（不会随父容器删除自动清理）

以下资源/模块**完全不依附**于 UI 对象树，删除父容器不会自动释放它们，必须**手动管理**其生命周期。

|                资源类型                | 是否自动释放 |                                       手动清理方法                                        |                                          备注                                          |
| ------------------------------------- | ----------- | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| **定时器 (Timer)**                    | ❌ 否       | `lv_timer_del(timer)`                                                                    | 回调中若操作已删除的 UI 对象可能引发逻辑错误。可设置重复次数为 1 或使用 `lv_timer_pause()`。 |
| **动画 (Animation)**                  | ⚠️ 半自动   | 当动画作用的对象被删除时，动画会自动停止并清理。但若动画是独立创建的实体（如路径动画），仍需注意。 | 通常无需手动删除，但建议验证对象有效性。                                                   |
| **样式 (Style)**                      | ❌ 否       | `lv_style_reset(style)` 或 `lv_style_remove_props()`，若为动态分配需 `lv_mem_free()`       | 样式是配置对象，可被多个控件共享。删除控件不影响样式本身。                                   |
| **字体 (Font)**                       | ❌ 否       | 内置字体无需删除；动态加载的字体需调用 `lv_font_free(font)`                                  | 极易遗漏，造成内存泄漏。                                                                 |
| **图片缓存 (Image Cache)**            | ❌ 否       | `lv_img_cache_invalidate_src(src)` 或 `lv_img_cache_set_size(0)` 清空                    | 全局解码缓存，存储解码后的图片数据。                                                      |
| **文件系统驱动 (File system driver)** | ❌ 否       | 需显式注销，通常没有标准 API，需根据实现自行清理                                             | 通过 `lv_fs_drv_register()` 注册的驱动结构体。                                           |
| **主题 (Theme)**                      | ❌ 否       | `lv_theme_set_current(NULL)` 并释放旧主题对象                                             | 若动态创建了新主题并替换，旧主题不会自动释放。                                             |
| **用户自定义数据 (User Data)**         | ❌ 否       | 必须在删除对象前手动 `free(obj->user_data)`                                                | LVGL 不会自动释放 `lv_obj_set_user_data()` 分配的内存。                                  |

> ⚠️ **核心原则**：凡是 `lv_xxx_create`、`lv_xxx_register` 获得的**非 UI 对象资源**，都需要找到对应的 `lv_xxx_del` / `lv_xxx_free` / `lv_xxx_unregister` 函数**成对调用**。

---

## 4. 典型陷阱与最佳实践

### 4.1 删除父容器 ≠ 删除定时器

```c
// 错误示例：定时器仍在运行，但父容器已被删除
static void timer_cb(lv_timer_t * t) {
    lv_obj_t * label = t->user_data;   // label 已被删除，成为野指针
    lv_label_set_text(label, "Hello"); // 可能导致崩溃
}

void create_ui(void) {
    lv_obj_t * parent = lv_obj_create(lv_scr_act());
    lv_obj_t * label = lv_label_create(parent);
    lv_timer_t * t = lv_timer_create(timer_cb, 1000, label);
    // ... 后面删除 parent
    lv_obj_del(parent);  // label 被删除，但 timer 还在运行
}