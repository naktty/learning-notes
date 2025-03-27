# useRouter

`useRouter`フックは、Next.jsのルーティング機能にアクセスするためのReactフックです。

## 基本的な使い方

```typescript
import { useRouter } from 'next/router'
```

## 戻り値

`useRouter`は以下のプロパティとメソッドを持つオブジェクトを返します：

### router.query

クエリパラメータを含むオブジェクトです。デフォルトでは空のオブジェクト（`{}`）です。

```typescript
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()
  const { id } = router.query
  return <p>Post: {id}</p>
}
```

### router.push

指定されたパスに遷移します。

```typescript
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  return (
    <button type="button" onClick={() => router.push('/about')}>
      Click here to read more
    </button>
  )
}
```

### router.back

ブラウザの履歴を1つ戻ります。`window.history.back()`を実行するのと同じです。

```typescript
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  return (
    <button type="button" onClick={() => router.back()}>
      Click here to go back
    </button>
  )
}
```

### router.reload

現在のURLを再読み込みします。ブラウザの更新ボタンをクリックするのと同じです。`window.location.reload()`を実行します。

```typescript
import { useRouter } from 'next/router'

export default function Page() {
  const router = useRouter()

  return (
    <button type="button" onClick={() => router.reload()}>
      Click here to reload
    </button>
  )
}
```

### router.events

Next.jsのルーター内で発生する様々なイベントをリッスンできます。サポートされているイベントは以下の通りです：

- `routeChangeStart(url, { shallow })` - ルートの変更が開始された時に発火
- `routeChangeComplete(url, { shallow })` - ルートの変更が完了した時に発火
- `routeChangeError(err, url, { shallow })` - ルート変更時にエラーが発生した場合、またはルートの読み込みがキャンセルされた場合に発火
  - `err.cancelled` - ナビゲーションがキャンセルされたかどうかを示す
- `beforeHistoryChange(url, { shallow })` - ブラウザの履歴が変更される前に発火
- `hashChangeStart(url, { shallow })` - ハッシュが変更されるが、ページは変更されない場合に発火
- `hashChangeComplete(url, { shallow })` - ハッシュが変更されたが、ページは変更されない場合に発火

> **注意**: ここでの`url`は、basePathを含むブラウザに表示されるURLです。

## next/compat/routerエクスポート

これは同じ`useRouter`フックですが、`app`と`pages`の両方のディレクトリで使用できます。

`next/router`との違いは、pagesルーターがマウントされていない場合にエラーをスローせず、代わりに`NextRouter | null`型を返すことです。これにより、開発者は`app`ルーターへの移行中に両方のディレクトリで動作するコンポーネントに変換できます。

## ESLintの潜在的なエラー

`router`オブジェクトの特定のメソッドはPromiseを返します。ESLintの`no-floating-promises`ルールが有効な場合、グローバルに無効にするか、影響を受ける行で無効にすることを検討してください。

影響を受けるメソッド：
- `router.push`
- `router.replace`
- `router.prefetch`

## withRouter

`useRouter`が最適でない場合、`withRouter`を使用して同じルーターオブジェクトを任意のコンポーネントに追加することもできます。

### 使用例

```typescript
import { withRouter } from 'next/router'

function Page({ router }) {
  return <p>{router.pathname}</p>
}

export default withRouter(Page)
```

### TypeScript

クラスコンポーネントで`withRouter`を使用する場合、コンポーネントは`router`プロパティを受け入れる必要があります：

```typescript
import React from 'react'
import { withRouter, NextRouter } from 'next/router'

interface WithRouterProps {
  router: NextRouter
}

interface MyComponentProps extends WithRouterProps {}

class MyComponent extends React.Component<MyComponentProps> {
  render() {
    return <p>{this.props.router.pathname}</p>
  }
}

export default withRouter(MyComponent)
```
