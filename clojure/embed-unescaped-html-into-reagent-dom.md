# ReagentでエスケープされていないHTMLを埋め込む

ClojureScriptのOmラッパーのひとつであるReagentでは、DOMをClojureScriptのデータ(ベクタやマップ)として書く。

このDOMを表すデータに含まれる文字列はデフォルトでHTMLエスケープされる。これを回避するにはReactの`dangerouslySetInnerHTML`の機能を使う。

```clj
[:div {:dangerouslySetInnerHTML {:__html "<b>This is an unescaped HTML!!</b>"}}]
```

## 参考
- https://github.com/reagent-project/reagent/issues/14
- https://facebook.github.io/react/tips/dangerously-set-inner-html.html
