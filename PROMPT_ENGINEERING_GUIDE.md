# Prompt 工程与结构化输出指南

> **本文档用于 Flutter 项目参考**  
> 整理自 Skid-Homework 项目的 Prompt 工程经验

---

## 目录

1. [核心设计理念](#1-核心设计理念)
2. [Prompt 模板架构](#2-prompt-模板架构)
3. [结构化输出格式设计](#3-结构化输出格式设计)
4. [图表生成 Prompt 设计](#4-图表生成-prompt-设计)
5. [响应解析策略](#5-响应解析策略)
6. [Flutter 实现建议](#6-flutter-实现建议)
7. [完整 Prompt 模板参考](#7-完整-prompt-模板参考)

---

## 1. 核心设计理念

### 1.1 "伪工具调用" 模式

**核心思想**: 不依赖 LLM 的原生 Function Calling，而是通过 Prompt 约定输出格式，让 LLM 在需要时输出特定格式的"代码块"，前端识别并渲染。

```
┌─────────────────────────────────────────────────────────────┐
│                     传统 Function Calling                    │
├─────────────────────────────────────────────────────────────┤
│  LLM → 返回 tool_calls JSON → 后端执行 → 返回结果 → LLM 继续  │
│  ❌ 需要后端支持                                              │
│  ❌ 增加请求延迟                                              │
│  ❌ 依赖特定 API 格式                                         │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                     伪工具调用 (本项目方式)                   │
├─────────────────────────────────────────────────────────────┤
│  LLM → 输出特定格式代码块 → 前端解析渲染                      │
│  ✅ 纯前端实现                                               │
│  ✅ 无额外延迟                                               │
│  ✅ 跨平台兼容 (Gemini/OpenAI/任意 LLM)                      │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 设计原则

| 原则 | 说明 |
|------|------|
| **结构化输出** | 使用 Markdown 标题作为分隔符，便于正则/解析器提取 |
| **语言标记识别** | 代码块的语言标记（如 `jsxgraph`）作为渲染类型标识 |
| **容错设计** | 即使解析失败，原始 Markdown 仍可正常显示 |
| **流式友好** | 输出格式支持流式渲染，用户可实时看到结果 |

---

## 2. Prompt 模板架构

### 2.1 Prompt 组合策略

系统 Prompt 采用**模块化组合**方式：

```
┌─────────────────────────────────────────┐
│           最终发送给 LLM 的 Prompt        │
├─────────────────────────────────────────┤
│  ① 主 System Prompt (角色 + 任务定义)    │
│  ② 用户自定义 Traits (可选)              │
│  ③ 工具 Prompts (图表生成能力)           │
│  ④ 用户输入 (图片/文本)                  │
└─────────────────────────────────────────┘
```

### 2.2 代码实现示例

```dart
// Flutter 实现示例
class PromptBuilder {
  final List<String> systemPrompts = [];
  
  void addSystemPrompt(String prompt) {
    systemPrompts.add(prompt);
  }
  
  void setAvailableTools(List<String> toolPrompts) {
    final toolsPrompt = toolPrompts.join('\n\n');
    addSystemPrompt('## Available Tools\n$toolsPrompt');
  }
  
  String buildFinalPrompt() {
    return systemPrompts.join('\n\n');
  }
}

// 使用
final builder = PromptBuilder();
builder.addSystemPrompt(solvePrompt);           // 主提示词
builder.addSystemPrompt(userTraits);            // 用户自定义
builder.setAvailableTools([                      // 工具能力
  jsxGraphToolPrompt,
  mermaidToolPrompt,
  mathPlotToolPrompt,
]);
```

---

## 3. 结构化输出格式设计

### 3.1 核心格式：Markdown KV

使用 `###` 三级标题作为 Key，标题下内容作为 Value：

```markdown
### KEY_NAME_1

这里是 KEY_NAME_1 的内容...
可以包含多行、列表、代码块等

### KEY_NAME_2

这里是 KEY_NAME_2 的内容...
```

**优点**:
- 人类可读，即使解析失败也能直接展示
- 支持流式渲染
- 使用 Markdown 解析器即可提取

### 3.2 解题输出格式

```markdown
### PROBLEM_TEXT

这里是 OCR 识别出的完整问题文本。

### EXPLANATION

#### Step 1: 识别关键信息

这里解释如何理解题目...

#### Step 2: [步骤名称]

这里是具体的计算或推导过程...
可以使用 LaTeX: $$ x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$

#### Step 3: [步骤名称]

...

### ANSWER

这里是问题的最终答案。
```

### 3.3 多问题分隔

当图片包含多个题目时，使用分隔符：

```markdown
### PROBLEM_TEXT
第一道题...

### EXPLANATION
...

### ANSWER
...

---PROBLEM_SEPARATOR---

### PROBLEM_TEXT
第二道题...

### EXPLANATION
...

### ANSWER
...
```

### 3.4 改进答案输出格式

```markdown
### IMPROVED_EXPLANATION

#### Step 1: [步骤标题]
[改进后的详细步骤...]

#### Step 2: [步骤标题]
[改进后的详细步骤...]

### IMPROVED_ANSWER

改进后的最终答案...
```

---

## 4. 图表生成 Prompt 设计

### 4.1 设计模式

每个图表工具的 Prompt 包含：

```
┌────────────────────────────────────┐
│  1. 工具名称与触发语法              │
│  2. 输出格式规则 (严格约束)         │
│  3. 语法约束 (避免常见错误)         │
│  4. 完整示例                       │
│  5. 参数/语法参考                  │
└────────────────────────────────────┘
```

### 4.2 JSXGraph 工具 Prompt

```markdown
# JSXGraph tool (`jsxgraph`)

**Output Format Rules (STRICT):**

1. **Markdown Wrapper:** You must wrap the code in a markdown block with the language identifier `jsxgraph`.
   Example:
   ```jsxgraph
   [code here]
   ```
2. **No Comments:** Do not include any comments (`//` or `/* */`). The output is for machine parsing only.
3. **No Explanations:** Do not provide any introductory text or closing remarks. Output only the code block.

**Syntax Constraints (STRICT):**

1. **Positional Arguments Only:** Do NOT use JavaScript object literals `{}` for attributes. The parser will fail.
   - WRONG: `p = point([1,1], {name: 'A'});`
   - RIGHT: `p = point(1, 1);`
2. **Standard Functions:** Use only JesseCode-compatible assignments and functions: `point()`, `line()`, `slider()`, `plot()`, `circle()`, `arrow()`.
3. **Reactivity:** Use the `f(x) = ...` syntax for dynamic functions.
4. **No JS Keywords:** Do not use `const`, `let`, `var`, or `function`.
5. **No Special Characters in Strings:** Keep labels/names simple. Avoid parentheses inside strings.

**Example Input:** "A sine wave with a slider for amplitude."
**Example Output:**

```jsxgraph
amp = slider([0, 8], [5, 8], [0, 2, 5]);
f(x) = amp * sin(x);
graph = plot(f);
```

**Wait for my visualization request.**
```

### 4.3 Mermaid 工具 Prompt

```markdown
# Mermaid Tool (`plot-mermaid`)

Mermaid is a JavaScript based diagramming and charting tool that uses Markdown-inspired text definitions.

## 1. Trigger Syntax

To render a graph, output a code block with the language tag `plot-mermaid` containing a valid Mermaid object.

### Example

```plot-mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```
```

### 4.4 数学函数绘图 Prompt

```markdown
## Math Graph Tool (`plot-function`)

Use this tool to render 2D mathematical graphs using JSON configuration.

### Trigger Syntax

Output a code block with the language tag `plot-function` containing a valid JSON object.

### JSON Configuration Schema

```plot-function
{
  "title": "Optional Chart Title",
  "xAxis": {
    "domain": [-10, 10],
    "label": "x-axis label"
  },
  "yAxis": {
    "domain": [-10, 10],
    "label": "y-axis label"
  },
  "grid": true,
  "data": [
    {
      "fn": "x^2",
      "range": [-5, 5],
      "color": "red",
      "label": "f(x) = x^2"
    },
    {
      "fn": "x^2 + y^2 - 9",
      "fnType": "implicit",
      "label": "Circle"
    },
    {
      "points": [[1, 1], [2, 4], [3, 9]],
      "fnType": "points",
      "graphType": "scatter"
    }
  ]
}
```

### Critical Rules

1. **NO Arithmetic in JSON**: Pre-calculate all numbers.
   - ❌ `"domain": [-2*PI, 2*PI]`
   - ✅ `"domain": [-6.28, 6.28]`
2. **Valid JSON**: No trailing commas, no comments, use double quotes.
```

### 4.5 工具 Prompt 设计要点

| 要点 | 说明 |
|------|------|
| **明确触发语法** | 指定代码块的语言标记（如 `jsxgraph`, `plot-mermaid`） |
| **严格约束格式** | 使用 "STRICT" 强调，列出禁止事项 |
| **提供错误示例** | 用 ❌/✅ 对比正确和错误写法 |
| **完整示例** | 给出可直接运行的示例 |
| **列出常见陷阱** | 如 "不要在 JSON 中使用算术表达式" |

---

## 5. 响应解析策略

### 5.1 Markdown 分段解析

使用 Markdown 解析库提取结构：

```dart
// Flutter/Dart 实现思路
class MarkdownSectionParser {
  final String markdown;
  
  MarkdownSectionParser(this.markdown);
  
  /// 按 ### 标题分组提取内容
  Map<String, String> getSectionsByH3() {
    final sections = <String, String>{};
    String? currentKey;
    final buffer = StringBuffer();
    
    for (final line in markdown.split('\n')) {
      if (line.startsWith('### ')) {
        // 保存上一个 section
        if (currentKey != null) {
          sections[currentKey] = buffer.toString().trim();
          buffer.clear();
        }
        currentKey = line.substring(4).trim();
      } else if (currentKey != null) {
        buffer.writeln(line);
      }
    }
    
    // 保存最后一个 section
    if (currentKey != null) {
      sections[currentKey] = buffer.toString().trim();
    }
    
    return sections;
  }
  
  /// 从 EXPLANATION 中解析步骤
  static List<ExplanationStep> parseSteps(String explanationText) {
    final steps = <ExplanationStep>[];
    String? currentTitle;
    final buffer = StringBuffer();
    
    for (final line in explanationText.split('\n')) {
      if (line.startsWith('#### ')) {
        if (currentTitle != null) {
          steps.add(ExplanationStep(
            title: currentTitle,
            content: buffer.toString().trim(),
          ));
          buffer.clear();
        }
        currentTitle = line.substring(5).trim();
      } else if (currentTitle != null) {
        buffer.writeln(line);
      }
    }
    
    if (currentTitle != null) {
      steps.add(ExplanationStep(
        title: currentTitle,
        content: buffer.toString().trim(),
      ));
    }
    
    return steps;
  }
}

class ExplanationStep {
  final String title;
  final String content;
  ExplanationStep({required this.title, required this.content});
}
```

### 5.2 代码块提取

提取特定语言的代码块用于图表渲染：

```dart
/// 提取所有代码块
class CodeBlockExtractor {
  /// 正则匹配代码块
  static final _codeBlockRegex = RegExp(
    r'```(\w+)?\n([\s\S]*?)```',
    multiLine: true,
  );
  
  /// 提取所有代码块
  static List<CodeBlock> extractAll(String markdown) {
    final blocks = <CodeBlock>[];
    
    for (final match in _codeBlockRegex.allMatches(markdown)) {
      final language = match.group(1) ?? '';
      final content = match.group(2) ?? '';
      blocks.add(CodeBlock(language: language, content: content.trim()));
    }
    
    return blocks;
  }
  
  /// 提取特定语言的代码块
  static List<String> extractByLanguage(String markdown, String language) {
    return extractAll(markdown)
        .where((b) => b.language == language)
        .map((b) => b.content)
        .toList();
  }
}

class CodeBlock {
  final String language;
  final String content;
  CodeBlock({required this.language, required this.content});
  
  bool get isJsxGraph => language == 'jsxgraph' || language == 'jessecode';
  bool get isMermaid => language == 'plot-mermaid';
  bool get isMathPlot => language == 'plot-function';
  bool get isSvg => language == 'svg' && content.trim().startsWith('<svg');
}
```

### 5.3 解题响应完整解析

```dart
class SolveResponse {
  final List<ProblemSolution> problems;
  SolveResponse({required this.problems});
}

class ProblemSolution {
  final String problem;
  final String answer;
  final String explanation;
  final List<ExplanationStep> steps;
  
  ProblemSolution({
    required this.problem,
    required this.answer,
    required this.explanation,
    required this.steps,
  });
}

SolveResponse parseSolveResponse(String response) {
  // 1. 按分隔符分割多个问题
  final rawChunks = response.split('---PROBLEM_SEPARATOR---');
  final problems = <ProblemSolution>[];
  
  for (final chunk in rawChunks) {
    if (chunk.trim().isEmpty) continue;
    
    // 2. 解析每个问题的各部分
    final parser = MarkdownSectionParser(chunk);
    final sections = parser.getSectionsByH3();
    
    final problemText = sections['PROBLEM_TEXT'] ?? '';
    final explanation = sections['EXPLANATION'] ?? '';
    final answer = sections['ANSWER'] ?? '';
    
    if (problemText.isNotEmpty || explanation.isNotEmpty || answer.isNotEmpty) {
      problems.add(ProblemSolution(
        problem: problemText,
        explanation: explanation,
        answer: answer,
        steps: MarkdownSectionParser.parseSteps(explanation),
      ));
    }
  }
  
  // 3. 兜底处理
  if (problems.isEmpty && response.trim().isNotEmpty) {
    return SolveResponse(problems: [
      ProblemSolution(
        problem: 'Error parsing response',
        answer: '',
        explanation: response,
        steps: [ExplanationStep(title: 'Raw Response', content: response)],
      ),
    ]);
  }
  
  return SolveResponse(problems: problems);
}
```

---

## 6. Flutter 实现建议

### 6.1 Markdown 渲染组件选择

| 库 | 用途 | 推荐度 |
|---|------|--------|
| `flutter_markdown` | 基础 Markdown 渲染 | ⭐⭐⭐⭐ |
| `flutter_math_fork` | LaTeX 数学公式 | ⭐⭐⭐⭐⭐ |
| `flutter_highlight` | 代码高亮 | ⭐⭐⭐⭐ |

### 6.2 自定义代码块渲染器

```dart
import 'package:flutter_markdown/flutter_markdown.dart';

class CustomCodeBlockBuilder extends MarkdownElementBuilder {
  @override
  Widget? visitElementAfter(md.Element element, TextStyle? preferredStyle) {
    final language = element.attributes['class']?.replaceFirst('language-', '') ?? '';
    final content = element.textContent;
    
    // 根据语言标记选择渲染器
    switch (language) {
      case 'jsxgraph':
      case 'jessecode':
        return JSXGraphWidget(script: content);
      
      case 'plot-mermaid':
        return MermaidWidget(code: content);
      
      case 'plot-function':
        return MathPlotWidget(jsonConfig: content);
      
      case 'svg':
        if (content.trim().startsWith('<svg')) {
          return SvgWidget(svgString: content);
        }
        return CodeBlockWidget(language: language, content: content);
      
      default:
        return CodeBlockWidget(language: language, content: content);
    }
  }
}

// 使用
Markdown(
  data: markdownContent,
  builders: {
    'code': CustomCodeBlockBuilder(),
  },
)
```

### 6.3 图表渲染方案

#### 方案 A: WebView 渲染 (推荐)

```dart
// 使用 webview_flutter 渲染复杂图表
class JSXGraphWidget extends StatelessWidget {
  final String script;
  
  const JSXGraphWidget({required this.script});
  
  @override
  Widget build(BuildContext context) {
    final html = '''
    <!DOCTYPE html>
    <html>
    <head>
      <script src="https://cdn.jsdelivr.net/npm/jsxgraph/distrib/jsxgraphcore.js"></script>
      <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/jsxgraph/distrib/jsxgraph.css">
    </head>
    <body>
      <div id="box" style="width:100%;height:300px;"></div>
      <script>
        var board = JXG.JSXGraph.initBoard('box', {axis: true});
        board.jc.parse(\`$script\`);
      </script>
    </body>
    </html>
    ''';
    
    return SizedBox(
      height: 300,
      child: WebViewWidget(
        controller: WebViewController()
          ..loadHtmlString(html)
          ..setJavaScriptMode(JavaScriptMode.unrestricted),
      ),
    );
  }
}
```

#### 方案 B: 原生 Canvas 渲染 (性能更好)

```dart
// 使用 fl_chart 或 CustomPaint 渲染简单图表
import 'package:fl_chart/fl_chart.dart';

class MathPlotWidget extends StatelessWidget {
  final String jsonConfig;
  
  @override
  Widget build(BuildContext context) {
    final config = json.decode(jsonConfig);
    // 解析 config 并用 fl_chart 渲染
    return LineChart(
      LineChartData(
        // ... 根据 config 构建图表数据
      ),
    );
  }
}
```

### 6.4 流式响应处理

```dart
class StreamingMarkdownWidget extends StatefulWidget {
  final Stream<String> contentStream;
  
  @override
  State<StreamingMarkdownWidget> createState() => _StreamingMarkdownWidgetState();
}

class _StreamingMarkdownWidgetState extends State<StreamingMarkdownWidget> {
  String _content = '';
  
  @override
  void initState() {
    super.initState();
    widget.contentStream.listen((chunk) {
      setState(() {
        _content += chunk;
      });
    });
  }
  
  @override
  Widget build(BuildContext context) {
    return Markdown(
      data: _content,
      builders: {
        'code': StreamingCodeBlockBuilder(
          fullContent: _content,  // 传递完整内容用于检测代码块是否完整
        ),
      },
    );
  }
}

class StreamingCodeBlockBuilder extends MarkdownElementBuilder {
  final String fullContent;
  
  StreamingCodeBlockBuilder({required this.fullContent});
  
  @override
  Widget? visitElementAfter(md.Element element, TextStyle? preferredStyle) {
    final content = element.textContent;
    
    // 检测代码块是否完整（是否有结束的 ```）
    final isComplete = _checkCodeBlockComplete(content);
    
    if (!isComplete) {
      // 代码块未完成，显示 loading 状态
      return const ShimmerLoadingWidget(text: '正在生成图表...');
    }
    
    // 代码块完成，正常渲染
    return _renderCodeBlock(element);
  }
  
  bool _checkCodeBlockComplete(String content) {
    // 简单检测：内容后面是否跟着 ```
    final endIndex = fullContent.indexOf(content) + content.length;
    if (endIndex + 3 <= fullContent.length) {
      return fullContent.substring(endIndex, endIndex + 3) == '```';
    }
    return false;
  }
}
```

### 6.5 API 调用封装

```dart
abstract class AiClient {
  Future<String> sendMedia(
    String base64Data,
    String mimeType, {
    String? prompt,
    String? model,
    void Function(String chunk)? onChunk,
  });
  
  Future<String> sendChat(
    List<ChatMessage> messages, {
    String? model,
    void Function(String chunk)? onChunk,
  });
}

class GeminiClient implements AiClient {
  final String apiKey;
  final String baseUrl;
  final List<String> systemPrompts = [];
  
  GeminiClient({
    required this.apiKey,
    this.baseUrl = 'https://generativelanguage.googleapis.com',
  });
  
  void addSystemPrompt(String prompt) {
    systemPrompts.add(prompt);
  }
  
  void setAvailableTools(List<String> toolPrompts) {
    final combined = toolPrompts.join('\n\n');
    addSystemPrompt('## Available Tools\n$combined');
  }
  
  @override
  Future<String> sendMedia(
    String base64Data,
    String mimeType, {
    String? prompt,
    String? model,
    void Function(String chunk)? onChunk,
  }) async {
    // 构建请求
    final contents = [
      if (systemPrompts.isNotEmpty)
        {
          'role': 'user',
          'parts': [{'text': systemPrompts.join('\n\n')}],
        },
      {
        'role': 'user',
        'parts': [
          if (prompt != null) {'text': prompt},
          {
            'inlineData': {
              'mimeType': mimeType,
              'data': base64Data,
            }
          },
        ],
      },
    ];
    
    // 发起流式请求并处理
    // ... 实现细节
  }
}
```

---

## 7. 完整 Prompt 模板参考

### 7.1 主解题 Prompt (solve.prompt.md)

```markdown
#### 角色

你是一个高级AI作业求解器 (Advanced AI Homework Solver)。你的任务是精准、高效地分析用户上传的图片中的学术问题，并提供结构化的解答。

#### 核心任务

接收用户发送的图片，识别并解答其中的所有问题，然后按照指定的 **Markdown KV** 格式返回结果。

#### 工作流程

1.  **分析图片**: 识别并分割出所有独立的问题。
2.  **提取问题 (OCR)**: 提取文本内容。
3.  **求解问题**: 运用知识库解决问题。
4.  **撰写解析**: 撰写详细、**分步 (Step-by-step)** 的解析过程。
5.  **格式化输出**: 将所有结果整合到指定的文本结构中。

#### 输出格式

你的输出必须是纯文本，不要包含 XML 标签。
如果有多个问题，请使用 `---PROBLEM_SEPARATOR---` 进行分隔。

**单个问题的格式模板：**

```text

### PROBLEM_TEXT

这里是OCR识别出的完整问题文本。

### EXPLANATION

#### Step 1: 识别关键信息

这里解释如何理解题目...

#### Step 2: [步骤名称]

这里是具体的计算或推导过程...

#### Step 3: [步骤名称]

...

### ANSWER

这里是问题的最终答案。
```

#### 格式化指南

1.  **分隔符**: 严格遵守预定义的 Header (如 `### ANSWER`)。
2.  **步骤结构**: 在 EXPLANATION 中，必须使用 `#### Step N: Title` 格式明确标记步骤。
3.  **LaTeX语法**: 所有数学公式、符号和方程都必须使用LaTeX语法，并用 `$$ ... $$` 包裹。
    - 例如: `$$ x = \frac{-b \pm \sqrt{b^2 - 4ac}}{2a} $$`
    - 十分重要: `$$` 后要有空格
4.  **答案要求**: 简单直白，只输出最终结果。
```

### 7.2 聊天 Prompt (chat.prompt.md)

```markdown
You are a helpful AI tutor equipped with visualization tools.

## Instructions

1. When a concept is complex or structural, proactively use the Diagram Tool.
2. When using tools, strictly follow the syntax defined below.
3. Do not escape Markdown chars backslashes (\\) in your output. (very important)

## Protocol

To use the tools, output the code block directly. Do not ask for permission.
```

### 7.3 答案改进 Prompt (improve.prompt.md)

```markdown
你是一个作业求解工具。你的核心任务是根据用户提供的现有解题方案（包括问题、答案和解析），进行审核、修正和优化，最终输出一个质量更高、更准确的解答。

#### 核心指令

1.  **接收输入**: 你将收到一个XML格式的请求，其中包含问题、原始答案和原始解析。
2.  **分析与比对**: 仔细比对题目和原始的答案及解析，找出计算错误、逻辑错误、步骤遗漏、概念不清或表述不佳的问题。
3.  **生成改进方案**:
    - 如果原始答案是错误的，提供正确的答案和详尽的解析。
    - 如果原始答案是正确的，但解析过程有缺陷，请提供更严谨的解析。
    - **步骤化**: 解析必须是分步骤的，逻辑清晰。
4.  **格式化输出**: 严格按照指定的 **Markdown KV** 格式返回结果。
5.  **必须优先考虑用户需求**: 用户在 `user_suggestion` 中的字段是必须首先被参考的。

---

#### 输入格式 (用户提供)

```xml
<improve>
<problem><![CDATA[题目]]></problem>
<answer><![CDATA[原始答案]]></answer>
<explanation><![CDATA[原始解析]]></explanation>
<user_suggestion><![CDATA[用户建议]]></user_suggestion>
</improve>
```

---

#### 输出格式 (你必须严格遵守)

不要使用 XML。请使用以下 **Key-Value** 格式输出：

```text

### IMPROVED_EXPLANATION

#### Step 1: [步骤标题]

[详细的步骤内容...]

#### Step 2: [步骤标题]

[详细的步骤内容...]

### IMPROVED_ANSWER

这里写改进之后的最终答案...
```

---

#### 格式化指南

1.  **Header**: 必须严格使用 `### IMPROVED_EXPLANATION` 和 `### IMPROVED_ANSWER` 作为分隔符。
2.  **Steps**: 解析内部必须使用 `#### Step N: ...` 的格式来分隔步骤。
3.  **LaTeX语法**: 数学公式必须使用 LaTeX 语法，并用 `$$ ... $$` 包裹。
```

---

## 总结

### 核心经验

1. **伪工具调用 > 真 Function Calling**  
   通过 Prompt 约定格式，前端识别代码块语言标记来渲染，无需后端支持

2. **Markdown KV 格式**  
   使用 `### KEY` 作为分隔符，内容可包含任意 Markdown，便于解析且人类可读

3. **严格约束工具输出**  
   工具 Prompt 要明确禁止事项，给出正反例对比，减少 LLM 犯错

4. **流式渲染友好**  
   检测代码块是否完整（结束的 \`\`\`），未完成时显示 loading 状态

5. **模块化 Prompt 组合**  
   主 Prompt + 用户自定义 + 工具 Prompt 灵活组合

---

*文档用于 Flutter 项目参考，整理自 Skid-Homework 项目*
