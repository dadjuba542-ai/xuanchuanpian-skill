<div align="center">

# 宣传片 Seedance 生产流水线

**宣传片类视频多步生产 SKILL 集合 —— 文案拆画面类型 → N 宫格图片提示词 → 生图 → 视频提示词 → 生成**

一套面向 **字节跳动即梦（Seedance 2.0）** 的宣传片视频生产工具链，包含两个配套 Agent SKILL。

[![Skill](https://img.shields.io/badge/SKILL-xuanchuan--seedance-6E56CF)](skills/xuanchuan-seedance/SKILL.md)
[![Skill](https://img.shields.io/badge/SKILL-seedance-0EA5E9)](skills/seedance/SKILL.md)
[![Platform](https://img.shields.io/badge/Platform-即梦%20Seedance%202.0-22C55E)](https://jimeng.jianying.com)

</div>

---

## 仓库结构

```
xuanchuanpian-skill/
├── README.md                          # 本文件
└── skills/
    ├── xuanchuan-seedance/            # 宣传片专用生产 SKILL（本仓库核心）
    │   └── SKILL.md
    └── seedance/                      # 通用 Seedance 提示词 SKILL（依赖）
        ├── SKILL.md
        ├── LICENSE
        ├── README.md
        └── references/                # 相机编码 / 美学约束 / 剪辑节奏 等完整方法论
```

## SKILL 关系

| SKILL | 定位 | 依赖关系 |
|-------|------|---------|
| **`xuanchuan-seedance`** | 宣传片专用多步生产管线 | 依赖 `seedance` 的提示词格式/相机编码/美学约束 |
| **`seedance`** | 通用 Seedance 提示词生成（底层能力库） | 独立可用 |

**`xuanchuan-seedance` 是面向「宣传片」场景的编排层，`seedance` 是提示词生成的方法论层。** 前者管理流程（6 个 Phase），后者提供工具（模板/编码/约束）。

---

## 六步工作流

```
Phase 1         Phase 2           Phase 3         Phase 4           Phase 5       Phase 6
文案拆画面类型 → N宫格图片提示词 → 外部生图(N张) → AI看N张图出      → 人工审改    → N张图+1条
                                                   1条视频提示词                   提示词→1条15s视频
```

| 阶段 | 内容 | 自动化程度 |
|------|------|-----------|
| **Phase 1** | 文案逐句拆出画面类型（叙事/产品/情绪/空镜等），AI 出候选，人确认 | 🔵 子 agent + 人工 |
| **Phase 2** | AI 建议宫格数（6/9/12），生成 N 条图片提示词（中英双语） | 🔵 子 agent + 人工微调 |
| **Phase 3** | 手动复制到生图工具（NanoBanana / MJ / 即梦图片）生图 | 🔴 纯人工 |
| **Phase 4** | 多模态模型分析 N 张图，生成 1 条视频提示词（中英双语），引用 `@图片1~@图片N` | 🔵 子 agent |
| **Phase 5** | 人工逐项审改（@编号/运镜/色调/字符数） | 🔴 纯人工 |
| **Phase 6** | 上传 N 图 + 1 条提示词到即梦，生成 1 条 15s 视频 | 🔴 纯人工 |

### 宫格数选择逻辑

宫格数 = 分镜精细度，类似漫画分格——**格数越多，每格信息越少，视频节奏越稳定**：

| 宫格数 | 分镜精细度 | 节奏效果 |
|--------|-----------|---------|
| **6 宫格** | 粗略拆分，每格承载较多画面信息 | 单格停留长，节奏偏慢 |
| **9 宫格** | 中等拆分，每格一个明确画面概念 | 节奏适中，每格 1.5-2s |
| **12 宫格** | 精细拆分，每格一个单一镜头/动作 | 节奏紧凑，每格 1-1.5s |

N 宫格 = N 张图作参考素材 + 1 条视频提示词 → 即梦生成 1 条 15s 视频。

### 提示词：中英双语结构

所有提示词采用 **中文（上）+ 英文（下）** 格式：
- **中文** — 写清楚细节描述，供人工参考确认
- **英文** — 简洁准确的关键词，复制到生图/视频平台（模型对英文支持更好）

---

## 安装

### 方式 1：Hermes Agent（推荐）

```bash
# 克隆到本地
git clone https://github.com/dadjuba542-ai/xuanchuanpian-skill.git
cd xuanchuanpian-skill

# 复制到 Hermes SKILL 目录
cp -r skills/xuanchuan-seedance ~/.hermes/skills/
cp -r skills/seedance ~/.hermes/skills/
```

然后用 Hermes 的 `skill_view(name='xuanchuan-seedance')` 加载。

### 方式 2：其他 Agent（Claude Code / Cursor 等）

将 `skills/` 目录下的两个 SKILL 放入对应 Agent 的 Skills 搜索路径即可。

---

## 快速使用

```bash
# 加载宣传片 SKILL
skill_view(name='xuanchuan-seedance')

# 底层方法论（相机编码/美学约束/剪辑公式）
skill_view(name='seedance')
skill_view(name='seedance', file_path='references/camera-codec.md')
skill_view(name='seedance', file_path='references/aesthetic-constraints.md')
```

---

## 翻车预防（摘要）

- **提示词不要包含文字** — 视频中的文字/LOGO/字幕等后期用剪映添加，Seedance 2.0 视频内文字渲染不可靠
- **先试片再定稿** — 第一次跑可能宫格切换顺序不对，先跑一版看效果，不满意回改提示词
- **宫格图风格必须统一** — N 张图画风不一致即梦难以弥合，生图时就用统一风格锚定
- **画幅比一致** — 生图画幅比 = 视频目标画幅比，否则裁切丢关键构图
- **字符数 ≤2000** — 英文 prompt 用 `len()` 实测，留 100 字符余量

---

## 依赖

- [Hermes Agent](https://hermes-agent.nousresearch.com/) 或支持 SKILL 规范的 Agent 工具
- `seedance` SKILL（本仓库已包含）
- 生图工具（NanoBanana / Midjourney / 即梦图片生成）
- 即梦平台（dreamina）账号

---

## 许可

`xuanchuan-seedance` SKILL 适用于 MIT License。
`seedance` SKILL 以其自身许可为准（详见 `skills/seedance/LICENSE`）。
