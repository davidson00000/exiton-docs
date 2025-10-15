🚀 EXITON SaaS 上位構想書（Review Ready v1.0）
🧭 概要

EXITONは、個人開発者が**「つくる」(Build)** と 「届ける」(Launch) を1つの流れで実現できるSaaSエコシステム。
AIに依存せず、人間の発想と判断を主軸に置いた開発・成長支援ツール群で構成される。

Vision:
一人でも、発想から製品リリースまで完走できる時代をつくる。
“Build Smart. Launch Bravely.”

🌐 EXITON Platform 全体構成
flowchart TD
    A[EXITON Platform] --> B[PM Tool 🧩]
    A --> C[Launch 🚀]
    B --> B1[Idea Graph]
    B --> B2[Project Selector]
    B --> B3[PM Core]
    B --> B4[Insight Dashboard]
    B --> B5[Launch Console]
    C --> C1[Profile Sync]
    C --> C2[Matching Board]
    C --> C3[Feedback Form]
    C --> C4[Community Thread]
    B5 --> C1
    C3 --> B4

🧩 EXITON PM Tool（つくる）

目的:
個人開発者がアイデアをMVPまで形にし、継続的に改善できるよう支援する。
AIは補助的存在。主役は“人間の思考と意思決定”。

モジュール	役割	現状
💡 Idea Graph	発想の構造化。Freemap＋Whiteboard統合。テンプレ＋例文支援。	統合設計中
🎯 Project Selector	アイデアのMVP昇格判定。ICE/RICE法などのスコア付け支援。	未実装（v3予定）
📋 PM Core	タスク・進捗・プロジェクト管理。	稼働中
📈 Insight Dashboard	継続率・完了率を自動可視化。週次ナッジ。	v2実装中
🚀 Launch Console	MVP→製品v1比較UI。左右構造・差分通知・タスク化。	v2実装中

目標:
「週末の3時間」で開発を進められる最軽量のPM体験。

🚀 EXITON Launch（届ける）

目的:
個人開発者が作ったMVPを初期ユーザーとマッチングし、実用的なフィードバックを得られる場を提供。
製品を“見てもらう”ことではなく、“改善のために使ってもらう”ことに焦点を置く。

モジュール	役割	状況
🧭 Profile Sync	PM Toolの成果をLaunchに自動同期（開発者プロフィール＋MVP概要）。	仕様策定中
🤝 Matching Board	開発者と初期ユーザーのマッチング（技術タグ＋課題カテゴリ）。	構想中
🧾 Feedback Form	初期ユーザーが直接FBを送信（定量＋定性）。	β版候補
💬 Community Thread	開発者間の情報交換。軽SNS的機能。	v3以降

目標:
最初の10人のユーザーを最短で見つけ、最初のフィードバックを最速で得る。

🔄 EXITON Flow（つくる→届ける）
フェーズ	主な操作	データ連携
1️⃣ MVP完成	開発者がLaunch Consoleで“公開可”を設定	PM Tool → Launch
2️⃣ 初期ユーザー登録	LaunchユーザーがMVPを試用・FB投稿	Launch → PM Tool (Insight反映)
3️⃣ 改善サイクル	開発者がInsightを見て修正・アップデート	内部循環
4️⃣ 製品v1公開	v1をLaunch上に再掲載し履歴管理	双方向同期

思想: 「AIが判断するPMツール」ではなく、「人間が判断し続けられる開発基盤」。

⚙️ 技術基盤（共通仕様）
層	採用技術	ポイント
Frontend	Next.js + Tailwind + D3.js	統一UI／静的生成／軽量構成
Backend	Supabase (PostgreSQL + Auth + Edge Functions)	共通Auth／週次ナッジ／課金Webhook
Storage	Supabase Storage	画像・添付保存
AI	オプション（ローカルLLM or クレジット制）	コストゼロ運用／必要時のみ
課金	Stripe + Supabase Webhook	Free／Pro／Team共通アカウント
Integration	GitHub / Slack / Calendar	Teamプランで拡張可能
💰 収益モデル（共通プラン）
プラン	料金	内容
Free	¥0	5プロジェクトまで／PM Tool基本機能／Launch掲載可（1件）／ローカルLLM可
Pro	¥900/月	無制限プロジェクト／週次AIレビュー／AIクレジット20回／Launch掲載3件まで
Team	¥3,000/月	複数メンバー／RLS権限管理／共同編集／Slack/GitHub連携／Launch無制限
🧱 アカウント設計・DB方針
項目	設計方針
認証	Supabase Auth（共通）
DB構造	Schema分離：pm. / launch.
ユーザ共有	usersテーブルは共通、user_settingsを各SaaSで拡張
課金共有	customer_id（Stripe）共通化、両サービスでPro/Team権限共有
セキュリティ	Row-Level Security（個人／チーム単位アクセス制御）
🗓️ ロードマップ（2025）
フェーズ	期間	内容
Phase 1	〜10月	PM Tool v2（Insight + Launch Console）公開
Phase 2	〜11月	Launch α（MVP掲載・FB収集）リリース
Phase 3	〜12月	共通Auth／共通課金／シームレス統合
Phase 4	2026 Q1	EXITON Hub（学習・コミュニティ）構想開始
💬 レビュー依頼（4 AI共通プロンプト）

あなたはSaaS設計、UX設計、PMツール構築、個人開発支援の専門家です。
以下のEXITON SaaS構想（PM Tool＋Launch）について、以下の観点でレビューしてください。

PM ToolとLaunchを分けた構成は妥当か？統合すべき箇所はあるか？

両者を連携させる最小のデータ連携点はどこにあるか？

個人開発者にとって、PM Tool→Launchの流れは自然で使いやすいか？

現行の収益モデルは持続可能か？Pro/Teamへの移行動機は十分か？

今後Hub構想を追加する際のボトルネック（技術・UX）はどこか？

出力フォーマット（統一）

### 総評
（3行以内）

### 強み
・
・

### 改善提案
1.
2.
3.

### 追加提案（任意）
（あれば）

📎 提出メモ

提出先: Claude, Gemini, Google AI Studio, ChatGPT

ファイル名: exiton_saas_architecture_review_v1.md

目的: 各AIの専門観点を比較し、MVP統合設計へ反映する

期限: 2025年10月15日（水）23:00 JST