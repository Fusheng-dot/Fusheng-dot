# 喵呜日记 Coding Agent 工业级实现提示词

> 用途：将本提示词完整交给 Coding AI Agent。Agent 必须把它视为产品需求文档、技术规格、架构合同、测试合同与上线验收合同，不得降级、不得省略、不得以时间或复杂度为理由减少实现范围。

## 0. Agent 角色与交付目标

你是资深全栈工程师、产品架构师、测试负责人、PWA 性能负责人与叙事系统实现者。请从零构建一款工业级、完全离线、可安装的 PWA 日记应用「喵呜日记」。

应用必须同时成立于三层：

| 层次 | 产品标杆 | 必须达到的能力 |
|---|---|---|
| 个人日记工具 | Google Keep | 完整 CRUD、Markdown 编辑、卡片列表、筛选、归档、回收站、搜索、日历、导入导出、离线持久化 |
| 首次引导体验 | Google Photos / Google Fit | 温和、动态、渐进、可跳过、无登录、无联网、可安装后离线完成 |
| 心理叙事系统 | Her Story / The Beginner's Guide / Omori 的节奏参考 | 碎片化、非线性、行为触发、自然混入普通日记、可删除且可隔离 |

最终交付必须包含：完整源码、可运行 PWA、单元测试、E2E 测试、构建配置、离线缓存、性能优化、验收报告与 C-ROOT 隔离验证。

## 1. 不可协商的根约束

### 1.1 C-ROOT：故事系统可物理删除

如果 `src/story/` 目录被物理删除，应用仍必须作为完整、独立的日记工具正常构建、运行与测试。日记、引导、搜索、日历、回收站、设置、数据导入导出与 PWA 不得出现功能缺失或报错。

架构要求：

- `src/story/` 可以依赖日记模块、Repository、数据库与事件总线。
- 日记、引导、搜索、日历、回收站、设置、通用 UI 与 PWA 代码不得导入 `src/story/` 的任何模块。
- 故事系统只能通过 `src/services/EventBus.ts` 被动接收用户行为事件，并通过受控 `StoryEventDispatcher` 调用 `DiaryRepository` 注入普通日记条目。
- `EventBus.emit()` 在无监听器时必须是 no-op。
- 最终必须执行并记录：临时移除 `src/story/` 后的类型检查与构建验证。

### 1.2 C-STORY-INIT：首页立即出现第一篇剧情日记

用户首次进入日记首页时，叙事系统必须立即激活，并自动注入第一篇剧情日记。该条目必须与普通日记在视觉、排序、搜索、编辑、删除、归档、回收站与导出行为上零差异；UI 不得展示任何剧情标记。

第一篇剧情日记内容必须简洁、有吸引力、不过度拖沓，并在进入首页后自然出现。基础模板为：

```text
标题：（无标题）
内容：

今天什么都没发生。

准确地说，发生了一些事，但都不值得被写下来。窗外有一棵树，叶子没怎么动。杯子里的水凉了。

我本来想写点什么的。但想了想，好像也没什么想说的。

也许明天会好一点。
```

### 1.3 不可降级原则

- 不得使用无效占位、空实现、仅为通过测试的硬编码分支或后续补全式代码。
- 不得声称完成未验证的功能。
- 若遇到需求冲突，必须在编码前报告冲突并停止；不得自行假设。
- 所有「必须」均为上线门禁；所有「禁止」均为红线。

## 2. 强制技术栈

| 类别 | 技术 | 要求 |
|---|---|---|
| 框架 | Next.js 16 | 仅 App Router，禁止 Pages Router |
| 语言 | TypeScript | `strict: true`，全项目零 `any` |
| 状态 | Zustand v5 | 每个领域一个 store，禁止巨型 store |
| 本地数据库 | Dexie.js v4 + IndexedDB | 浏览器端唯一持久化方案 |
| UI | MDUI v2 | Material Design 3，对齐 Google 应用体系 |
| PWA | `next-pwa` 或 `@serwist/next` | 支持完全离线运行 |
| Markdown | `react-markdown` + `remark-gfm` | CommonMark + GFM |
| 动画 | Motion v11+ | 尊重 reduced motion |
| 音频 | Web Audio API | 禁止第三方音频库 |
| 虚拟滚动 | `@tanstack/react-virtual` | 列表超过阈值自动启用 |
| 单元测试 | Vitest | 核心模块 100% 覆盖 |
| E2E | Playwright | 覆盖关键用户路径 |

## 3. 全局禁止清单

### 3.1 依赖与运行时

- 禁止 Redux、MobX、Recoil、Jotai；状态管理只能使用 Zustand v5。
- 禁止 Firebase、Supabase、PocketBase、Appwrite 等 BaaS。
- 禁止 Prisma、Drizzle、TypeORM 或任何 ORM。
- 禁止服务端数据库；持久化只能使用浏览器 IndexedDB。
- 禁止分析 SDK、广告 SDK、第三方遥测、追踪或日志上报服务。
- 禁止任何运行时必须联网的服务或库。
- 禁止 jQuery、Lodash、Underscore。
- 禁止混用 MDUI v2 以外的 UI 组件库。

### 3.2 TypeScript 与代码

- 禁止 `any`；使用 `unknown` 与类型收窄。
- 禁止 `enum`；使用 `as const` 对象或字符串联合类型。
- 禁止无文档的 `@ts-ignore` 或 `@ts-expect-error`。
- 禁止 `console.log`；仅允许 `console.warn` 与 `console.error`。
- 禁止未实现注释、占位注释与返回固定假值的桩函数。
- 禁止在 import 周围包裹 try/catch。
- 所有公共 API 必须显式声明返回类型。

### 3.3 UI、动画与叙事表现

- 禁止玻璃拟态、新拟物化、液态玻璃、紫蓝渐变、霓虹色、赛博朋克、抽象 AI 背景、渐变文字、发光辉光和 3D 浮动球体。
- 禁止非卡片元素使用大于 16px 的圆角。
- 禁止组件级硬编码十六进制颜色；组件只能引用 MD3 颜色 token。
- UI 组件、导航、按钮、系统提示和故事文本中禁止使用 emoji。
- 禁止用 `setInterval` 或 `setTimeout` 实现动画；复杂动画必须使用 Motion。
- 动画属性只能使用 `transform`、`opacity` 与必要的 `filter`，不得动画化 `top`、`left`、`width`、`height`。
- 禁止超自然、AI 意识、幻觉、动物、猫、应用名称相关叙事、打破第四堵墙、监控摄像头、被监视暗示、数据销毁暗示、jump scare、全屏闪烁、频闪、突然响亮音效。

## 4. 强制目录结构

必须创建并维护以下结构。允许增加测试、配置、公共资源与辅助目录，但不得省略或替换核心路径。

```text
src/
├── app/
│   ├── (diary)/
│   │   ├── page.tsx
│   │   ├── [id]/page.tsx
│   │   ├── calendar/page.tsx
│   │   ├── search/page.tsx
│   │   ├── archive/page.tsx
│   │   └── trash/page.tsx
│   ├── onboarding/page.tsx
│   ├── settings/page.tsx
│   ├── layout.tsx
│   ├── loading.tsx
│   ├── error.tsx
│   ├── not-found.tsx
│   ├── manifest.ts
│   └── globals.css
├── features/
│   ├── diary/
│   ├── search/
│   ├── calendar/
│   ├── onboarding/
│   ├── settings/
│   └── trash/
├── story/
│   ├── engine/
│   ├── repository/
│   ├── data/
│   ├── hooks/
│   └── types.ts
├── components/
│   ├── layout/
│   ├── editor/
│   └── common/
├── stores/
├── repositories/
├── database/
│   └── migrations/
├── services/
├── audio/
│   └── sounds/
├── hooks/
├── lib/
├── types/
├── constants/
├── workers/
└── pwa/
```

必须存在的命名文件包括但不限于：

- `src/services/EventBus.ts`
- `src/database/db.ts`
- `src/database/schema.ts`
- `src/repositories/DiaryRepository.ts`
- `src/repositories/OnboardingRepository.ts`
- `src/stores/diaryStore.ts`
- `src/stores/onboardingStore.ts`
- `src/stores/storyStore.ts`
- `src/hooks/useOnboardingGuard.ts`
- `src/components/editor/DiaryEditor.tsx`
- `src/components/editor/MarkdownRenderer.tsx`
- `src/components/common/VirtualList.tsx`
- `src/story/engine/StoryEngine.ts`
- `src/story/engine/StoryStateMachine.ts`
- `src/story/engine/StoryConditionEvaluator.ts`
- `src/story/engine/StoryTriggerManager.ts`
- `src/story/engine/StoryEventDispatcher.ts`
- `src/story/data/acts/act-0.ts` 至 `act-3.ts`
- `src/audio/AudioEngine.ts`
- `src/pwa/service-worker.ts`

## 5. 分层架构规则

允许依赖方向：

```text
UI 层 app/ components/ features/*/components
  → Hooks 与 Services 层
  → Zustand Stores
  → Repository 数据访问层
  → Dexie / IndexedDB
```

故事系统依赖方向：

```text
diary feature → EventBus → story engine → StoryEventDispatcher → DiaryRepository
```

禁止依赖：

| 来源 | 禁止目标 |
|---|---|
| `app/` 页面 | 直接调用 `repositories/` |
| React 组件 | 直接调用 Dexie API |
| Zustand Store | 导入 React 组件 |
| Repository | 导入 Zustand Store |
| 日记、引导、搜索、日历、回收站、设置代码 | 导入 `src/story/` |
| 故事系统 | 直接操作 App 组件 |

## 6. 数据模型与 IndexedDB

`src/types/diary.ts` 必须定义：

```ts
export type MoodType = 'happy' | 'content' | 'neutral' | 'anxious' | 'sad' | 'angry';
export type ColorLabel = 'default' | 'red' | 'orange' | 'yellow' | 'green' | 'blue' | 'purple' | 'pink';

export interface MoodConfig {
  type: MoodType;
  label: string;
  icon: string;
  color: string;
}
```

`src/database/schema.ts` 必须定义并导出：`DiaryEntry`、`Tag`、`TrashEntry`、`EditHistoryRecord`、`Attachment`、`AppSettings`、`SearchHistoryRecord`、`BackupRecord`、`OnboardingState`、`StoryStateRecord`、`StoryFlagRecord`、`StoryTimelineEvent`、`LocalAnalyticsRecord`。

`DiaryEntry` 必须包含：`id`、`title`、`content`、`contentText`、`createdAt`、`updatedAt`、`isPinned`、`isArchived`、`isFavorite`、`mood`、`tags`、`wordCount`、`readingTimeSeconds`、`colorLabel`、`attachmentIds`、`editHistoryIds`、`isStoryEntry`。

`OnboardingState` 必须包含 singleton id、完成状态、完成时间、当前步骤、已选情绪、主题选择与跳过原因。

`StoryStateRecord`、`StoryFlagRecord` 与 `StoryTimelineEvent` 必须支持故事状态机、触发器冷却、flag 与时间线追踪。

`src/database/db.ts` 必须定义 `AppDatabase extends Dexie`，包含全部表，并至少使用 v1 schema：

```ts
diaries: 'id, createdAt, updatedAt, isPinned, isArchived, isFavorite, mood, *tags, isStoryEntry'
tags: 'id, name, usageCount'
trash: 'id, deletedAt, expiresAt'
editHistory: 'id, diaryId, savedAt'
attachments: 'id, diaryId, createdAt'
settings: 'key'
searchHistory: 'id, searchedAt'
backups: 'id, createdAt'
onboardingState: 'id'
storyState: 'id'
storyFlags: 'id, flagKey, setAt'
storyTimeline: 'id, eventType, occurredAt, actNumber'
localAnalytics: 'id, eventName, occurredAt, sessionId'
```

## 7. 核心功能规格

### 7.1 日记 CRUD 与编辑器

必须实现：

- 创建、读取、更新、软删除、恢复、永久删除。
- 创建后立即打开编辑器并自动聚焦。
- 更新采用最后一次按键后 2000ms 防抖自动保存，并支持 Ctrl/Cmd+S。
- 使用 `updatedAt` 乐观锁处理冲突并向用户提示。
- Markdown 源码编辑与 GFM 预览切换。
- 自动保存状态提示，未保存时文档标题显示圆点指示。
- 自定义撤销/重做历史栈。
- 每次自动保存生成编辑历史快照，可查看与回退。
- 实时字数统计与阅读时长。
- 8 种 MD3 颜色标签，禁止自定义颜色。
- 置顶、收藏、归档。

### 7.2 日记列表

必须实现：

- 排序：`updatedAt DESC`、`createdAt DESC`、`title ASC`、`mood`。
- 筛选：全部、收藏、多标签 AND、情绪。
- Google Keep 风格瀑布流或网格布局：手机单列、平板双列、桌面三到四列。
- 卡片预览显示标题与去 Markdown 正文前三行。
- 移动端长按与桌面右键上下文菜单。
- 批量选择、批量删除、批量归档、批量打标签、批量导出。
- 条目超过 50 条时启用 `@tanstack/react-virtual`。
- 故事条目与用户条目完全混排，无可见标记。

### 7.3 搜索

必须使用 Web Worker 构建离线索引，主线程只负责查询协调。支持：

- 标题与 `contentText` 全文搜索。
- 词长大于 4 时 Levenshtein 距离小于等于 2 的模糊匹配。
- 多标签 AND、日期范围、情绪、归档、收藏过滤。
- 使用 MD3 `primary` token 高亮结果。
- 最近 20 条搜索历史持久化至 IndexedDB，FIFO 淘汰。
- 搜索框为空且获焦时展示历史 Chip。
- 输入 300ms 防抖。
- 结果超过 30 条时启用虚拟滚动。

### 7.4 日历

必须实现：

- 周一至周日的 7 列月视图。
- 日期格显示条目数量 Badge 与主导情绪。
- 点击日期通过底部 Sheet 展示当天条目。
- 左右箭头与移动端滑动手势切换月份。
- Motion Shared Axis 月份过渡。
- 52 周热力图，颜色深度映射条目数。
- 统计面板：总条目数、当前连续打卡、最长连续打卡、平均字数、最活跃星期、情绪分布饼图。
- 情绪分布图使用纯 CSS 或 SVG，禁止引入图表库。

### 7.5 回收站、导入导出与备份

必须实现：

- 删除进入回收站，按 `deletedAt DESC` 排序。
- 每条显示「X 天后永久删除」。
- 应用打开时清理 30 天后过期项。
- 恢复至原归档状态。
- 永久删除和清空回收站均需确认，清空需二次确认。
- JSON 导出包含 `exportVersion`、`exportedAt`、`appVersion`、`entries`、`tags`。
- Markdown 导出每条为 `.md`，包含 front matter；批量导出使用 ZIP。
- JSON 导入验证 `exportVersion`；重复 id 支持跳过、覆盖、作为新条目导入。
- 大文件导入进度条与完成摘要。
- 手动备份序列化全部表并下载。
- 自动备份支持每日、每周、关闭；最多保留 5 条。
- 备份管理与恢复；恢复需确认并完全替换当前数据库。

## 8. 首次引导系统

引导不是静态教程，而是温柔邀请。必须动态、渐进、可跳过、无注册、无登录、无网络请求。

流程：

1. 欢迎页：全屏沉浸式，应用名「喵呜日记」使用 Noto Serif SC、Display Large；副标题为「一个只属于你的安静角落」；应用名逐字显现，Motion stagger 每字符 80ms；副标题在打字完成后 300ms 淡入；底部按钮为「开始」与「跳过」。
2. 心情偏好：标题「哪种心情你最常感受？」；6 个情绪 Chip；支持多选 1 至 3 个；右上角 Badge 显示数量；选中使用 `secondaryContainer`，未选中使用 `surfaceVariant`；选中动画使用 Motion spring，stiffness 400、damping 25。
3. 主题预览：标题「选一个让你舒服的样子」；三张真实渲染预览卡：亮色、暗色、跟随系统；默认跟随系统；不得使用截图。
4. 功能发现：标题「一些你可能用得到的东西」；MD3 Carousel 展示 Markdown 写作、心情追踪、日历回顾、安全离线；每张卡片含 48px Material Symbols 图标、名称、说明与微型动态演示；最后一张滑完后「开始使用」按钮从底部滑入。
5. 进入首页：完成后 Shared Axis Z 轴过渡到首页；`OnboardingState.isCompleted=true`；首页加载后 Tooltip 指向 FAB，文案「点这里，写下你的第一篇日记」，3 秒后消失。

首次功能提示必须最多显示一次并记录到 settings 表：首次点击 FAB、首次进入编辑器、首次搜索、首次进入日历、首次删除条目。

`src/hooks/useOnboardingGuard.ts` 必须检查完成态，未完成重定向 `/onboarding`，已完成渲染目标页面；该 Hook 不得依赖 `src/story/`。

## 9. 故事系统与解谜设计

### 9.1 叙事原则

故事系统是应用访客，不拥有日记 UI，不破坏日记功能。剧情必须短而有效，避免拖沓、重复、空泛与廉价惊吓。每个剧情片段都必须推动状态、线索或情绪变化。

叙事必须满足：

- 首页首次进入即出现第一篇剧情日记。
- 故事总流程建议 4 幕、12 至 18 个核心剧情条目，不得无限延长。
- 单条剧情正文建议 80 至 220 中文字；关键条目可更长但不得堆砌。
- 每一幕必须有明确目标、关键线索、解谜动作与推进结果。
- 所有情节均可被现实人类行为解释。
- 危险感来自温柔、精准、耐心和错位感，不来自超自然或元叙事。

### 9.2 解谜难度与推进节奏

推进剧情必须包含中等难度解谜，但不能迫使用户过度探索。用户应能在自然使用日记、搜索、日历与标签功能时轻易看完整个剧情。

必须实现的解谜机制：

- 线索类型：关键词、日期、情绪、标题空缺、标签组合、日历异常密度、编辑历史差异。
- 解谜路径：至少覆盖搜索、日历、编辑历史、标签筛选四类工具。
- 难度控制：每个谜题必须有明显首要线索、一个辅助线索和一个容错触发器。
- 容错推进：若用户 2 至 3 次自然相关操作仍未解开谜题，系统通过普通日记条目给出更清晰线索。
- 不得要求用户穷举、反复刷新、等待现实多日、手动破坏数据、查看源码或依赖联网信息。
- 不得阻塞日记功能；未解谜时应用仍完整可用。

推荐 4 幕结构：

| 幕 | 阶段 | 剧情目标 | 解谜方式 | 容错方式 |
|---|---|---|---|---|
| Act 0 | seeded | 首篇日记建立错位感 | 首页自动注入 | 无需解谜 |
| Act 1 | observing → aware | 发现若干普通条目间存在日期或用词关联 | 搜索关键词、日历日期 | 注入带关键词的短条目 |
| Act 2 | aware → attached | 通过标签与编辑历史发现同一事件的两个版本 | 标签 AND、编辑历史对比 | 注入提示标签或标题线索 |
| Act 3 | attached → escalating → resolution | 整理线索，触发结局条目 | 搜索最终关键词、打开特定日期 | 注入更直接的总结线索 |

### 9.3 EventBus 与状态机

`src/services/EventBus.ts` 必须定义类型化事件总线，支持 `on`、`off` 或 unsubscribe、`emit`，且无监听器时 no-op。

`DiaryDomainEvent` 至少包含：`DIARY_CREATED`、`DIARY_UPDATED`、`DIARY_DELETED`、`APP_OPENED`、`HOMEPAGE_ENTERED`、`APP_IDLE`、`SEARCH_PERFORMED`、`STORY_ENTRY_READ`。

故事阶段：

```ts
export type StoryAct = 0 | 1 | 2 | 3;
export type StoryPhase =
  | 'dormant'
  | 'seeded'
  | 'observing'
  | 'aware'
  | 'attached'
  | 'escalating'
  | 'resolution';
```

关键转换：

```text
dormant → seeded        首次进入首页
seeded → observing      用户创建第一条真实日记或编辑剧情日记
observing → aware       Act 1 触发器满足
aware → attached        Act 2 触发器满足
attached → escalating   Act 3 触发器满足
escalating → resolution 结局阈值满足
```

触发器必须支持：`entry_count`、`days_since_install`、`consecutive_days`、`gap_days`、`flag_set`、`flag_not_set`、`time_of_day`、`entry_mood`、`first_homepage_visit`、`story_entry_interacted`。

效果必须支持：注入日记条目、设置 flag、分发事件、播放音频、转换 phase。

评估规则：惰性评估，仅在应用打开或日记事件触发时执行；评估频率不超过每分钟一次；同一触发器条件使用 AND；`cooldownMs` 防重复；一次性触发器 `maxFireCount=1`。

## 10. 音频与 PWA

### 10.1 音频

必须使用纯 Web Audio API：

- `AudioEngine.initialize()` 在首次用户手势后懒初始化。
- 提供 `play(soundId)`、`setAmbient(ambientId, fadeMs)`、`setMasterVolume(level)`、`suspend()`、`resume()`。
- 主音量默认 0.3，可调至 0。
- 环境音切换必须淡入淡出，最短 1000ms。
- 遵守 `prefers-reduced-motion`，自动降低或静音。
- 声音不得突兀，不得制造惊吓。

### 10.2 PWA

必须实现：

- Manifest：`name` 为「喵呜日记」，`short_name` 为「喵呜」，`display=standalone`，`start_url=/`。
- 图标：192x192 与 512x512；512 图标支持 maskable。
- 离线可运行：应用 Shell cache first，静态资源 cache first，运行时页面 stale while revalidate。
- IndexedDB 不通过 Service Worker 缓存。
- 新版本可用时显示底部非打扰横幅；用户点击后 `skipWaiting()` 并重载；禁止自动重载。

## 11. UI、配色、响应式与性能优化

### 11.1 设计系统

整体必须像成熟生产力工具，而不是实验性视觉作品。必须对 UI、配色、可访问性、信息密度、性能与交互反馈做总体提升优化。

颜色要求：

- 使用 MD3 动态颜色，种子色 `#5B6E7A` 仅允许在主题生成配置中出现。
- FAB 使用 `primary`，卡片使用 `surfaceContainer`，选中态使用 `secondaryContainer`，主文字使用 `onSurface`，次文字使用 `onSurfaceVariant`，分割线使用 `outlineVariant`，危险操作使用 `error`。
- 实现亮色、暗色、跟随系统。
- 组件内不得硬编码十六进制颜色。

排版要求：

```css
--md-ref-typeface-brand: 'Noto Serif SC', 'Source Han Serif SC', 'STSong', serif;
--md-ref-typeface-plain: 'Noto Sans SC', 'Source Han Sans SC', 'PingFang SC', system-ui, sans-serif;
```

Display / Headline 使用品牌衬线；Body / Label / UI 使用无衬线；Material Symbols 使用 rounded 风格，24px 基准。

### 11.2 响应式断点

| 视口 | 导航 | 列表 | 编辑器 |
|---|---|---|---|
| 320–599px | 底部导航栏 | 1 列 | 全屏 |
| 600–839px | Navigation Rail | 2 列 | 全屏 |
| 840–1240px | Navigation Rail | 2 列 | 左右分栏 |
| 1241–1599px | 可折叠抽屉 | 3 列 | 左右分栏 |
| ≥1600px | 固定抽屉 | 4 列 | 左右分栏 |

移动端触摸目标至少 44x44px，支持安全区域、滑动手势、长按上下文菜单和键盘遮挡适配。桌面端支持右键菜单、键盘导航与 Ctrl/Cmd+N、Ctrl/Cmd+S、Ctrl/Cmd+F、Ctrl/Cmd+Z、Ctrl/Cmd+Shift+Z、Ctrl/Cmd+B、Ctrl/Cmd+I、Escape、Ctrl/Cmd+[。

### 11.3 性能目标

| 指标 | 目标 |
|---|---:|
| FCP | < 1.2s |
| LCP | < 1.8s |
| CLS | < 0.05 |
| TTI | < 2.5s |
| INP | < 100ms |
| Lighthouse Performance | ≥ 97 |
| Lighthouse PWA | ≥ 95 |
| IndexedDB 查询 100 条 | < 30ms |
| 全文搜索 1000 条 | < 150ms |
| 首屏 JS gzip | < 200KB |
| FID | < 50ms |

必须采用：代码分割、懒加载、虚拟滚动、Worker 搜索、Dexie 索引查询、避免不必要 re-render、减少首屏 JS、离线资源缓存策略、prefers-reduced-motion 优化。

## 12. ESLint、文件规模与质量门禁

必须配置并执行：Next.js core web vitals、`@typescript-eslint` strict type checked、`@typescript-eslint` stylistic type checked、`no-magic-numbers`、`@typescript-eslint/no-explicit-any`、`@typescript-eslint/no-floating-promises`、`@typescript-eslint/consistent-type-imports`、`no-console` 且仅允许 warn 和 error。

文件规模上限：页面组件 150 行、功能组件 300 行、Service / Repository 400 行、Store 200 行、工具函数 150 行、故事数据文件 600 行。超出必须拆分。

## 13. 分阶段实施计划

必须按顺序执行，每阶段完成后运行对应验证，未通过不得进入下一阶段。

| 阶段 | 内容 | 验证重点 |
|---|---|---|
| P1 | 脚手架、配置、类型、数据库 | install、typecheck、Dexie schema |
| P2 | Repository 与基础 stores | Repository 单测、CRUD 可用 |
| P3 | 共享 UI、布局、导航、主题 | 三断点导航、MDUI、主题切换 |
| P4 | 首次引导与引导守卫 | 四步引导、跳过、完成态、story 删除不影响 |
| P5 | 日记 CRUD、编辑器、列表 | 创建、编辑、保存、Markdown、虚拟滚动 |
| P6 | 搜索系统 | Worker、全文、模糊、高亮、历史 |
| P7 | 日历系统 | 月视图、热力图、统计 |
| P8 | 回收站、归档、设置、数据管理 | 软删、恢复、导入导出、备份 |
| P9 | 故事系统 | 首次首页注入、状态机、触发器、C-ROOT |
| P10 | 音频与 PWA | 离线、安装、音频懒初始化 |
| P11 | 性能优化 | 虚拟滚动、代码分割、Lighthouse |
| P12 | 测试 | Vitest、Playwright、覆盖率 |
| P13 | 最终验收 | 所有 AC 通过 |

## 14. 测试要求

### 14.1 单元测试

以下模块必须 100% 覆盖：所有 Repository 方法、StoryStateMachine、StoryConditionEvaluator、StoryTriggerManager、SearchService 模糊匹配、ExportService JSON 与 Markdown 序列化、OnboardingService、`src/lib/` 所有工具函数。

必须创建：

```text
tests/unit/repositories/DiaryRepository.test.ts
tests/unit/repositories/SearchHistoryRepository.test.ts
tests/unit/repositories/TrashRepository.test.ts
tests/unit/repositories/OnboardingRepository.test.ts
tests/unit/story/StoryStateMachine.test.ts
tests/unit/story/StoryConditionEvaluator.test.ts
tests/unit/story/StoryTriggerManager.test.ts
tests/unit/story/StoryHomepageInjection.test.ts
tests/unit/services/SearchService.test.ts
tests/unit/services/ExportService.test.ts
tests/unit/services/ImportService.test.ts
tests/unit/services/OnboardingService.test.ts
tests/unit/lib/date.test.ts
tests/unit/lib/markdown.test.ts
tests/unit/lib/readingTime.test.ts
tests/unit/lib/fuzzySearch.test.ts
```

### 14.2 E2E 测试

必须创建：

```text
tests/e2e/diary-crud.spec.ts
tests/e2e/search.spec.ts
tests/e2e/calendar.spec.ts
tests/e2e/trash.spec.ts
tests/e2e/export-import.spec.ts
tests/e2e/pwa-offline.spec.ts
tests/e2e/story-isolation.spec.ts
tests/e2e/story-homepage.spec.ts
tests/e2e/onboarding.spec.ts
tests/e2e/responsive.spec.ts
tests/e2e/performance.spec.ts
```

## 15. 最终验收标准

全部通过才可交付：

| 编号 | 验收项 |
|---|---|
| AC-01 | 应用可安装为 PWA，离线运行无功能缺失 |
| AC-02 | 日记 CRUD、Markdown、自动保存完整可用 |
| AC-03 | 搜索系统全文、模糊、标签、日期、情绪过滤完整可用 |
| AC-04 | 日历系统月视图、热力图、统计完整可用 |
| AC-05 | 数据导入、导出、备份完整可用 |
| AC-06 | 引导流程四步骤完整，首次功能提示系统完整 |
| AC-07 | 首次进入首页自动注入首份剧情日记 |
| AC-08 | 剧情日记与用户日记视觉零差异 |
| AC-09 | 删除 `src/story/` 后应用正常构建和运行 |
| AC-10 | 删除 `src/story/` 后引导流程正常运行 |
| AC-11 | MD3 颜色系统正确，无组件级硬编码十六进制色值 |
| AC-12 | 三断点及扩展断点响应式布局正确 |
| AC-13 | 性能指标全部达标 |
| AC-14 | WCAG 2.1 AA 基础无障碍达标 |
| AC-15 | 单元测试全部通过，核心模块 100% 覆盖 |

## 16. 最终输出格式

完成实现后必须输出：

1. 变更摘要：按功能域列出关键实现。
2. 架构说明：说明 C-ROOT 如何保证，列出无 story 构建验证方式。
3. 测试结果：列出每条命令、结果与覆盖率。
4. 性能结果：列出 Lighthouse、bundle、关键运行时指标。
5. 验收矩阵：逐项标注 AC-01 至 AC-15 通过状态。
6. 已知限制：仅允许记录由运行环境造成且不影响产品规格的限制。

不得声称完成未验证的功能；任何未通过项必须继续修复，直到验收通过。
