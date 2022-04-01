---
title: "Testing SSR Next.js application with Cypress and MSW" # 記事のタイトル
emoji: "🍏" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["nextjs", "cypress", "msw", "test"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

`getServerSideProps()` で SSR する場合、`cy.intercept()` でスタブを用意することができません。
`cy.intercept()` で作れるスタブは、ブラウザからのリクエスト用だけです。`getServerSideProps()` は Node
プロセスで実行されるため、そもそも傍受することができません。

本記事では、Cypress でテストごとにハンドラを登録し、SSR のためのスタブを用意する 1 つの方法を提案します。
（あまり情報がないため、他の皆さんはどうやってテストしているか気になります。）

なお、Cypress 公式が SSR 用のスタブを作る方法を開発していたようですが、2022 年 3 月 12 日現在は半年ほどメンテされておらず、npm
パッケージとしても未公開の状態なので今回は使用を見送りました。

https://github.com/cypress-io/cypress-mock-ssr

# 環境

```
$ yarn run next -v
Next.js v11.0.1
$ yarn run cypress -v
Cypress package version: 9.0.0
Cypress binary version: 9.0.0
Electron version: 15.2.0
Bundled Node version: 16.5.0
$ yarn run msw --version
0.36.3
```

# 方針

`getServerSideProps()` へのスタブを用意するためには、以下の 2 つを行います。

1. Node プロセスで MSW を起動する
2. Node プロセスの MSW サーバーにハンドラを渡す

## 1. Node プロセスで MSW を起動する

`getServerSideProps()` 用のスタブを用意するためには、ブラウザの外で動く Node.js のプロセスから MSW
を起動する必要があります。 Cypress
では、[Plugin](https://docs.cypress.io/guides/tooling/plugins-guide) を使って Node.js
のプロセスからプログラムを実行できます。

> Plugins enable you to tap into the Node process running outside of the
> browser.

Plugin は plugins ディレクトリ以下に実装します。 初期状態では、およそ以下のような空っぽの plugins/index.js
が用意されています。なお、本記事では筆者が TypeScript で書き直した plugins/index.ts を例として使います。

```typescript
/// <reference types="cypress" />

/**
 * @type {Cypress.PluginConfig}
 */
module.exports = async (
  on: Cypress.PluginEvents,
  config: Cypress.PluginConfigOptions,
) => {
  // `on` is used to hook into various events Cypress emits
  // `config` is the resolved Cypress config
};
```

plugin/index.ts は Cypress の起動と共に実行されます。 下記のようにして MSW サーバーを起動します。

```typescript
/// <reference types="cypress" />
import { setupServer } from "msw/node";

/**
 * @type {Cypress.PluginConfig}
 */
module.exports = async (
  on: Cypress.PluginEvents,
  config: Cypress.PluginConfigOptions,
) => {
  // `on` is used to hook into various events Cypress emits
  // `config` is the resolved Cypress config
  const mswServer = setupServer();
  mswServer.listen();
  console.log("> MSW server is listening");
};
```

## 2. Node プロセスの MSW サーバーにハンドラを渡す

Cypress の `cy.task()` を使うと、イベントを通してテストコードから Plugin（Node プロセス）へ値を渡すことができます。

簡単な例を示します。`cy.task('greet', message)` は、Plugin で登録された `greet`
イベントハンドラを呼び出し、`message` を引数として渡します。

```typescript
// in spec
it("is a sample test", () => {
  const message = "hello";
  cy.task("greet", message);
});
```

```typescript
// plugins/index.ts
module.exports = async (
  on: Cypress.PluginEvents,
  config: Cypress.PluginConfigOptions,
) => {
  on("task", {
    greet(message: string) {
      console.log(message);
      return null; // null を返すと、イベントが正常に処理されたことになります。
    },
  });
};
```

MSW のハンドラを登録するイベントを定義することにより、テストコードから任意のリクエストへスタブを用意できるようになります。MSW
サーバーについては、spec が実行される前に起動し、終わったら停止するように書き換えました。

```typescript
/// <reference types="cypress" />
import { setupServer } from "msw/node";

/**
 * @type {Cypress.PluginConfig}
 */
module.exports = async (
  on: Cypress.PluginEvents,
  config: Cypress.PluginConfigOptions,
) => {
  // `on` is used to hook into various events Cypress emits
  // `config` is the resolved Cypress config

  on("before:spec", () => {
    mswServer.listen();
    console.log("> MSW server is listening");
  });

  on("after:spec", () => {
    mswServer.close();
    console.log("> MSW server is closed");
  });

  on("task", {
    "msw:reset:handlers": () => {
      mswServer.resetHandlers();
      return null;
    },
    "msw:set:query:handlers": (
      queries: {
        name: string;
        payload: Record<string, any>;
      }[],
    ) => {
      // NOTE: `qraphql.query()` によるハンドラの生成をテストコード内で行うと傍受できない。
      // 生成された `GraphQLHandler` の `ctx`, `resolver` プロパティが空になってしまう。
      const handlers = queries.map(({ name, payload }) =>
        graphql.query(name, (_req, res, ctx) => {
          return res(ctx.data(payload));
        })
      );
      mswServer.use(...handlers);
      return null;
    },
  });
};
```

（上記のコードで動作するのですが、コメントにある通り筆者が解決できなかった問題があります。 `graphql.query()`
をテストコードから呼び出した場合、リクエストを捕まえられないのです。 この点について情報をお持ちの方がいたら教えてください。）

以上の Plugin を実装することで、テストコードからハンドラを登録できるようになります。

```typescript
describe('sample test', () => {
  afterEach(() => {
    cy.task('msw:reset:handlers');
  });

  it('should list all articles', () => {
    const payload = {
      __typename: 'EnumArticles'
      items: [
        // some articles...
      ]
    };

    cy.task('msw:set:query:handlers', [{ name: 'enumArticles', payload }]);

    cy.visit('/');
    // some tests...
  });
});
```

# 今後の課題

テストコード内にて `qraphql.query()` でハンドラを定義できず `payload`
しか渡せない点を解決したいです。フィードバックお待ちしております。

# 参考

1. https://glebbahmutov.com/blog/mock-network-from-server/
2. https://docs.cypress.io/guides/tooling/plugins-guide
3. https://docs.cypress.io/api/plugins/writing-a-plugin
4. https://docs.cypress.io/api/commands/task
