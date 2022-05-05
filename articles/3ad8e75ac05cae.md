---
title: "React Nativeのライブラリ作成を楽にしてくれるcreate-react-native-library"
emoji: "🍉"
type: "tech"
topics: ["reactnative", "react"]
published: true
---

React Nativeに限らずライブラリを作成する場合、かなりの事前準備が必要になります。
もちろん公開してから体裁を整えていく事もできますが、一旦の完成となるまでにはざっと以下のような作業が必要になります。

- 公開するライブラリ本体のコードの記述及び、ビルドの設定
- ESLint等のフォーマッターの導入
- LICENSEの設定
- README・Exampleの用意
- バージョニング
- CI
- コミットルールの制定
- コントリビュートガイドの追加
- リリース作業の自動化
- etc...

私は経験がありませんが、React NativeでiOS・Androidのネイティブコードを使用するライブラリを作成する場合やネイティブコードを使用するライブラリに依存するライブラリを作成する場合は、依存関係の解決などでさらにライブラリの公開難易度が上がります。
素早くライブラリを作成するためにも、できれば上記にまつわる面倒な作業は最小限に抑えたいところです。

## create-react-native-library

React×TypeScriptでライブラリの雛形を作成する場合は[tsdx](https://github.com/formium/tsdx)のようなツールが有名ですが、React Nativeでライブラリを作成する場合は、[公式](https://reactnative.dev/docs/native-modules-setup)でも紹介されている`create-react-native-library`が便利です。
`create-react-native-library`は`react-native-builder-bob`というリポジトリから提供されています。

https://github.com/callstack/react-native-builder-bob

## 使用してみる

`react-native-builder-bob`は既存のReact Nativeプロジェクトをライブラリとしてビルドする機能を提供していますが、今回は1からライブラリを作成するため `create-react-native-library` コマンドを実行してライブラリの雛形を作成します。
`example-library` という名前でプロジェクトを作成する場合は以下のとおりです。

```
npx create-react-native-library example-library
```

## JavaScript onlyで作成

CLIでパッケージ名等の質問にいくつか回答していくと、どの言語（AndroidはJava/Kotlin、iOSはObjective-C/SwiftもしくはJavaScript onlyなど）を使用するかについて質問されます。
ここの選択肢によってできあがるサンプルコードが変わります。
JavaScript onlyかつをExpoでサンプルアプリを作成すると以下のような構成で作成されます。

- TypeScript
- ESLint&Prettierの設定
- huskyとcommitlintの設定
- Release itの設定
- CircleCIの設定
- ライセンス（デフォルトはMIT）
- コントリビュートガイドライン&README
- Expoでライブラリを試せるExample

JavaScript onlyを選択するとExpoで自分が作成しているライブラリを確認することもできるので、開発・確認の面で便利です。

ESLintやhuskyなどの設定はプロジェクトや好みによっても変わってくると思いますが、デフォルトでも十分実用的ですし、雛形があればカスタマイズして変更するのもやりやすいです。

### ネイティブの言語を選択した場合

先程の言語選択でJavaScript onlyではなく、ネイティブのAPIもしくはコンポーネントを使用する選択をすると、ESLintやhusky等の基本的な設定は同じですが、Expoではなく `ios/android` ディレクトリが作成されネイティブのコードを呼び出すサンプルが作成されます。

後は公式のNative Module作成のドキュメントを参考にしながら、ネイティブのAPI・コンポーネントをReact Nativeで使用するようにライブラリを構築していきます。

https://reactnative.dev/docs/native-modules-intro


## さいごに

React Nativeはクロスプラットフォーム対応ということもあり、ライブラリの雛形を手動で作成するのはかなり大変なのでこういったツールを提供していただけるのは、非常にありがたいなと思います。
