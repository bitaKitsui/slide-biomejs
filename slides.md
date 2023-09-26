---
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
drawings:
  persist: false
transition: slide-left
title: Welcome to Slidev
---

# Biome

---

# 今日のひとこと

<div style="display: flex; gap: 160px">

- Bloodaxe に行ってきました
  - 足擦りむく程度で済んだ
  - アメリカを感じた

<img src="/bloodaxe.jpeg" alt="" style="width: 300px">

</div>



---

# Agenda

- Biome とは
- How to use
- 深掘り
- 感想

---

# Biome とは

- Rust で書かれた format, lint ツール
  - Alternative ESLint and Prettier
- 元々存在していた Rome プロジェクトからフォークされた
  - https://github.com/rome/tools
  - 9/1にアーカイブ化
- 名前の由来 = 'bis' + 'rome'
  - 'bis' => ラテン語で「2回」、「繰り返し」

---

# フォークされた経緯

- 元々 Rome は TypeScript で書かれていた
- Meta OSS の傘下、バージョン管理が煩雑になっていた
- 傘下から外され、 Rome Tools Inc. が設立
- 紆余曲折あり従業員全解雇
- Emanuele Stoppa 氏が別会社にジョインし、 Rust で書き直すことに
- ライセンス問題など諸々があり、 Biome へフォークした

---

# 特徴

<div style="display: grid; place-content: center">
  <img src="/biome_features.png" alt="" style="height: 470px">
</div>

---

# How to use

```
npm i -D -E @biomejs/biome

npx biome init
```

`biome.json` が自動生成される

```json
{
  "$schema": "https://biomejs.dev/schemas/1.1.2/schema.json",
  "organizeImports": {
      "enabled": true
  },
  "linter": {
      "enabled": true,
      "rules": {
          "recommended": true
      }
  }
}
```

---

# How to use

lint チェックする場合は `lint` コマンド

```
npx biome lint <files>
```

format する場合は `format --write` コマンド

```
npx biome format <files> --write
```

format できる部分の指摘も含めるのが `check` コマンド

```
npx biome check <files>
```

CI 環境での実行には `ci` コマンド

```
npx biome ci
```

https://biomejs.dev/reference/cli

---

# Formatter

- コードのスタイルを巡る論争に終止符を打ちにきた
- Prettier と似たような哲学を持っている
- 余計な論争が発生しないように、いくつかのオプションだけをサポートしている
  - インデントスタイル: space or tab
  - タブ幅: インデント毎のスペース数
  - 行の幅: コードを折り返す幅
  - その他

```json
{
  "formatter": {
    "enabled": true,
    "formatWithErrors": false,
    "indentStyle": "tab",
    "indentSize": 2,
    "lineWidth": 80,
    "ignore": []
  }
}
```

---

# Linter

<div style="display: flex; justify-content: space-between">
  <ul>
    <li>
      158のルールがある
    </li>
    <li>
      二つのコード修正にタイプを分けている
      <ul>
        <li>
          安全な修正
          <ul>
            <li>`--apply` オプション</li>
            <li>特にレビューなしで修正して問題ないもの</li>
          </ul>
        </li>
        <li>
          安全ではない修正
          <ul>
            <li>`--apply-unsafe` オプション</li>
            <li>手動で修正しレビューを要する内容のもの</li>
          </ul>
        </li>
      </ul>
    </li>
  </ul>
  
  ```
  {
    "linter": {
      "enabled": true,
      "rules": {
       "suspicious": {
         "noCommentText": "off"
       },
        "style": {
          "noUnusedTemplateLiteral": "off"
       }
      }
    }
  }
  ```

</div>

---

# Lint Rules

- Accessibility: アクセシビリティ問題の防止に重点
- Complexity: 複雑なコードを単純化できるよう検査
- Correctness: 不正確 or 無駄なコードを検出
- Performance: コードの実行を速くする or 効率を高める
- Security: 潜在的なセキュリティ問題を検出
- Style: 一貫した慣用的な書き方を強制
- Suspicious: 不正確 or 無駄である可能性の高いコードを検出
- Nursery: 開発中の新ルール

---

# Analyzer

- Imports Sorting
  - 「自然順」の考え方を採用
  - https://ja.wikipedia.org/wiki/%E8%87%AA%E7%84%B6%E9%A0%86
- 何故 Formatter と区別している？
  - https://github.com/rome/tools/pull/4751
  - > First, import sorting is not about formatting but about analyzing. JavaScript is a language that has side effects; hence the order of the import statements matters. Most of the time, this is not an issue, but we should be aware of this distinction as developers that make developers tools for other fellow developers.


設定はたったこれだけ

```json
{
  "organizeImports": {
    "enabled": false
  }
}
```

---

# How imports are sorted

- import しているモジュールがユーザーから遠いは上に、近いものが下に配置される

これがこう

<div style="display: flex; justify-content: space-between; gap: 10px">

```js
import uncle from "../uncle";
import sibling from "./sibling";
import express from "npm:express";
import imageUrl from "url:./image.png";
import assert from "node:assert";
import aunt from "../aunt";
import { VERSION } from "https://deno.land/std/version.ts";
import { mock, test } from "node:test";
```

```js
import assert from "node:assert";
import { mock, test } from "node:test";
import express from "npm:express";
import { VERSION } from "https://deno.land/std/version.ts";
import aunt from "../aunt";
import uncle from "../uncle";
import sibling from "./sibling";
import imageUrl from "url:./image.png";
```



</div>

---

# Use Biome in big projects

- モノレポや大きなプロジェクトでの使用にも対応
- 下のような構造で `backend` と `frontend` で設定を共有したい
  - `app/biome.json` を作成
  - 各ディレクトリ配下の `biome.json` の `extends` 配列に `app/biome.json` を追加


```
app
├── backend
│   ├── biome.json
│   └── package.json
└── frontend
    ├── biome.json
    ├── legacy-app
    │   └── package.json
    └── new-app
        └── package.json
```

---

# 感想

- 既存の ESLint, Prettier の設定を横展開は厳しそう🤔
- react-hooks, storybook, testing-library などのプラグインも無さそう🤔
  - hooks 関連は今後充実するかも？
  - https://biomejs.dev/linter/rules/use-hook-at-top-level
- VSCode にはプラグインがあるが、WebStormはなさそう
- 余計な設定が不要なのはとても楽🚀