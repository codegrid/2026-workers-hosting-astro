# AstroのサイトをWorkersにデプロイする

CodeGridの連載「Cloudflare Workersで作るWebサイトのホスティング」第2回のサンプルです。Astroで作ったサイトをCloudflare Workersにデプロイし、SSG・SSRの両方を動かすところまでを扱います。

記事の手順を1ステップずつコミットに分けていますので、`git log`を追いながら、どのファイルがどの段階で変わったのかを確認できます。

## コミットの流れ

| コミット | 内容 | 記事の対応箇所 |
| :--- | :--- | :--- |
| `Initial commit from Astro` | `npm create astro@latest`（basicsテンプレート） | Astroの準備 |
| Wranglerを追加して… | wranglerの導入と、静的アセットを配信する`wrangler.jsonc` | SSGの場合 |
| Cloudflareアダプターを追加して… | `astro add cloudflare`と、`main`・`nodejs_compat`・`binding`の追記 | SSRを使う場合 |
| オンデマンドレンダリングの… | `prerender = false`を指定した`ssr.astro` | SSRなページを追加する |
| 404ページを追加して… | `not_found_handling`とビルド時に出力される404ページ | 運用時のポイント |

## 動かし方

依存パッケージをインストールします。

```
npm install
```

Astroの開発サーバーで確認する場合は次のとおりです。ただし、この方法ではWorkersのランタイム（workerd）は使われません。

```
npm run dev
```

workerd上での挙動を確認したい場合は、ビルドしてからWranglerの開発サーバーを起動します。`Astro.request.cf`の値が取得できるのはこちらです。

```
npm run build
npx wrangler dev
```

`/`は静的なページ、`/ssr`はリクエストのたびにレンダリングされるページです。リロードするたびに表示時刻が変わることを確認してみてください。存在しないパス（`/no-such-page`など）にアクセスすると、ビルド時に生成された静的な404ページが返されます。

## デプロイ

Cloudflareアカウントにログインした状態で、ビルドしてからデプロイします。

```
npm run build
npx wrangler deploy
```

いきなり本番に反映せず、プレビュー用のURLで確認したい場合は次のようにします。実行後に出力される`Version Preview URL:`のURLでプレビューできます。

```
npx wrangler versions upload
```

なお、`wrangler.jsonc`の`name`は`demo-astro-workers`にしてありますので、自分のアカウントにデプロイする際は適宜変更してください。

## 補足

`worker-configuration.d.ts`は自動生成されるファイルのため、リポジトリには含めていません。型が必要な場合は次のコマンドで生成できます。

```
npm run generate-types
```
