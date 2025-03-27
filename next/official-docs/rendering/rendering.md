# レンダリング

デフォルトでは、Next.jsはすべてのページを**プリレンダリング**します。これは、Next.jsが各ページのHTMLを事前に生成することを意味し、すべてをクライアントサイドのJavaScriptで行うのではありません。プリレンダリングにより、パフォーマンスとSEOが向上する可能性があります。

生成された各HTMLは、そのページに必要な最小限のJavaScriptコードと関連付けられています。ブラウザでページが読み込まれると、そのJavaScriptコードが実行され、ページが完全にインタラクティブになります（このプロセスはReactではハイドレーションと呼ばれます）。

## プリレンダリング

Next.jsには2つのプリレンダリング形式があります：**静的生成**と**サーバーサイドレンダリング**です。違いは、ページのHTMLを**いつ**生成するかです。

- 静的生成：HTMLは**ビルド時**に生成され、各リクエストで再利用されます。
- サーバーサイドレンダリング：HTMLは**各リクエスト**で生成されます。

重要なのは、Next.jsでは各ページに対して使用したいプリレンダリング形式を選択できることです。ほとんどのページに静的生成を使用し、他のページにサーバーサイドレンダリングを使用することで、「ハイブリッド」なNext.jsアプリを作成できます。

パフォーマンスの理由から、サーバーサイドレンダリングよりも静的生成の使用を推奨します。静的に生成されたページは、追加の設定なしでCDNでキャッシュでき、パフォーマンスを向上させることができます。ただし、場合によってはサーバーサイドレンダリングが唯一の選択肢となることもあります。

また、静的生成やサーバーサイドレンダリングと一緒にクライアントサイドのデータフェッチングを使用することもできます。これは、ページの一部を完全にクライアントサイドのJavaScriptでレンダリングできることを意味します。詳細については、データフェッチングのドキュメントをご覧ください。

## 関連ドキュメント

- [サーバーサイドレンダリング (SSR)](https://nextjs.org/docs/pages/building-your-application/rendering/server-side-rendering) - 各リクエストでページをレンダリングする方法
- [静的サイト生成 (SSG)](https://nextjs.org/docs/pages/building-your-application/rendering/static-site-generation) - ビルド時にページをプリレンダリングする方法
- [自動静的最適化](https://nextjs.org/docs/pages/building-your-application/rendering/automatic-static-optimization) - Next.jsが可能な限り静的HTMLに最適化する方法
- [クライアントサイドレンダリング (CSR)](https://nextjs.org/docs/pages/building-your-application/rendering/client-side-rendering) - Pages Routerでクライアントサイドレンダリングを実装する方法
- [Edge と Node.js ランタイム](https://nextjs.org/docs/pages/building-your-application/rendering/edge-and-nodejs-runtimes) - Next.jsの切り替え可能なランタイム（Edge と Node.js）について
