---
title: "React Nativeのライブラリ作成を楽にしてくれるcreate-react-native-library"
emoji: "🍉"
type: "tech"
topics: ["reactnative", "react"]
published: true
publication_name: "gemcook"
---

React Native に限らずライブラリを作成する場合、かなりの事前準備が必要になります。
もちろん公開してから体裁を整えていく事もできますが、一旦の完成となるまでにはざっと以下のような作業が必要になります。

- 公開するライブラリ本体のコードの記述及び、ビルドの設定
- ESLint 等のフォーマッターの導入
- LICENSE の設定
- README・Example の用意
- バージョニング
- CI
- コミットルールの制定
- コントリビュートガイドの追加
- リリース作業の自動化
- etc...

私は経験がありませんが、React Native で iOS・Android のネイティブコードを使用するライブラリを作成する場合やネイティブコードを使用するライブラリに依存するライブラリを作成する場合は、依存関係の解決などでさらにライブラリの公開難易度が上がります。
素早くライブラリを作成するためにも、できれば上記にまつわる面倒な作業は最小限に抑えたいところです。

## create-react-native-library

React×TypeScript でライブラリの雛形を作成する場合は[tsdx](https://github.com/formium/tsdx)のようなツールが有名ですが、React Native でライブラリを作成する場合は、[公式](https://reactnative.dev/docs/native-modules-setup)でも紹介されている`create-react-native-library`が便利です。
`create-react-native-library`は`react-native-builder-bob`というリポジトリから提供されています。

https://github.com/callstack/react-native-builder-bob

## 使用してみる

`react-native-builder-bob`は既存の React Native プロジェクトをライブラリとしてビルドする機能を提供していますが、今回は 1 からライブラリを作成するため `create-react-native-library` コマンドを実行してライブラリの雛形を作成します。
`example-library` という名前でプロジェクトを作成する場合は以下のとおりです。

```
npx create-react-native-library example-library
```

## JavaScript only で作成

CLI でパッケージ名等の質問にいくつか回答していくと、どの言語（Android は Java/Kotlin、iOS は Objective-C/Swift もしくは JavaScript only など）を使用するかについて質問されます。
ここの選択肢によってできあがるサンプルコードが変わります。
JavaScript only かつを Expo でサンプルアプリを作成すると以下のような構成で作成されます。

- TypeScript
- ESLint&Prettier の設定
- husky と commitlint の設定
- Release it の設定
- CircleCI の設定
- ライセンス（デフォルトは MIT）
- コントリビュートガイドライン&README
- Expo でライブラリを試せる Example

JavaScript only を選択すると Expo で自分が作成しているライブラリを確認することもできるので、開発・確認の面で便利です。

ESLint や husky などの設定はプロジェクトや好みによっても変わってくると思いますが、デフォルトでも十分実用的ですし、雛形があればカスタマイズして変更するのもやりやすいです。

### ネイティブの言語を選択した場合

先程の言語選択で JavaScript only ではなく、ネイティブの API もしくはコンポーネントを使用する選択をすると、ESLint や husky 等の基本的な設定は同じですが、Expo ではなく `ios/android` ディレクトリが作成されネイティブのコードを呼び出すサンプルが作成されます。

後は公式の Native Module 作成のドキュメントを参考にしながら、ネイティブの API・コンポーネントを React Native で使用するようにライブラリを構築していきます。

https://reactnative.dev/docs/native-modules-intro

## さいごに

React Native はクロスプラットフォーム対応ということもあり、ライブラリの雛形を手動で作成するのはかなり大変なのでこういったツールを提供していただけるのは、非常にありがたいなと思います。
