🎨 EXITON UX Design Specification v1.0
Document Type: Design System & Interface Specification
Designer: Gemini CLI (Lead UX) + Claude Code (Architecture Review)
Date: 2025-10-15
Status: Figma Implementation Ready
Tone: AI-Bebop — 静かな熱 × Controlled Glow

Executive Summary
EXITON UIは**「構造化された即興演奏の場」である。
人間が指揮者として中心に立ち、AIが熟練した楽器奏者として応答する。
この設計仕様は、その哲学を視覚言語とインタラクションパターン**に翻訳する。
デザイン原則:

Minimal but Dense — 認知負荷を減らしつつ、文脈情報は豊富に
Human-in-Command — 全ての重要な決定は人間が行う
Transparent AI States — AIの状態を直感的に可視化
Flow State Enabler — 深い集中を妨げない流れるようなUI


1. UX 哲学
1.1 コアコンセプト: "The Studio"
EXITONは生産性ツールではなく、創造のためのスタジオ。
音楽スタジオで楽曲を作るように、開発者はこの場でコードを「演奏」する。
人間（Kousuke） = 指揮者・プロデューサー
   ↓
AIエージェント = 熟練した楽器奏者
   ▲ ChatGPT = 構成担当（構造、全体像）
   {} Claude  = ロジック担当（コード、レビュー）
   👁 Gemini  = ビジュアル担当（UI、デザイン）
```

### 1.2 デザインメタファー

| メタファー | UI要素 | 意味 |
|-----------|--------|------|
| **The Studio** | Dashboard | 全体を俯瞰する制作室 |
| **The Score** | Task Board | 楽譜＝プロジェクトの進行計画 |
| **The Session** | Focus View | レコーディングブース＝深い集中空間 |
| **The Stage** | Launch Console | ステージ＝出荷・公開の場 |

### 1.3 視覚的トーン: 静かな熱
```
静かな = ミニマル、暗い背景、過度な装飾なし
熱 = ネオングロー、パルスアニメーション、インタラクティブ性

結果 = 落ち着いた集中空間に、必要な時だけ光が宿る
```

---

## 2. 情報アーキテクチャ

### 2.1 サイト構造
```
/ (Root)
  ↓
/studio (Dashboard — メインハブ)
  │
  ├─ /[project_slug]/score (Task Board — カンバン)
  │   │
  │   └─ /[project_slug]/session/[task_id] (Focus View)
  │
  ├─ /[project_slug]/stage (Launch Console)
  │
  ├─ /academy (ナレッジベース)
  │
  └─ /settings (設定)

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
共有コンポーネント:
- AI Companion Panel (右サイド、コンテキスト依存で表示)
- Command Palette (Cmd+K で全画面から呼び出し)
```

### 2.2 ナビゲーション戦略

**Left Column Navigation (固定80px)**
```
┌────────┐
│   ≡    │ ← Hamburger (モバイル用)
├────────┤
│   🏠   │ ← Studio (Dashboard)
├────────┤
│   📋   │ ← Projects (ドロップダウン)
├────────┤
│   🚀   │ ← Launch
├────────┤
│   🎓   │ ← Academy
├────────┤
│   ⚙️   │ ← Settings
└────────┘
```

**特徴:**
- アイコンのみ表示、ホバーでツールチップ
- 現在地は `Neon Purple #A855F7` でハイライト
- 80px幅固定でスペース効率化

---

## 3. 画面別デザイン仕様

### 3.1 Dashboard — "The Studio"

#### レイアウトグリッド
```
┌────┬─────────────────────────────────────────┬──────────┐
│    │                                         │          │
│ N  │         Main Content                    │   AI     │
│ a  │         (12-column grid)                │   Panel  │
│ v  │                                         │  (320px) │
│    │                                         │          │
│80px│                                         │collapsed │
└────┴─────────────────────────────────────────┴──────────┘
主要コンポーネント
1. Milestone Progress Meter
typescript// 大きな円形プログレスバー
<CircularProgress 
  value={75} 
  size={240}
  strokeWidth={8}
  color="var(--primary-human)" // Purple
  glowOnActive={true}
/>
```

- 現在のマイルストーン達成度を中央に大きく表示
- 75%以上で微細なパルスグロー
- クリックで詳細モーダル

**2. Motivation Loop Widget**
```
┌────────────────────────────────────┐
│  🔥 Current Streak: 12 days        │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━   │
│                                    │
│  Recent Badges:                    │
│  🏆 Week Warrior  ⚡ Speed Coder   │
│                                    │
│  Activity Graph (7 days)           │
│  ▃▅▇▆▇▅▃                          │
└────────────────────────────────────┘

ゲーミフィケーション要素を控えめに
グラフは Cyan #00E0FF の線グラフ
過度な通知は避け、静かに存在

3. Project Cards Grid
typescriptinterface ProjectCard {
  title: string;
  progress: number; // 0-100
  activeAISigils: Array<'chatgpt' | 'claude' | 'gemini'>;
  lastActivity: Date;
}

// グリッドレイアウト
<Grid 
  columns={3} 
  gap={24}
  minCardWidth={280}
>
  {projects.map(p => <ProjectCard {...p} />)}
</Grid>
```

**カードデザイン:**
- 背景: `#0A0A0A` + `1px border rgba(255,255,255,0.1)`
- ホバー: `box-shadow: 0 0 16px 0 rgba(168, 85, 247, 0.3)`
- アクティブAI Sigilsは右上に小さく表示（発光状態なら輝く）

**4. AI Status Panel**
```
AI Status:
  ▲ ChatGPT  ● Idle
  {} Claude   ● Active (2 tasks)
  👁 Gemini   ● Idle

ドット色: Idle=Gray, Active=Cyan, Error=Red
クリックでAI全体ダッシュボードへ

Auto-Layout & Responsiveness
css.studio-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 24px;
  padding: 32px;
}

@media (max-width: 768px) {
  .studio-grid {
    grid-template-columns: 1fr;
  }
}
```

---

### 3.2 Task Board — "The Score"

#### レイアウト構造
```
┌────┬───────────────────────────────────────────────────┬────────┐
│    │  [Backlog] [This Week] [In Progress] [AI Review]  │        │
│    │  [Human Review] [Done]                            │   AI   │
│Nav │  ←  Horizontal Scrollable Kanban  →               │ Panel  │
│    │                                                    │        │
│80px│  AI Sigil Palette (Fixed Bottom-Left)             │ 320px  │
└────┴───────────────────────────────────────────────────┴────────┘
カンバン列定義
列名意味カード色アクセントBacklog未着手なしThis Week今週予定薄い紫In Progress作業中紫AI ReviewAI処理中シアングローHuman Review人間承認待ち紫グロー（強調）Done完了グレーアウト
Task Card 仕様
typescriptinterface TaskCard {
  id: string;
  title: string;
  description?: string;
  assignee: 'human' | 'ai' | null;
  aiSigilDocks: [SigilDock, SigilDock, SigilDock];
  dueDate?: Date;
  tags: string[];
  confidence?: number; // AI生成の場合
}

interface SigilDock {
  agent: 'chatgpt' | 'claude' | 'gemini' | null;
  state: 'idle' | 'working' | 'ready' | 'error';
}
```

**カードビジュアル:**
```
┌──────────────────────────────────┐
│ #123  Fix login bug             │ ← タイトル (Inter Medium)
│                                  │
│ Due: Oct 20                      │
│ Tags: [urgent] [backend]         │
│                                  │
│ AI Sigils:                       │
│ [▲] [{} ✓] [👁]                  │ ← Dockエリア
│  ↑    ↑     ↑                   │
│ 未割当 完了  未割当              │
└──────────────────────────────────┘
```

#### AI Sigil インタラクション

**1. AI Sigil Palette（固定パレット）**
```
┌────────────────────────┐
│  Drag an AI to a task: │
│                        │
│   ▲        {}      👁  │
│  ChatGPT  Claude  Gemini│
└────────────────────────┘
```

- 画面左下に固定表示
- 各Sigilはドラッグ可能
- ホバー時に役割ツールチップ表示

**2. ドラッグ&ドロップフロー**
```
[Palette] ▲ Sigil
    ↓ (User drags)
[Task Card] Dock Area
    ↓ (Drop)
✓ Snap animation + subtle sound
✓ Sigil docked, begins to pulse Cyan
✓ Task moves to "AI Review" column
3. 状態別ビジュアル
StateVisualAnimationIdleGray, no glowNoneWorkingCyan pulseopacity: 0.5 ↔ 1.0, 2s yoyoReadySolid Cyan + badge "Review Ready"NoneErrorRed/AmberNone
CSS実装例:
css.sigil-working {
  color: #00E0FF;
  animation: pulse 2s ease-in-out infinite alternate;
  box-shadow: 0 0 12px 2px rgba(0, 224, 255, 0.6);
}

@keyframes pulse {
  0% { opacity: 0.5; }
  100% { opacity: 1; }
}
Command Palette (Cmd+K)
typescript// 実装イメージ
<CommandPalette
  shortcuts={[
    { key: 'n', label: 'New Task', action: createTask },
    { key: 'a', label: 'Assign AI', action: openSigilSelector },
    { key: 'f', label: 'Filter', action: openFilter },
  ]}
  placeholder="Type a command or search..."
  theme="dark"
/>
```

**デザイン:**
- フルスクリーンオーバーレイ（半透明黒背景）
- 中央に検索ボックス（`backdrop-filter: blur(8px)`）
- 候補リストはキーボードナビゲーション対応

---

### 3.3 Focus View — "The Session"

#### レイアウト（フルスクリーン）
```
┌────────────────────────────────────────────────────────────┐
│  ← Back to Board              [Task #123: Fix login bug]   │
├──────────────────────┬─────────────────────────────────────┤
│                      │                                     │
│  Task Details        │   Code Editor / Preview             │
│  (Left Panel)        │   (Main Workspace)                  │
│                      │                                     │
│  - Title             │   ┌─────────────────────────────┐  │
│  - Description       │   │ // AI-generated code        │  │
│  - Sub-tasks         │   │ function authenticate() {   │  │
│  - Comments          │   │   // ...                    │  │
│                      │   │ }                           │  │
│  AI Outputs:         │   └─────────────────────────────┘  │
│  [{} Claude]         │                                     │
│  Confidence: 0.89    │   [Accept] [Request Revision]       │
│  [View Reasoning]    │                                     │
│                      │                                     │
└──────────────────────┴─────────────────────────────────────┘
主要機能
1. Task Detail Panel（左）

タスク情報の完全表示
サブタスクチェックリスト
コメントスレッド

2. Code Editor Pane（中央）
typescript// Monaco Editor統合
import Editor from '@monaco-editor/react';

<Editor
  height="100vh"
  defaultLanguage="typescript"
  theme="exiton-dark"
  value={code}
  options={{
    fontSize: 14,
    fontFamily: 'JetBrains Mono',
    minimap: { enabled: false },
    lineNumbers: 'on',
  }}
/>
AI生成コードの表示:
typescript// エディタ内でAI生成部分をハイライト
const decorations = [
  {
    range: new monaco.Range(10, 1, 25, 1),
    options: {
      isWholeLine: true,
      className: 'ai-generated-code',
      glyphMarginClassName: 'ai-glyph-claude',
      hoverMessage: { value: '**Claude**: Confidence 0.89\n[View reasoning]' }
    }
  }
];
CSS:
css.ai-generated-code {
  background-color: rgba(0, 224, 255, 0.1);
  border-left: 3px solid #00E0FF;
}

.ai-glyph-claude::before {
  content: '{}';
  color: #00E0FF;
  font-weight: bold;
}
```

**3. AI Companion Panel（右、またはフローティング）**
- 長文対話用チャットUI
- 履歴保持
- コンテキスト表示（「このセッションで使用されたトークン: 1,247」）

**4. 承認コントロール**
```
┌──────────────────────────────────────────┐
│  AI Output by Claude (Confidence: 0.89)  │
│                                          │
│  [✓ Accept]  [✎ Request Revision]  [✗ Reject] │
│                                          │
│  Reasoning Chain: [Expand ▼]            │
└──────────────────────────────────────────┘
```

**ボタンデザイン:**
- Accept: `#A855F7` (Purple)
- Request Revision: `#00E0FF` (Cyan)
- Reject: `#EF4444` (Red)

---

### 3.4 Launch Console — "The Stage"

#### コンセプト
出荷前の最終チェックダッシュボード。  
本番デプロイに必要な全てのチェック項目を可視化。

#### レイアウト
```
┌────────────────────────────────────────────────────────────┐
│  Launch Console: Project XYZ                              │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  Pre-Launch Checklist:                                    │
│  ✓ All tests passed                                       │
│  ✓ Code review completed                                  │
│  ✓ Documentation updated                                  │
│  ⚠ Performance benchmarks: 2/3 passed                     │
│  ✗ Security scan: 1 vulnerability found                   │
│                                                            │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                            │
│  Customer Analytics:                                      │
│  - Active Users: 1,247                                    │
│  - Churn Rate: 2.3%                                       │
│                                                            │
│  [🚀 Deploy to Production]                                │
│  (Disabled until all checks pass)                         │
│                                                            │
└────────────────────────────────────────────────────────────┘

4. AI Interaction Design System
4.1 AI Sigil 定義
ビジュアルデザイン
css/* ▲ ChatGPT (Conductor) */
.sigil-chatgpt {
  content: '▲';
  font-size: 24px;
  color: #A855F7; /* 構造を象徴する紫 */
  font-family: 'Orbitron', sans-serif;
}

/* {} Claude (Logic) */
.sigil-claude {
  content: '{}';
  font-size: 24px;
  color: #00E0FF; /* ロジックを象徴するシアン */
  font-family: 'JetBrains Mono', monospace;
}

/* 👁 Gemini (Design) */
.sigil-gemini {
  content: '👁';
  font-size: 24px;
  filter: drop-shadow(0 0 8px #00E0FF);
}
状態マシン
typescripttype SigilState = 
  | 'unassigned'  // パレット内
  | 'assigned'    // タスクにドック済み、待機中
  | 'working'     // AI実行中
  | 'ready'       // レビュー準備完了
  | 'error';      // エラー発生

const sigilStateColors: Record<SigilState, string> = {
  unassigned: '#888888',
  assigned: '#FFFFFF',
  working: '#00E0FF',
  ready: '#00E0FF',
  error: '#EF4444',
};
```

### 4.2 Visual Task Flow（ユーザージャーニー）
```
1. Assignment (割り当て)
   User drags {} Claude → Task Card
   ↓
   ✓ Snap animation (0.2s ease-out)
   ✓ Subtle sound effect (optional)
   ✓ Sigil docks in designated area

2. Working State (作業中)
   {} Claude begins processing
   ↓
   ✓ Sigil pulses Cyan (opacity 0.5 ↔ 1.0)
   ✓ Task card edge glows faintly
   ✓ Circular progress indicator around sigil (optional)

3. Response Ready (完了)
   AI completes task
   ↓
   ✓ Pulse stops
   ✓ Sigil becomes solid Cyan
   ✓ Badge "Review Ready" appears
   ✓ Non-intrusive notification

4. Human Review (人間レビュー)
   User clicks task card
   ↓
   ✓ Opens Focus View
   ✓ AI output highlighted with Cyan background
   ✓ {} Claude sigil shown with provenance info
   ✓ [Accept] / [Request Revision] / [Reject] buttons

5. Error State (エラー)
   If AI fails
   ↓
   ✓ Sigil turns Red/Amber
   ✓ Click sigil → shows error log in Companion Panel
   ✓ Option to retry or fallback to another agent
4.3 Notification System
typescriptinterface Notification {
  id: string;
  type: 'ai_ready' | 'human_review_required' | 'error' | 'milestone';
  title: string;
  message: string;
  actionUrl?: string;
  priority: 'low' | 'medium' | 'high';
  timestamp: Date;
}

// 表示ルール
const notificationRules = {
  ai_ready: {
    sound: 'subtle-chime.mp3',
    duration: 5000, // 5秒後に自動消去
    position: 'bottom-right',
  },
  human_review_required: {
    sound: 'attention.mp3',
    duration: null, // 手動で閉じるまで表示
    position: 'top-center',
    style: 'pulsing-purple-border',
  },
};
```

**デザイン:**
```
┌────────────────────────────────────┐
│  🔔 Review Required                │
│  Task #123 needs your approval     │
│                                    │
│  [View Now]  [Later]               │
└────────────────────────────────────┘

5. Design System
5.1 Color Palette
css:root {
  /* Backgrounds */
  --bg-primary: #0A0A0A;        /* Jet Black */
  --bg-secondary: #1A1A1A;      /* Slightly lighter for cards */
  --bg-tertiary: #2A2A2A;       /* Hover states */

  /* Text */
  --text-primary: rgba(255, 255, 255, 0.87);
  --text-secondary: #888888;
  --text-tertiary: #555555;

  /* Brand Colors */
  --primary-human: #A855F7;     /* Neon Purple - Human actions */
  --secondary-ai: #00E0FF;      /* Cyan Glow - AI outputs */
  
  /* Semantic Colors */
  --success: #10B981;           /* Green */
  --warning: #F59E0B;           /* Amber */
  --error: #EF4444;             /* Red */
  
  /* Borders & Dividers */
  --border: rgba(255, 255, 255, 0.1);
  --divider: rgba(255, 255, 255, 0.05);
}
使用ガイドライン:

--primary-human: ユーザーが操作するボタン、アクティブ状態、選択
--secondary-ai: AI生成コンテンツ、グロー、通知
背景は常に --bg-primary をベースに

5.2 Typography
css/* Font Imports */
@import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@700;900&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');
@import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;700&display=swap');

/* Type Scale */
:root {
  /* H1 - Brand / Major Titles */
  --font-h1: 700 32px/1.2 'Orbitron', sans-serif;
  --font-h1-mobile: 700 24px/1.2 'Orbitron', sans-serif;
  
  /* H2 - Section Headers */
  --font-h2: 600 24px/1.3 'Inter', sans-serif;
  
  /* H3 - Card Titles */
  --font-h3: 600 18px/1.4 'Inter', sans-serif;
  
  /* Body - Primary Text */
  --font-body: 400 16px/1.6 'Inter', sans-serif;
  --font-body-medium: 500 16px/1.6 'Inter', sans-serif;
  
  /* Small - Labels, Captions */
  --font-small: 400 14px/1.5 'Inter', sans-serif;
  
  /* Code - Monospace */
  --font-code: 400 14px/1.6 'JetBrains Mono', monospace;
}

/* Usage Examples */
.page-title {
  font: var(--font-h1);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: var(--text-primary);
}

.card-title {
  font: var(--font-h3);
  color: var(--text-primary);
}

.code-block {
  font: var(--font-code);
  color: var(--secondary-ai);
}
5.3 Component Standards
Cards
css.card {
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  border-radius: 8px;
  padding: 16px;
  transition: all 0.2s ease-out;
}

.card:hover {
  border-color: var(--primary-human);
  box-shadow: 0 0 16px 0 rgba(168, 85, 247, 0.3);
  transform: translateY(-2px);
}

.card--active {
  border-color: var(--secondary-ai);
  box-shadow: 0 0 16px 0 rgba(0, 224, 255, 0.4);
}
Buttons
css.btn {
  font: var(--font-body-medium);
  padding: 12px 24px;
  border-radius: 6px;
  border: none;
  cursor: pointer;
  transition: all 0.2s ease-out;
}

.btn--primary {
  background: var(--primary-human);
  color: white;
}

.btn--primary:hover {
  background: #9333EA;
  box-shadow: 0 0 20px 0 rgba(168, 85, 247, 0.6);
}

.btn--secondary {
  background: transparent;
  border: 1px solid var(--border);
  color: var(--text-primary);
}

.btn--secondary:hover {
  border-color: var(--secondary-ai);
  color: var(--secondary-ai);
}
Modals
css.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(8px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: var(--bg-secondary);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 32px;
  max-width: 600px;
  width: 90%;
  box-shadow: 0 0 40px 0 rgba(168, 85, 247, 0.3);
}
Badges
css.badge {
  display: inline-flex;
  align-items: center;
  font: var(--font-small);
  font-weight: 500;
  padding: 4px 12px;
  border-radius: 12px;
  background: var(--bg-tertiary);
  color: var(--text-primary);
}

.badge--ai-ready {
  background: var(--secondary-ai);
  color: var(--bg-primary);
}

.badge--urgent {
  background: var(--error);
  color: white;
}
5.4 Neon Glow & Motion Principles
Glow Effects
css/* Contained Glow - AI活動中 */
.glow-ai {
  box-shadow: 0 0 12px 2px rgba(0, 224, 255, 0.6);
}

/* Strong Glow - 人間アクション必要 */
.glow-human-attention {
  box-shadow: 0 0 20px 4px rgba(168, 85, 247, 0.8);
}

/* Subtle Glow - ホバー状態 */
.glow-subtle {
  box-shadow: 0 0 8px 1px rgba(255, 255, 255, 0.1);
}
Animations
css/* Pulse Animation - AI作業中 */
@keyframes pulse-ai {
  0%, 100% {
    opacity: 0.5;
    box-shadow: 0 0 12px 2px rgba(0, 224, 255, 0.6);
  }
  50% {
    opacity: 1;
    box-shadow: 0 0 20px 4px rgba(0, 224, 255, 0.8);
  }
}

.ai-working {
  animation: pulse-ai 2s ease-in-out infinite;
}

/* Slide In - 通知 */
@keyframes slide-in-right {
  from {
    transform: translateX(100%);
    opacity: 0;
  }
  to {
    transform: translateX(0);
    opacity: 1;
  }
}

.notification-enter {
  animation: slide-in-right 0.3s ease-out;
}
Framer Motion推奨設定
typescript// モーション設定
const motionConfig = {
  initial: { opacity: 0, y: 20 },
  animate: { opacity: 1, y: 0 },
  exit: { opacity: 0, y: -20 },
  transition: { 
    duration: 0.2, 
    ease: 'easeOut' 
  },
};

// 使用例
<motion.div {...motionConfig}>
  <Card />
</motion.div>
5.5 Accessibility Guidelines
typescript// Color Contrast Requirements
const contrastChecks = {
  normalText: 4.5,      // WCAG AA
  largeText: 3.0,       // 18px+ or 14px+ bold
  uiComponents: 3.0,
};

// Example validation
function validateContrast(fg: string, bg: string): boolean {
  const ratio = getContrastRatio(fg, bg);
  return ratio >= 4.5;
}

// Neon colors for accent only, never primary text
const accessibleTextOnDark = {
  primary: 'rgba(255, 255, 255, 0.87)',   // 16.1:1 contrast
  secondary: '#888888',                    // 4.6:1 contrast
  // ✗ BAD: '#00E0FF' for body text (too low contrast)
  // ✓ GOOD: '#00E0FF' for highlights only
};
```

---

## 6. Figma Implementation Roadmap

### Sprint 1: 構造ワイヤーフレーム (1週間)

**目標:** 情報アーキテクチャと3カラムレイアウトの検証

**成果物:**
- グレースケールワイヤーフレーム（色なし）
- Studio → Score → Session の遷移フロー
- クリッカブルプロトタイプ

**検証項目:**
- ナビゲーションは直感的か？
- レイアウトバランスは適切か？
- 3カラム構造は機能するか？

**Figmaファイル構成:**
```
📁 EXITON-Wireframes-v1
  ├─ 🖼 Cover Page
  ├─ 📐 Grid System (80px / 12-col / 320px)
  ├─ 🧭 Navigation Flow Diagram
  ├─ 📱 Frame: Studio (Desktop)
  ├─ 📱 Frame: Score (Desktop)
  ├─ 📱 Frame: Session (Desktop)
  └─ 🔗 Prototype Links

Sprint 2: AI連携フロー高忠実度プロトタイプ (2週間)
目標: コアインタラクション「AI-Bebop」の検証
成果物:

Task Board完全版（カラー、アニメーション）
Focus View完全版
AI Sigilドラッグ&ドロップ実装
状態変化アニメーション

検証項目:

Sigilインタラクションは明確で満足感があるか？
AI状態の視覚的フィードバックは理解しやすいか？
Human Reviewフローは直感的か？

重点実装:
typescript// Figmaプラグイン: Smart Animate使用
// 各状態を別フレームとして作成

Frame 1: Sigil Idle (パレット内)
  ↓ Smart Animate
Frame 2: Sigil Dragging (ドラッグ中)
  ↓ Smart Animate
Frame 3: Sigil Docked (タスクにドック)
  ↓ Smart Animate
Frame 4: Sigil Working (パルスアニメーション)
  ↓ Smart Animate
Frame 5: Sigil Ready (完了状態)
```

---

### Sprint 3: デザインシステム・コンポーネントライブラリ (1週間)

**目標:** 再利用可能なコンポーネント体系化

**成果物:**
- Figma Libraryとして公開
- 全カラー定義（Color Styles）
- 全テキストスタイル（Text Styles）
- コンポーネント一式（Variants付き）

**Component List:**
```
📚 EXITON Design System v1.0

Components:
  ├─ 🎨 Colors
  │   ├─ Background / Primary
  │   ├─ Background / Secondary
  │   ├─ Text / Primary
  │   ├─ Brand / Human (Purple)
  │   └─ Brand / AI (Cyan)
  │
  ├─ 📝 Typography
  │   ├─ H1 / Orbitron Bold
  │   ├─ Body / Inter Regular
  │   └─ Code / JetBrains Mono
  │
  ├─ 🃏 Cards
  │   ├─ Project Card (Default, Hover, Active)
  │   └─ Task Card (Default, AI Working, Review Ready)
  │
  ├─ 🔘 Buttons
  │   ├─ Primary (Default, Hover, Disabled)
  │   ├─ Secondary
  │   └─ Ghost
  │
  ├─ 🏷 Badges
  │   ├─ Default
  │   ├─ AI Ready (Cyan)
  │   └─ Urgent (Red)
  │
  ├─ 🎭 AI Sigils
  │   ├─ ChatGPT ▲ (Idle, Working, Ready, Error)
  │   ├─ Claude {} (Idle, Working, Ready, Error)
  │   └─ Gemini 👁 (Idle, Working, Ready, Error)
  │
  └─ 🔔 Notifications
      ├─ Info
      ├─ Success
      └─ Error
```

---

### Sprint 4: オンボーディングプロトタイプ (1週間)

**目標:** 初回体験の設計

**成果物:**
- サインアップフロー
- 初回プロジェクト作成ガイド
- AI紹介チュートリアル

**フロー:**
```
1. Welcome Screen
   "Welcome to EXITON — Your AI Band is Ready"
   
2. AI Introduction
   "Meet your band members:"
   ▲ ChatGPT - The Conductor
   {} Claude - The Coder
   👁 Gemini - The Designer
   
3. First Project Setup
   ▲ ChatGPT: "Let's start with your first project.
               What are you building?"
   [User inputs goal]
   
4. First Task Creation
   "I've created your first milestone.
    Drag an AI to get started."
   [Interactive tutorial]
   
5. Complete First Review
   "Great! You've completed your first AI collaboration.
    This is how EXITON works — you conduct, AI performs."

7. 開発者向けドキュメント
7.1 Component API Examples
typescript// TaskCard Component
interface TaskCardProps {
  task: Task;
  onSigilDrop: (agent: AIAgent) => void;
  onCardClick: () => void;
}

export const TaskCard: React.FC<TaskCardProps> = ({ 
  task, 
  onSigilDrop, 
  onCardClick 
}) => {
  return (
    <motion.div
      className="task-card"
      onClick={onCardClick}
      whileHover={{ scale: 1.02 }}
      transition={{ duration: 0.2 }}
    >
      <h3>{task.title}</h3>
      
      <SigilDockArea
        sigils={task.aiSigilDocks}
        onDrop={onSigilDrop}
      />
      
      {task.confidence && (
        <Badge variant="ai-ready">
          Confidence: {(task.confidence * 100).toFixed(0)}%
        </Badge>
      )}
    </motion.div>
  );
};
7.2 Animation Utilities
typescript// Framer Motion Variants
export const fadeInUp = {
  initial: { opacity: 0, y: 20 },
  animate: { opacity: 1, y: 0 },
  exit: { opacity: 0, y: -20 },
  transition: { duration: 0.2, ease: 'easeOut' },
};

export const pulseGlow = {
  initial: { opacity: 0.5 },
  animate: { 
    opacity: [0.5, 1, 0.5],
    boxShadow: [
      '0 0 12px 2px rgba(0, 224, 255, 0.6)',
      '0 0 20px 4px rgba(0, 224, 255, 0.8)',
      '0 0 12px 2px rgba(0, 224, 255, 0.6)',
    ],
  },
  transition: { 
    duration: 2, 
    repeat: Infinity,
    ease: 'easeInOut',
  },
};
7.3 CSS Variables Integration
typescript// Tailwind Config
module.exports = {
  theme: {
    extend: {
      colors: {
        'bg-primary': 'var(--bg-primary)',
        'primary-human': 'var(--primary-human)',
        'secondary-ai': 'var(--secondary-ai)',
      },
      fontFamily: {
        orbitron: ['Orbitron', 'sans-serif'],
        inter: ['Inter', 'sans-serif'],
        mono: ['JetBrains Mono', 'monospace'],
      },
    },
  },
};

8. 結論: 静かな熱の実装
この仕様書は、EXITON の哲学を視覚的現実に変換するブループリントである。
実装時の最重要原則:

Human Agency First — 人間が常に主導権を持つ
Transparent AI States — AI状態は常に明確に可視化
Minimal but Dense — 最小限のUIで最大限の情報
Controlled Glow — ネオンは使いすぎない、効果的に

次のステップ:

Figma Sprint 1開始（ワイヤーフレーム）
開発チームとのコンポーネント仕様レビュー
ユーザビリティテスト計画立案


Status: ✅ UX Specification Complete — Figma Implementation Ready
Designer: Gemini CLI + Claude Code
Philosophy: 構造が即興を可能にする。規律が創造を生む。
"The interface is not a barrier. It's a stage where humans and AIs perform together."