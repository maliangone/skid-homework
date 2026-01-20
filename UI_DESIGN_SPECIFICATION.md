# AI 作业助手 - Flutter Mobile App UI 设计规范

> **用途**: 提供给 Google Stitch / Figma AI 生成 UI 组件  
> **平台**: Flutter (iOS & Android)  
> **设计风格**: 现代简约、教育友好、深色/浅色主题支持

---

## 1. 设计系统 (Design System)

### 1.1 颜色方案

```
Light Theme:
- Primary: #6366F1 (Indigo 500) - 主要操作按钮、强调色
- Secondary: #10B981 (Emerald 500) - 成功状态、辅助操作
- Accent: #F59E0B (Amber 500) - 提示、解释区域高亮
- Background: #FFFFFF - 页面背景
- Surface: #F8FAFC - 卡片、输入框背景
- Border: #E2E8F0 - 分割线、边框
- Text Primary: #1E293B - 主要文字
- Text Secondary: #64748B - 次要文字、提示
- Text Hint: #94A3B8 - 占位符文字
- Error: #EF4444 - 错误状态
- Success: #22C55E - 成功状态

Dark Theme:
- Primary: #818CF8 (Indigo 400)
- Secondary: #34D399 (Emerald 400)
- Accent: #FBBF24 (Amber 400)
- Background: #0F172A - 页面背景
- Surface: #1E293B - 卡片背景
- Border: #334155 - 分割线
- Text Primary: #F1F5F9
- Text Secondary: #94A3B8
```

### 1.2 字体规范

```
Font Family: Inter (Google Fonts) / SF Pro (iOS fallback)

Text Styles:
- Display Large: 32px, Bold, Line Height 1.2
- Display Medium: 28px, Bold, Line Height 1.2
- Headline Large: 24px, SemiBold, Line Height 1.3
- Headline Medium: 20px, SemiBold, Line Height 1.3
- Title Large: 18px, SemiBold, Line Height 1.4
- Title Medium: 16px, Medium, Line Height 1.4
- Body Large: 16px, Regular, Line Height 1.5
- Body Medium: 14px, Regular, Line Height 1.5
- Body Small: 12px, Regular, Line Height 1.4
- Label: 12px, Medium, Line Height 1.2
- Caption: 11px, Regular, Line Height 1.3
```

### 1.3 间距系统

```
Spacing Scale (8px base):
- xs: 4px
- sm: 8px
- md: 16px
- lg: 24px
- xl: 32px
- 2xl: 48px
- 3xl: 64px

Border Radius:
- Small: 8px (按钮、标签)
- Medium: 12px (卡片、输入框)
- Large: 16px (模态框、大卡片)
- Full: 9999px (圆形头像、药丸按钮)
```

### 1.4 阴影

```
Shadow Small: 0 1px 2px rgba(0,0,0,0.05)
Shadow Medium: 0 4px 6px rgba(0,0,0,0.07)
Shadow Large: 0 10px 15px rgba(0,0,0,0.1)
```

---

## 2. 页面结构 (Page Structure)

### 2.1 页面导航架构

```
App Navigation Structure:

├── Onboarding Flow (首次使用)
│   └── OnboardingPage (4个引导页面)
│
├── Auth Flow (认证)
│   ├── LoginPage
│   └── RegisterPage
│
├── Main Shell (底部导航栏)
│   ├── Tab 1: HomePage (首页)
│   ├── Tab 2: HistoryPage (历史记录)
│   ├── Tab 3: SubscriptionPage (订阅)
│   └── Tab 4: ProfilePage (个人中心)
│
├── Question Flow (全屏流程)
│   ├── CameraPage (拍照/相册)
│   ├── CropPage (裁剪问题区域)
│   └── QuestionPage (问答对话)
│
└── Detail Pages
    ├── HistoryDetailPage (历史详情)
    ├── SolutionDetailPage (解答详情 - 分题目展示)
    └── SettingsPage (设置)
```

---

## 3. 页面详细设计

### 3.1 Onboarding Page (引导页)

**布局**: 全屏滑动页面，底部有页面指示器和导航按钮

```
┌─────────────────────────────────┐
│  [右上角] Skip 文字按钮          │
├─────────────────────────────────┤
│                                 │
│     ┌───────────────────┐       │
│     │                   │       │
│     │   大圆形背景       │       │
│     │   中心图标/插画    │       │
│     │   (120px icon)    │       │
│     │                   │       │
│     └───────────────────┘       │
│                                 │
│     Welcome to LearnAI          │
│     (Display Medium, Bold)      │
│                                 │
│     Your AI-powered education   │
│     companion that helps you    │
│     understand any question     │
│     (Body Large, Secondary)     │
│                                 │
├─────────────────────────────────┤
│         ●──●──●──○              │
│      (Page Indicator)           │
├─────────────────────────────────┤
│  [Previous]        [Next →]     │
│  (Outline)         (Filled)     │
└─────────────────────────────────┘

引导内容 (4页):
1. Welcome - 欢迎介绍 (学校图标)
2. Capture - 拍照功能 (相机图标)  
3. Crop - 智能裁剪 (裁剪图标)
4. Answer - AI解答 (灯泡图标)
```

**组件**:
- `OnboardingPageView` - 可滑动的页面容器
- `OnboardingSlide` - 单个引导页
- `PageIndicator` - 底部圆点指示器
- `SkipButton` - 跳过按钮

---

### 3.2 Home Page (首页)

**布局**: 可滚动页面，带浮动操作按钮

```
┌─────────────────────────────────┐
│  Hello, Student! 👋             │
│  What would you like to learn?  │
│                        [🔔]     │
├─────────────────────────────────┤
│  ┌─────────────────────────┐    │
│  │ 📊 Usage This Month     │    │
│  │ ████████░░░░ 67/100     │    │
│  │ 33 questions remaining  │    │
│  └─────────────────────────┘    │
├─────────────────────────────────┤
│  Quick Actions                  │
│  ┌───────────┐ ┌───────────┐   │
│  │ 📷        │ │ ✏️        │   │
│  │ Take      │ │ Type      │   │
│  │ Photo     │ │ Question  │   │
│  │           │ │           │   │
│  └───────────┘ └───────────┘   │
├─────────────────────────────────┤
│  Recent Questions    [View All] │
│  ┌─────────────────────────┐   │
│  │ 📐 What is derivative?  │   │
│  │    Mathematics · 2h ago │   │
│  └─────────────────────────┘   │
│  ┌─────────────────────────┐   │
│  │ 🧬 Explain photosynthesis│  │
│  │    Biology · 1d ago     │   │
│  └─────────────────────────┘   │
├─────────────────────────────────┤
│  Features                       │
│  ┌─────────────────────────┐   │
│  │ 📸 Multi-Image Capture  │   │
│  │    Up to 3 images/Q     │   │
│  └─────────────────────────┘   │
│                                 │
│                         [+ FAB] │
└─────────────────────────────────┘

底部导航栏:
┌─────┬─────┬─────┬─────┐
│ 🏠  │ 📜  │ 💎  │ 👤  │
│Home │Hist │Sub  │Prof │
└─────┴─────┴─────┴─────┘
```

**组件**:
- `UsageIndicatorCard` - 用量显示卡片（进度条 + 数字）
- `QuickActionCard` - 快捷操作卡片（渐变背景 + 图标 + 标题）
- `RecentQuestionCard` - 最近问题卡片（缩略图 + 标题 + 学科标签 + 时间）
- `FeatureListTile` - 功能介绍列表项
- `FloatingActionButton` - 新建问题的 FAB

---

### 3.3 Camera Page (拍照页)

**布局**: 全屏深色背景，相机风格界面

```
┌─────────────────────────────────┐
│ [✕]  Capture Question  [Next(2)]│
│      (白色文字，黑色背景)         │
├─────────────────────────────────┤
│  ┌─────────────────────────┐    │
│  │ 📷 Capture up to 3      │    │
│  │    images per question  │    │
│  │    Make it clearly      │    │
│  └─────────────────────────┘    │
│         (提示卡片,半透明)        │
├─────────────────────────────────┤
│  Selected Images (横向滚动):    │
│  ┌────┐ ┌────┐ ┌────┐          │
│  │img1│ │img2│ │    │          │
│  │ ✕  │ │ ✕  │ │    │          │
│  │ 1  │ │ 2  │ │    │          │
│  └────┘ └────┘ └────┘          │
├─────────────────────────────────┤
│                                 │
│                                 │
│         ┌─────────┐             │
│         │         │             │
│         │  ○ 📷   │  <- 拍照按钮│
│         │  (大圆) │             │
│         └─────────┘             │
│                                 │
│    ┌──────┐      ┌──────┐      │
│    │ 🖼️   │      │ 📷   │      │
│    │Gallery│     │Camera │      │
│    └──────┘      └──────┘      │
│                                 │
├─────────────────────────────────┤
│  [ Continue with 2 images  →]   │
│          (主要按钮)              │
│     Type question instead       │
│          (文字链接)              │
└─────────────────────────────────┘
```

**组件**:
- `CameraCaptureButton` - 大圆形拍照按钮（白色边框 + 内圆）
- `ImageThumbnailCard` - 已选图片缩略图（带删除按钮 + 序号）
- `ActionIconButton` - 底部操作按钮（图标 + 标签）
- `InstructionBanner` - 顶部提示条

---

### 3.4 Crop Page (裁剪页)

**布局**: 全屏图片查看 + 可拖动裁剪框

```
┌─────────────────────────────────┐
│ [←] Crop Questions (1/3) [Submit]│
├─────────────────────────────────┤
│  ┌─────────────────────────┐    │
│  │ ✂️ Drag corners to      │    │
│  │    select question area │    │
│  └─────────────────────────┘    │
├─────────────────────────────────┤
│  ┌─────────────────────────┐    │
│  │                         │    │
│  │     ┌─ ─ ─ ─ ─ ─┐      │    │
│  │     │ ●        ● │      │    │
│  │     │            │      │    │
│  │     │  裁剪区域  │      │    │
│  │     │  (蓝色边框) │      │    │
│  │     │            │      │    │
│  │     │ ●        ● │      │    │
│  │     └─ ─ ─ ─ ─ ─┘      │    │
│  │                         │    │
│  │   (图片 + 半透明遮罩)    │    │
│  │                         │    │
│  └─────────────────────────┘    │
├─────────────────────────────────┤
│         ●──●──○                 │
│      (Page Indicator)           │
├─────────────────────────────────┤
│  [Previous]    [Next Image →]   │
│                                 │
│       Reset crop area           │
└─────────────────────────────────┘
```

**组件**:
- `ImageCropWidget` - 图片裁剪组件
  - 可缩放/平移的图片
  - 四个角可拖动的裁剪框
  - 裁剪区域外的半透明遮罩
- `CropHandleCorner` - 裁剪框角落拖动点（圆形，Primary 色）
- `CropOverlay` - 半透明遮罩层

---

### 3.5 Question Page (问答对话页) ⭐ 核心页面

**布局**: 类 ChatGPT 对话界面 + 底部输入框

```
┌─────────────────────────────────┐
│ [←] Ask Question      [📷] [📜]│
├─────────────────────────────────┤
│                                 │
│  ┌──┐ AI Tutor · just now      │
│  │🤖│ ┌─────────────────────┐  │
│  └──┘ │ Hi! I'm your AI     │  │
│       │ tutor. What would   │  │
│       │ you like to learn?  │  │
│       └─────────────────────┘  │
│                                 │
│       ┌──┐ You · 2m ago        │
│       │👤│ ┌─────────────────┐ │
│       └──┘ │ [图片预览]       │ │
│            │ ┌────┐ ┌────┐   │ │
│            │ │img1│ │img2│   │ │
│            │ └────┘ └────┘   │ │
│            │ How to solve    │ │
│            │ this equation?  │ │
│            └─────────────────┘ │
│                                 │
│  ┌──┐ AI Tutor · 1m ago        │
│  │🤖│ ┌─────────────────────┐  │
│  └──┘ │ ## Answer           │  │
│       │ x = 5               │  │
│       │                     │  │
│       │ ┌─ Explanation ───┐ │  │
│       │ │ 💡 Step 1: ...  │ │  │
│       │ │ Step 2: ...     │ │  │
│       │ └─────────────────┘ │  │
│       │                     │  │
│       │ [📋Copy][👍][👎]    │  │
│       └─────────────────────┘  │
│                                 │
├─────────────────────────────────┤
│ ┌─────────────────────────────┐│
│ │📎│ Type your question...  │↑││
│ └─────────────────────────────┘│
└─────────────────────────────────┘
```

**组件**:
- `ChatMessageBubble` - 消息气泡
  - 用户消息: Primary 色背景，右对齐头像
  - AI消息: Surface 背景 + 边框，左对齐头像
- `MessageAvatar` - 圆形头像（用户/AI 图标）
- `ImageGalleryPreview` - 消息内的图片预览（横向滚动）
- `ExplanationCard` - 解释区域卡片（Accent 浅色背景 + 边框）
- `MessageActionBar` - 消息操作按钮（复制、点赞、点踩）
- `ChatInputBar` - 底部输入栏
  - 附件按钮
  - 多行文本输入框
  - 发送按钮
- `TypingIndicator` - AI 正在输入的加载动画

---

### 3.6 Solution Detail Page (解答详情页) ⭐ 来自 Skid-Homework

**布局**: 分题目展示，支持流式输出

```
┌─────────────────────────────────┐
│ [←] Solutions          [Export] │
├─────────────────────────────────┤
│  ┌─────────────────────────┐    │
│  │ [img1] [img2] [img3]    │ <- 图片Tab切换
│  └─────────────────────────┘    │
│  [← Prev Image] [Next Image →]  │
├─────────────────────────────────┤
│  Photo 1 · Camera               │
│  ┌─────────────────────────┐    │
│  │ [点击展开/收起图片预览]  │    │
│  │  ▼ Toggle Preview        │    │
│  └─────────────────────────┘    │
├─────────────────────────────────┤
│  ⏳ Streaming Output            │
│  ┌─────────────────────────┐    │
│  │ ### PROBLEM_TEXT        │    │
│  │ 求 f(x)=x² 在 x=2...    │    │
│  │ ▊ (闪烁光标)            │    │
│  └─────────────────────────┘    │
├─────────────────────────────────┤
│  🔢 Problems Found: 3           │
│                                 │
│  Problem List (左侧):           │
│  ┌────────────────┐             │
│  │ ● Q1: 求导数   │ ← 选中态    │
│  │ ○ Q2: 求极值   │             │
│  │ ○ Q3: 画图     │             │
│  └────────────────┘             │
│                                 │
│  Solution Viewer (右侧):        │
│  ┌─────────────────────────┐    │
│  │ ## Problem              │    │
│  │ 求 f(x)=x² 的导数       │    │
│  │                         │    │
│  │ ## Answer               │    │
│  │ f'(x) = 2x              │    │
│  │                         │    │
│  │ ## Explanation          │    │
│  │ ┌─ Step 1 ─────────┐   │    │
│  │ │ 应用幂函数求导法则 │   │    │
│  │ │ d/dx(x^n) = nx^...│   │    │
│  │ └──────────────────┘   │    │
│  │                         │    │
│  │ ┌─ Step 2 ─────────┐   │    │
│  │ │ 代入 n=2 ...     │   │    │
│  │ └──────────────────┘   │    │
│  │                         │    │
│  │ ┌─ 图表 ──────────┐    │    │
│  │ │  [JSXGraph图表]  │    │    │
│  │ │  (可交互缩放)    │    │    │
│  │ │  [View Code]     │    │    │
│  │ └──────────────────┘   │    │
│  └─────────────────────────┘    │
├─────────────────────────────────┤
│  [Space: Next Q] [/: Improve]   │
│           (快捷键提示)           │
├─────────────────────────────────┤
│  [◀ Prev Problem] [Next ▶]      │
└─────────────────────────────────┘
```

**组件**:
- `ImageTabBar` - 图片切换标签栏
- `CollapsibleImagePreview` - 可折叠图片预览区
- `StreamingOutputCard` - 流式输出显示卡片（带闪烁光标）
- `ProblemListSidebar` - 问题列表侧边栏
- `SolutionViewerCard` - 解答查看卡片
- `StepAccordion` - 步骤折叠面板
- `DiagramContainer` - 图表容器
  - 支持 JSXGraph (交互式数学图表)
  - 支持 Mermaid (流程图)
  - 支持 SVG (矢量图)
  - 代码/图表视图切换按钮
- `ShortcutHintBar` - 快捷键提示栏
- `ProblemNavigationBar` - 问题导航按钮

---

### 3.7 History Page (历史记录页)

**布局**: 列表页，按日期分组

```
┌─────────────────────────────────┐
│ History                    [🔍] │
├─────────────────────────────────┤
│  🔍 Search questions...         │
├─────────────────────────────────┤
│  Today                          │
│  ┌─────────────────────────┐    │
│  │ 📐 Derivative of x²     │    │
│  │    Mathematics · 2h ago │🗑️ │
│  └─────────────────────────┘    │
│                                 │
│  Yesterday                      │
│  ┌─────────────────────────┐    │
│  │ 🧬 Photosynthesis       │    │
│  │    Biology · 1d ago     │🗑️ │
│  └─────────────────────────┘    │
│  ┌─────────────────────────┐    │
│  │ 📊 Statistics problem   │    │
│  │    Math · 1d ago        │🗑️ │
│  └─────────────────────────┘    │
│                                 │
│  Last Week                      │
│  ...                            │
├─────────────────────────────────┤
│  Empty State (无记录时):        │
│  ┌─────────────────────────┐    │
│  │       📜                │    │
│  │  No history yet         │    │
│  │  Start by asking your   │    │
│  │  first question!        │    │
│  └─────────────────────────┘    │
└─────────────────────────────────┘
```

**组件**:
- `SearchBar` - 搜索栏
- `DateSectionHeader` - 日期分组标题
- `HistoryQuestionCard` - 历史记录卡片
  - 学科图标
  - 问题标题（截断显示）
  - 时间戳
  - 删除按钮（滑动或点击）
- `EmptyStateWidget` - 空状态展示

---

### 3.8 Subscription Page (订阅页)

**布局**: 订阅计划展示 + 用量统计

```
┌─────────────────────────────────┐
│ Subscription                    │
├─────────────────────────────────┤
│  Current Plan                   │
│  ┌─────────────────────────┐    │
│  │ 💎 Standard Plan        │    │
│  │    $19.99/month         │    │
│  │                         │    │
│  │    ✅ Active            │    │
│  │    Expires: Jan 31      │    │
│  └─────────────────────────┘    │
├─────────────────────────────────┤
│  Usage This Month               │
│  ┌─────────────────────────┐    │
│  │ Questions Used          │    │
│  │ ████████████░░░ 67/100  │    │
│  │                         │    │
│  │ 🔔 Alert at 80%         │    │
│  └─────────────────────────┘    │
├─────────────────────────────────┤
│  Available Plans                │
│                                 │
│  ┌─────────────────────────┐    │
│  │ Free                    │    │
│  │ $0/month · 10 Q/month   │    │
│  │         [Current]       │    │
│  └─────────────────────────┘    │
│                                 │
│  ┌─────────────────────────┐    │
│  │ ⭐ Standard (推荐)       │    │
│  │ $19.99/mo · 100 Q/month │    │
│  │         [Upgrade]       │    │
│  └─────────────────────────┘    │
│                                 │
│  ┌─────────────────────────┐    │
│  │ 💎 Pro                  │    │
│  │ $39.99/mo · 500 Q/month │    │
│  │         [Upgrade]       │    │
│  └─────────────────────────┘    │
├─────────────────────────────────┤
│  [Restore Purchases]            │
└─────────────────────────────────┘
```

**组件**:
- `CurrentPlanCard` - 当前计划卡片
- `UsageProgressCard` - 用量进度卡片（带进度条）
- `PlanOptionCard` - 计划选项卡片
  - 推荐标签（带星星）
  - 价格和额度
  - 操作按钮（Current/Upgrade）
- `RestorePurchasesButton` - 恢复购买按钮

---

### 3.9 Profile Page (个人中心页)

**布局**: 用户信息 + 设置列表

```
┌─────────────────────────────────┐
│ Profile                         │
├─────────────────────────────────┤
│      ┌─────────┐                │
│      │  👤     │                │
│      │ Avatar  │                │
│      └─────────┘                │
│      John Doe                   │
│      john@example.com           │
│      [Edit Profile]             │
├─────────────────────────────────┤
│  Settings                       │
│  ┌─────────────────────────┐    │
│  │ 🎨 Appearance           │    │
│  │    Theme: System     >  │    │
│  └─────────────────────────┘    │
│  ┌─────────────────────────┐    │
│  │ 🌐 Language             │    │
│  │    English           >  │    │
│  └─────────────────────────┘    │
│  ┌─────────────────────────┐    │
│  │ 🔔 Notifications        │    │
│  │                    [ON] │    │
│  └─────────────────────────┘    │
│  ┌─────────────────────────┐    │
│  │ 🤖 AI Provider          │    │
│  │    Configure API     >  │    │
│  └─────────────────────────┘    │
├─────────────────────────────────┤
│  Support                        │
│  ┌─────────────────────────┐    │
│  │ ❓ Help & FAQ           │    │
│  └─────────────────────────┘    │
│  ┌─────────────────────────┐    │
│  │ 📧 Contact Us           │    │
│  └─────────────────────────┘    │
│  ┌─────────────────────────┐    │
│  │ 📜 Privacy Policy       │    │
│  └─────────────────────────┘    │
├─────────────────────────────────┤
│  [Sign Out]                     │
│                                 │
│  App Version 1.0.0              │
└─────────────────────────────────┘
```

**组件**:
- `ProfileHeader` - 用户头像和信息区
- `SettingsSection` - 设置分组区域
- `SettingsListTile` - 设置列表项
  - 图标 + 标题 + 当前值/开关
  - 右侧箭头或 Toggle
- `SignOutButton` - 登出按钮
- `AppVersionLabel` - 版本号标签

---

### 3.10 Settings Detail: AI Provider (AI 配置页)

**布局**: AI API 配置管理

```
┌─────────────────────────────────┐
│ [←] AI Provider                 │
├─────────────────────────────────┤
│  Active Provider                │
│  ┌─────────────────────────┐    │
│  │ ● Gemini               ✓│    │
│  │ ○ OpenAI Compatible     │    │
│  │ ○ Custom API            │    │
│  └─────────────────────────┘    │
├─────────────────────────────────┤
│  Gemini Configuration           │
│  ┌─────────────────────────┐    │
│  │ API Key                 │    │
│  │ ┌───────────────────┐   │    │
│  │ │ ••••••••••••      │   │    │
│  │ └───────────────────┘   │    │
│  │ [Get API Key ↗]         │    │
│  │                         │    │
│  │ Model                   │    │
│  │ ┌───────────────────┐   │    │
│  │ │ gemini-2.5-flash ▼│   │    │
│  │ └───────────────────┘   │    │
│  │                         │    │
│  │ Thinking Budget         │    │
│  │ ●────────────○ 8192    │    │
│  └─────────────────────────┘    │
├─────────────────────────────────┤
│  Custom Prompt (Global)         │
│  ┌─────────────────────────┐    │
│  │ Add custom instructions │    │
│  │ for all questions...    │    │
│  │                         │    │
│  │                         │    │
│  └─────────────────────────┘    │
├─────────────────────────────────┤
│  [Test Connection]              │
│  [Save Settings]                │
└─────────────────────────────────┘
```

**组件**:
- `ProviderRadioGroup` - AI 提供商单选组
- `ApiKeyInput` - API Key 输入框（带遮蔽和显示切换）
- `ModelDropdown` - 模型选择下拉框
- `SliderWithValue` - 带数值显示的滑块
- `MultilineTextInput` - 多行文本输入框（自定义 Prompt）

---

## 4. 通用组件库

### 4.1 按钮 (Buttons)

```
Primary Button:
- 背景: Primary 色
- 文字: 白色
- 圆角: 12px
- 高度: 48px (默认), 56px (大)
- 状态: Normal / Hover / Pressed / Disabled / Loading

Secondary Button (Outline):
- 边框: Primary 色
- 背景: 透明
- 文字: Primary 色

Ghost Button:
- 无边框
- 文字: Primary 或 Secondary 色

Icon Button:
- 圆形
- 大小: 40px / 48px
```

### 4.2 输入框 (Inputs)

```
Text Input:
- 背景: Surface 色
- 边框: Border 色 (聚焦时 Primary)
- 圆角: 12px
- 高度: 48px
- Label 在顶部
- 错误状态: Error 色边框 + 提示文字

Text Area:
- 同 Text Input
- 高度: 自适应 (min 100px)

Search Input:
- 带搜索图标前缀
- 圆角: Full (药丸形)
```

### 4.3 卡片 (Cards)

```
Base Card:
- 背景: Surface 色
- 边框: Border 色 (可选)
- 圆角: 16px
- 阴影: Small 或 Medium
- 内边距: 16px

Gradient Card:
- 渐变背景 (Primary 10% -> 5%)
- 用于 Quick Actions
```

### 4.4 对话框 (Dialogs)

```
Bottom Sheet:
- 从底部滑入
- 圆角: 顶部 24px
- 拖动条: 居中 40x4px 灰色条

Modal Dialog:
- 居中显示
- 背景遮罩
- 圆角: 16px
- 标题 + 内容 + 操作按钮
```

### 4.5 加载状态 (Loading)

```
Circular Progress:
- Primary 色
- 大小: 24px / 40px

Shimmer Loading:
- 骨架屏效果
- 用于内容加载占位

Typing Indicator:
- 三个跳动的圆点
- 用于 AI 回复加载中
```

### 4.6 Toast / Snackbar

```
位置: 底部居中
背景: 深色 (Dark) 或根据类型
类型:
- Success: 绿色 + ✓ 图标
- Error: 红色 + ✗ 图标
- Info: 蓝色 + ℹ 图标
持续: 3秒自动消失
```

---

## 5. 动画规范

### 5.1 页面转场

```
Push Navigation:
- 新页面从右侧滑入
- 当前页面向左淡出
- 时长: 300ms
- 曲线: easeInOut

Bottom Sheet:
- 从底部滑入
- 时长: 250ms
- 曲线: easeOut
```

### 5.2 组件动画

```
Button Press:
- Scale: 0.98
- 时长: 100ms

Card Hover/Press:
- 轻微提升阴影
- 时长: 150ms

Page Indicator:
- 宽度变化: 8px -> 32px
- 时长: 300ms
- 曲线: easeInOut

Staggered List:
- 列表项依次淡入 + 上滑
- 每项间隔: 50ms
- 单项时长: 375ms
```

---

## 6. 响应式断点

```
Mobile (默认):
- 宽度 < 600px
- 单列布局
- 底部导航栏

Tablet:
- 宽度 >= 600px
- 可选双列布局 (Solution Detail: 问题列表 | 解答)
- 底部导航栏 或 侧边栏

Desktop (Web):
- 宽度 >= 1024px
- 最大内容宽度: 1200px
- 侧边栏导航
```

---

## 7. 无障碍 (Accessibility)

```
- 所有图片有 alt text
- 按钮有 semantic label
- 颜色对比度 >= 4.5:1
- 支持屏幕阅读器
- 触摸目标 >= 44x44px
- 支持动态字体大小
```

---

## 8. 国际化 (i18n)

```
支持语言:
- English (en) - 默认
- 中文简体 (zh-CN)
- 中文繁体 (zh-TW)

文本容器需支持:
- RTL 布局 (未来)
- 文本长度变化 (预留空间)
```

---

## 生成提示 (For AI Design Tools)

**给 Google Stitch / Figma AI 的提示模板**:

```
Create a mobile app UI design for an AI-powered homework helper app with the following specifications:

Design Style:
- Modern, clean, minimalist
- Education-friendly with a professional feel
- Support both light and dark themes
- Primary color: Indigo (#6366F1)
- Font: Inter

Pages to generate:
1. Onboarding (4 slides with skip button)
2. Home page with usage stats and quick actions
3. Camera capture page (dark theme)
4. Image crop page with draggable corners
5. Chat-style Q&A page
6. Solution detail page with:
   - Multiple problems tab navigation
   - Step-by-step explanation accordion
   - Interactive diagram container
   - Streaming output display
7. History list page with date grouping
8. Subscription page with plan comparison
9. Profile/Settings page

Key Features:
- Multi-image support (up to 3 per question)
- Real-time streaming output display
- Math formula rendering (LaTeX)
- Interactive diagrams (JSXGraph, Mermaid)
- Keyboard shortcut hints
- Problem navigation (prev/next)

Component Library:
- Cards with gradients
- Chat message bubbles (user/AI)
- Progress indicators
- Collapsible sections
- Tab navigation
- Bottom navigation bar
- Floating action button
```

---

*文档版本: 1.0*  
*生成日期: 2026-01-20*  
*用于: Google Stitch / Figma AI 生成 Flutter UI 组件*
