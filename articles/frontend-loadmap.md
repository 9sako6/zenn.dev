---
title: "バックエンドエンジニアが Next.js でフロントエンド開発を始めるにあたり学習したこと"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['frontend']
published: false
---

2021年8月ごろ、副業のような形で Next.js によるフロントエンド開発のお手伝いをさせていただくことになりました。
ひとまずフロントエンド開発ができるようになってきた気がするので、これまで学んできたことをご紹介します。

基本の TypeScript, React, Next.js だけでなく、GraphQL の周辺ツールやテストについても学習しました。

## これまで

当時、Web 系の受託開発会社にて主に Ruby on Rails でバックエンドの開発をしていました。
TypeScript, React は学生の頃から趣味で書いていました。
テストは、Rails での開発なら RSpec や Capybara で書いていましたが、JS ではほぼやったことがありませんでした。
GraphQL は全くの未経験でした。

## やったこと

### テスト

### 素振りする

ヘッドレス CMS の Contentful をバックエンドに使いました。
Contentful の GraphQL API から Apolloc Client をつかって記事を取得する構成にしました。

## これから

パフォーマンスに関する知識をもっとつけたいです。

具体的には、React で再レンダリングを最小限にする実装であったり、CDN の活用であったりです。
