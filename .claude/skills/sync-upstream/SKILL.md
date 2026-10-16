---
name: sync-upstream
description: upstream の最新の main を origin の main に merge して push する。
disable-model-invocation: true
---

# upstream の取り込み

`upstream/main` を `origin/main` に merge し、検証を通して push するまでを、ユーザーに確認せず一度で完遂する。conflict も検証の失敗も、下の方針で自分で判断して直す。判断に迷った箇所は、選んだ解決と理由を最後の報告に書く。それがユーザーの見直す窓口になる。

## 手順

1. **作業場所を作る。** `git fetch --multiple upstream origin` のあと、scratchpad（無ければ `$TMPDIR`）に `origin/main` から worktree を切り、以降の作業はすべてそこで行う。ユーザーの checkout は触らない。worktree で `bun run bootstrap` を流す。
   - `git merge-base --is-ancestor upstream/main origin/main` が成功したら、取り込むものは無い。worktree を消し、そう報告して終える。
2. **merge する。** `git merge --no-edit upstream/main`。origin の履歴は push 済みなので、rebase ではなく merge で積む。fork 側に独自の commit が無く fast-forward で済んだときは（`git rev-parse -q --verify MERGE_HEAD` が失敗する）、手順 4 へ進む。
3. **conflict を解く。** 下の「conflict の解き方」に従う。完了条件: `git diff --name-only --diff-filter=U` が空で、`git grep -nE '^(<<<<<<<|>>>>>>>) '` が何も返さない。そのうえで `git commit --no-edit` で merge commit を作る。
4. **検証する。** `bun run bootstrap` を流し直してから、`bun run format:check`・`lint`・`typecheck`・`test` を流す。完了条件: 4 つすべてが通る。
   - 失敗したら、`upstream/main` の worktree でも同じ失敗が出るかを確かめる。upstream でも出る失敗は upstream 由来なので直さず、報告に載せる。upstream では出ない失敗は merge が生んだ失敗なので、直して別の commit に積み、4 つを流し直す。
5. **push する。** `git push origin HEAD:main`。origin が先に進んでいて拒否されたら、`git fetch origin` して `git merge --no-edit origin/main` を足し、手順 3 から繰り返す。origin の履歴を残すため、push は常に fast-forward で行う。
6. **片付ける。** worktree を消す。ユーザーの checkout が `main` にいて未 commit の変更が無ければ、そこで `git merge --ff-only origin/main` して手元も揃える。
7. **報告する。** 取り込んだ範囲（`<旧 origin/main>..<新 origin/main>` と upstream の commit 数）、conflict が出た file ごとの解き方と理由、merge 後に足した修正 commit、検証の結果（upstream 由来で残した失敗を含む）、push した SHA。

## conflict の解き方

この fork の変更は、upstream の file には配線の数行だけを足し、本体は独自の file に置く方針で作られている（`CLAUDE.local.md` を参照）。conflict はほぼ、upstream が動かした土台と、そこに足した配線の衝突になる。

- **upstream の形を土台にし、自分の変更をその上に載せ直す。** まず `git log -p upstream/main..origin/main -- <file>` で、fork 側がその file に何を足したかを読む。次に `git log -p origin/main..upstream/main -- <file>` で upstream 側の変更を読み、upstream の新しい形に、fork の意図した振る舞いを最小の行で足し直す。
- **upstream が file を移動・改名・削除した**: fork の変更を移動先に載せ直す。機能ごと消えた場合は、fork の独自機能が依存していた API を新しい API に繋ぎ替え、その経緯を報告に書く。
- **lockfile（`bun.lock`）**: upstream 側を採ってから `bun install` で作り直す。手で merge しない。
- **生成物**: upstream 側を採ってから、生成元の script で作り直す。
- 解いたあとの file は、fork 側の変更が意図どおり残っているかを diff で確かめる。`git diff upstream/main -- <file>` が、fork の足した配線だけになっていれば正しい。
