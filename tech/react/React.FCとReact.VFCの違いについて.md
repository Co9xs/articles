タイトルの通り、Reactコンポーネントの型定義である両者の違いを備忘録として簡単にまとめておきます。

（※本記事ではクラスコンポーネントの型については触れません。）



いきなりですが結論から書くと、

「今はReact.VFCを使っといて、将来的に@types/reactのv18のアップデートが来たらReact.FCに置換しましょう。」です。

以下では型定義ファイルを見つつ違いをまとめていきます。

なお今回参照するのは@types/reactのv17.0.3(2021年4月時点の最新版)です。



## React.FCの型定義ファイルを見てみる
以下がReact.FCの定義部分です。

React.FCはReact.FunctionComponentのショートハンドになっているみたいなので、FunctionComponentの方だけ見れば良さそうです。

```
type FC<P = {}> = FunctionComponent<P>;    
interface FunctionComponent<P = {}> {
  (props: PropsWithChildren<P>, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P>;
  contextTypes?: ValidationMap<any>;
  defaultProps?: Partial<P>;
  displayName?: string;
}
```


React.FC<Props>としてジェネリクスで渡したPropsをさらにPropsWithChildren<P>という型定義にして返しています。

ではPropsWithChildren<P>では何をしているのか見てみます。

```type PropsWithChildren<P> = P & { children?: ReactNode };```


名前の通りですが、propsにoptionalなchildrenを追加しているだけですね。

つまり、Propsにchildrenを含まないように自分で定義したとしても、実際に関数の引数として渡るpropsには暗黙的にchildrenが含まれているので使用する際にchildrenを渡してもエラーにならないということです。



仮に以下のようなTestComponentを定義したとします。

```
type Props = {
  hoge: string;
  huga: number;
}

const TestComponent: React.FC<Props> = (props) => {
  return 〜  
} 
```


このTestコンポーネントは単体で使うことを想定して作りましたが、実際にはchildrenも暗黙的に定義されているので、以下のような使い方をしてもエラーになりません。

```
<TestComponent>
  <ChildComponent/>
</TestComponent>
```

これだと実際のPropsの型定義からchildrenを使うのかどうかがわかりにくいという問題があります。



## React.VFCの型定義ファイルを見てみる
対してReact.VFCを見てみましょう。

こちらもReact.VFCはReact.VoidFunctionComponentのショートハンドになっていので、VoidFunctionComponentの方だけ見れば良さそうです。

```
type VFC<P = {}> = VoidFunctionComponent<P>;    
interface VoidFunctionComponent<P = {}> {
  (props: P, context?: any): ReactElement<any, any> | null;
  propTypes?: WeakValidationMap<P>;
  contextTypes?: ValidationMap<any>;
  defaultProps?: Partial<P>;
  displayName?: string;
}
```


先ほどのReact.FCと違うのはpropsの部分だけです。

こちらは実際に関数に渡るpropsの型は自分で定義したPropsそのものです。



先ほどのTestComponentをReact.VFCに置き換えてみましょう。

```
type Props = {
  hoge: string;
  huga: number;
}

const TestComponent: React.VFC<Props> = (props) => {
  return 〜  
} 
```


こうすることで実際にコンポーネントに渡るpropsにもchildrenは含まれないので、以下の使い方はエラーになります。

```
// コンパイルエラー
<TestComponent>
  <ChildComponent/>
</TestComponent>
```


## まとめ
Reactコンポーネントの型定義にはReact.VFCを使い, childrenを取るコンポーネントのときは明示的にchildren: React.ReactNodeをPropsに定義する。

将来的には@types/reactのアップデートにより、React.FCから暗黙的なchildrenの定義が削除される破壊的変更が入るので、そしたら置き換える。
