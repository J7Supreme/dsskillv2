# Design Token 管理架构 PRD

**Single Source of Truth + Adaptive Token Pipeline**

作者：James Hou
版本：v1.0
状态：Draft

---

# 1. 概述

本文档定义了产品设计系统中的 **Design Token 管理架构**。

目标是在 Git 中建立一个 **Single Source of Truth（SSOT）** 的 Design Token 源文件，并通过自动化流程生成适配不同环境的 Token 格式，包括：

* Figma
* Runtime（Web / CSS / Tailwind / App）
* AI Native 系统

该架构确保：

* 所有设计决策来自统一 Token Source
* 设计与代码保持一致
* AI 能够理解并生成符合 Design System 的 UI

---

# 2. 核心原则

## 2.1 Single Source of Truth

所有 Design Token 存储在 Git 中的唯一源文件：

```
source/tokens.json
```

该文件是 **唯一允许手动编辑 Token 的地方**。

其他 Token 文件均为 **自动生成产物**。

---

## 2.2 单向同步

Token 只允许单向流动：

```
Git Source
    ↓
Token Compiler
    ↓
Adapters
    ↓
Consumers
```

消费者包括：

* Figma
* Runtime
* AI 系统

Figma **不是 Token Source**。

---

## 2.3 Adapter 架构

不同系统需要不同 Token 格式，因此通过 Adapter 生成：

```
figma.tokens.json
runtime.tokens.json
ai.tokens.json
```

---

# 3. Token 架构

Canonical Token Schema 使用四层结构：

```
primitive
semantic
pattern
component
```

---

## 3.1 Primitive Tokens

Primitive Token 表示基础视觉数值。

例如：

```
primitive.color.blue.500
primitive.spacing.8
primitive.radius.md
primitive.shadow.sm
```

特点：

* 表示视觉 scale
* 不包含 UI 语义
* 可复用

---

## 3.2 Semantic Tokens

Semantic Token 表示设计语义。

例如：

```
semantic.text.primary
semantic.background.card
semantic.border.default
semantic.icon.primary
```

Semantic Token 通常引用 Primitive Token。

例如：

```
semantic.text.primary → primitive.color.navy.900
```

---

## 3.3 Pattern Tokens

Pattern Token 表示 UI Pattern。

例如：

```
pattern.surface.card
pattern.surface.modal
pattern.layout.container
```

---

## 3.4 Component Tokens

Component Token 表示组件级设计规则。

例如：

```
component.button.primary.background
component.button.primary.padding
component.button.primary.radius
component.card.shadow
```

这些 Token 允许 Runtime 与 AI 系统重建组件。

---

# 4. Design Token System Architecture

本章节描述 **Design Token 系统整体架构**，展示 Token 从 Source 到各系统消费端的完整流程。

---

## 4.1 高层系统架构

```
                 ┌─────────────────────┐
                 │   Design Token      │
                 │   Source of Truth   │
                 │                     │
                 │   source/tokens.json│
                 │   (Git Repository)  │
                 └──────────┬──────────┘
                            │
                            ▼
                 ┌─────────────────────┐
                 │    Token Compiler   │
                 │                     │
                 │  - Resolve tokens  │
                 │  - Validate schema │
                 │  - Expand alias    │
                 │  - Build adapters  │
                 └──────────┬──────────┘
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼

         Figma Adapter   Runtime Adapter   AI Adapter

              │             │               │
              ▼             ▼               ▼

      figma.tokens.json  runtime.tokens.json  ai.tokens.json

              │             │               │
              ▼             ▼               ▼

       Figma Variables     CSS / Tailwind     AI UI Generation
       Tokens Studio       React / App        AI Reasoning
```

---

## 4.2 架构组件说明

### Token Source

Token Source 是整个系统的 **唯一设计数据源**。

```
source/tokens.json
```

特点：

* 存储在 Git
* 所有 Token 修改必须通过 PR
* 不依赖任何工具

---

### Token Compiler

Token Compiler 负责：

* 解析 Token 引用
* 校验 Schema
* 构建 Token 层级
* 生成适配不同系统的 Token

核心流程：

```
load tokens
   ↓
resolve references
   ↓
validate schema
   ↓
generate adapters
```

---

### Adapter Layer

Adapter Layer 负责生成不同系统需要的 Token。

#### Figma Adapter

生成：

```
figma.tokens.json
```

用于：

* Tokens Studio
* Figma Variables

---

#### Runtime Adapter

生成：

```
runtime.tokens.json
```

用于：

* CSS Variables
* Tailwind
* React
* Mobile

---

#### AI Adapter

生成：

```
ai.tokens.json
```

用于：

* AI UI generation
* AI reasoning
* AI design validation

---

# 5. Repository 结构

推荐结构：

```
design-tokens
│
├── source
│   └── tokens.json
│
├── adapters
│   ├── figma
│   │   └── tokens.figmatokens.json
│   │
│   ├── runtime
│   │   └── tokens.runtime.json
│   │
│   └── ai
│       └── tokens.ai.json
│
├── schema
│   ├── dtcg.schema.json
│   ├── figma.schema.json
│   ├── tokenstudio.schema.json
│   └── runtime.schema.json
│
└── scripts
    └── build-tokens.ts
```

---

# 6. Token Schema 设计

Source Token Schema 需要兼容：

* Figma Native Variables
* Tokens Studio
* Runtime Token
* W3C DTCG
* AI Native Token

Schema 以 **Reference 文档形式独立维护**。

Token Compiler 会根据 schema 自动生成 Token。

---

# 7. Figma 集成

Figma Token 通过 Adapter 自动生成。

流程：

```
Git tokens
      ↓
generate figma.tokens.json
      ↓
Tokens Studio import
      ↓
Apply to Figma Variables
```

规则：

* Figma Token **只读**
* 不允许在 Figma 修改 Token
* Token 只允许 **从 Git 同步**

---

# 8. Runtime 集成

Runtime Token 用于：

* CSS Variables
* Tailwind
* React Theme
* Mobile Design Token

示例：

```
--color-text-primary
--spacing-sm
--radius-md
```

---

# 9. AI Native Token Layer

AI Adapter 生成：

```
ai.tokens.json
```

AI 可以通过这些 Token：

* 理解 UI
* 生成组件
* 保持设计一致性

---

# 10. Token Flow 示例

```
primitive.color.blue.500
        ↓
semantic.text.primary
        ↓
component.button.primary.background
```

消费者：

```
Figma
Runtime
AI
```

---

# 11. Design Governance

Token 修改流程：

```
token change
     ↓
pull request
     ↓
design review
     ↓
merge
     ↓
token rebuild
```

保证：

* 设计治理
* 版本可追溯

---

# 12. AI Token Consumption Model

AI 在生成 UI 时 **不直接生成视觉数值**。

AI 必须引用 Token。

例如：

```
background: semantic.background.primary
padding: primitive.spacing.8
radius: primitive.radius.md
```

而不是：

```
background: #007FFF
padding: 8px
```

---

# 13. AI UI Generation Flow

AI UI 生成流程：

```
User prompt
     ↓
AI reasoning
     ↓
Token lookup
     ↓
Component resolution
     ↓
Generate UI
```

---

# 14. AI Component Reconstruction

AI 可以通过 Token 层级重建组件：

```
component.button.primary
```

AI 可以解析：

```
layout
background
typography
spacing
```

---

# 15. AI Design Consistency

AI 生成 UI 必须遵守：

1. 不生成未定义设计值
2. 不直接写 hex color
3. spacing 必须来自 token scale

错误：

```
padding: 7px
```

正确：

```
padding: primitive.spacing.8
```

---

# 16. AI Token Pipeline

```
source tokens
      ↓
token compiler
      ↓
ai.tokens.json
      ↓
AI system
      ↓
UI generation
```

---

# 17. Token Naming Convention

Token 命名结构：

```
layer.category.element.variant.property
```

示例：

```
primitive.color.blue.500
semantic.text.primary
component.button.primary.background
```

---

# 18. 总结

该架构建立了一套 **AI Native Design Token Infrastructure**。

核心特点：

* Git Single Source of Truth
* Adapter Token Pipeline
* Schema Reference Architecture
* AI Native Token Consumption
* Scalable Design System

Design Token 成为：

```
Design System
Runtime
AI UI Generation
```

之间的统一语义层。
