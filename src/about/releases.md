---
outline: deep
---

<script setup>
import { onMounted } from 'vue'

let version = $ref()

onMounted(async () => {
  const res = await fetch('https://api.github.com/repos/vuejs/core/releases?per_page=1')
  version = (await res.json())[0].name
})
</script>

# リリース {#releases}

<p v-if="version">
現在の最新版の Vue の安定バージョンは <strong>{{ version }}</strong> です。
</p>
<p v-else>
Checking latest version...
</p>

過去のリリースの完全な変更履歴は、[GitHub](https://github.com/vuejs/core/blob/main/CHANGELOG.md) にて利用できます。

## リリースサイクル {#release-cycle}

Vue は、一定のリリースサイクルがありません。

- パッチリリースは、必要に応じて公開されます。

- マイナーリリースには常に新しい機能が含まれており、通常 3 〜 6 か月間の期間を伴っています。マイナーリリースは、常にベータプレリリース段階を通過します。

- メジャーリリースは事前に発表されて、初期の検討段階とアルファ / ベータプレリリース段階を通過します。

## セマンティックバージョニングエッジケース {#semantic-versioning-edge-cases}

Vue のリリースは、少しのエッジケースを伴う[セマンティックバージョニング](https://semver.org/lang/ja/)に従います。

### TypeScript 定義 {#typescript-definitions}

**マイナー**バージョンの間で TypeScript 定義に互換性のない変更を出す可能性があります。その理由は:

1. TypeScript 自体がマイナーバージョンの間に互換性のない変更を出すことがあり、 TypeScript の新しいバージョンをサポートするように型を調整せざるを得ない可能性があります。

2. まれに、TypeScript の新しいバージョンでのみ利用可能な機能を採用する必要が出てくる可能性があり、必要な TypeScript の最小バージョンが引き上げられます。

TypeScript を使用している場合、現在のマイナーを固定させる semver 範囲を使うことができ、 Vue の新しいマイナーバージョンが公開されるときに手動でアップグレードすることができます。

### コンパイルされたコードの以前のランタイムとの互換性 {#compiled-code-compatibility-with-older-runtime}

Vue コンパイラーの新しい**マイナー**バージョンは、以前のマイナーバージョンでの Vue ランタイムとは互換性のないコードが生成する可能性があります。たとえば、Vue 3.2 コンパイラーによって生成されたコードが Vue 3.1 のランタイムで利用される場合、完全な互換性がない可能性があります。

アプリケーションでは、コンパイラーバージョンとランタイムのバージョンは常に同じなので、このことはライブラリー作成者に対してのみの懸念事項です。バージョンの不一致は、コンパイル済みの Vue コンポーネントコードをパッケージとして出荷し、利用者が古いバージョンの Vue を使用するプロジェクトでそれを使用する場合にのみ発生します。結果として、あなたのパッケージ Vue のマイナーバージョンの最小条件を明示的に宣言する必要が出てくる可能性があります。

## プレリリース {#pre-releases}

マイナーリリースは通常決められていない数のベータ版を通過します。メジャーリリースはアルファ段階とベータ段階を通過します。

プレリリースは統合 / 安定性テストのため、そして不安定な機能に対するフィードバックを提供するアーリーアダプターのために意味をなします。本番でプレリリースを使用しないでください。すべてのプレリリースは不安定であると考えられ、途中で破壊的変更を出す可能性があるため、プレリリースを使用するときは、常に正確なバージョンをピン留めします。

## 非推奨 {#deprecations}

マイナーリリースでは定期的に、新しく優れた代替品のある機能が非推奨になる可能性があります。非推奨の機能は動き続けますが、それが非推奨ステータスになったあとで次のメジャーリリースにおいて除去されます。

## RFC {#rfcs}

実質的な API に対する新機能と Vue への主要な変更は、**Request for Comments** (RFC) プロセスを通過します。RFC プロセスは、新機能がフレームワークに導入されるための一貫して管理された経路を提供し、ユーザーが設計プロセスに参加してフィードバックをする機会を与えることを目的とします。

RFC プロセスは、GitHub 上の [vuejs/rfcs](https://github.com/vuejs/rfcs) レポジトリで行われます。

## 実験的機能 {#experimental-features}

Vue の安定したバージョンで出され、ドキュメント化されますが、実験的として指定される機能もあります。実験的機能は通常、理論的には設計上の問題の大部分が解決された、関連する RFC の議論がある機能ですが、実際の使用からのフィードバックはまだ不足しています。

実験的機能のゴールは、ユーザーが本番のセッティングでテストすることによって、 Vue の不安定なバージョンを使用することなく、フィードバックを提供できるようにすることです。実験的機能自体は不安定であると考えられ、どんなリリースタイプの中でも変更の可能性があると予想されますので、コントロールされた方法だけで使用する必要があります。
