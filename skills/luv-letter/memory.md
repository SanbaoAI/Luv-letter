# Memory — 切片化存储（Hermes 思路）

Memory **不是** Session/Thread/Episodic 三个 YAML 桶，也不是聊天记录。

它是 **按用途分层的切片（Slice）存储系统** — 每条 Slice 是可独立 add / replace / remove / search 的原子记忆单元，由 Memory Tool 管理，Orchestrator 调度读写。

> 参考 Hermes：**热记忆进 prompt、温记忆在线程边界更新、冷记忆按需检索、压缩前 flush 蒸馏 durable slices。**

---

## 设计原则（来自 Hermes）

| 原则 | Luv Letter 落地 |
|------|----------------|
| **Curation over capture** | 不存原文对话；只存蒸馏后的 slice |
| **Separate by purpose** | 内心洞察 / 线程状态 / 历史切片 / 回合归档 — 分开存 |
| **Frozen snapshot** | L1 会话启动时冻结注入；会话内写入落盘但不改 frozen |
| **Bounded hot stores** | 字符上限，满了 consolidate，不 silent drop |
| **On-demand cold recall** | 跨线程/跨会话用 `slice_search`，不整库灌进 context |
| **Flush before loss** | Pause / Gate 深度切换 / 上下文压缩前，先 flush durable slices |

---

## 五层记忆架构

```
┌─────────────────────────────────────────────────────────────┐
│ L1 HOT — Frozen Snapshot（会话启动注入，会话内不变）          │
│   INNER.md   用户内在世界 — 已确认主题、锚词、偏好            │
│   LUV.md     反思系统笔记 — 拒框记录、反思惯例                │
├─────────────────────────────────────────────────────────────┤
│ L2 WARM — Active Thread（线程边界更新：pause / 换题 / flush） │
│   THREAD.md  当前单线程蒸馏态                                │
│   thread_buffer[]  本轮待 commit 的 slice 草稿               │
├─────────────────────────────────────────────────────────────┤
│ L3 COLD — Slice Archive（按需 slice_search 检索）            │
│   slices/index.json + slices/*.slice.json                  │
├─────────────────────────────────────────────────────────────┤
│ L4 TURN — Session Archive（ episodic 召回，类似 Hermes FTS） │
│   sessions/{session_id}.jsonl  回合级原文归档                 │
├─────────────────────────────────────────────────────────────┤
│ L5 WORKING — 当前轮（不持久化）                               │
│   observe 输入、route_plan、validate 草稿                     │
└─────────────────────────────────────────────────────────────┘
```

**不要 collapse 成一层。** 向量库式「全塞进去再 retrieve」不是本系统路径。

---

## Slice — 原子记忆单元

每条 slice 是 **一个** 可寻址、可替换、可搜索的记忆片段。

### Schema

```yaml
slice:
  id: sl_{date}_{seq}           # e.g. sl_20250625_003
  store: inner | luv | thread | archive
  type:                         # 见下表
  content: string               # 密集陈述，单条 ≤ 280 chars
  user_anchor: string | null    # 用户原话锚点（优先存这个）
  meta:
    thread_id: string | null
    session_id: string
    turn_id: int | null
    dimension: emotion | trigger | assumption | tension | unspoken | null
    confidence: confirmed | rejected | offered | distilled
    tags: [string]              # e.g. ["availability", "rest"]
  created_at: ISO8601
  superseded_by: string | null  # replace 链；旧 slice 保留但标记失效
```

### Slice Types

| type | store | 何时写入 | 示例 |
|------|-------|----------|------|
| `confirmed_insight` | inner → archive | 用户确认 Layer 2 某维 | `§必须随时回应才会安全` |
| `rejected_frame` | luv | 用户拒框 | `§曾误判为焦虑；用户说是烦` |
| `theme_anchor` | inner | pause flush / 用户重复出现 | `§「必须有用」反复出现` |
| `thread_distill` | thread | 每 2–3 轮或换题前 | `§谈耗竭与回应压力，未到结论` |
| `pause_snapshot` | archive | 用户选 pause | `§触到必须有用；用户选择停下` |
| `preference` | inner | 用户明确偏好 | `§偏好中文慢反思` |
| `image_felt` | archive | 用户描述图片内在感受后 | `§画面像内在阴天，非物体描述` |

### § 分隔符（Hermes 同款）

热存储文件（INNER.md / LUV.md / THREAD.md）多条 entry 用 `§` 分隔，每条 = 一个 slice 的 `content` 视图：

```text
══════════════════════════════════════════════
INNER WORLD [72% — 1,080/1,500 chars]
══════════════════════════════════════════════
§必须随时回应才有安全感（用户原话：不回就焦虑）
§「必须有用」是反复主题；暂停本身带来内疚
§偏好慢反思、不要给建议
```

---

## 存储布局

```
.luv-letter/memory/
├── INNER.md                 # L1 1500 chars
├── LUV.md                   # L1 800 chars
├── THREAD.md                # L2 600 chars
├── .snapshot/               # 会话启动时复制
│   ├── INNER.frozen.md
│   └── LUV.frozen.md
├── slices/
│   ├── index.json           # 检索索引 {id, type, tags, preview, thread_id, created_at}
│   └── sl_20250625_003.slice.json
└── sessions/
    └── sess_abc.jsonl       # L4 每行一个 turn record
```

### 容量上限

| Store | 上限 | 典型条数 | 注入方式 |
|-------|------|----------|----------|
| INNER.md | 1,500 chars | 5–8 | Frozen L1 |
| LUV.md | 800 chars | 3–6 | Frozen L1 |
| THREAD.md | 600 chars | 2–4 | L2 可变 |
| archive | 无硬上限 | — | `slice_search` 按需 |
| sessions | 无硬上限 | — | `turn_search` 按需 |

满容时 **返回 error**，Agent 必须 `slice_consolidate` 或 `slice_remove` 后再 add — 不 silent truncate。

---

## Memory Tool

Orchestrator 通过 ReAct **Act** 阶段调用；Reflection 模块 **不直接写盘**。

完整 API：[memory-tool.md](memory-tool.md)

| Action | 用途 |
|--------|------|
| `slice_add` | 新增一条 slice |
| `slice_replace` | substring 匹配替换（Hermes 同款） |
| `slice_remove` | substring 匹配删除 |
| `slice_search` | 冷存储检索（tags / type / keyword） |
| `slice_consolidate` | 合并重叠 slice，腾出 hot store 空间 |
| `flush` | 压缩/Pause 前蒸馏 durable slices |
| `turn_append` | L4 归档当前回合（原文，不参与 prompt 注入） |

### 写入门禁

| slice type | 门禁 |
|------------|------|
| `confirmed_insight` | **必须** user 明确确认，或 pause flush 且含 `user_anchor` |
| `rejected_frame` | 用户拒框即可写 |
| `theme_anchor` | flush 或同一 theme 出现 ≥2 次 |
| `offered` 未确认假设 | **禁止** 进 INNER.md；最多留 thread_buffer 一轮 |
| 危机内容 | **禁止** 任何 store |

---

## Frozen Snapshot 模式

```
Session Start:
  1. 读 INNER.md + LUV.md 磁盘 live 态
  2. 复制 → .snapshot/*.frozen.md
  3. ReAct Observe 注入 frozen 内容（非 live）

Mid-Session Write:
  1. slice_add → 立即落盘，更新 live 文件
  2. frozen snapshot **不变**
  3. 当前轮通过 tool response + slice_search 感知新 slice
  4. 下一会话启动时进入 frozen

Session End / Pause:
  1. flush() 蒸馏 → INNER / archive
  2. 可选：invalidate snapshot，重建 frozen（缓存一次 miss 可接受）
```

**trade-off：** 会话内新确认的 insight 不在 frozen L1 里，但在 **tool response + thread_buffer + slice_search** 可见。生产取向：保护 prompt 前缀稳定。

---

## Flush — 压缩前蒸馏（Hermes compression flush）

在以下时机 **必须先 flush**，再 trim context / 结束线程：

| 触发 | flush 目标 |
|------|------------|
| `user_choose_pause` | `pause_snapshot` + `confirmed_insight` → inner/archive |
| `round_count >= 4` 且即将 trim | `thread_distill` → thread.md |
| 换题确认 | 旧 thread flush → archive；THREAD.md 清空 |
| 上下文窗口压缩 | 合成指令：「会话将被压缩，提取值得保留的内在洞察、用户校正、重复主题」|

Flush 流程：

```
1. Orchestrator 注入 flush 指令（内部，不对用户展示）
2. 仅开放 memory tool（slice_add / consolidate）
3. 从 thread_buffer + context_window 蒸馏 durable slices
4. 写入 INNER / archive / THREAD
5. trim context_window
6. 继续或 pause
```

**优先保留：** 用户偏好、用户校正、重复主题、confirmed insight。  
**丢弃：** 临时 offered 假设、任务进度、未确认 Deconstruction。

---

## Load 协议（每轮 Observe）

ReAct Observe **不读整库**，只组装 `memory_bundle`：

```yaml
memory_bundle:
  # L1 — frozen，always in Observe
  inner_frozen: string        # .snapshot/INNER.frozen.md
  luv_frozen: string

  # L2 — live thread
  thread_live: string         # THREAD.md
  thread_buffer: [slice]      # 本线程未 flush 草稿

  # L3 — on-demand（Reason 决定要不要搜）
  search_queries: []          # 由 Reason 填充
  search_results: []          # slice_search 返回，max 5 slices

  # Session index（非 slice）
  session:
    phase: INIT | ACTIVE | GATED | PAUSED
    round_count: int
    thread_id: string
    language: zh | en

  # L4 — 仅当用户引用过去时
  turn_search_results: []     # turn_search，max 3 windows
```

### 检索策略

| 场景 | 调用 |
|------|------|
| 新会话开始 | 读 frozen INNER；`slice_search(type=theme_anchor, limit=3)` |
| 继续同主题 | thread_live + thread_buffer |
| 用户说「上次」「之前谈过」 | `turn_search` + `slice_search(keyword=…)` |
| reject 后 | `slice_search(type=rejected_frame, thread_id=current)` 避免重犯 |

---

## Commit 协议（Validate 后）

```
1. turn_append → sessions/{id}.jsonl     # L4 原文归档
2. 执行 queued slice ops（add/replace/remove）
3. 若触发 flush 条件 → flush() 先于此步
4. 清空 working / thread_buffer 已落盘项
5. 更新 slices/index.json
6. 更新 session index（phase, round_count）
```

**原子性：** 单轮 slice ops 要么全成功要么全回滚；写盘用 temp + rename（Hermes `os.replace` 模式）。

---

## 与旧设计的对照

| 旧（YAML 三桶） | 新（切片化） |
|----------------|-------------|
| `frames_confirmed[]` 数组 | `confirmed_insight` slices in INNER + archive |
| `frames_rejected[]` | `rejected_frame` slices in LUV |
| `episodic.entries[]` | `theme_anchor` + `pause_snapshot` in archive，按需 search |
| `context_window` 滑动 | L4 turn archive + 最近 3 轮 working；压缩前 flush |
| `memory.load()` 一把梭 | `memory_bundle` = frozen + thread + optional search |
| 整包 snapshot 给 Reason | 分层加载；冷记忆按需检索 |

---

## 禁止事项

- 把整段对话塞进 slice
- 未确认假设写入 INNER.md
- 诊断标签、危机细节入任何 store
- 图片物体清单写入 slice
- 超 cap silent drop
- 混淆 store 用途（inner 里塞系统调试笔记 → 应进 LUV.md）

---

## 相关文档

- Memory Tool API：[memory-tool.md](memory-tool.md)
- 调度如何使用 bundle：[orchestrator.md](orchestrator.md)
- ReAct Observe 消费方式：[react-loop.md](react-loop.md)
