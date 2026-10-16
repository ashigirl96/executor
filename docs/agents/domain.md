# Domain Docs

engineering 系の skill が codebase を調べるときに、この repo の domain docs をどう読むかを決める。

## 調べ始める前に読むもの

- **`GLOSSARY.md`**（repo の root）
- **`docs/adr/`**: これから手を入れる領域に触れる ADR を読む。

どれかが無ければ、**何も言わずに進める**。無いことを指摘したり、先回りして作るよう勧めたりしない。`/domain-modeling` skill（`/grill-with-docs` と `/improve-codebase-architecture` から呼ばれる）が、用語や決定が実際に固まった時点で作る。

## File の配置

この repo は single-context で扱う。`packages/*` と `apps/*` は分かれているが、1 つのプロダクトとして用語を共有するため。

```
/
├── GLOSSARY.md
├── docs/adr/
│   ├── 0001-....md
│   └── 0002-....md
├── packages/
└── apps/
```

`GLOSSARY.md` の用語の見出しは、code に出てくる英語の用語を先に、日本語訳を括弧に入れて書く（例: `## Scope（スコープ）`）。

## Glossary の語彙を使う

出力で domain の概念に名前を付けるとき（issue の title、refactor の提案、仮説、テスト名）は、`GLOSSARY.md` で定義した用語を使う。glossary が避けると決めた同義語に流れない。

必要な概念が glossary にまだ無いなら、それは合図になる。project が使っていない言葉を作っている（考え直す）か、本当に抜けている（`/domain-modeling` 向けに書き留める）かのどちらか。

## ADR との矛盾を示す

出力が既存の ADR と食い違うなら、黙って上書きせず、はっきり示す。

> _ADR-0007（event-sourced orders）と食い違うが、次の理由で見直す価値がある…_
