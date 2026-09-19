# Issue追跡

**共通規約（ファイル名 `NNN-<type>-<slug>.md`・type 語彙・状態ディレクトリ `next/` `pending/` `waiting/`
`done/` `epic/<name>/`・`期限:`・`human`・`retro`・本文の書き方）は `~/dotfiles/_claude/issue-rules.md` が正本**で、
issues/ を持つ repo のセッションに SessionStart hook が注入する。**ここへ共通規約を写さない**
（写したコピーは更新が伝播せず古いまま残る。dotfiles issue 401）。

このファイルには、この repo でだけ効く道具・固有ディレクトリ・経緯を書く。

## この repo の道具

- 命名チェック: `make lint-issues`（Makefile に target がある場合）
- 採番: `issues/` 配下を全サブディレクトリまで走査し、最大番号 + 1

## Issue索引（`INDEX.md` がある場合）

Issue一覧・優先度・ステータスはアプリ固有の `INDEX.md` で管理する。
**issue を追加・完了・ステータス変更したら `INDEX.md` を必ず更新すること。**

- **純粋なインデックス**: 番号・タイトル・ステータス・優先度のテーブルだけ。詳細な概要や解説は書かない（トークン節約）
- Open Issues を種類別に整理する。Done 一覧は書かない

---

> この雛形は `project-templates/{macos,ios}/issues/README.md` で管理している。repo 固有の節を足してよい
> （共通規約の変更は dotfiles `_claude/issue-rules.md` へ）。
