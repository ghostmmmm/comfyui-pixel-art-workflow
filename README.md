# GPT Image 2 像素素材工作流

一套面向游戏素材制作的 ComfyUI 工作流，使用 **GPT Image 2** 生成基础图，再经过自动抠图、像素化、限色、硬 Alpha 和 1px 黑色描边处理，输出可直接继续编辑或导入游戏项目的透明 PNG。

仓库包含物品、技能图标、人物头像和全身人物四类工作流。每一类均支持：

- 文生图与参考图重绘两种模式
- 多种最终输出尺寸
- 透明背景 RGBA PNG
- 无抖动限色，保持清晰硬边
- 自动生成 1px 黑色外轮廓
- 最近邻放大预览，方便检查真实像素结构
- 可选 Perfect Pixel 网格修复

> [!IMPORTANT]
> 这些工作流调用 ComfyUI 的 **OpenAI GPT Image 2 Partner Node**。运行生成节点会使用联网服务并产生相应费用，不是纯本地、免费工作流。

## 工作流列表

| 类型 | 工作流 | 可选尺寸 | 默认尺寸 | 色彩设置 | 默认输出目录 |
|---|---|---:|---:|---|---|
| 游戏物品 | [gpt物品生图32，64，128.json](./gpt物品生图32，64，128.json) | 32 / 64 / 128 | 64 | 主体 15 色，加入黑边后最多约 16 色 | `output/pixel_items/` |
| 技能/状态图标 | [gpt图标生图64，100，128.json](./gpt图标生图64，100，128%20.json) | 64 / 100 / 128 | 100 | 主体 15 色，加入黑边后最多约 16 色 | `output/pixel_icons/` |
| 人物头像 | [gpt人物头像生图64，128，256.json](./gpt人物头像生图64，128%20，256.json) | 64 / 128 / 256 | 64 | 16 / 32 / 48 色档位 | `output/pixel_portraits/` |
| 全身人物 | [gpt人物生图64，80，100，128.json](./gpt人物生图64，80，100，128.json) | 64 / 80 / 100 / 128 / 256 | 64 | 总色数可在 16–32 之间调整，默认 24 | `output/pixel_characters/` |

工作流内部的 `Save Image` 节点还会继续建立各自的英文子目录。

## 处理流程

```text
GPT Image 2 文生图 / 参考图重绘
                ↓
         BiRefNet 自动抠图
                ↓
       Area 缩小到目标分辨率
                ↓
    Perfect Pixel（默认旁路，可选）
                ↓
       最近邻锁定最终尺寸
                ↓
       无抖动调色板量化
                ↓
      Alpha 硬化 + 1px 黑边
                ↓
        保存透明 RGBA PNG
```

## 环境要求

- 较新的 [ComfyUI](https://github.com/Comfy-Org/ComfyUI)
- 可正常访问 Partner Nodes 的网络环境
- 已登录且具备可用额度的 Comfy 账号
- 推荐使用 ComfyUI-Manager 安装缺失节点

GPT Image 2 是 ComfyUI 的 Partner Node。请先更新 ComfyUI，并在界面中登录 Comfy 账号。官方说明：

- [OpenAI GPT Image 2 节点](https://docs.comfy.org/zh/tutorials/partner-nodes/openai/gpt-image-2)
- [Partner Nodes 说明](https://docs.comfy.org/tutorials/partner-nodes/overview)

## 自定义节点

四份工作流会用到以下扩展：

| 扩展 | 使用的节点 | 安装地址 |
|---|---|---|
| ComfyUI-RMBG | `BiRefNetRMBG` | [1038lab/ComfyUI-RMBG](https://github.com/1038lab/ComfyUI-RMBG) |
| ComfyUI-Pixelate | `ComfyUIPixelate` | [flycarl/ComfyUI-Pixelate](https://github.com/flycarl/ComfyUI-Pixelate) |
| ComfyUI Impact Pack | `ImpactSwitch`、`ImpactInt` | [ltdrdata/ComfyUI-Impact-Pack](https://github.com/ltdrdata/ComfyUI-Impact-Pack) |
| ComfyUI-Custom-Scripts | `MathExpression\|pysssss` | [pythongosssss/ComfyUI-Custom-Scripts](https://github.com/pythongosssss/ComfyUI-Custom-Scripts) |
| perfectPixel-ComfyUI | `PerfectPixel` | [TobyKSKGD/perfectPixel-ComfyUI](https://github.com/TobyKSKGD/perfectPixel-ComfyUI) |

其中 `ImpactInt` 和 `MathExpression` 只在“全身人物”工作流中用于控制总色数。

安装 `perfectPixel-ComfyUI` 后，还需要在 **ComfyUI 正在使用的 Python 环境**中安装：

```bash
pip install "perfect-pixel[opencv]"
```

首次运行 `BiRefNetRMBG` 时可能会自动下载 `BiRefNet_toonout` 模型。请按照 ComfyUI-RMBG 项目的说明完成模型安装。

## 使用方法

1. 下载需要的 `.json` 文件。
2. 启动 ComfyUI。
3. 将 JSON 拖入 ComfyUI 页面，或使用菜单中的“打开”载入工作流。
4. 如果出现红色节点，使用 ComfyUI-Manager 的“安装缺失节点”，然后完整重启 ComfyUI。
5. 在 `00B 生图模式` 中选择生成方式：
   - `1`：文生图
   - `2`：参考图重绘
6. 文生图时，修改 `01A GPT Image 2 文生图` 提示词的首行主题。
7. 图生图时，在 `00A 可选参考图` 中载入自己的图片，并修改 `01B` 提示词。
8. 在 `20A 最终尺寸` 中选择输出尺寸。
9. 点击“运行”或“加入队列”，等待生成和后处理完成。

> [!TIP]
> 工作流自带的 `Load Image` 节点可能保留了作者本机使用过的参考图文件名。文生图模式不需要它；图生图模式请换成你自己的参考图。

## 关键参数

### 生图模式

`00B 生图模式`：

- `1`：使用 `01A`，只根据文字生成
- `2`：使用 `01B`，根据 `00A` 中的参考图重绘

### 输出尺寸

`20A 最终尺寸` 的数字与节点标题中标注的尺寸一一对应。例如物品工作流：

```text
1 = 32×32
2 = 64×64
3 = 128×128
```

请以当前工作流中 `20A` 节点的标题为准。

### 全身人物色数

全身人物工作流提供 `20B 输出总色数`：

- 允许范围：16–32
- 默认值：24
- 节点会自动为黑色描边预留 1 个颜色位置
- 超出范围的输入会自动限制到 16–32

### Perfect Pixel

Perfect Pixel 分支默认旁路。它适合修复不规则像素网格，但可能改变局部结构或输出尺寸。建议先比较旁路结果和启用结果，再决定是否使用。

## 提示词修改建议

工作流里的英文提示词已经包含构图、占画面比例、像素硬边、纯色洋红背景和负面约束。通常只需要修改第一行中的主题，不要轻易删除后面的结构约束。

示例：

```text
Create one iron sword as a production-ready 2D game inventory item.
```

```text
Create one blue flame magic ability symbol as a production-ready 2D pixel-art game UI icon.
```

```text
Create one forest ranger as a production-ready square pixel-art dialogue portrait.
```

如果使用参考图，建议明确要求保留：

- 人物或物品身份
- 轮廓和主要设计特征
- 原有配色、服装和配件
- 居中构图与完整边缘

## 输出说明

最终成品为透明背景 PNG。默认输出位置在：

```text
ComfyUI/output/
├─ pixel_items/
├─ pixel_icons/
├─ pixel_portraits/
└─ pixel_characters/
```

工作流中的 512×512 图像仅用于最近邻放大预览，不是最终成品尺寸。

## 常见问题

### 找不到 GPT Image 2 节点

更新 ComfyUI。当前节点在界面中可能显示为 **OpenAI GPT Image 1.5**，在它的 `model` 参数中选择 `gpt-image-2`。还要确认启动参数没有使用 `--disable-api-nodes`。

### GPT Image 2 无法运行或提示未登录

检查 ComfyUI 账号登录状态、Partner Node 可用区域、网络环境和账号额度。登录 ComfyUI 账号与填写 OpenAI Platform 的 `sk-...` API Key 不是同一件事。

### BiRefNet 模型缺失

首次使用时保持网络连接，或按照 ComfyUI-RMBG 文档手动放置模型。工作流使用的是 `BiRefNet_toonout`。

### 打开工作流后出现红色节点

先通过 ComfyUI-Manager 安装缺失节点，完整重启 ComfyUI，再重新载入 JSON。只刷新浏览器不一定能加载刚安装的 Python 节点。

### 成品边缘缺失或出现残留背景

检查 `P2B RMBG前景Mask预览`。如果主体没有完整变白，可调整 RMBG 参数；如果洋红背景残留，可让提示词继续保持纯色 `#FF00FF` 背景，并避免阴影、光晕和半透明效果。

### 成品看起来模糊

请查看工作流内的最近邻放大预览，不要让图片查看器使用平滑插值。导入游戏引擎后，也应将纹理过滤设置为 Point/Nearest。

## 注意事项

- GPT Image 2 的生成具有随机性，同一提示词不保证每次完全一致。
- 工作流会把 AI 生成图后处理为更规整的像素素材，但不能替代人工像素级修图。
- 透明边缘、1px 描边和限色会改变非常细小的原始细节。
- 生成内容及其使用方式应遵守 ComfyUI、模型服务和相关平台的条款。
- 本仓库只分享工作流，不包含 GPT Image 2、BiRefNet 或其他模型权重。

## 致谢

感谢 ComfyUI 与上述自定义节点的作者。工作流将云端图像生成与本地像素化后处理组合在一起；各节点和模型的版权及许可证归其各自作者所有。

