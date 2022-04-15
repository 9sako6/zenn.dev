---
title: "バックエンドエンジニアが Next.js でモダンなフロントエンド開発を始めるにあたり学習したこと"
emoji: "🧸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['frontend']
published: true
---

2021年秋ごろ、副業のような形で Next.js による新規フロントエンド開発のお手伝いをさせていただくことになりました。プライベートの空き時間でフロントエンドの学習をし、今はひとまず開発できるようになってきた気がするので、これまで学んできたことをご紹介します。

基本の TypeScript, React, Next.js だけでなく、GraphQL の周辺ツールやテストについても学習しました。

## これまで

当時、Web 系の受託開発会社にて主に Ruby on Rails でバックエンドの開発をしていました。TypeScript, React は学生の頃から趣味で書いていました。
テストは、Rails での開発なら RSpec や Capybara で書いていましたが、JS ではほぼやったことがありませんでした。GraphQL は全くの未経験でした。

## やったこと

### React チュートリアル

所要時間: 2時間

- 公式の [チュートリアル：React の導入 – React](https://ja.reactjs.org/tutorial/tutorial.html) で小さいゲームを作りながら基礎を復習しました
- React の状態管理やイベント処理など、基本を学べます

https://ja.reactjs.org/tutorial/tutorial.html

### React 公式ドキュメントを読み込む

所要時間: 1週間

- [Getting Started – React](https://ja.reactjs.org/docs/getting-started.html) の各章を読みました
- 状態管理、イベント処理、コンテクスト、hooks、パフォーマンス最適化、設計原則など、重要な事項が書いてあります
- それなりの分量があります。移動時間等の隙間時間を使ってちょこちょこ読んでいました

https://ja.reactjs.org/docs/hooks-reference.html

### Next.js チュートリアル

所要時間: 半日

- 公式の [Introduction | Learn Next.js](https://nextjs.org/learn/foundations/about-nextjs?utm_source=next-site&utm_medium=nav-cta&utm_campaign=next-website) でブログを作りながら Next.js を学びました
- チュートリアルは英語で書いてありますが、素直な文章なのでそれほど苦戦せず読めました
- Next.js でのアセット管理、レンダリングの仕組み、動的ルーティングなど、基本を学ぶことができます

https://nextjs.org/learn/foundations/about-nextjs?utm_source=next-site&utm_medium=nav-cta&utm_campaign=next-website

### CSS

所要時間: 1日

- CSS は雰囲気で書いていたので、mdn web docs で理解が甘かった箇所を読みました
- [フレックスボックス - ウェブ開発を学ぶ | MDN](https://developer.mozilla.org/ja/docs/Learn/CSS/CSS_layout/Flexbox) や [グリッド - ウェブ開発を学ぶ | MDN](https://developer.mozilla.org/ja/docs/Learn/CSS/CSS_layout/Grids) でレイアウトの基本を学習しました


https://developer.mozilla.org/ja/docs/Learn/CSS/CSS_layout/Grids

### GraphQL

所要時間: 半日

- クライアントサイドで使うのに最低限必要な知識を得るため、公式のガイド [Queries and Mutations | GraphQL](https://graphql.org/learn/queries/), [Schemas and Types | GraphQL](https://graphql.org/learn/schema/) を読みました
- とはいえクエリを書かないことにはなかなか覚えられないし、サーバー側の処理までは追えていないしで、まだまだ勉強不足です

https://graphql.org/learn/queries/

### GraphQL 周辺ツール

所要時間: 1, 2週間

- 実際にプロジェクトに導入しながら必要に応じて学びました。導入過程で想定外にハマったり、新たな疑問が湧いてきたりするので実践は大事ですね
- [Apollo Client](https://www.apollographql.com/docs/react)
  - クライアントサイドで GraphQL API を叩くための hooks を提供したり、状態管理したり、キャッシュ管理したりしてくれるツールです
  - 公式ドキュメント [Introduction to Apollo Client - Apollo GraphQL Docs](https://www.apollographql.com/docs/react) をかいつまんで読みました
  - API の使い方であったり、Cache の仕組み、設定方法などを読みました

https://www.apollographql.com/docs/react

- [GraphQL Code Generator](https://www.graphql-code-generator.com/)
  - GraphQL のスキーマから TypeScript の型を生成するためのライブラリです
  - プラグインを入れれば、Aollo Client で使える `useXXX` hooks も自動生成できます


### テストの種類を知る

- ネットでフロントエンドのテスト戦略について調べました
- visual regression test, E2E test, integration test, unit test のように、テストしたい問題の大きさや内容によっていくつか種類がありました
  - 例えば、Testing Library 作者の Kent C. Dodds さんの下記記事をご覧ください

https://kentcdodds.com/blog/static-vs-unit-vs-integration-vs-e2e-tests


### React Testing Library, Jest

- [Jest](https://jestjs.io/)
  - 公式の [API ドキュメント](https://jestjs.io/ja/docs/api) を読みました
  - 最近は高速で Jest API 互換な [Vitest](https://github.com/vitest-dev/vitest) が気になってます
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro)
  - 公式の [API | Testing Library](https://testing-library.com/docs/react-testing-library/api) を読みました

https://jestjs.io/ja/docs/api

https://testing-library.com/docs/react-testing-library/intro/

### Mock Service Worker

- API をモックするツールで、SSR, CSR どちらでも使えます
- 設定が楽で API はシンプルで使いやすいです
- Apollo にもモック機能はありますが、`MockedProvider` コンテキストが必要だったりでコンポーネントに一手間加える必要があります。MSW は Service Worker 上で動作しており、アプリとは独立した環境にいます。MSW によって API クライアントへの依存性を減らすことができます

https://mswjs.io/docs/

### Cypress

- E2E テスト用のフレームワークです。React Testing Library, Jest ではテストできない箇所、例えばページ遷移が発生するようなテストに使っています
- 公式の短いガイド [Writing Your First Test | Cypress Documentation](https://docs.cypress.io/guides/getting-started/writing-your-first-test#Step-4-Make-an-assertion) を読みました
- その後プロジェクトに導入したり、ハマったりしながら学んでいきました
  - [Testing SSR Next.js application with Cypress and MSW - Zenn](https://zenn.dev/9sako6/articles/testing-ssr-nextjs-with-cypress-and-msw)

https://docs.cypress.io/guides/overview/why-cypress

### Storybook

所要時間: 3時間

- 公式の [How to write stories](https://storybook.js.org/docs/react/writing-stories/introduction) を読んで、ストーリーの書き方を学びました
- visual regression test に使えたり、非エンジニアの方にデザインを共有しやすかったりで心強いツールです

https://storybook.js.org/docs/react/get-started/introduction

### 素振りする

所要時間: 数日

- 学んだ内容をもとに、個人ブログ [9sako6.com](https://9sako6.com/) を Next.js で再構築しました
  - ※ 現在はここに書いてある技術スタックではない可能性があります
- ヘッドレス CMS の Contentful をバックエンドに使いました

## まとめ

私はまだまだ発展途上ですが、辿ってきた足跡がどなたかの参考になると幸いです。特にテストは独学だと疎かになりそうな箇所です。個人的な感覚としては、例えば RSpec などと比べるとまだ情報が少ない気がします。私も何か発見があれば適宜発信していく所存です。

これからは、パフォーマンスに関する知識をもっとつけたいです。具体的には、React で再レンダリングを最小限にする実装であったり、CDN の活用であったりです。もっと DevTools を使い倒したり、GitHub で OSS になっているフロントエンドアプリがあれば実装を読みたいです。
