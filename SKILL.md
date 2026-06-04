---
name: memoir-river
description: >-
  为长辈制作个人回忆录——对话引导、语音录制转写、文字整理修饰、生成"记忆河流"可视化个人网站。
  Use when the user wants to create a memoir, record voice memories, transcribe and beautify text,
  or generate a personal memory timeline website with particle river visualization.
  Triggers: 回忆录, memoir, 记忆, memory river, 语音记录, voice diary, 个人网站, personal site.
---

# 记忆河流 · Memoir River

> 为长辈打造的回忆录工具——用对话引出记忆碎片，用声音留住时光，用文字雕琢故事，用粒子河流呈现一生。

## 概述

本技能完成四件事：
1. **对话引导** — 用温暖的问题引出回忆者的记忆片段
2. **语音录制 & 转写** — 通过 `bl speech recognize` 将口述转为文字
3. **文字整理 & 修饰** — 通过 `bl text chat` 保留口语韵味、修饰为可读文本
4. **网站生成** — 输出一个自包含的 HTML 文件，以"记忆河流"粒子可视化呈现所有记忆

---

## 工作流程

### Phase 1: 建立记忆档案

在项目根目录创建 `memoir/` 文件夹：

```
memoir/
├── memories.json        # 所有记忆数据（结构化）
├── audio/               # 原始音频片段
├── site/                # 生成的网站
│   └── index.html       # 自包含单文件网站
└── transcripts/         # 转写原文
```

初始化 `memories.json`：
```json
{
  "subject": {
    "name": "回忆者姓名",
    "birthYear": 1961,
    "birthPlace": "出生地"
  },
  "memories": []
}
```

### Phase 2: 对话引导（引出记忆）

使用"记忆锚点"技术引导对话。按时间线从童年到现在，用具体、感官化的问题引出回忆：

```bash
bl text chat --system "你是一位温暖的回忆录访谈者。根据回忆者的年代背景（1961年出生），用贴近生活的问题引导回忆。每次只问一个问题，问题要具体、有画面感。避免空泛提问。" \
  --message "回忆者刚聊到了[上次话题]，请生成下一个引导问题"
```

**引导问题分类（见 [reference/interview-guide.md](reference/interview-guide.md)）：**
- 童年：气味、声音、游戏、食物
- 青年：第一次（工作、恋爱、离家）
- 壮年：重大选择、意外转折
- 新闻锚点：用历史事件勾连个人记忆

### Phase 3: 语音录制 & 转写

录制音频后，使用 ASR 转写：

```bash
# 单文件转写
bl speech recognize --url ./memoir/audio/memory_001.wav --language zh --out ./memoir/transcripts/memory_001.json

# 多人对话（如有访谈者）
bl speech recognize --url ./memoir/audio/memory_001.wav --language zh --diarization --speaker-count 2 --out ./memoir/transcripts/memory_001.json
```

### Phase 4: 文字整理 & 修饰

将转写原文整理为流畅、有温度的文字，保留口语特色：

```bash
bl text chat --system "你是一位文学编辑，专长是将口述回忆整理为可读文本。规则：1.保留说话人的语气和口头禅 2.去除重复和语气词 3.补充必要的时间/地点上下文 4.分段、加小标题 5.不编造任何未提及的细节" \
  --message "请整理以下口述内容：[转写文本]" \
  --max-tokens 8192
```

**生成一句话摘要（用于时间线悬停）：**
```bash
bl text chat --message "为以下回忆生成一句话摘要（不超过20字，有画面感）：[整理后文本]"
```

### Phase 5: 写入 memories.json

每条记忆的数据结构：
```json
{
  "id": "mem_001",
  "date": {
    "year": 1975,
    "month": 6,
    "day": null
  },
  "title": "一句话摘要（20字内）",
  "content": "整理后的完整文本",
  "rawTranscript": "原始转写文本",
  "audioFile": "audio/memory_001.wav",
  "audioSnippet": {
    "start": 12.5,
    "end": 25.0
  },
  "tags": ["童年", "食物", "夏天"],
  "newsAnchor": "1975年全国推广杂交水稻",
  "mood": "温暖",
  "location": "湖南长沙"
}
```

### Phase 6: 生成记忆河流网站

使用 [reference/site-template.html](reference/site-template.html) 模板，将 `memories.json` 数据注入生成最终网站：

```bash
# 网站文件输出位置
memoir/site/index.html
```

**网站特性：**
- 全屏粒子河流动画（每个粒子代表一段记忆）
- 时间线导航（年份/月份刻度）
- 鼠标悬停显示一句话摘要
- 点击粒子展开完整记忆文本
- 内嵌音频播放器（可试听原始语音片段）
- 响应式设计，手机/平板/桌面均可访问
- 纯静态单文件，可直接部署到任何托管服务

---

## 增量更新

每次新增记忆后：
1. 追加到 `memories.json`
2. 重新生成 `site/index.html`（模板自动读取所有数据）
3. 网站自然"生长"——新粒子出现在河流中

---

## 部署建议

```bash
# 本地预览
npx serve memoir/site/

# 部署到 Vercel（推荐）
cd memoir/site && npx vercel --prod

# 部署到 GitHub Pages
# 将 memoir/site/ 设为 GitHub Pages 根目录
```

---

## 注意事项

- 音频格式支持：wav, mp3, m4a, flac, ogg
- 单次转写最大文件 100MB / 2小时
- memories.json 单条 content 建议控制在 2000 字内（网站渲染性能）
- 音频片段裁剪：使用 ffmpeg 截取精华段落
  ```bash
  ffmpeg -i memory_001.wav -ss 12.5 -to 25.0 -c copy memoir/audio/snippet_001.wav
  ```
- 隐私保护：建议为网站设置密码访问或仅生成本地文件

---

## 完整示例

```bash
# 1. 初始化项目
mkdir -p memoir/{audio,transcripts,site}

# 2. 录制后转写
bl speech recognize --url ./memoir/audio/dad_childhood.wav --language zh --out ./memoir/transcripts/dad_childhood.json

# 3. 整理文字
bl text chat --system "整理口述回忆为可读文本，保留口语韵味" --message "$(cat ./memoir/transcripts/dad_childhood.json | jq -r '.text')" > ./memoir/transcripts/dad_childhood_edited.md

# 4. 生成摘要
bl text chat --message "一句话摘要（20字内）：$(head -5 ./memoir/transcripts/dad_childhood_edited.md)"

# 5. 更新 memories.json 并重新生成网站
# （由 Agent 自动完成 JSON 更新和 HTML 注入）
```

---

## 参考文件

- [reference/site-template.html](reference/site-template.html) — 记忆河流网站完整模板
- [reference/interview-guide.md](reference/interview-guide.md) — 对话引导问题库
- [reference/memory-schema.json](reference/memory-schema.json) — 数据结构定义
