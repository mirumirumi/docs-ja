<script setup>
import TestingApiSwitcher from './TestingApiSwitcher.vue'
</script>

# テスト {#testing}

## なぜテストをするのか？ {#why-test}

自動テストは、アプリケーションをテスト可能な関数、モジュール、クラス、およびコンポーネントに分解することを奨励することによって、手戻りを防ぎ、あなたとチームが複雑な Vue アプリケーションを迅速かつ自信を持って構築するのを助けてくれます。他のアプリケーションと同様、新しい Vue アプリケーションはさまざまなことで壊れる可能性があるため、リリース前に問題をキャッチして修正可能であることが重要になります。

このガイドでは、基本的な用語解説と、Vue 3 アプリケーションにどのようなツールを選択すべきかについて、私たちのおすすめの方法を紹介します。

コンポーザブル関数をカバーする Vue 固有のセクションが 1 つあります。詳しくは、以下の [コンポーザブル関数のテスト](#testing-composables)を参照してください。

## いつテストをするか？ {#when-to-test}

早くテストを始めましょう！私たちは、できる限り早くテストを書き始めることをおすすめします。アプリケーションにテストを追加するのが遅れれば遅れるほど、 アプリケーションの依存性が高まり、始めるのが困難になります。

## テストの種類 {#testing-types}

Vue アプリケーションのテスト戦略を設計する際には、以下のような種類のテストを活用する必要があります:

- **ユニット**: 渡された関数、クラス、あるいはコンポーザブル関数への入力が、期待される出力や副作用を作っているかどうかをチェックします。
- **コンポーネント**: コンポーネントのマウント、レンダリング、インタラクティブであるか、期待通りのふるまいであるかをチェックします。これらのテストは単体テストよりも多くのコードをインポートし、より複雑で、実行に多くの時間を必要とします。
- **エンドツーエンド**: 複数のページにまたがる機能や、実際のネットワークリクエストを、実用環境と同等に構築された Vue アプリケーションに対してチェックします。これらのテストでは、データベースや他のバックエンドを立ち上げることがよくあります。

アプリケーションのテスト戦略においては各テストが種別ごとの役割を果たし、それぞれ異なるタイプの問題からあなたを守ってくれます。

## 概要 {#overview}

各テストがどのようなものか、Vue アプリケーションにどのように実装できるかを簡単に説明し、一般的な推奨事項を説明します。

## 単体テスト {#unit-testing}

単体テストは、小さく分離したコードユニットが期待通りに動作していることを検証するために書かれます。単体テストは通常、単一の関数、クラス、コンポーザブル、またはモジュールをカバーします。単体テストは、論理的な正しさに焦点を当て、アプリケーション全体のごく一部の機能にしか関心を持ちません。単体テストは、アプリケーションの環境の大部分 (たとえば、初期状態、複雑なクラス、サードパーティーモジュール、ネットワークリクエストなど) をモックすることがあります。

一般に、単体テストは関数のビジネスロジックや論理的な正しさに関する問題を検出します。

この `increment` 関数を例にしてみます:

```js
// helpers.js
export function increment (current, max = 10) {
  if (current < max) {
    return current + 1
  }
  return current
}
```

この関数は内容が自己完結しているので、インクリメント関数を呼び出して、それが想定しているものを返すかどうかアサーションするのは簡単でしょう。さっそく単体テストを書いてみます。

これらのアサーションのいずれかが失敗した場合、問題が `increment` 関数の中にあることは明らかです。

```js{4-16}
// helpers.spec.js
import { increment } from './helpers'

describe('increment', () => {
  test('increments the current number by 1', () => {
    expect(increment(0, 10)).toBe(1)
  })

  test('does not increment the current number over the max', () => {
    expect(increment(10, 10)).toBe(10)
  })

  test('has a default max of 10', () => {
    expect(increment(10)).toBe(10)
  })
})
```

先ほど書いた通り、単体テストは通常、UI レンダリングやネットワークリクエストや他の環境に関係しない、自己完結型のビジネスロジック、コンポーネント、クラス、モジュール、もしくは関数に適用されます。

これらは通常、Vue とは関係のないプレーンな JavaScript / TypeScript モジュールです。一般的に言って、Vue アプリケーションのビジネスロジックの単体テストを書くことは、他のフレームワークを使用したアプリケーションと大きく異なるものではありません。

Vue 固有の機能を単体テストする場合、2 つのケースがあります:

1. コンポーザブル
2. コンポーネント

### コンポーザブル {#composables}

Vue アプリケーションに特有の関数のカテゴリーの 1 つに[コンポーザブル](/guide/reusability/composables.html)があり、こちらはテスト時に特別な処理を必要とする場合があります。
詳細は、以下の[Composablesのテスト](#testing-composables)を参照してください。

### コンポーネントの単体テスト {#unit-testing-components}

コンポーネントは 2 つの方法でテストできます:

1. ホワイトボックス: 単体テスト

   "ホワイトボックステスト"は、コンポーネントの実装の詳細や依存関係を認識した上でテストを行います。このテストは、テスト対象のコンポーネントを **分離** することに重点を置いています。これらのテストでは通常、コンポーネントの子要素のいくつかをモックしたり、プラグインの状態や依存関係（例: Vuex）を設定したりすることになります。

2. ブラックボックス: コンポーネントのテスト

   "ブラックボックステスト"は、コンポーネントの実装の詳細については意識しません。これらのテストは、コンポーネントとシステム全体の統合をテストするために、できるだけモックを少なくします。通常、子コンポーネントをすべてレンダリングするため、より"統合テスト"に近いものと考えられています。以下、[コンポーネントテストの推奨事項](#component-testing)を参照してください。

### 推奨事項 {#recommendation}

- [Vitest](https://vitest.dev/)

  `create-vue` で作成される公式の構成は [Vite](https://vitejs.dev/) をベースにしているので、Vite と同じ設定・変換パイプラインを直接利用できるユニットテストフレームワークを使用することをお勧めします。[Vitest](https://vitest.dev/) は、この目的のために特別に設計されたユニットテストフレームワークで、Vue / Vite チームのメンバーによって開発、メンテナンスされています。Vite ベースのプロジェクトと最小限の労力で統合でき、非常に高速です。

### その他の選択肢 {#other-options}

- [Peeky](https://peeky.dev/) は最も Vite と統合されている、もう 1 つの高速単体テストランナーです。こちらも Vue のコアチームのメンバーによって作成されていて、GUI ベースのテストインターフェイスを提供します。

- [Jest](https://jestjs.io/) はよく知られたユニットテストフレームワークで、[vite-jest](https://github.com/sodatea/vite-jest) パッケージによって Vite で動作させることができます。ですが、Jest を推奨するのは既存の Jest テストスイートを Vite ベースのプロジェクトに移行する必要がある場合のみです。Vitest はよりシームレスな統合と優れたパフォーマンスを提供するからです。

## コンポーネントのテスト {#component-testing}

Vue アプリケーションでは、コンポーネントは UI の主要なビルディングブロックです。したがって、アプリケーションの動作を検証する時の分離の単位として、コンポーネントは自然な単位といえます。粒度の観点からは、コンポーネントテストは単体テストの上のどこかに位置し、かつ統合テストの一形態とも考えることができます。Vue アプリケーションの多くはコンポーネントテストでカバーされるべきであり、各 Vue コンポーネントは専用の spec ファイルを持つようにすることをお勧めします。

コンポーネントテストは、コンポーネントのプロパティ、イベント、提供するスロット、スタイル、クラス、ライフサイクルフックなどに関する問題を検出するようにする必要があります。

コンポーネントテストでは、子コンポーネントをモックするのではなく、代わりにユーザーと同じようにコンポーネントと相互的な操作を行い、コンポーネントとその子コンポーネントの間のインタラクションをテストするのが望ましいです。例えば、コンポーネントテストでは、プログラムでコンポーネントとやりとりするのではなく、ユーザーが行うように要素をクリックするようにしなければなりません。

コンポーネントのテストでは、内部実装の詳細よりもそのコンポーネントのパブリックなインターフェイスに注目する必要があります。ほとんどのコンポーネントでは、公開インターフェースは発行されるイベント、プロパティ、スロットに限定されています。テストする際は**コンポーネントが何をするのかをテストするのであって、どのようにするのかをテストするのではない** ことを忘れないようにしましょう。

**やること**

- **ビジュアル** のロジック: 入力されたプロパティとスロットに基づくレンダリング結果の正しさを評価します。
- **ふるまい** のロジック: ユーザーの入力イベントに対応した更新内容のレンダリングやイベント発行を評価します。

  以下の例では、"increment" というラベルの付いた DOM 要素を持ち、クリックすることができるステッパー (Stepper: 値が増えていく) コンポーネントを示しています。`max` というプロパティを渡すことで、ステッパーが `2` を超えてインクリメントすることを防ぎます。したがって、ボタンを 3 回クリックしても、UI は `2` と表示されるはずです。

  私たちはステッパーの実装について何も知りません。ただ、"入力"が `max` プロパティで、"出力"がユーザーが見る DOM の状態であることだけが分かっています。

<TestingApiSwitcher>

<div class="testing-library-api">

```js
const { getByText } = render(Stepper, {
  props: {
    max: 1
  }
})

getByText('0') // コンポーネント内に "0 "があることを暗黙のうちに評価します

const button = getByText('increment')

// インクリメントボタンにクリックイベントをディスパッチします。
await fireEvent.click(button)

getByText('1')

await fireEvent.click(button)
```

</div>

<div class="vtu-api">

```js
const valueSelector = '[data-testid=stepper-value]'
const buttonSelector = '[data-testid=increment]'

const wrapper = mount(Stepper, {
  props: {
    max: 1
  }
})

expect(wrapper.find(valueSelector).text()).toContain('0')

await wrapper.find(buttonSelector).trigger('click')

expect(wrapper.find(valueSelector).text()).toContain('1')
```

</div>

<div class="cypress-api">

```js
const valueSelector = '[data-testid=stepper-value]'
const buttonSelector = '[data-testid=increment]'

mount(Stepper, {
  props: {
    max: 1
  }
})

cy.get(valueSelector).should('be.visible').and('contain.text', '0')
  .get(buttonSelector).click()
  .get(valueSelector).should('contain.text', '1')
```

</div>

</TestingApiSwitcher>

- **やってはいけないこと**

  コンポーネントインスタンスのプライベートな状態をアサーションしたり、プライベートなメソッドをテストしないでください。実装の詳細をテストするとテストが壊れやすくなり、実装が変更されたときにテストの更新が必要になる可能性が高くなります。

  コンポーネントの最終的な仕事は正しい DOM をレンダリングすることです。DOM 出力に焦点を当てたテストは同じレベルの正しさを保証すると同時に、（少なくとも）より堅牢で変化に強くなります。

  スナップショットテストだけに頼らないでください。HTML 文字列の評価は正しさを表すものではありません。意図を持ってテストを書いてください。

  もしメソッドを徹底的にテストする必要があるなら、スタンドアローンのユーティリティー関数に抽出し、専用の単体テストを書くことを検討してください。きれいに抽出できない場合は、それをカバーするコンポーネントテスト、統合テスト、またはエンドツーエンドテストの一部としてテストすることができます。

### 推奨事項 {#recommendation-1}

- [Vitest](https://vitest.dev/) はヘッドレスでレンダリングするコンポーネントやコンポーザブルのためのライブラリーです（例: VueUse の[`useFavicon`](https://vueuse.org/core/useFavicon/#usefavicon)関数など）。コンポーネントと DOM は [@testing-library/vue](https://testing-library.com/docs/vue-testing-library/intro) を使うことでテストできます。

- [Cypress Component Testing](https://on.cypress.io/component) はテストで期待する動作が、スタイルの適切なレンダリングやネイティブ DOM イベントのトリガーに依存するようなコンポーネントのためのライブラリーです。テストライブラリーは、[@testing-library/cypress](https://testing-library.com/docs/cypress-testing-library/intro) を介して使用することができます。

Vitest とブラウザーベースのランナーの主な違いは、スピードと実行コンテキストです。つまり、Cypress のようなブラウザーベースのランナーは Vitest のようなノードベースのランナーでは捕捉できない問題（例: スタイルの問題、本物のネイティブ DOM イベント、クッキー、ローカルストレージ、ネットワーク障害など）をキャッチすることができます。ですが、ブラウザーベースのランナーは、ブラウザーを開き、スタイルシートをコンパイルなどをするので *Vitest より桁違いに遅い* のです。Cypress はコンポーネントテストをサポートするブラウザーベースのランナーです。Vitest と Cypress の比較に関する最新情報は [Vitest の比較ページ](https://vitest.dev/guide/comparisons.html#cypress)をお読みください。

### ライブラリーのマウント {#mounting-libraries}

コンポーネントのテストでは、テスト対象のコンポーネントを単独でマウントし、シミュレーションされたユーザー入力イベントをトリガーし、レンダリングした DOM に対してアサーションを行うことがよくあります。これらの作業を簡単にする専用のユーティリティーライブラリーがあります。

- [`@testing-library/vue`](https://github.com/testing-library/vue-testing-library) は、実装の詳細に依存しないコンポーネントのテストに重点を置いた Vue のテストライブラリーです。アクセシビリティーを念頭に置いて構築されており、そのアプローチによりリファクタリングも容易になります。このガイドの指針は、テストがソフトウェアの使われ方に似ていればいるほど、より信頼性を高めることができるというものです。

- [`@vue/test-utils`](https://github.com/vuejs/test-utils) は、Vue 固有の API へのアクセスをユーザーに提供するために書かれた、公式のローレベルコンポーネントテストライブラリーです。`@testing-library/vue` はこのライブラリー上に構築されているローレベルのライブラリーでもあります。

アプリケーションのテストの優先順位と一致するため、アプリケーションのコンポーネントテストには `@testing-library/vue` を使用することをお勧めします。Vue 固有の内部機能をテストする必要があるような、高度なコンポーネントを構築する場合にのみ `@vue/test-utils` を使用してください。

### その他の選択肢 {#other-options-1}

- [Nightwatch](https://v2.nightwatchjs.org/) は、Vue コンポーネントテストをサポートする E2E テストランナーです。(Nightwatch バージョン 2 の[プロジェクトの例](https://github.com/nightwatchjs-community/todo-vue)）

## E2E テスト {#e2e-testing}

単体テストは開発者にある程度の信頼性を与えますが、単体テストやコンポーネントテストは、本番環境にデプロイしたときにアプリケーションの総合的なカバレッジを提供する性能が制限されます。その結果として、E2E（エンドツーエンド）テストは、アプリケーションの最も重要な側面に関するカバレッジを提供します: すなわち、ユーザーが実際にアプリケーションを使用するときに何が起こるか、です。

エンドツーエンドテストは、本番と同等に構築された Vue アプリケーションに対してネットワークリクエストを行う、複数ページのアプリケーションの動作に焦点を当てます。多くの場合、データベースやその他のバックエンドを立ち上げる必要があり、稼働中のステージング環境に対して実行されることもあります。

エンドツーエンドテストは、ルーター、状態管理ライブラリー、トップレベルのコンポーネント（例 : App や Layout）、パブリックアセット、またはリクエスト処理に関する問題をしばしば捕捉します。上記のように、単体テストやコンポーネントテストではできない重要な問題をキャッチすることができます。

エンドツーエンドテストは Vue アプリケーションのコードを一切インポートせず、代わりに実際のブラウザーでページ全体を操作してアプリケーションをテストすることに完全に依存しています。

エンドツーエンドテストは、アプリケーションの多くのレイヤーを検証します。ローカルでビルドしたアプリケーションを対象とすることも、稼働中のステージング環境を対象とすることもできます。ステージング環境に対するテストは、フロントエンドのコードと静的サーバーだけでなく、関連するすべてのバックエンドのサービスとインフラストラクチャーもテスト対象に含みます。

> テストがソフトウェアの使われ方に似ていればいるほど、テストはより信頼できるものになる。- [Kent C. Dodds](https://twitter.com/kentcdodds/status/977018512689455106) - テスティングライブラリーの著者

ユーザーの行動がアプリケーションにどのような影響を与えるかをテストすることで、E2E テストは、アプリケーションが適切に機能しているかどうかの信頼性を高める鍵となることが多いです。

### E2E テストソリューションの選択 {#choosing-an-e2e-testing-solution}

ウェブ上のエンドツーエンド (E2E) テストは、信頼性の低い（脆弱な）テストで開発プロセスの遅延を起こす、というネガティブな評判があります。ですが、最近の E2E ツールは、より信頼性が高く、インタラクティブで、役に立つテストを作成できるよう発展してきています。E2E テストフレームワークを選択するときのため、以下のセクションでは、アプリケーションのためのテストフレームワーク選定にあたって心に留めておくべきガイドラインを記載します。

#### クロスブラウザーテスト {#cross-browser-testing}

エンドツーエンド（E2E）テストで知られている最も大きな利点の 1 つは、複数のブラウザーにわたってアプリケーションをテストできることです。100% のクロスブラウザーカバレッジが望ましいと思われるかもしれませんが、クロスブラウザーテストを一貫して実行するために必要な時間とマシンパワーが増えるため、チームのリソースに対するリターンが減る点に注意することが重要です。そのため、アプリケーションに必要なクロスブラウザーテストの量を選択する際には、このトレードオフを意識することが重要です。

#### フィードバックループの高速化 {#faster-feedback-loops}

エンドツーエンド（E2E）テストと開発における主な問題の 1 つは、スイート全体を実行するのに長い時間がかかるということです。一般的に、これは継続的インテグレーションとデプロイメント（CI/CD）パイプラインでのみ実行されます。最新の E2E テストフレームワークは、並列化などの機能を追加することでこの問題を解決し、CI/CD パイプラインを以前より数段速く実行できるようになりました。さらに、ローカルで開発する場合、作業中のページに対して単一のテストを選択的に実行し、テストのホットリロードを提供する機能は、開発者のワークフローと生産性を向上させるのに役立ちます。

#### 一級のデバッグ体験 {#first-class-debugging-experience}

開発者は従来、テストで何が問題だったかを判断するための方法としてターミナルウィンドウのログ調査に頼っていましたが、最新のエンドツーエンド（E2E）テストフレームワークでは、開発者がすでに使い慣れているツール、例えばブラウザーの開発者ツールを活用することができます。

#### ヘッドレスモードでの可視性 {#visibility-in-headless-mode}

エンドツーエンド (E2E) テストが継続的インテグレーション / デプロイメントパイプラインで実行されるとき、しばしばヘッドレスブラウザーで実行されます（すなわち、ユーザーが見るためのブラウザーが開かれません）。最新の E2E テストフレームワークの重要な機能では、テスト中にアプリケーションのスナップショットやビデオを見て、エラーが発生した理由に対する何らかの洞察を得られる機能があります。歴史的に、これらの機能の統合を維持するのは面倒なことでした。

### 推奨事項 {#recommendation-2}

- [Cypress](https://www.cypress.io/)

  全体的に見て Cypress は、有益なグラフィカルインターフェース、優れたデバッグ性、組み込みアサーションとスタブ、耐フレーク性、並列化、スナップショットなどの機能により、最も完成した E2E ソリューションを提供すると私たちは考えています。また、前述の通り[コンポーネントのテスト](https://docs.cypress.io/guides/component-testing/introduction)もサポートしています。ですが、サポートしているのは Chromium ベースのブラウザーと Firefox のみです。

### その他の選択肢 {#other-options-2}

- [Playwright](https://playwright.dev/) は、より幅広いブラウザー（主に WebKit）をサポートする、優れた E2E テストソリューションでもあります。詳しくは[なぜ Playwright なのか](https://playwright.dev/docs/why-playwright)をご覧ください。

- [Nightwatch v2](https://v2.nightwatchjs.org/) は、[Selenium WebDriver](https://www.npmjs.com/package/selenium-webdriver) をベースとした E2E テストソリューションです。こちらは最も広い範囲のブラウザーをサポートしています。

## レシピ {#recipes}

### Vitest プロジェクトに追加する {#adding-vitest-to-a-project}

Vite ベースの Vue プロジェクトでは、以下を実行します:

```sh
> npm install -D vitest happy-dom @testing-library/vue
```

次に、Vite の設定を更新して `test` オプションブロックを追加します:

```js{6-12}
// vite.config.js
import { defineConfig } from 'vite'

export default defineConfig({
  // ...
  test: {
    // jest ライクなグローバルテスト API を有効化
    globals: true,
    // happy-dom で DOM をシミュレーションします
    // （peer dependency として happy-dom のインストールが必要です）
    environment: 'happy-dom'
  }
})
```

:::tip
TypeScript を使用している場合は、`tsconfig.json` の `types` フィールドに `vitest/globals` を追加してください。

```json
// tsconfig.json

{
 "compilerOptions": {
    "types": ["vitest/globals"]
  }
}
```
:::

そして、`*.test.js` で終わるファイルをプロジェクト内に作成します。すべてのテストファイルは、プロジェクトのルートにあるテストディレクトリー、またはソースファイルの隣接するテストディレクトリーに置くことができます。Vitest は命名規則を使用して自動的にファイルを検索してくれます。

```js
// MyComponent.test.js
import { render } from '@testing-library/vue'
import MyComponent from './MyComponent.vue'

test('it should work', () => {
  const { getByText } = render(MyComponent, {
    props: {
      /* ... */
    }
  })

  // assert output
  getByText('...')
})
```

最後に、`package.json` を更新して、テストスクリプトを追加し、実行します:

```json{4}
{
  // ...
  "scripts": {
    "test": "vitest"
  }
}
```

```sh
> npm test
```

### コンポーザブルのテスト {#testing-composables}

> このセクションは[コンポーザブル](/guide/reusability/composables.html)セクションを既に読んでいることを想定しています。

コンポーザブルをテストする場合、2 つのカテゴリーに分割することができます: ホストコンポーネントのインスタンスに依存しないコンポーザブルと、依存するコンポーザブルです。

以下の API を使用する場合、コンポーザブルはホストコンポーネントインスタンスに依存します:

- ライフサイクルフック
- Provide / Inject

Reactivity API のみを使用するコンポーザブルの場合、それを直接呼び出して、返された状態やメソッドをアサーションすることでテストすることができます:

```js
// counter.js
import { ref } from 'vue'

export function useCounter() {
  const count = ref(0)
  const increment = () => count.value++

  return {
    count,
    increment
  }
}
```

```js
// counter.test.js
import { useCounter } from './counter.js'

test('useCounter', () => {
  const { count, increment } = useCounter()
  expect(count.value).toBe(0)

  increment()
  expect(count.value).toBe(1)
})
```

ライフサイクルフックや Provide / Inject に依存するコンポーザブルは、テスト対象となるホストコンポーネントでラップする必要があります。以下のようなヘルパーを作成するといいでしょう:

```js
// test-utils.js
import { createApp } from 'vue'

export function withSetup(composable) {
  let result
  const app = createApp({
    setup() {
      result = composable()
      // template の警告を抑えます
      return () => {}
    }
  })
  app.mount(document.createElement('div'))
  // provide / unmount のテストのため
  // result と app インスタンスを返却します
  return [result, app]
}
```
```js
import { withSetup } from './test-utils'
import { useFoo } from './foo'

test('useFoo', () => {
  const [result, app] = withSetup(() => useFoo(123))
  // injection のテストのため provide をモック
  app.provide(...)
  // アサーションを実行
  expect(result.foo.value).toBe(1)
  // 必要に応じて onUnmounted フックを実行します
  app.unmount()
})
```

より複雑なコンポーザブルの場合、[コンポーネントテスト](#component-testing)のテクニックを使ってラッパーコンポーネントに対するテストを書くことで、より簡単にテストすることもできます。

<!--
TODO more testing recipes can be added in the future e.g.
- How to set up CI via GitHub actions
- How to do mocking in component testing
-->
