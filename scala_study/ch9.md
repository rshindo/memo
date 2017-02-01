# 第９章　制御の抽象化

## 重複するコードの削減

パラメータとしての関数、すなわち高階関数の使用はコードの単純化に役立つことがある。

以下は、ファイル名の末尾が特定の文字列になっているファイルを探すAPIである。

```scala
object FileMatcher {
  private def filesHere = (new java.io.File(".")).listFiles
  def filesEnding(query: String) = {
    for(file <- filesHere; if file.getName.endsWith(query)) yield file
  }
}
```

ここで、ファイル名の任意の場所に特定の文字列が含まれているファイルを探すAPIを追加する。

```scala
object FileMatcher {
  private def filesHere = (new java.io.File(".")).listFiles
  def filesEnding(query: String) = {
    for(file <- filesHere; if file.getName.endsWith(query)) yield file
  }
  def filesContaining(query: String) = {
    for(file <- filesHere; if file.getName.contains(query)) yield file
  }
}
```

filesEnding と filesContaining には似たような処理がある。この処理をヘルパー関数でまとめることができないだろうか？
高階関数を使って、以下のように書き換えることができる。

```scala
object FileMatcher {
  private def filesHere = (new java.io.File(".")).listFiles
  private def filesMatching(matcher: String => Boolean) = {
    for(file <- filesHere; if matcher(file.getName)) yield file
  }
  def filesEnding(query: String) = filesMatching(_.endsWith(query))
  def filesContaining(query: String) = filesMatching(_.contains(query))
  def filesRegex(query: String) = filesMatching(_.matches(query))
}
```

## 9.2 クライアントコードの単純化

Scalaのコレクションには高階関数をパラメーターにとるAPIが多く用意されている。

* リストに負数が含まれていればtrueを返す関数

```scala
def containsNeg(nums: List[Int]): Boolean = {
  var exists = false
  for(num <- nums)
    if(num < 0)
      exists = true
  exists
}
containsNeg(List(1, 2, 3, 4))
containsNeg(List(1, 2, -3, 4))
```

このコードはListのexistsを呼び出したほうが簡潔に書ける

```scala
def containsNeg(nums: List[Int]): Boolean = nums.exists(_ < 0)
containsNeg(List(1, 2, 3, 4))
containsNeg(List(1, 2, -3, 4))
```

## 9.3 カリー化

普通の関数定義

```scala
def plainOldSum(x: Int, y: Int) = x + y
plainOldSum(1, 2)
```

カリー化された関数定義

```
def curriedSum(x: Int)(y: Int) = x + y
curriedSum(1)(2)
```

カリー化された関数を呼び出したときに実際に起きていることは、２回の関数呼び出しの連続である。

## 9.4 新しい制御構造を作る

リソースのオープン、操作、クローズを一つの制御構造にまとめてみる。

```scala
def withPrintWriter(file: File, op: PrintWriter => Unit) = {
  val writer = new PrintWriter(file)
  try {
    op(writer)
  } finally {
    writer.close()
  }
}
```

このような、関数にリソースを貸し出す制御構造をローンパターンと呼ぶ。

なお、引数が１個だけ渡すメソッド呼び出しでは丸括弧を中括弧に変えてもよい。

また、上の関数をカリー化すると次のようになる。

```scala
def withPrintWriter(file: File)(op: PrintWriter => Unit) = {
  val writer = new PrintWriter(file)
  try {
    op(writer)
  } finally {
    writer.close()
  }
}
val file = new File("date.txt")
withPrintWriter(file) { writer =>
  writer.println(new java.util.Date)
}
```

## 9.5 名前渡しパラメーター

中括弧の中のコードに値を渡さない制御構造を作るために、Scalaでは名前渡しパラメーターが提供されている。

名前渡しパラメーターを使わない場合

```scala
var assertionsEnabled = true
def myAssert(predicate: () => Boolean) = {
  if (assertionsEnabled && !predicate()) throw new AssertionError
}
```

この関数は

```scala
myAssert(() => 5 > 3)
```

というように呼び出すことができるが、高階関数のパラメーターの () => を外して書きたい。

パラメーターの関数を => から始めることで、名前渡しパラメーターを作ることができる。

```scala
var assertionsEnabled = true
def myAssert(predicate: => Boolean) = {
  if (assertionsEnabled && !predicate) throw new AssertionError
}
```

この関数呼び出しは

```scala
myAssert(5 > 3)
```

というように簡潔に書くことができる。
なお、名前渡しパラメーターの場合は、 5 > 3 の部分は関数呼び出しの**後**に評価される。myAssert を単純にBooleanの値をパラメーターに取る関数として定義した場合には、パラメーターの値は関数呼び出しの**前**に評価されることになる。
