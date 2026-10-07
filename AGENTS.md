## Development

When starting the dev server, use background mode:

```
astro dev --background
```

Manage the background server with `astro dev stop`, `astro dev status`, and `astro dev logs`.

## Documentation

Full documentation: https://docs.astro.build

Consult these guides before working on related tasks:

- [Adding pages, dynamic routes, or middleware](https://docs.astro.build/en/guides/routing/)
- [Working with Astro components](https://docs.astro.build/en/basics/astro-components/)
- [Using React, Vue, Svelte, or other framework components](https://docs.astro.build/en/guides/framework-components/)
- [Adding or managing content](https://docs.astro.build/en/guides/content-collections/)
- [Adding styles or using Tailwind](https://docs.astro.build/en/guides/styling/)
- [Supporting multiple languages](https://docs.astro.build/en/guides/internationalization/)

## Tailwind CSS 使用规范

本项目已通过 `@tailwindcss/vite` 集成 Tailwind CSS v4。入口与设计 token 集中在 `src/styles/global.css`（`@import "tailwindcss"` 与 `@theme { --color-paper / --color-ink / --color-signal-red / --color-signal-blue / --color-signal-yellow }`）。以下规范约束 Tailwind 在本项目中的使用边界。

### 定位

- Tailwind 主要承担**常规布局、间距、排版、边框、颜色、响应式和基础视觉**的实现。
- 普通 CSS 负责**真正特殊的视觉细节**：流体 `clamp()` 排版、`minmax()` + `fr` 复合网格轨道、伪元素（如 `.playground::after` 的网格底纹）、装饰性绝对定位与 `clip-path`、`:hover` 兄弟选择器变换（如 `.playground:hover .shape--*`）、非标准断点下的网格重排等。
- **不要把 Tailwind 当成设计系统本身。** 设计 token（`@theme` 中的 `--color-*` 等）与全局 reset / 字体 / 主题变量统一在 `global.css` 维护，不在 markup 中另造一套互相不一致的任意色值、字号、间距。

### 何时直接写 utility classes

常规、且能被设计 token 或标准工具类**精确覆盖**的样式，直接在 Astro markup 中组合 utility classes，不要为此新建 CSS class。例如：

- 布局：`grid items-center`、`flex items-baseline justify-between`、`self-start`、`justify-self-end`
- 间距：`gap-4`、`pt-4`、`py-4`、`mb-3`、`m-0`
- 颜色（来自 `@theme` token）：`bg-signal-yellow`、`bg-signal-blue`、`text-signal-red`、`text-signal-blue`、`text-ink`
- 排版：`font-black`、`font-bold`、`uppercase`、`text-base`
- 视觉/交互：`no-underline`、`hover:text-signal-blue`、`w-full`、`relative isolate overflow-hidden`

不要为很短的一组 class 特意新建 CSS class；也不要为了“看起来 Tailwind”硬把复杂 CSS 拆成大量 arbitrary value。

### 何时使用 Astro 组件复用

- 当一组 utility class 具有**明确的 UI 语义**且在多处重复出现时，优先抽成 Astro 组件（例如 header / footer / card），而不是用 `@apply` 把同一串 class 塞回 CSS。
- **不要因为迁移样式而改变现有 Astro 组件边界。** 本项目当前 `.masthead` / `.site-footer` 等跨页共享样式仍由 `global.css` 统一承担，保持其职责不变。

### @apply 的使用边界

- 默认**不使用** `@apply`。
- 仅当“同一组样式在多个地方重复出现、且具有稳定语义”时才考虑 `@apply`；单个元素或只出现一次的组合**不要**使用。
- 不要用 `@apply` 复制 Tailwind 默认工具类已有的能力（例如不要 `@apply grid` 之类）。

### arbitrary values 的使用边界

- `w-[...]`、`top-[...]`、`text-[clamp(...)]` 等 arbitrary values 仅用于**确实需要的单次特殊值**。
- **不要把整个设计系统建立在 arbitrary values 上**：当某个值属于稳定的设计系统约定时（如 `clamp()` 流体字号、`minmax()` 复合网格轨道、`3px` 厚边框、`760px / 650px / 430px` 自定义断点），应保留在 CSS 中，或沉淀为 `@theme` token，而不是到处散写 arbitrary value。

### 可以合理保留的 CSS

- 全局 reset、`:root` / `html` / `body` / `a` 基础规则、`@theme` token、`prefers-reduced-motion` 等全局规则。
- 跨页面共享且已语义化的全局类（如 `.site-shell`、`.masthead`、`.masthead__mark`、`.site-footer`）。
- 流体排版（`clamp()`）、复合网格（`minmax()` + `fr`）、伪元素（`::after`）、装饰性绝对定位与 `clip-path`、`:hover` 兄弟选择器变换、自定义断点下的网格重排等真正特殊的视觉规则。
- 复合选择器 hover（如 `.resource-item--blue a:hover`）——保留在 CSS 比把条件 class 推进模板更清晰。

### 响应式

- 保持 mobile-first，优先使用 Tailwind 断点处理响应式。
- 当现有断点为非标准值（如 760px / 650px / 430px）且涉及复合网格重排时，可继续以媒体查询保留在 CSS 中，不强行改写为 arbitrary breakpoint。

### 目标

最终目标是 **“Tailwind 负责常规 UI 实现，普通 CSS 负责真正特殊的视觉细节”**，而不是 100% CSS 清零。
