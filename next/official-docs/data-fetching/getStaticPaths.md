# getStaticPaths

動的ルートを使用し、`getStaticProps`を使用するページでは、静的に生成するパスのリストを定義する必要があります。

動的ルートを使用するページから`getStaticPaths`（Static Site Generation）関数をエクスポートすると、Next.jsは`getStaticPaths`で指定されたすべてのパスを静的にプリレンダリングします。

```typescript
// pages/repo/[name].tsx
import type {
  InferGetStaticPropsType,
  GetStaticProps,
  GetStaticPaths,
} from 'next'
 
type Repo = {
  name: string
  stargazers_count: number
}
 
export const getStaticPaths = (async () => {
  return {
    paths: [
      {
        params: {
          name: 'next.js',
        },
      }, // 「paths」セクションを参照
    ],
    fallback: true, // false または "blocking"
  }
}) satisfies GetStaticPaths
 
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

`getStaticPaths`のAPIリファレンスでは、`getStaticPaths`で使用できるすべてのパラメータとプロパティをカバーしています。

## getStaticPathsはいつ使用すべきか？

以下の場合に`getStaticPaths`を使用すべきです：

- ヘッドレスCMSからのデータ
- データベースからのデータ
- ファイルシステムからのデータ
- パブリックにキャッシュできるデータ（ユーザー固有でない）
- SEOのためにプリレンダリングが必要で、非常に高速である必要があるページ — `getStaticProps`は`HTML`と`JSON`ファイルを生成し、両方ともCDNでキャッシュしてパフォーマンスを向上させることができます

## getStaticPathsはいつ実行されるか

`getStaticPaths`は本番環境ではビルド時にのみ実行され、実行時には呼び出されません。このツールを使用して、`getStaticPaths`内に書かれたコードがクライアントサイドバンドルから削除されていることを確認できます。

### getStaticPathsに関連してgetStaticPropsはどのように実行されるか

- `getStaticProps`は`next build`時に、ビルド時に返された`paths`に対して実行されます
- `fallback: true`を使用している場合、`getStaticProps`はバックグラウンドで実行されます
- `fallback: blocking`を使用している場合、`getStaticProps`は初期レンダリング前に呼び出されます

## getStaticPathsはどこで使用できるか

- `getStaticPaths`は**必ず**`getStaticProps`と一緒に使用する必要があります
- `getStaticPaths`を`getServerSideProps`と一緒に使用することは**できません**
- `getStaticProps`を使用する動的ルートから`getStaticPaths`をエクスポートできます
- ページ以外のファイル（例：`components`フォルダ）から`getStaticPaths`をエクスポートすることは**できません**
- `getStaticPaths`はページコンポーネントのプロパティではなく、スタンドアロン関数としてエクスポートする必要があります

## 開発環境では毎回のリクエストで実行される

開発環境（`next dev`）では、`getStaticPaths`は毎回のリクエストで呼び出されます。

## オンデマンドでのパス生成

`getStaticPaths`を使用すると、ビルド時にどのページを生成するかを制御できます。ビルド時に生成するページが多いほど、ビルドが遅くなります。

`paths`に空の配列を返すことで、すべてのページの生成をオンデマンドに延期することができます。これは特にNext.jsアプリケーションを複数の環境にデプロイする場合に役立ちます。例えば、プレビュー環境ではすべてのページをオンデマンドで生成し（本番ビルドは除く）、ビルドを高速化することができます。これは数百/数千の静的ページを持つサイトに特に有効です。

```typescript
// pages/posts/[id].js
export async function getStaticPaths() {
  // プレビュー環境では（この値がtrueの場合）
  // 静的ページをプリレンダリングしない
  // （ビルドは高速だが、初期ページロードは遅い）
  if (process.env.SKIP_BUILD_STATIC_GENERATION) {
    return {
      paths: [],
      fallback: 'blocking',
    }
  }
 
  // 外部APIエンドポイントを呼び出して投稿を取得
  const res = await fetch('https://.../posts')
  const posts = await res.json()
 
  // 投稿に基づいてプリレンダリングするパスを取得
  // 本番環境では、すべてのページをプリレンダリング
  // （ビルドは遅いが、初期ページロードは高速）
  const paths = posts.map((post) => ({
    params: { id: post.id },
  }))
 
  // { fallback: false } は他のルートが404になることを意味します
  return { paths, fallback: false }
}
```
