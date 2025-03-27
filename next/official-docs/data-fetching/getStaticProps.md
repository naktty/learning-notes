# getStaticProps

ページから`getStaticProps`（Static Site Generation）という関数をエクスポートすると、Next.jsはビルド時に`getStaticProps`が返すpropsを使用してそのページを事前レンダリングします。

```typescript
// pages/index.tsx
import type { InferGetStaticPropsType, GetStaticProps } from 'next'
 
type Repo = {
  name: string
  stargazers_count: number
}
 
export const getStaticProps = (async (context) => {
  const res = await fetch('https://api.github.com/repos/vercel/next.js')
  const repo = await res.json()
  return { props: { repo } }
}) satisfies GetStaticProps<{
  repo: Repo
}>
 
export default function Page({
  repo,
}: InferGetStaticPropsType<typeof getStaticProps>) {
  return repo.stargazers_count
}
```

> 注意：レンダリングタイプに関係なく、すべての`props`はページコンポーネントに渡され、初期HTMLの一部としてクライアントサイドで表示できます。これはページを正しくハイドレーションするためです。クライアントで利用可能であってはならない機密情報を`props`に渡さないように注意してください。

getStaticProps APIリファレンスでは、`getStaticProps`で使用できるすべてのパラメータとpropsについて説明しています。

## getStaticPropsはいつ使用すべきか？

以下の場合に`getStaticProps`を使用すべきです：

* ページのレンダリングに必要なデータが、ユーザーのリクエスト前にビルド時に利用可能な場合
* データがヘッドレスCMSから取得される場合
* ページが事前レンダリングされ（SEOのため）、非常に高速である必要がある場合 — `getStaticProps`は`HTML`と`JSON`ファイルを生成し、両方ともCDNでキャッシュしてパフォーマンスを向上させることができます
* データを公開キャッシュできる場合（ユーザー固有でない場合）。この条件は、特定の状況でMiddlewareを使用してパスを書き換えることで回避できます

## getStaticPropsはいつ実行されるか

`getStaticProps`は常にサーバーサイドで実行され、クライアントサイドでは実行されません。このツールを使用して、`getStaticProps`内に書かれたコードがクライアントサイドバンドルから削除されていることを確認できます。

* `getStaticProps`は常に`next build`中に実行されます
* `fallback: true`を使用する場合、`getStaticProps`はバックグラウンドで実行されます
* `fallback: blocking`を使用する場合、`getStaticProps`は初期レンダリング前に呼び出されます
* `revalidate`を使用する場合、`getStaticProps`はバックグラウンドで実行されます
* `revalidate()`を使用する場合、`getStaticProps`はオンデマンドでバックグラウンドで実行されます

Incremental Static Regenerationと組み合わせた場合、`getStaticProps`は古いページが再検証されている間バックグラウンドで実行され、新しいページがブラウザに提供されます。

`getStaticProps`は静的HTMLを生成するため、受信リクエスト（クエリパラメータやHTTPヘッダーなど）にアクセスできません。ページでリクエストにアクセスする必要がある場合は、`getStaticProps`に加えてMiddlewareの使用を検討してください。

## CMSからデータを取得するためのgetStaticPropsの使用

以下の例は、CMSからブログ投稿のリストを取得する方法を示しています。

```typescript
// pages/blog.tsx
// postsはgetStaticProps()によってビルド時に設定されます
export default function Blog({ posts }) {
  return (
    <ul>
      {posts.map((post) => (
        <li>{post.title}</li>
      ))}
    </ul>
  )
}
 
// この関数はビルド時にサーバーサイドで呼び出されます
// クライアントサイドでは呼び出されないため、
// 直接データベースクエリを実行することもできます
export async function getStaticProps() {
  // 投稿を取得するために外部APIエンドポイントを呼び出します
  // 任意のデータ取得ライブラリを使用できます
  const res = await fetch('https://.../posts')
  const posts = await res.json()
 
  // { props: { posts } }を返すことで、Blogコンポーネントは
  // ビルド時に`posts`をpropsとして受け取ります
  return {
    props: {
      posts,
    },
  }
}
```

## サーバーサイドコードを直接書く

`getStaticProps`はサーバーサイドでのみ実行され、クライアントサイドでは実行されません。ブラウザのJSバンドルにも含まれないため、直接データベースクエリを書くことができ、それらはブラウザに送信されません。

これは、`getStaticProps`から**APIルート**を呼び出して（それ自体が外部ソースからデータを取得する）のではなく、`getStaticProps`内に直接サーバーサイドコードを書くことができることを意味します。

以下の例を見てみましょう。APIルートを使用してCMSからデータを取得し、そのAPIルートを`getStaticProps`から直接呼び出しています。これにより追加の呼び出しが発生し、パフォーマンスが低下します。代わりに、CMSからのデータ取得ロジックを`lib/`ディレクトリを使用して共有できます。その後、`getStaticProps`と共有できます。

```javascript
// lib/load-posts.js
// 以下の関数は`lib/`ディレクトリから
// getStaticPropsとAPIルートで共有されます
export async function loadPosts() {
  // 投稿を取得するために外部APIエンドポイントを呼び出します
  const res = await fetch('https://.../posts/')
  const data = await res.json()
 
  return data
}
```

```javascript
// pages/blog.js
import { loadPosts } from '../lib/load-posts'
 
// この関数はサーバーサイドでのみ実行されます
export async function getStaticProps() {
  // `/api`ルートを呼び出す代わりに、
  // `getStaticProps`内で同じ関数を直接呼び出すことができます
  const posts = await loadPosts()
 
  // 返されたpropsはページコンポーネントに渡されます
  return { props: { posts } }
}
```

また、データ取得にAPIルートを使用**していない**場合は、`getStaticProps`内で直接fetch() APIを使用してデータを取得することができます。

Next.jsがクライアントサイドバンドルから何を削除しているかを確認するには、next-code-eliminationツールを使用できます。

## HTMLとJSONの両方を静的に生成

`getStaticProps`を持つページがビルド時に事前レンダリングされると、ページのHTMLファイルに加えて、`getStaticProps`の実行結果を保持するJSONファイルもNext.jsが生成します。

このJSONファイルは、next/linkやnext/routerを通じたクライアントサイドルーティングで使用されます。`getStaticProps`を使用して事前レンダリングされたページに移動すると、Next.jsはこのJSONファイル（ビルド時に事前計算）を取得し、それをページコンポーネントのpropsとして使用します。これは、クライアントサイドのページ遷移では`getStaticProps`が呼び出されないことを意味します。エクスポートされたJSONのみが使用されるためです。

Incremental Static Generationを使用する場合、`getStaticProps`はクライアントサイドナビゲーションに必要なJSONを生成するためにバックグラウンドで実行されます。同じページに対して複数のリクエストが行われているように見える場合がありますが、これは意図された動作であり、エンドユーザーのパフォーマンスには影響しません。

## getStaticPropsはどこで使用できるか

`getStaticProps`は**ページ**からのみエクスポートできます。非ページファイル、`_app`、`_document`、または`_error`からはエクスポートできません。

この制限の理由の1つは、Reactがページをレンダリングする前にすべての必要なデータを持っている必要があるためです。

また、`getStaticProps`は独立した関数としてエクスポートする必要があります — ページコンポーネントのプロパティとして`getStaticProps`を追加しても**機能しません**。

> **知っておくと良いこと**：カスタムアプリを作成した場合、リンクされたドキュメントに示されているように`pageProps`をページコンポーネントに渡していることを確認してください。そうしないと、propsが空になります。

## 開発時は毎回のリクエストで実行される

開発時（`next dev`）では、`getStaticProps`は毎回のリクエストで呼び出されます。

## プレビューモード

**プレビューモード**を使用すると、ビルド時ではなく**リクエスト時**にページをレンダリングすることで、一時的に静的生成をバイパスすることができます。例えば、ヘッドレスCMSを使用していて、公開前に下書きをプレビューしたい場合に使用できます。
