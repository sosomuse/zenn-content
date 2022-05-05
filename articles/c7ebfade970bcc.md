---
title: "useRefを使った値管理ガイド"
emoji: "🐇"
type: "tech"
topics: ["react"]
published: true
---

## 概要

useRefはDOMにアクセスするために使用できますが、コンポーネント内に値を保持するためにも使えます。
この記事では useState と useRef の違いを見ながら、どのようなユースケースで useRef が有効かをサンプルを見ながら解説します。

> しかしながら useRef() は ref 属性で使うだけではなく、より便利に使えます。これはクラスでインスタンス変数を使うのと同様にして、あらゆる書き換え可能な値を保持しておくのに便利です。

https://ja.reactjs.org/docs/hooks-reference.html#useref

## はじめに

DOMやコンポーネントインスタンスの参照を得る以外に useRef を使用する機会はそこまで多くありません。大抵の場合 useState やライブラリによる状態管理で事足りるからです。
ですが特定のケースにおいて ref を使用するとパフォーマンスに優れ、シンプルな実装にできることがあります。useRefが有効なケースを知るためにまずは useState との違いを確認します。

## useStateとuseRefの違い

useState と比較したとき useRef の重要な特徴は3つです。

1. 更新による再レンダリングが発生しない
2. 値が同期的に更新される
3. 返却されるRefオブジェクトは同一である

### 1. 更新による再レンダリングが発生しない

useState は setState を実行した後コンポーネントが再レンダリングされますが useRef は更新しても再レンダリングされることはありません。
DOMの更新が発生しない分パフォーマンスには良いです。当然ですがUI上で使用すると 「更新したのに画面に反映されない！」 なんてことになるので **描画に使用しない値** や **他の処理で使用する一時データ** の管理に向いています。

### 2. 値が同期的に更新される

「値が同期的に更新される...」なにを当然のことを言ってるんだと思うかもしれません。
しかし useState はそうではありません。
例えば、一度は以下のようなコードを書いてみたことがあるのではないでしょうか？

```tsx:Counter.tsx
const Counter = () => {
  const [count, setCount] = useState(0);
  
  return (
    <button
      onClick={() => {
        console.log(count);
        setCount(1);
        console.log(count);
      }}
    >
      カウント1
    </button>
  );
};
```

このボタンをクリックした結果としては setCount を呼び出したにも関わらず count は以前の値を保っています。

![Result](https://storage.googleapis.com/zenn-user-upload/a4805c1b9ba3-20211130.png)

これは useState が同一の `render` 内で同じ値を保持し続けるためです。
Reactは主にパフォーマンスのために同一のイベント処理が全て終了した後にコンポーネントを一度に再レンダリングします。
そして新しくレンダリングされたコンポーネントで初めて更新された state を参照することができるようになります。

このレンダリング間における状態の詳しい仕組みは以下の記事が参考になります。

https://overreacted.io/ja/a-complete-guide-to-useeffect/

それでは先程のコードを useRef に書き換えてみましょう。

```tsx:Counter.tsx
const Counter = () => {
  const count = useRef(0);

  return (
    <button
      onClick={() => {
        console.log(count.current);
        count.current = 1;
        console.log(count.current);
      }}
    >
      カウント1
    </button>
  );
};
```

![Result2](https://storage.googleapis.com/zenn-user-upload/0fb3d19a31aa-20211130.png)

値が同期的に更新されていることが確認できます。
useState による状態管理では値を更新しても次のレンダリングまで最新の状態を参照することができません。しかし useRef ならば常に最新の値を参照することができます。

### 3. 返却されるRefオブジェクトは同一である

useState は初期値に渡した値または setState で更新した状態がそのまま返却されますが useRef はRefオブジェクトが返却されます。
useRef で返却されるRefオブジェクトは呼び出したコンポーネントが存在する限り同一のオブジェクトです。
これは useCallback, useMemo, useEffect など依存関係を持つHooksを使用する際に便利です。
上記のような依存関係を持つHooksは `===` で前回の値と新しい値が同一かどうか判定します。

そのため useRef で返却されるRefオブジェクトは依存関係に含めても依存関係に影響を起こさず、`eslint` の `react-hooks/exhaustive-deps` ルールを有効にしていてもRefは依存関係に含めるように勧められることもありません。

## useRef使用例

useState と useRef の違いを踏まえた上で、useRefの使用例を見ていきます。

### 基本的な使用方法

**「更新による再レンダリングが発生しない」** と **「値が同期的に更新される」** 点から画面に描画しない値であれば非常にシンプルに取り回せます。

コンポーネントのマウント状態、特定の処理が何度呼び出されたか、送信前のログデータを保持しておくなど様々な使用用途が考えられます。

例えばコンポーネント呼び出し時にどうしても一度だけ呼び出したい処理があったとします。
以下の記事でも触れられていますが、React v18 からは StrictMode 下の開発環境で依存配列が `[]` だったとしても useEffect が複数回呼び出される可能性があります。
（現在でもFast Refresh等で再現します）

[React 18 alpha版発表まとめ
](https://zenn.dev/uhyo/articles/react-18-alpha-essentials#strictmode-%E3%81%A7%E3%81%AE-useeffect-%E3%81%AE%E6%8C%99%E5%8B%95%E3%81%AE%E5%A4%89%E5%8C%96)

これはrefでフラグを管理すれば共通フックとして簡単に実装できます。

```ts:useEffectOnce.ts

const useEffectOnce = (effect: React.EffectCallback) => {
  const called = useRef(false);

  useEffect(() => {
    if (!called.current) {
      called.current = true;
      return effect();
    }
  }, []);
};
```

### usePrevious

usePreviousは[公式でも紹介されている](https://ja.reactjs.org/docs/hooks-faq.html#how-to-get-the-previous-props-or-state)有名なカスタムフックで、前回のレンダリング時の値を保存して使用できます。
また、この前回の値は変数として保持されるためUI上に描画しても意図通りに動作します。

```ts:usePrevious.ts
const usePrevious = <T extends unknown>(value: T) => {
  const ref = useRef<T>();
  useEffect(() => {
    ref.current = value;
  });
  return ref.current;
}
```

```tsx
const Counter = () => {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    ...
  );
};
```

このusePreviousの実装を見て「Refの値は値が同期的に更新されるのでは？これで前回の値を返せるの？」と疑問を持つ方もいるかも知れません。
これは return で ref.current を返却しており、レンダリング後の useEffect が実行される前の値を返却しているためです。
以下のように書き換えるとわかりやすいでしょうか。

```ts:usePrevious.ts
const usePrevious = <T extends unknown>(value: T) => {
  const ref = useRef<T>();
  // もともとの実装で return していた以前の値
  const prev = ref.current;
  useEffect(() => {
    ref.current = value;
  });
  
  return { ref, prev };
}
```

```tsx
const Counter = () => {
  const [count, setCount] = useState(0);
  const { ref, prev } = usePrevious(count);
  
  return (
    <button
      onClick={() => {
        console.log(ref, prev);
        setCount((prevCount) => prevCount + 1);
      }}
    >
      カウントアップ
    </button>
};
```

実行結果は以下のとおりです。
useEffectで更新される前に変数に代入した prev では前回の値が参照でき、refには最新の値が入っていることが確認できます。

![](https://storage.googleapis.com/zenn-user-upload/aad5378c516d-20211130.png)


### refに関数を保存する

カスタムフックを使用していると度々 callback を受け取るフックを作りたくなることがあります。
例えば useApi のような外部にリクエストを行う関数を受け取り、レスポンスデータを返却するようなカスタムフックを作るとします。

```ts:useApi.ts
type PromiseFunc = (...cb: any) => Promise<any>;
type PromiseReturnType<T extends PromiseFunc> = ReturnType<T> extends Promise<infer T> ? T : never;

const useApi = <T extends PromiseFunc>(callback: T) => {
  const [data, setData] = useState<PromiseReturnType<T>>();

  const request = useCallback(async () => {
    const data = await callback();
    setData(data);
  }, [callback]);

  useEffect(() => {
    request();
  }, [request]);

  return data;
}
```

引数に callback を受け取り、レスポンスデータを useState で保存するだけのカスタムフックです。
ですが、このコードは**無限ループ**を引き起こす危険性を含んでいます。

```tsx:Page.tsx
// ダミーのAPIリクエスト
const apiRequest = (): Promise<number> => {
  return new Promise((resolve) =>
    setTimeout(() => resolve(Math.floor(Math.random() * 1000)), 3000)
  );
};

const Counter = () => {
  const data = useApi(() => apiRequest());
  console.log(data);

  return ...;
};
```

例えば useApi フックの callback にメモ化されていない関数が渡されると非常に危険です。
リクエスト終了後に setData でレンダリングされてメモ化されていない callback が再生成されることにより、再度 request 関数が実行されて無限にAPIリクエストしてしまうことが想定されます。

この無限ループを安直に解決しようとすると useEffect の第2引数を空にしてしまう方法が考えられます。`react-hooks/exhaustive-deps` ルールに指摘はされますが `eslint-disable` で無効にしてしまえば実装としては問題ないでしょう。
しかし今後オプションを受け取るような機能追加がされたときに何に依存しているかを考え直す必要があるため、依存配列に手を入れるのはできれば最終手段にしておきたいところです。

```ts:useApi.ts
  useEffect(() => {
    request();
  }, []);
```

こういったときに便利なのが useRef に関数を保存する方法です。

```ts:useApi.ts
const useApi = <T extends PromiseFunc>(callback: T) => {
  const [data, setData] = useState<PromiseReturnType<T>>();
  const ref = useRef<T>();

  useEffect(() => {
    ref.current = callback;
  }, [callback]);

  const request = useCallback(async () => {
    const data = await ref.current?.();
    setData(data);
  }, []);

  useEffect(() => {
    request();
  }, [request]);

  return data;
}
```

callbackが更新される毎に ref に保持している関数を更新して、ref.currentで関数を呼び出します。
refオブジェクトは同一である事が保証されているため、関数を ref に入れておくことで依存配列を気にせず実装することができました。

## 最後に

useRef は useState のように状態管理の主役というわけではないですが、非常に小回りの効く便利なHooksです。
useState との違いを意識した上で適切に使用すれば実装の幅が広がります。

## 参考

- https://ja.reactjs.org/docs/hooks-reference.html
- https://overreacted.io/ja/a-complete-guide-to-useeffect/
- https://overreacted.io/react-as-a-ui-runtime/
