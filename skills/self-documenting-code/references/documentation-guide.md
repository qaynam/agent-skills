# API ドキュメント（JSDoc / TSDoc）ガイド

JSDoc/TSDoc・docstring を書く前に読む。

## Step 0：ファイル種別の分類（必須）

ルールはファイル種別で変わる。最初に分類する。

| 種別 | 型情報の扱い |
| --- | --- |
| `.ts` / `.tsx` | 型は TS 構文が真実。JSDoc に型を書かない（`@param {Type}` 禁止、`@type` 禁止）。 |
| `.js` / `.jsx`（checkJs / JSDoc 型付け運用） | JSDoc が型システムの一部。`@param {Type}`、`@returns {Type}`、`@typedef` は**正当**であり必須になりうる。 |
| `.d.ts` / npm 公開ライブラリ | TSDoc 準拠。API Extractor / TypeDoc 等、実際に使われているツールの設定に従う。 |

TSDoc タグには Core / Extended / Discretionary の区分があり、`@throws`
`@example` `@defaultValue` `{@inheritDoc}` 等は Extended。doc 生成ツールを
使うプロジェクトでは、そのツールが対応しているかを設定から確認する。
アプリケーションコード（エディタ表示が主目的）では気にしなくてよい。

## Step 1：API 境界の特定

`export` ≠ 公開 API。以下で判定する：

- npm パッケージの export map / 公開エントリポイントから到達可能か
- 外部（別チーム・別リポジトリ・OSS 利用者）が呼ぶか
- モノレポ内・ファイル間共有のためだけの export か
- テスト用 export、フレームワークが参照するだけのシンボルか

| 分類 | ドキュメント方針 |
| --- | --- |
| 公開 API | 契約を文書化する（下の契約チェックリスト参照） |
| 内部の exported | 名前＋シグネチャで足りなければ summary 程度 |
| 非 export ヘルパー | 原則不要。目的が読めない場合のみ |

## Step 2：doc の置き場所

- 契約は**利用者に最も近い宣言**に書く。複数実装が同じ契約を共有するなら
  インターフェース／基底型に置き、ツールが対応するなら `{@inheritDoc}` を使う。
- 実装がインターフェースの契約から逸脱しているのを見つけたら、doc で
  ごまかす前に、契約か実装のどちらを直すべきかをまず疑う。直せない正当な
  差異（性能特性など）だけを実装側に書く。
- ファイル／モジュール全体の説明：
  - Google/JSDoc 系の規約があるリポジトリ → `@fileoverview`
  - TSDoc/API Extractor のライブラリ → エントリポイントに `@packageDocumentation`
  - 通常のアプリコード → モジュール境界や不変条件が exports から読めない場合のみ

## Summary（1 行目）の書き方

一文。利用者の抽象度で「何をするか／何を保証するか」。実装方法ではなく、
偶発的な呼び出し元でもない。動詞から始める（「〜を返す。」「〜を削除する。」）。

関数名と同じ内容でも、抽象度が違えば価値がある。名前の逐語的言い換えだけが
ノイズである：

```ts
// ❌ 名前の言い換え（情報ゼロ）
/** id でユーザーを取得する。 */
function getUserById(id: string): Promise<User>

// ✅ 「何を」の説明だが、契約（トークン再利用不可）を足している
/**
 * 指定時刻より前に期限切れとなったセッションを削除する。
 *
 * 削除されたセッションのトークンは再利用できない。
 */
function deleteExpiredSessions(before: Date): Promise<number>
```

## タグ別ルール（.ts / .tsx）

- **`@param name - 説明`** — 名前＋型を超える意味がある引数だけ。単位、
  有効範囲、役割、`undefined` の意味、**省略時のデフォルト値**もここ。
  部分的なリストで構わない。どれも不要なら `@param` 自体を書かない。

  ```ts
  /**
   * チャンネル内でユーザーをミュートする。
   * @param target - ミュートされる側のユーザー（実行する側ではない）。
   * @param durationMs - `Infinity` で恒久ミュート。省略時は 10 分。
   */
  function mute(target: User, durationMs?: number): void
  ```

- **`@returns`** — 値の意味が非自明なときのみ。特に `null`・空配列・
  番兵値が何を意味するか。
- **`@throws`** — 呼び出し側にハンドリングを期待する例外型ごとに 1 ブロック。
  シグネチャに現れないため、しばしば最も価値が高い。
- **`@example`** — 使い方が自明でない公開 API に。1 例 1 ブロック、
  最小で、コンパイルが通る形に。
- **`@remarks`** — summary は短く保ち、長い文脈（設計上の制約、性能、
  仕様リンク）はここへ。
- **`@deprecated`** — 必ず移行先とセット：`@deprecated 代わりに {@link newThing} を使う。`
- **`@defaultValue`** — **インターフェース／クラスのフィールド・プロパティ専用**
  （TSDoc 仕様）。関数引数のデフォルトは `@param` 内に書く。

  ```ts
  interface SearchOptions {
    /**
     * 検索結果の最大件数。
     * @defaultValue 20
     */
    limit?: number;
  }
  ```

- **`@internal`** — doc 生成・API レポートから除外する目的で可。
- **可視性・修飾タグ** — TypeScript 構文（`private`、`readonly`、`override`、
  `implements`、`enum`）と**同じ情報を重複させる目的**では書かない。
  ただし API Extractor 等が `@public` / `@beta` / `@alpha` / `@internal` を
  **リリース段階タグ**として要求する場合はプロジェクト設定に従う
  （それは可視性の重複ではなく安定性の宣言）。

## 公開 API の契約チェックリスト

良い doc は型が語らない契約を語る。公開 API では該当するものを検討する：

- 副作用（I/O、グローバル状態、イベント発火とそのタイミング）
- 入力を変更するか（mutation）／返り値の所有権（呼び出し側が変更してよいか）
- 冪等性、リトライして安全か
- 結果の順序と安定性（「スコア降順。同点の順序は保証しない」）
- `null`・空・エラー時のそれぞれの意味の違い
- Promise の解決・拒否条件、キャンセル時の挙動
- キャッシュの有無と無効化条件
- 並行呼び出し・再入の安全性
- 日時のタイムゾーン前提
- セキュリティ上の前提（入力はサニタイズ済みか等）
- 性能特性（計算量、大量データでの挙動）が利用判断に影響する場合
- 使用後に破棄（dispose / close）が必要か

```ts
/**
 * 現在の検索条件に一致する記事を返す。
 *
 * @remarks
 * 結果はスコア降順。同点の場合の順序は保証しない。
 * 入力の `filters` は変更しない。
 *
 * @throws {@link SearchTimeoutError}
 * 制限時間内に検索が完了しなかった場合。
 */
function searchArticles(filters: Filters): Promise<Article[]>
```

## React コンポーネント

- doc は props の**型のメンバー**に書く（そこが公開 API）。`@param props` は書かない。
- コンポーネント本体の doc ＝ 何をレンダリングするか＋特筆すべき挙動
  （ポータル、副作用、必要な context、非制御/制御の別）。
- optional props のデフォルトは `@defaultValue`（プロパティなので仕様上も正しい）。
