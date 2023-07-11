# Playground Vite

## 目的

- Viteを体験する
- [ドキュメント](https://ja.vitejs.dev/)を読んだ際のメモ書きをする

## 学習メモ

### Viteの理念

- 無駄のない拡張可能なコア
  - 最も一般的なパターンをすぐにサポートすることを目指しているが、すべてのユースケースをカバーするつもりはない
  - プロジェクtの長期的な保守性を維持するために、Viteのコアは小さいAPIで無駄のない状態を維持する
  - ユースケースに応じて適切なプラグインを利用してくださいというスタンス
- モダンWebを推し進める
  - ソースコードはESMのみで書ける
  - Webワーカーは最新の標準の`new Worker`構文で書ける
  - Node.jsモジュールはブラウザでは使用できない
  - etc.
- パフォーマンスに重点をおく
  - プロジェクトの規模が大きくなっても高速なまま動くHMR
  - esbuild,SWCの採用
  - APIの安定性を保ちつつ、DXを向上させる新たなライブラリが登場したら積極的に採用していく
- Vite上にフレームワークを構築する
  - Viteを直接使用することもできるが、フレームワークを作成するためのツールとして輝く

### 特徴

Viteの速さの秘密はesbuildによる事前バンドル

ビルドの最適化もしてくれる（@see <https://ja.vitejs.dev/guide/features.html#%E3%83%92%E3%82%99%E3%83%AB%E3%83%88%E3%82%99%E3%81%AE%E6%9C%80%E9%81%A9%E5%8C%96>）

- CSSのコード分割
- 非同期チャンク読み込みの最適化
- プリロードディレクティブの生成

### TypeScript

ViteはTypeScriptサポートをしているが、トランスパイルのみ。
別途、型チェックする機構をいれこんでおくことが推奨される。

- プロダクションビルドの場合は、`$ tsc && vite build`
- 開発中は、別プロセスで`$ tsc --noEmit --watch`か、[vite-plugin-checker](https://vite-plugin-checker.netlify.app/)を使用

`tsconfig.json`の設定に注意が必要👀

<https://ja.vitejs.dev/guide/features.html#typescript-%E3%82%B3%E3%83%B3%E3%83%8F%E3%82%9A%E3%82%A4%E3%83%A9%E3%82%AA%E3%83%95%E3%82%9A%E3%82%B7%E3%83%A7%E3%83%B3>

### キャッシュ

@see <https://ja.vitejs.dev/guide/dep-pre-bundling.html#%E3%82%AD%E3%83%A3%E3%83%83%E3%82%B7%E3%83%A5>

- File Systemキャッシュ
  - `node_modules/.vite`に事前バンドル済みの依存関係をキャッシュする
  - キャッシュが切られる基準
    - パッケージマネージャーのロックファイル
    - vite.config.jsの関連フィールド
    - パッチフォルダの変更時間
    - `NODE_ENV`
- ブラウザキャッシュ
  - 開発サーバは`max-age = 31536000, immutable`で積極的にキャッシュされる

キャッシュ消したい場合は`--force`コマンドオプションで再起動

### 静的アセットの扱い

@see <https://ja.vitejs.dev/guide/assets.html>

- Importing Asset as URL
  - 本番ビルド時にはハッシュ化されたファイル命になる
  - 一般的な静的アセットの拡張子は自動的に検出される
- publicディレクトリ
  - 以下のようなアセットの場合、こちらに配置する
    - ソースコードで参照されない（ex. `robots.txt`）
    - 全く同じファイル名を保持する必要がある

### ブラウザサポート

@see <https://ja.vitejs.dev/guide/build.html#%E3%83%95%E3%82%99%E3%83%A9%E3%82%A6%E3%82%B5%E3%82%99%E3%81%AE%E4%BA%92%E6%8F%9B%E6%80%A7>

デフォルトでは以下がサポート対象

- Chrome >=87
- Firefox >=78
- Safari >=14
- Edge >=88

### チャンク戦略

@see <https://ja.vitejs.dev/guide/build.html#%E3%83%81%E3%83%A3%E3%83%B3%E3%82%AF%E6%88%A6%E7%95%A5>

`build.rollupOptions.output.manualChunks`で設定できる。

Vite2.9以降はデフォルトではチャンク分割せず、`splitVendorChunkPlugin`を追加すると、`index`と`vendor`にチャンク分割されるようになる。

### 環境変数

@see <https://ja.vitejs.dev/guide/env-and-mode.html>

`import.meta.env`オブジェクトにViteの環境変数が公開される。

- `import.meta.env.MODE`: アプリが動作しているモード
- `import.meta.env.BASE_URL`: アプリが配信されているベースURL
- `import.meta.env.PROD`: アプリがプロダクションで動作しているかどうか
- `import.meta.env.DEV`: アプリが開発で動作しているかどうか
- `import.meta.env.SSR`: アプリがサーバで動作しているかどうか

追加の環境変数を読み込むために、dotenvを利用する。

`VITE_`prefixがついた環境変数のみ、クライアントソースコードに公開され、ついていない環境変数は公開されない。

#### TypeScript用の自動補完

@see <https://ja.vitejs.dev/guide/env-and-mode.html#typescript-%E7%94%A8%E3%81%AE%E8%87%AA%E5%8B%95%E8%A3%9C%E5%AE%8C>

デフォルトで`vite/client.d.ts`で`import.meta.env`のための型定義を提供するが、`.env`ファイルで定義した環境変数の型は、`src/env.d.ts`を作成することで補完することができる。

```ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_APP_TITLE: string
  // その他の環境変数...
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```