---
title: "React + TypeScriptでpropsと型を便利に扱うTips集"
emoji: "🤖"
type: "tech"
topics: ["react", "typescript"]
published: true
---

# はじめに

React + TypeScriptでコンポーネントを書く際に便利だと思うTips集。

# 環境

```
"react": "17.0.2",
"@types/react": "17.0.9",
"typescript": "4.1.2"
```

# Tips集 

## React.ComponentProps

`ComponentProps`は対象のコンポーネントのpropsの型を取得できる型です。

```tsx
type AppProps = React.ComponentProps<typeof App>;
```

コンポーネントの型がexportされていない場合でも型情報を取得できます。
子コンポーネントにpropsを受け渡す際に、交差型を使って新しいpropsの型を作成する際に使用することが多いです。

```tsx:Table.tsx
import List from "./List";

type ListProps = React.ComponentProps<typeof List>;

type TableProps = {
  color: string;
} & ListProps;

const Table: React.VFC<TableProps> = ({ color, data }) => {
  return (
    <div>
      ...
      <List data={data} />;
    </div>
  );
};
```

## React.FC or React.VFC 

:::message alert
React v18 から React.VFC は非推奨になりました。
代わりに Rect.FC から children の型がなくなっているため、React.VFC を React.FC にそのまま置き換えてください。
:::

関数コンポーネントの型付けを便利にしてくれます。
FCは`FunctionComponent`の略称、VFCは`VoidFunctionComponent`の略称です。

```tsx:FC.tsx
const Table: React.FC<Props> = ({ data }) => {
  return (
    <div>
      ...
    </div>
  );
};
```

```tsx:VFC.tsx
const Table: React.VFC<Props> = ({ data }) => {
  return (
    <div>
      ...
    </div>
  );
};
```

この2つの方の違いはchildrenの型が定義されているかどうかのみです。
型定義を見るとFCはpropsの型に`PropsWithChildren`が指定されており、デフォルトでchildrenに`ReactNode`が指定されるようになっています。

```ts
type PropsWithChildren<P> = P & { children?: ReactNode };
```

個人的にはchildrenの型を明示的に指定する必要のあるVFCの方が好きですが、プロジェクトで統一されていれば問題ないでしょう。

## default props

関数コンポーネントの場合、通常の関数の引数を指定するのと同様の記法でdefault propsを定義できます。
以下の例はアロー関数ですがfunction構文でも通常の関数と同じです。

```tsx:Table.tsx
type Props = {
  color?: string;
}

const Table: React.VFC<Props> = ({ color = "red" }) => {
  return (
    <div style={{ color }}>
      ...
    </div>
  );
};
```

## 分割代入

propsを分割代入することで子コンポーネントに一部（または全て）のpropsを受け渡すことができます。
薄いWrapperコンポーネントなどを作成する際に便利です。
残りのpropsをこのコンポーネントに渡すというのが明示的に分かりやすい記述になっていると良いかなと思います。

```tsx:Button.tsx
type ButtonProps = React.HTMLAttributes<HTMLButtonElement>;

type Props = {
  color?: string;
} & ButtonProps;

const Button: React.VFC<Props> = ({ color = "red", children, ...buttonProps }) => {
  return <button style={{ color }} {...buttonProps}>{children}</button>;
};
```

# おわりに

Reactでコンポーネントを作る際に便利だと思った構文・型をまとめてみました。
コンポーネント作成時の一助となれば幸いです。
