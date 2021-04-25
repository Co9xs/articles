## 結論
親にmin-height100%を設定していても、子にはmin-heightプロパティは継承されない。

そのため子でheight100%しても、親のmin-heightの値までは広がらず、親のheightの設定値までしか伸びない。

また、この状態で親にheightを設定していない場合は、暗黙的にheight100%になる。
