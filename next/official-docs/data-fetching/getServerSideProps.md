# getServerSideProps

`getServerSideProps`は、リクエスト時にデータを取得し、ページのコンテンツをレンダリングするために使用できるNext.jsの関数です。

## 例

`getServerSideProps`は、ページコンポーネントからエクスポートすることで使用できます。以下の例では、`getServerSideProps`でサードパーティAPIからデータを取得し、そのデータをページにpropsとして渡す方法を示しています：

```typescript
// pages/index.tsx
import type { InferGetServerSidePropsType, GetServerSideProps } from 'next'
 
type Repo = {
  name: string
  stargazers_count: number
}
 
export const getServerSideProps = (async () => {
  // 外部APIからデータを取得
  const res = await fetch('https://api.github.com/repos/vercel/next.js')
  const repo: Repo = await res.json()
  // propsを通じてページにデータを渡す
  return { props: { repo } }
}) satisfies GetServerSideProps<{ repo: Repo }>
 
export default function Page({
  repo,
}: InferGetServerSidePropsType<typeof getServerSideProps>) {
  return (
    <main>
      <p>{repo.stargazers_count}</p>
    </main>
  )
}
```

## getServerSidePropsはいつ使用すべきか？

パーソナライズされたユーザーデータや、リクエスト時にのみ知ることができる情報（認証ヘッダーやジオロケーションなど）に依存するページをレンダリングする必要がある場合は、`getServerSideProps`を使用すべきです。

リクエスト時にデータを取得する必要がない場合、またはデータとプリレンダリングされたHTMLをキャッシュしたい場合は、`getStaticProps`の使用をお勧めします。

## 動作

- `getServerSideProps`はサーバーサイドで実行されます
- `getServerSideProps`は**ページ**からのみエクスポートできます
- `getServerSideProps`はJSONを返します
- ユーザーがページにアクセスすると、`getServerSideProps`がリクエスト時にデータを取得し、そのデータを使用してページの初期HTMLがレンダリングされます
- ページコンポーネントに渡される`props`は、初期HTMLの一部としてクライアントで表示できます。これはページを正しくハイドレーションするためです。クライアントで利用可能であってはならない機密情報を`props`に渡さないように注意してください
- ユーザーが`next/link`や`next/router`を通じてページにアクセスすると、Next.jsはサーバーにAPIリクエストを送信し、`getServerSideProps`が実行されます
- `getServerSideProps`はサーバーで実行されるため、データを取得するためにNext.jsのAPIルートを呼び出す必要はありません。代わりに、CMS、データベース、その他のサードパーティAPIを直接`getServerSideProps`内から呼び出すことができます

> **知っておくと良いこと：**
> 
> - `getServerSideProps`で使用できるパラメータとpropsについては、getServerSideProps APIリファレンスを参照してください
> - next-code-eliminationツールを使用して、Next.jsがクライアントサイドバンドルから何を削除しているかを確認できます

## エラーハンドリング

`getServerSideProps`内でエラーがスローされた場合、`pages/500.js`ファイルが表示されます。500ページの作成方法については、500ページのドキュメントをご覧ください。開発中は、このファイルは使用されず、代わりに開発用のエラーオーバーレイが表示されます。

## エッジケース

### サーバーサイドレンダリング（SSR）でのキャッシュ

`getServerSideProps`内でキャッシュヘッダー（`Cache-Control`）を使用して、動的レスポンスをキャッシュすることができます。例えば、stale-while-revalidateを使用する場合：

```typescript
// この値は10秒間新鮮とみなされます（s-maxage=10）
// 次の10秒以内にリクエストが繰り返された場合、以前にキャッシュされた値がまだ新鮮な状態で使用されます
// 59秒以内にリクエストが繰り返された場合、キャッシュされた値は古くなっていますが、まだレンダリングされます（stale-while-revalidate=59）
//
// バックグラウンドで、キャッシュを新しい値で更新するための再検証リクエストが行われます
// ページを更新すると、新しい値が表示されます
export async function getServerSideProps({ req, res }) {
  res.setHeader(
    'Cache-Control',
    'public, s-maxage=10, stale-while-revalidate=59'
  )
 
  return {
    props: {},
  }
}
```

ただし、`cache-control`を使用する前に、`getStaticProps`とISRがユースケースにより適しているかどうかを検討することをお勧めします。
