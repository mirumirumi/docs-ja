---
pageClass: api
---

# ビルトインのコンポーネント {#built-in-components}

:::info 登録と使用
組み込みコンポーネントは登録する必要なくテンプレート内で直接使用できます。ツリーシェイクも可能で、使用されたときだけビルドに含まれます。

[レンダー関数](/guide/extras/render-function.html)で使用する場合は明示的にインポートする必要があります。例えば:

```js
import { h, Transition } from 'vue'

h(Transition, {
  /* プロパティ */
})
```

:::

## `<Transition>` {#transition}

**単一の**要素またはコンポーネントにアニメーションのトランジションを提供します。

- **プロパティ**

  ```ts
  interface TransitionProps {
    /**
     * トランジションの CSS クラス名を自動生成するために使用します。
     * 例: `name: 'fade'` は `.fade-enter` や `.fade-enter-active`
     * などに自動展開されます。
     */
    name?: string
    /**
     * CSS のトランジションクラスを適用するかどうか。
     * デフォルト: true
     */
    css?: boolean
    /**
     * トランジション終了タイミングを決定するために待機する、
     * トランジションイベントの種類を指定します。
     * デフォルトの動作は、持続時間がより長い方のタイプを
     * 自動検出します。
     */
    type?: 'transition' | 'animation'
    /**
     * トランジションの持続時間を明示的に指定します。
     * デフォルトの動作は、ルートトランジション要素の最初の
     * `transitionend` または `animationend` イベントを待ちます。
     */
    duration?: number | { enter: number; leave: number }
    /**
     * leaving/entering トランジションのタイミングシーケンスを制御。
     * デフォルトの動作は同時です。
     */
    mode?: 'in-out' | 'out-in' | 'default'
    /**
     * 初回レンダリング時にトランジションを適用するかどうか。
     * デフォルト: false
     */
    appear?: boolean

    /**
     * トランジションクラスをカスタマイズするためのプロパティ。
     * テンプレートでは kebab-case を使用（例: enter-from-class="xxx"）
     */
    enterFromClass?: string
    enterActiveClass?: string
    enterToClass?: string
    appearFromClass?: string
    appearActiveClass?: string
    appearToClass?: string
    leaveFromClass?: string
    leaveActiveClass?: string
    leaveToClass?: string
  }
  ```

- **イベント**

  - `@before-enter`
  - `@before-leave`
  - `@enter`
  - `@leave`
  - `@appear`
  - `@after-enter`
  - `@after-leave`
  - `@after-appear`
  - `@enter-cancelled`
  - `@leave-cancelled`（`v-show` のみ）
  - `@appear-cancelled`

- **例**

  シンプルな要素:

  ```vue-html
  <Transition>
    <div v-if="ok">toggled content</div>
  </Transition>
  ```

  トランジションモードと出現時のアニメーションを備えているダイナミックコンポーネント:

  ```vue-html
  <Transition name="fade" mode="out-in" appear>
    <component :is="view"></component>
  </Transition>
  ```

  トランジションイベントを購読する:

  ```vue-html
  <Transition @after-enter="onTransitionComplete">
    <div v-show="ok">toggled content</div>
  </Transition>
  ```

- **参照:** [`<Transition>` ガイド](/guide/built-ins/transition.html)

## `<TransitionGroup>` {#transitiongroup}

リスト内の**複数**の要素またはコンポーネントにトランジション効果を提供する。

- **プロパティ**

  `<TransitionGroup>` は `<Transition>` と同じプロパティ（`mode` 以外）と追加の 2 つのプロパティを受け取ります:

  ```ts
  interface TransitionGroupProps extends Omit<TransitionProps, 'mode'> {
    /**
     * 未定義の場合はフラグメントとしてレンダリングされます。
     */
    tag?: string
    /**
     * 移動のトランジション中に適用される CSS クラスのカスタマイズ。
     * テンプレートでは kebab-case を使用（例: move-class="xxx"）
     */
    moveClass?: string
  }
  ```

- **イベント**

  `<TransitionGroup>` は `<Transition>` と同じイベントを発行します。

- **詳細**

  デフォルトでは、`<TransitionGroup>` はラッパー DOM 要素をレンダリングしませんが、 `tag` プロパティによって定義できます。

  アニメーションが正しく動作するためには、`<transition-group>` 内のすべての子に[**一意なキーを指定**](/guide/essentials/list.html#maintaining-state-with-key)する必要があることに注意してください。

  `<TransitionGroup>` は CSS の transform による移動トランジションに対応しています。更新後に画面上の子の位置が変化した場合、移動用の CSS クラス（`name` 属性から自動生成されるか、`move-class` プロパティで設定）が適用されます。移動用のクラスが適用されたときに、CSS の `transform` プロパティが「トランジション可能」であれば、その要素は [FLIP テクニック](https://aerotwist.com/blog/flip-your-animations/)を使って移動先までスムーズにアニメーションします。

- **例**

  ```vue-html
  <TransitionGroup tag="ul" name="slide">
    <li v-for="item in items" :key="item.id">
      {{ item.text }}
    </li>
  </TransitionGroup>
  ```

- **参照:** [ガイド - TransitionGroup](/guide/built-ins/transition-group.html)

## `<KeepAlive>` {#keepalive}

動的に切り替えられる、内側のコンポーネントをキャッシュします。

- **プロパティ**

  ```ts
  interface KeepAliveProps {
    /**
     * 指定された場合、`include` でマッチした名前の
     * コンポーネントのみがキャッシュされます。
     */
    include?: MatchPattern
    /**
     * `exclude` でマッチした名前のコンポーネントは
     * キャッシュされません。
     */
    exclude?: MatchPattern
    /**
     * キャッシュするコンポーネントインスタンスの最大数。
     */
    max?: number | string
  }

  type MatchPattern = string | RegExp | (string | RegExp)[]
  ```

- **詳細**

  動的コンポーネントをラップすると、`<KeepAlive>` は非アクティブなコンポーネントインスタンスを破棄せずにキャッシュします。

  `<KeepAlive>` の直接の子として、アクティブなコンポーネントのインスタンスは常に 1 つだけです。

  `<KeepAlive>` の内部でコンポーネントが切り替えられると、その `activated` と `deactivated` ライフサイクルフックが呼び出されます（`mounted` と `unmounted` は呼び出されず、その代わりとして提供されています）。これは `<KeepAlive>` の直接の子だけでなく、そのすべての子孫にも適用されます。

- **例**

  基本的な使い方:

  ```vue-html
  <KeepAlive>
    <component :is="view"></component>
  </KeepAlive>
  ```

  `v-if` / `v-else` の分岐を使用する場合、一度にレンダリングされるコンポーネントは 1 つだけである必要があります:

  ```vue-html
  <KeepAlive>
    <comp-a v-if="a > 1"></comp-a>
    <comp-b v-else></comp-b>
  </KeepAlive>
  ```

  `<Transition>` と共に使用:

  ```vue-html
  <Transition>
    <KeepAlive>
      <component :is="view"></component>
    </KeepAlive>
  </Transition>
  ```

  `include` / `exclude` の使用:

  ```vue-html
  <!-- カンマ区切りの文字列 -->
  <KeepAlive include="a,b">
    <component :is="view"></component>
  </KeepAlive>

  <!-- 正規表現（`v-bind` を使用） -->
  <KeepAlive :include="/a|b/">
    <component :is="view"></component>
  </KeepAlive>

  <!-- 配列（`v-bind` を使用） -->
  <KeepAlive :include="['a', 'b']">
    <component :is="view"></component>
  </KeepAlive>
  ```

  `max` の使用:

  ```vue-html
  <KeepAlive :max="10">
    <component :is="view"></component>
  </KeepAlive>
  ```

- **参照:** [ガイド - KeepAlive](/guide/built-ins/keep-alive.html)

## `<Teleport>` {#teleport}

スロットの内容を DOM の別の場所にレンダリングします。

- **プロパティ**

  ```ts
  interface TeleportProps {
    /**
     * 必須。ターゲットコンテナーを指定します。
     * セレクターまたは実際の要素のいずれかを指定できます。
     */
    to: string | HTMLElement
    /**
     * `true` の場合、コンテンツはターゲットコンテナーに
     * 移動せずに元の場所に残ります。
     * 動的に変更できます。
     */
    disabled?: boolean
  }
  ```

- **例**

  ターゲットコンテナーの指定:

  ```vue-html
  <teleport to="#some-id" />
  <teleport to=".some-class" />
  <teleport to="[data-teleport]" />
  ```

  条件によって無効化:

  ```vue-html
  <teleport to="#popup" :disabled="displayVideoInline">
    <video src="./my-movie.mp4">
  </teleport>
  ```

- **参照:** [ガイド - Teleport](/guide/built-ins/teleport.html)

## `<Suspense>` <sup class="vt-badge experimental" /> {#suspense}

コンポーネントツリー内のネストした非同期な依存関係を管理するために使用します。

- **プロパティ**

  ```ts
  interface SuspenseProps {
    timeout?: string | number
  }
  ```

- **イベント**

  - `@resolve`
  - `@pending`
  - `@fallback`

- **詳細**

  `<Suspense>` は `#default` スロットと `#fallback` スロットの 2 つのスロットを受け付けます。default スロットをメモリー内にレンダリングする間、fallback スロットの内容を表示します。

  デフォルトスロットのレンダリング中に非同期な依存関係（[非同期コンポーネント](/guide/components/async.html)や [`async setup()`](/guide/built-ins/suspense.html#async-setup) のコンポーネント）が発生すると、それらが全て解決するまで待ってからデフォルトスロットを表示します。

- **参照:** [ガイド - Suspense](/guide/built-ins/suspense.html)
