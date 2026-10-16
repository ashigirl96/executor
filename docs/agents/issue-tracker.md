# Issue tracker: GitHub

この repo の issue と spec は GitHub issue に置く。操作はすべて `gh` CLI で行う。

issue は `ashigirl96/executor`（origin）に立てる。`UsefulSoftwareCo/executor`（upstream）には立てない。この checkout の `gh` は origin が既定になっているが、別の clone や worktree で `gh repo set-default --view` が upstream を返したら、各コマンドに `-R ashigirl96/executor` を付ける。

## 操作

- **issue を作る**: `gh issue create --title "..." --body "..."`。複数行の本文は heredoc で渡す。
- **issue を読む**: `gh issue view <number> --json number,title,body,labels,comments`。
- **issue を一覧する**: `gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`。必要に応じて `--label` と `--state` で絞る。
- **sub-issue にする**: `gh issue create --parent <parent> ...`、または作った後で `gh issue edit <parent> --add-sub-issue <child>`（`gh` 2.94 以降）。古い `gh` では `gh api --method POST repos/<owner>/<repo>/issues/<parent>/sub_issues -F sub_issue_id=<child-db-id>`（database id は下の **Blocking** と同じ取り方）。sub-issue が使えないときは、子の本文の先頭に `Part of #<parent>` と書く。
- **comment する**: `gh issue comment <number> --body "..."`
- **label を付ける・外す**: `gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **閉じる**: `gh issue close <number> --comment "..."`

## Pull requests as a triage surface

**PRs as a request surface: no.** _（外部からの PR を機能要望として扱う repo なら `yes` にする。`/triage` がこの値を読む。）_

`yes` のときは、PR も issue と同じ label と状態で扱い、`gh pr` の同等のコマンドを使う。

- **PR を読む**: `gh pr view <number> --comments`、diff は `gh pr diff <number>`。
- **triage 対象の外部 PR を一覧する**: `gh api --paginate 'repos/{owner}/{repo}/pulls?state=open' --jq '.[] | select(.author_association | IN("OWNER","MEMBER","COLLABORATOR") | not) | {number, title, author: .user.login, author_association, labels: [.labels[].name]}'`。
- **comment・label・close**: `gh pr comment`、`gh pr edit --add-label`/`--remove-label`、`gh pr close`。

GitHub は issue と PR で番号を共有するので、`#42` だけではどちらか分からない。`gh pr view 42` で確かめ、無ければ `gh issue view 42` を使う。

## skill が「issue tracker に publish する」と言ったとき

GitHub issue を作る。

## skill が「関連する ticket を取ってくる」と言ったとき

上の **issue を読む** のとおりに読む。

## Wayfinding operations

`/wayfinder` が使う。**map** は 1 つの issue で、その **child** issue が ticket になる。

- **Map**: `wayfinder:map` label を付けた 1 つの issue。本文に Notes / Decisions-so-far / Fog を持つ。`gh issue create --label wayfinder:map`。
- **Child ticket**: map の sub-issue としてつないだ issue（**sub-issue にする** を参照）。sub-issue が使えないときは、map の本文の task list に子を足し、子の本文の先頭に `Part of #<map>` と書く。label は `wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）。claim したら、動かしている開発者を assign する。
- **Blocking**: GitHub の **native issue dependencies** を正とする。辺を足すには `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`。`<blocker-db-id>` は blocker の数値の **database id**（`gh api repos/<owner>/<repo>/issues/<n> --jq .id`。`#number` や `node_id` ではない）。GitHub は `issue_dependencies_summary.blocked_by`（open な blocker だけを数える、現在の関門）を返す。dependencies が使えないときは、子の本文の先頭に `Blocked by: #<n>, #<n>` と書く。blocker がすべて閉じたら unblocked になる。
- **Frontier query**: map の open な子を一覧し（`gh issue list --state open` を map の sub-issue か task list に絞る）、open な blocker がある（`issue_dependencies_summary.blocked_by > 0`、または `Blocked by` 行に open な issue がある）ものと assignee がいるものを除く。map の順で最初のものを取る。
- **Claim**: `gh issue edit <n> --add-assignee @me`。session の最初の書き込みにする。
- **Resolve**: `gh issue comment <n> --body "<answer>"`、次に `gh issue close <n>`、最後に map の Decisions-so-far に要点とリンクを足す。
