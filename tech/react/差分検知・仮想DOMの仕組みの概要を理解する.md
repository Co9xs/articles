Reactのソースコードや内部実装を理解したくて色々と奮闘した記録。

間違いがあったらTwitterで指摘してくれると嬉しいです。

## やったこと
build-your-own-reactを翻訳記事も参考にしつつ写経してみた。

内容はDidactと名付けた単純化されたReactを自作してみようというハンズオン形式の記事。

後述するがスケジューリングの部分がややこしくて理解するのが難しかった。というより半分くらいしか理解できてないかも。

ここからはDidactをコーディングしていくにあたって得られた自分なりの理解をまとめておく。



## Didactの全体像
### JSXはどのように解釈されるか？

以下のようなJSXを考える。

```const element = <h2 id="title">HelloWorld</h2>```

上記のコードはBabelのツールでReactならReact.createElement(), preactならh()のようなピュアなJavaScript関数に変換される。

関数と言ってもやっていることは単純で、タグ名、属性、子要素をJavaScriptのObjectとして構造化して返しているだけ。

例えばさっきのJSXコードは、

```const element = <h2 id="title">HelloWorld</h2>```


こんな感じに関数に変換されて、

```Didact.createElement("h2", { id: "title" }, "HelloWorld")```


構造化されたObjectになって返ってくる。
```
{
  type: "h2",
  props: {
    id: "title",
    children: "HelloWorld"
  }
}
```
JSXで宣言的に書いたUIがJavaScriptのObjectとして構造化されるイメージ。

このObjectこそが一つの仮想DOM（正確に言うとVirtual Node）であるという理解で良さそう？



### 差分検知(reconciliation)
さっきの例では静的なデータだけをJSXで扱っていたが、実際の開発ではstateやコンポーネントに渡されるpropsがJSXに渡ってくることがほとんどである。これらは変化しうるものであり、当然変化があったら再描画が必要になってくる。

ここで使われるのがreconciliationと呼ばれる差分検知のアルゴリズムである。

状態が変化すると、最新の仮想DOMツリーと前回の仮想DOMツリーの差分検知を行い、どの部分に再描画が必要なのかを計算する。



今まではDOMツリー全体に対して差分検知を行うために、一番親の要素から再帰的に処理を行っていたが、それではすべての差分検知の処理が終わるまで他の処理がブロックされてUIが固まるなどの問題が発生していた。

つまり、DOMツリー全体の差分検知の処理が終わるまで他の処理が進まない同期的な処理だった。

この問題の解決としてReactは後述するFiberという新しいアーキテクチャを採用することで、差分検知を非同期な処理として扱っている。



UIのブロッキングについては以下のページで体験できる。

async 非同期な処理の方では比較的スムーズに入力できるが、 sync 同期処理の方では引っかかりが感じられる。

（違いが分かりにくい場合はchrome dev toolのperformanceタブの歯車からCPUの欄を4x slowdownに設定してみると良い。）

https://koba04.github.io/react-fiber-resources



### Fiberとはなにか？
Fiberとは一言でいうと、差分検知の処理を行うための最小のデータ構造のこと。

Fiberは他のFIberへの参照や、前回のFiberへの参照、どんな変化が起こったか（新規作成or 更新 or 削除）、新しく描画したいElementのデータなどを持っているので、今まで再帰的に行っていた1つの大きな処理を、優先度付きの連結リストと考えて小さな更新処理に分けて行うことができる。

Fiberを利用することで、一つ一つの更新処理を独立したものとして扱えるようになるので、途中停止もできるし、それぞれのジョブに優先度をつけておくことで、適切なスケジューリングを行っている。

優先度の高い処理から行って、優先度の低い処理はブラウザが暇な時に行うイメージ？



### シングルスレッドなJavaScriptでスケジューリングを行う仕組み
requestIdleCallback()やrequestAnimationFrame()といったWebAPIを使って擬似的に優先度付きのスケジューリングをしていた。

現在ReactはrequestIdleCallback()を使用しておらず、スケジューラパッケージを使用しているらしい。 



### 実DOMへの反映
今まで見てきた差分検知から仮想DOMツリーの生成までをrendering phaseと呼ばれ、実DOMへの反映のフェーズはcommit phaseと呼ばれる。

前述したようにFiberの利用によって差分検出は非同期的な処理になったが実DOMへの反映はこのcommit phaseで一気に行われるので、更新途中のデータでDOMが更新されることはない。



## まとめ
優先度をつけてスケジューリングするの部分のコードはすごくややこしく感じて、完全に理解したとは言いがたい。

とはいえ、一連の流れやFiberの概念などを体系的に理解できたのはよかった。

本来の目的であったReactのコードリーディングにもチャンレンジしたい。



## 参考
https://pomb.us/build-your-own-react/

https://zenn.dev/akatsuki/articles/a2cbd26488fa151b828b#step-2-render%E9%96%A2%E6%95%B0

https://www.koharakazuya.net/posts/2020-05-24-didact/

https://qiita.com/seya/items/a655adb340af3b6690b5

https://html5experts.jp/shumpei-shiraishi/23265/

https://reactjs.org/docs/reconciliation.html
