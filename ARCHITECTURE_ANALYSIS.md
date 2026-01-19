# Skid-Homework 项目架构分析

本文档详细分析 Skid-Homework 项目的技术架构、处理流程、渲染机制和持久化策略。

---

## 目录

1. [整体架构概述](#1-整体架构概述)
2. [前端技术方案与响应式适配](#2-前端技术方案与响应式适配)
3. [图片上传与AI处理流程](#3-图片上传与ai处理流程)
4. [LLM响应渲染与绘图功能](#4-llm响应渲染与绘图功能)
5. [持久化策略](#5-持久化策略)
6. [官方部署实例分析](#6-官方部署实例分析)

---

## 1. 整体架构概述

### 1.1 技术栈总览

| 类别 | 技术选型 | 版本 |
|------|----------|------|
| **框架** | Next.js | 16.x |
| **UI 库** | React | 19.x |
| **样式** | TailwindCSS | 4.x |
| **状态管理** | Zustand | 5.x |
| **本地数据库** | Dexie (IndexedDB) | 4.x |
| **国际化** | i18next + react-i18next | 25.x |
| **AI SDK** | @google/genai, openai | 最新版 |
| **Markdown 渲染** | react-markdown | 10.x |
| **数学公式** | KaTeX + remark-math | 最新版 |
| **图表绘制** | JSXGraph, Mermaid, function-plot | 各最新版 |

### 1.2 项目结构

```
src/
├── ai/                    # AI 相关逻辑
│   ├── gemini.ts          # Gemini API 客户端
│   ├── openai.ts          # OpenAI 兼容 API 客户端
│   ├── prompts/           # System Prompts
│   │   ├── solve.prompt.md    # 解题主提示词
│   │   ├── chat.prompt.md     # 聊天提示词
│   │   └── tools/             # 工具调用提示词
│   └── response.ts        # 响应解析器
├── app/                   # Next.js App Router 页面
├── components/            # React 组件
│   ├── areas/             # 主要区域组件
│   ├── cards/             # 卡片组件
│   ├── chat/              # 聊天功能组件
│   ├── markdown/          # Markdown 渲染组件
│   │   └── diagram/       # 图表渲染器
│   └── ui/                # 基础 UI 组件 (shadcn/ui)
├── hooks/                 # 自定义 Hooks
├── store/                 # Zustand 状态管理
│   ├── ai-store.ts        # AI 配置状态
│   ├── problems-store.ts  # 题目/解答状态
│   ├── settings-store.ts  # 设置状态
│   ├── chat-db.ts         # IndexedDB Schema
│   └── chat-store.ts      # 聊天状态
└── utils/                 # 工具函数
```

### 1.3 架构特点

1. **纯前端应用**: 所有 AI API 调用直接从浏览器发起，无后端服务器
2. **Serverless 部署**: 支持 Vercel、Cloudflare Workers、Docker 多种部署方式
3. **客户端存储**: 使用 localStorage 和 IndexedDB 进行本地持久化
4. **多 AI 源支持**: 同时支持 Gemini 和 OpenAI 兼容 API

---

## 2. 前端技术方案与响应式适配

### 2.1 响应式设计策略

项目采用 **Mobile-First** 设计理念，通过自定义 Hook 检测设备类型：

```typescript
// src/hooks/use-media-query.ts
export function useMediaQuery(query: string) {
  const [matches, setMatches] = useState(false);
  
  useEffect(() => {
    const mql = window.matchMedia(query);
    setMatches(mql.matches);
    mql.addEventListener("change", listener);
    return () => mql.removeEventListener("change", listener);
  }, [query]);
  
  return matches;
}

// 使用示例
const isMobile = useMediaQuery("(max-width: 640px)");
const prefersTouch = useMediaQuery("(pointer: coarse)");
```

### 2.2 桌面端布局

桌面端采用 **三栏网格布局**：

```jsx
// src/components/pages/ScanPage.tsx
<div className="grid grid-cols-1 gap-6 md:grid-cols-3 lg:gap-8">
  <ActionsCard />  {/* 操作区 */}
  <PreviewCard />  {/* 预览区 - 占 2 列 */}
</div>
```

特点：
- 三列网格布局 (`md:grid-cols-3`)
- 键盘快捷键支持（Ctrl+1~5 快捷操作）
- Tab/方向键导航

### 2.3 移动端布局

移动端采用 **Tab 切换布局**：

```jsx
// src/components/pages/ScanPage.tsx
{isMobile ? (
  <Tabs value={activeTab} onValueChange={setActiveTab}>
    <TabsList className="grid w-full grid-cols-2">
      <TabsTrigger value="capture">拍摄</TabsTrigger>
      <TabsTrigger value="preview">预览</TabsTrigger>
    </TabsList>
    <TabsContent value="capture">
      <ActionsCard layout="mobile" />
    </TabsContent>
    <TabsContent value="preview">
      <PreviewCard layout="mobile" />
    </TabsContent>
  </Tabs>
) : (
  // 桌面端布局...
)}
```

### 2.4 触控手势支持

移动端支持滑动切换题目：

```typescript
// src/components/areas/SolutionsArea.tsx
import { useDrag } from "@use-gesture/react";
import { useSpring, animated } from "@react-spring/web";

const bindDrag = useDrag(
  ({ down, movement: [mx], elapsedTime }) => {
    if (!prefersTouch) return;
    
    api.start({ x: down ? mx : 0, immediate: down });
    
    if (down) return;
    if (elapsedTime > 450 || Math.abs(mx) < 60) return;
    
    if (mx < 0) goNextProblem();
    else goPrevProblem();
  },
  { enabled: prefersTouch, filterTaps: true, threshold: 25 }
);
```

### 2.5 安全区域适配

支持 iOS 刘海屏等设备：

```css
/* src/index.css */
:root {
  --safe-top: max(env(safe-area-inset-top, 0px), 0px);
}

.safe-area {
  padding-top: var(--safe-top);
}
```

---

## 3. 图片上传与AI处理流程

### 3.1 完整处理流程图

```
┌─────────────────┐
│  用户上传图片    │  (文件选择器 / 相机拍摄)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  创建 FileItem   │  生成 UUID, Object URL
└────────┬────────┘
         │
         ▼
┌─────────────────┐     开启?     ┌─────────────────┐
│ 图像增强 (可选)  │ ─────────────▶│  OpenCV.js 处理  │
│ imageEnhancement │              │  灰度化/二值化   │
└────────┬────────┘              └────────┬────────┘
         │                                │
         ▼                                ▼
┌─────────────────┐
│  转换为 Base64   │  arrayBuffer → uint8ToBase64
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  构建 AI 请求    │  System Prompt + Tool Prompts + 图片
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  发送到 LLM      │  Gemini / OpenAI Vision API
│  (Streaming)    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  解析响应       │  parseSolveResponse()
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  渲染解答       │  MarkdownRenderer + 图表组件
└─────────────────┘
```

### 3.2 图像预处理（可选）

使用 OpenCV.js 进行图像增强：

```typescript
// src/utils/image-post-processing.ts
export const processImage = async (imageFile: File) => {
  const cv = (window as any).cv;
  
  // 1. 转灰度
  cv.cvtColor(src, src, cv.COLOR_RGBA2GRAY);
  
  // 2. 背景估计与光照校正 (去除阴影)
  kernelBig = cv.getStructuringElement(cv.MORPH_RECT, new cv.Size(50, 50));
  cv.morphologyEx(src, bg, cv.MORPH_CLOSE, kernelBig);
  cv.divide(src, bg, dst, 255, -1);
  
  // 3. 二值化 (Otsu 自动阈值)
  cv.threshold(dst, dst, 0, 255, cv.THRESH_BINARY | cv.THRESH_OTSU);
  
  return { file, url: URL.createObjectURL(file) };
};
```

### 3.3 默认模型配置

**项目默认使用多模态模型**，不是单纯的语言模型：

| Provider | 默认模型 | 类型 |
|----------|----------|------|
| Gemini | `models/gemini-2.5-flash` | 多模态 (Vision) |
| OpenAI | `gpt-4.1-mini` | 多模态 (Vision) |

```typescript
// src/store/ai-store.ts
export const DEFAULT_GEMINI_MODEL = "models/gemini-2.5-flash";
export const DEFAULT_OPENAI_MODEL = "gpt-4.1-mini";
```

### 3.4 System Prompt 设定

主要的解题提示词位于 `src/ai/prompts/solve.prompt.md`：

```markdown
#### 角色

你是一个高级AI作业求解器 (Advanced AI Homework Solver)。
你的任务是精准、高效地分析用户上传的图片中的学术问题，并提供结构化的解答。

#### 核心任务

接收用户发送的图片，识别并解答其中的所有问题，
然后按照指定的 **Markdown KV** 格式返回结果。

#### 工作流程

1. **分析图片**: 识别并分割出所有独立的问题。
2. **提取问题 (OCR)**: 提取文本内容。
3. **求解问题**: 运用知识库解决问题。
4. **撰写解析**: 撰写详细、**分步 (Step-by-step)** 的解析过程。
5. **格式化输出**: 将所有结果整合到指定的文本结构中。

#### 输出格式

### PROBLEM_TEXT
这里是OCR识别出的完整问题文本。

### EXPLANATION
#### Step 1: 识别关键信息
...

### ANSWER
这里是问题的最终答案。
```

### 3.5 工具调用提示词

项目通过提示词实现"伪工具调用"，让 LLM 输出特定格式的代码块：

```typescript
// src/ai/prompts/prompt-manager.ts
export function getEnabledToolCallingPrompts() {
  return [jsxGraphToolPrompt, diagramToolPrompt, mermaidToolPrompt];
}
```

这些提示词指导 LLM 在需要绘图时输出特定语言标记的代码块，如：
- `\`\`\`jsxgraph` - 数学图形
- `\`\`\`plot-mermaid` - 流程图
- `\`\`\`svg` - 直接输出 SVG

### 3.6 响应解析

使用 `marked.js` 的 lexer 解析 LLM 响应：

```typescript
// src/ai/response.ts
export function parseSolveResponse(response: string): SolveResponse {
  // 按分隔符分割多个问题
  const rawChunks = response.split("---PROBLEM_SEPARATOR---");
  
  for (const chunk of rawChunks) {
    const parser = new MarkdownSectionParser(chunk);
    const sections = parser.getSectionsByH3();  // 按 ### 标题分组
    
    problems.push({
      problem: sections["PROBLEM_TEXT"],
      explanation: sections["EXPLANATION"],
      answer: sections["ANSWER"],
      steps: MarkdownSectionParser.parseSteps(sections["EXPLANATION"]),
    });
  }
  
  return { problems };
}
```

---

## 4. LLM响应渲染与绘图功能

### 4.1 Markdown 渲染管线

```
LLM 响应 (Markdown)
       │
       ▼
┌─────────────────────────────────┐
│        react-markdown           │
│  ┌───────────────────────────┐  │
│  │     remark-gfm            │  │  GitHub Flavored Markdown
│  ├───────────────────────────┤  │
│  │     remark-math           │  │  数学公式识别
│  ├───────────────────────────┤  │
│  │     rehype-katex          │  │  LaTeX 公式渲染
│  └───────────────────────────┘  │
│              │                  │
│              ▼                  │
│  ┌───────────────────────────┐  │
│  │   Custom Code Renderer    │  │
│  │   (CodeBlock 组件)         │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

### 4.2 核心渲染组件

```typescript
// src/components/markdown/MarkdownRenderer.tsx
const MarkdownRenderer = ({ source }: { source: string }) => {
  return (
    <Markdown
      remarkPlugins={[remarkGfm, remarkMath]}
      rehypePlugins={[[rehypeKatex, { output: "html" }]]}
      components={{ code: CodeBlock }}
    >
      {source}
    </Markdown>
  );
};
```

### 4.3 代码块与图表渲染

`CodeBlock` 组件根据语言标记决定渲染方式：

```typescript
// src/components/markdown/MarkdownRenderer.tsx
const CodeBlock = ({ className, children, node }) => {
  const lang = /language-([\w-]+)/.exec(className || "")?.[1];
  const content = String(children).replace(/\n$/, "");
  
  // 检测代码块是否完整（用于流式渲染）
  const isBlockComplete = checkFenceComplete(node, source);
  
  const isPlot = lang.startsWith("plot-");
  const isSvg = (lang === "svg" || lang === "xml") && content.startsWith("<svg");
  const isJessecode = lang === "jessecode" || lang === "jsxgraph";
  
  // 未完成的块显示"生成中"动画
  if ((isPlot || isSvg || isJessecode) && !isBlockComplete) {
    return <TextShimmer>正在生成图表...</TextShimmer>;
  }
  
  // 图表渲染
  if (isPlot) {
    switch (lang) {
      case "plot-function":
        return <MathPlotDiagram code={content} />;
      case "plot-mermaid":
        return <MermaidDiagram code={content} />;
    }
  }
  
  if (isSvg) {
    const cleanSvg = DOMPurify.sanitize(content);
    return <div dangerouslySetInnerHTML={{ __html: cleanSvg }} />;
  }
  
  if (isJessecode) {
    return <JSXGraphDiagram jesseScript={content} />;
  }
  
  // 普通代码高亮
  return <CodeRenderer language={lang} content={content} />;
};
```

### 4.4 绘图功能详解

#### 4.4.1 JSXGraph (交互式数学图形)

**用途**: 函数图像、几何图形、动态演示

```typescript
// src/components/markdown/diagram/JSXGraphDiagram.tsx
export default function JSXGraphDiagram({ jesseScript }: Props) {
  const boardId = `jxgbox-${useId().replace(/:/g, "")}`;
  
  const initBoard = () => {
    const board = JXG.JSXGraph.initBoard(boardId, {
      axis: true,
      pan: { enabled: true, needShift: false },
      zoom: { wheel: true, needShift: false },
    });
    
    // 执行 JessieCode 脚本
    board.jc.parse(jesseScript);
  };
  
  // 支持键盘导航 (方向键/hjkl)
  const handleKeyDown = (e: KeyboardEvent) => {
    // 平移视图
  };
  
  return (
    <div
      id={boardId}
      ref={boardRef}
      className="w-full aspect-3/2 rounded-lg bg-white"
      tabIndex={0}  // 可聚焦以接收键盘事件
    />
  );
}
```

**LLM 输出示例**:
```jsxgraph
amp = slider([0, 8], [5, 8], [0, 2, 5]);
f(x) = amp * sin(x);
graph = plot(f);
```

#### 4.4.2 Mermaid (流程图/图表)

**用途**: 流程图、时序图、状态图、思维导图

```typescript
// src/components/markdown/diagram/MermaidDiagram.tsx
export default function MermaidDiagram({ code }: Props) {
  const ref = useRef<HTMLDivElement>(null);
  
  useEffect(() => {
    if (ref.current && code) {
      mermaid.initialize({ startOnLoad: false, theme: "default" });
      mermaid.run({ nodes: [ref.current] });
    }
  }, [code]);
  
  return (
    <TransformWrapper>  {/* 支持缩放和平移 */}
      <TransformComponent>
        <div ref={ref} className="mermaid">{code}</div>
      </TransformComponent>
    </TransformWrapper>
  );
}
```

**LLM 输出示例**:
```plot-mermaid
graph TD;
    A[开始] --> B{条件判断};
    B -->|是| C[执行A];
    B -->|否| D[执行B];
```

#### 4.4.3 function-plot (数学函数)

**用途**: 简单的数学函数绘制（正在逐步被 JSXGraph 替代）

```typescript
// src/components/markdown/diagram/MathPlotDiagram.tsx
import functionPlot from "function-plot";

// 解析 JSON 配置并绘制函数图像
```

#### 4.4.4 SVG 直接渲染

**用途**: 自定义矢量图形

```typescript
// 使用 DOMPurify 消毒后直接注入
const cleanSvg = DOMPurify.sanitize(content);
return <div dangerouslySetInnerHTML={{ __html: cleanSvg }} />;
```

### 4.5 图表渲染包装器

所有图表都通过 `DiagramRenderer` 包装，提供代码/图表切换功能：

```typescript
// src/components/markdown/diagram/DiagramRenderer.tsx
export default function DiagramRenderer({ content, language, children }) {
  const [isCodeView, setIsCodeView] = useState(false);
  
  return (
    <div className="mermaid-container">
      <Button onClick={() => setIsCodeView(!isCodeView)}>
        {isCodeView ? "查看图表" : "查看代码"}
      </Button>
      {isCodeView ? (
        <CodeRenderer language={language} content={content} />
      ) : (
        children  // 图表组件
      )}
    </div>
  );
}
```

---

## 5. 持久化策略

### 5.1 持久化总览

| 数据类型 | 存储位置 | 持久性 | 技术实现 |
|----------|----------|--------|----------|
| AI 配置 (API Key, 模型) | localStorage | ✅ 持久 | Zustand persist |
| 用户设置 (主题, 快捷键) | localStorage | ✅ 持久 | Zustand persist |
| 聊天记录 | IndexedDB | ✅ 持久 | Dexie |
| 上传的图片 | 内存 (Object URL) | ❌ 临时 | URL.createObjectURL |
| 解题结果 | 内存 (Zustand) | ❌ 临时 | Zustand (无 persist) |

### 5.2 localStorage 持久化

#### AI 配置存储

```typescript
// src/store/ai-store.ts
export const useAiStore = create<AiStore>()(
  persist(
    (set, get) => ({
      sources: createDefaultSources(),
      activeSourceId: "gemini-default",
      // ... actions
    }),
    {
      name: "ai-storage",  // localStorage key
      storage: createJSONStorage(() => localStorage),
      partialize: (state) => ({
        sources: state.sources,
        activeSourceId: state.activeSourceId,
      }),
    },
  ),
);
```

#### 用户设置存储

```typescript
// src/store/settings-store.ts
export const useSettingsStore = create<SettingsState>()(
  persist(
    (set) => ({
      imageEnhancement: false,
      theme: "system",
      language: "en",
      keybindings: DEFAULT_SHORTCUTS,
      traits: "",  // 全局提示词
      // ...
    }),
    {
      name: "skidhw-storage",
      version: 5,  // 支持 migration
    },
  ),
);
```

### 5.3 IndexedDB 持久化 (聊天记录)

使用 Dexie 作为 IndexedDB 的封装：

```typescript
// src/store/chat-db.ts
class ChatDatabase extends Dexie {
  threads!: Table<ChatThreadRecord, string>;
  messages!: Table<ChatMessageRecord, string>;

  constructor() {
    super("skid-homework-chat-db");
    
    this.version(1).stores({
      threads: "id, updatedAt, createdAt",
      messages: "id, chatId, createdAt, [chatId+createdAt]",
    });
  }
}

export const chatDb = new ChatDatabase();
```

### 5.4 ⚠️ 不持久化的数据

**上传的图片和解题结果不会持久化！**

```typescript
// src/store/problems-store.ts
export const useProblemsStore = create<ProblemsState>((set) => ({
  imageItems: [],           // 仅内存
  imageSolutions: new Map(), // 仅内存
  // 没有使用 persist 中间件！
}));
```

**原因**:
1. Object URL 在页面刷新后失效
2. 避免存储大量图片数据消耗空间
3. 隐私考量 - 不留痕

**刷新页面后**:
- 上传的图片丢失
- 解题结果丢失
- 但聊天记录保留

### 5.5 关于图片分割

**项目不执行图片分割**。

题目分割是由 LLM 在云端完成的：
1. 用户上传完整图片
2. LLM 识别图片中的多个题目
3. 返回结构化的多题目响应（用 `---PROBLEM_SEPARATOR---` 分隔）

图片本身不会被裁剪或分割存储。

---

## 6. 官方部署实例分析

### 6.1 官方实例

官方实例部署在: **https://skid.996every.day**

### 6.2 部署架构

根据 `wrangler.toml` 配置，官方实例使用 **Cloudflare Workers** 部署：

```toml
# wrangler.toml
name = "skid-homework"
compatibility_date = "2025-12-13"
compatibility_flags = ["nodejs_compat"]
send_metrics = false

[assets]
directory = ".vercel/output/static"
binding = "ASSETS"
```

### 6.3 架构特点

```
                    ┌─────────────────────────────────┐
                    │        Cloudflare CDN           │
                    │   (全球边缘节点缓存静态资源)     │
                    └────────────┬────────────────────┘
                                 │
                    ┌────────────▼────────────────────┐
                    │      Cloudflare Workers         │
                    │   (Serverless Edge Runtime)     │
                    │      - SSR / 路由处理           │
                    │      - 静态资源服务              │
                    └────────────┬────────────────────┘
                                 │
          ┌──────────────────────┼──────────────────────┐
          │                      │                      │
          ▼                      ▼                      ▼
    ┌──────────┐          ┌──────────┐          ┌──────────┐
    │  用户 A   │          │  用户 B   │          │  用户 C   │
    │ 浏览器    │          │ 浏览器    │          │ 浏览器    │
    │  - 本地存储│          │  - 本地存储│          │  - 本地存储│
    │  - API Key│          │  - API Key│          │  - API Key│
    └─────┬────┘          └─────┬────┘          └─────┬────┘
          │                      │                      │
          ▼                      ▼                      ▼
    ┌──────────┐          ┌──────────┐          ┌──────────┐
    │ Gemini   │          │ Gemini   │          │ OpenAI   │
    │   API    │          │   API    │          │   API    │
    └──────────┘          └──────────┘          └──────────┘
```

### 6.4 官方实例的持久化

官方实例 **没有服务端持久化**：

- ❌ 没有用户数据库
- ❌ 没有会话存储
- ❌ API Key 存储在用户浏览器 localStorage 中
- ❌ 聊天记录存储在用户浏览器 IndexedDB 中

**换设备/清除浏览器数据 = 数据丢失**

### 6.5 并发处理策略

**关键点: 官方实例本身不处理 AI 请求**

```
用户浏览器 ──────► Gemini/OpenAI API (直接调用)
     │
     │ (只请求静态资源)
     ▼
Cloudflare Workers
```

#### 为什么能应付大量并发？

1. **无状态架构**: 
   - 服务器只提供静态资源
   - 所有 AI 调用由客户端直接发起到 Google/OpenAI

2. **Cloudflare Edge 缓存**:
   - 静态资源 (JS/CSS/图片) 在全球 CDN 缓存
   - 边缘节点响应，减少源站压力

3. **Serverless 自动伸缩**:
   - Cloudflare Workers 按需扩展
   - 无需管理服务器容量

4. **AI 负载分散**:
   - 每个用户使用自己的 API Key
   - 请求分散到 Google/OpenAI 的基础设施
   - 不存在单点瓶颈

5. **轻量级请求**:
   - 首次加载后，只有 API 调用
   - 页面资源已被浏览器缓存

### 6.6 部署选项对比

| 部署方式 | 配置难度 | 成本 | 特点 |
|----------|----------|------|------|
| **Cloudflare Workers** | 中 | 免费层可用 | 全球边缘部署, 低延迟 |
| **Vercel** | 低 | 免费层可用 | 一键部署, Next.js 原生支持 |
| **Docker** | 高 | 自行承担 | 完全控制, 可私有化部署 |

```bash
# Docker 部署
docker run -p 3000:3000 ghcr.io/cubewhy/skid-homework:sha-<commit_hash>

# Vercel 一键部署
# 点击 README 中的 Deploy 按钮
```

---

## 总结

| 问题 | 答案 |
|------|------|
| **架构类型** | 纯前端 SPA + Serverless 部署 |
| **默认 AI 模型** | Gemini 2.5 Flash (多模态) |
| **图片处理** | 可选 OpenCV 增强，不分割 |
| **响应渲染** | react-markdown + KaTeX + 自定义图表组件 |
| **绘图支持** | JSXGraph, Mermaid, function-plot, SVG |
| **本地持久化** | 设置/API Key (localStorage) + 聊天 (IndexedDB) |
| **图片持久化** | ❌ 不持久化 (内存 Object URL) |
| **服务端持久化** | ❌ 无 |
| **并发策略** | CDN + Serverless + 用户自带 API Key |

---

*文档生成时间: 2026-01-19*
