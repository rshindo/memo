# 第８章　関数とクロージャー

## 8.1 メソッド
メソッド・・・何らかのオブジェクトのメンバー関数
下記のメソッドprocessFileはファイル名と１行の長さの下限を受け付け、長ければファイル名、コロン、行を出力する。
```scala
import scala.io.Source
object LongLines {
  def processFile(filename: String, width: Int) = {
    val source = Source.fromFile(filename)
    for(line <- source.getLines())
      processLine(filename, width, line)
  }
  private def processLine(filename: String, width: Int, line: String) = {
    if(line.length > width)
      println(filename + ": " + line.trim)
  }
}
```

## 8.2 ローカル関数

ヘルパー関数はクラスの外からは見えないように隠しておきたい。Javaではそのためにprivateメソッドを使う方法がある。
Scalaではprivateメソッドの他に、関数の中に定義されたローカル関数というアプローチがある。

```scala
def processFile(filename: String, width: Int) = {
  def processLine(filename: String, width: Int, line: String) = {
    if(line.length > width)
      println(filename + ": " + line.trim)
  }
  val source = Source.fromFile(filename)
  for(line <- source.getLines()) {
    processLine(filename, width, line)
  }
}
```
さらに、ローカル関数から外側の関数のパラメーターにアクセスできるので、```processLine```からfilenameとwidthを引数から消すことができる。

```scala
def processFile(filename: String, width: Int) = {
  def processLine(line: String) = {
    if(line.length > width)
      println(filename + ": " + line.trim)
  }
  // 省略
}
```

## 8.3 一人前の存在としての関数

Scalaにおける関数はファーストクラスである。

* 関数を定義して呼び出せる
* 名前の付けられていないリテラルとして関数を書ける
* 関数を値として渡すことができる

```scala
var increase = (x: Int) => x + 1
increase(10)
increase = (x: Int) => x + 9999
increase(10)
increase = (x: Int) => {
  println("Hello world!")
  x + 1
}
increase(10)
```
コレクションの操作で関数リテラルはよく出てくる。

```
val someNumbers = List(-11, -10, -5, 0, 5, 10)
someNumbers.filter((x: Int) => x > 0)
```

## 8.4 関数リテラルの短縮形

上のfilterの例では、関数のパラメータの型を省略できる。

```scala
val someNumbers = List(-11, -10, -5, 0, 5, 10)
someNumbers.filter(x => x > 0)
```

## 8.5 プレースホルダー構文

パラメーターが関数リテラル内で１度しか使われない場合には、１個以上のパラメーターのプレースホルダーとしてアンダースコアを使うことができる。

```scala
val someNumbers = List(-11, -10, -5, 0, 5, 10)
someNumbers.filter(_ > 0)
```

プレースホルダー構文を使うためには、コンパイラーが書かれていないパラメーターの型を推論できるだけの情報を持っている必要がある。たとえば次の場合はエラーとなる。

```scala
val f = _ + _
```

型を指定すればエラーは生じない。下記の関数は２個のパラメーターを取る。

``` scala
val f = (_: Int) + (_: Int)
```


### 8.6 部分適用された関数

関数のパラメーターの一部、または全体をプレースホルダーにすることで、部分適用された関数をつくることができる。

```scala
def sum(a: Int, b: Int, c: Int) = a + b + c
val a = sum _
```

sum _ で部分適用関数をつくるとき、関数オブジェクトが生成されている。この関数オブジェクトは引数を３つ取り、もとのsum関数を呼び出している。
上の例ではパラメーター全体をプレースホルダーにする（＝0個の引数に適用している）が、一部のパラメーターのみをプレースホルダーにすることもできる。

```scala
def sum(a: Int, b: Int, c: Int) = a + b + c
val b = sum(1, _: Int, 3)
```

また、すべてのパラメーターを省略した部分適用関数式を書く場合、その位置で関数呼び出しが必要なら、アンダースコアを省略することができる。

```scala
List(1, 2, 3).foreach(println _)
```

※関数名とアンダースコアの間にはスペースが必要

## 8.7 クロージャー

Scalaでは、関数リテラルは関数内で定義されたパラメーターだけではなく、どこか別の場所で定義された変数を参照することができる。

```scala
val more = 1
val addMore = (x: Int) => x + more
addMore(10)
```

この関数リテラルから実行時に作られる値としての関数（オブジェクト）をクロージャーと呼ぶ。
※厳密には関数内部の変数のみを参照する関数を「閉じた項」、外部の変数を参照する関数を「開いた項」と呼び、開いた項のみがクロージャーである。

Scalaのクロージャーは、変数が参照している値ではなく、変数自体をつかんでいる。クロージャーが参照している外部の変数に変更が加えられれは、クロージャーの中の変数も変わる。

```scala
val someNumbers = List(1, 2, 3)
var sum = 0
someNumbers.foreach(sum += _)
sum
```

## 8.8 関数呼び出しの特殊な形態

### 8.8.1 連続パラメーター

連続パラメーター＝可変長引数は次のように書く。

```scala
def echo(args: String*) = for (arg <- args) println(arg)
echo("hello", "world!")
```

連続パラメーターはechoの中ではArray[String]となっている。
しかし、Array[String]のパラメーターをechoに渡すとコンパイルエラーになる。

```scala
val arr = Array("What's", "up", "doc?")
echo(arr)
```

このような渡し方をしたい場合は配列引数に型注釈を足す必要がある。

```scala
echo(arr: _*)
```

### 8.8.2 名前付き引数

関数を呼び出す際に、名前付き引数を使ってパラメーターと異なる順序で引数を渡せる。

```scala
def speed(distance: Float, time: Float): Float = distance / time
speed(time = 10, distance = 100)
```

### 8.3 パラメーターのデフォルト値

関数パラメーターにデフォルトの値を指定できる。

```scala
def printTime(out: java.io.PrintStream = Console.out) =
  out.println("time = " + System.currentTimeMillis())
```

この関数を```printTime()```という形式で呼び出せば、outにはデフォルト値のConsole.outが使われる。```printTime(Console.err)```という形式で呼び出せば、標準エラー出力が使われる。

パラメーターのデフォルト値は名前付き引数とセットで使うと役に立つ。

```scala
def printTime2(out: java.io.PrintStream = Console.out, divisor: Int = 1) =
  out.println("time = " + System.currentTimeMillis() / divisor)
printTime2(out = Console.err)
printTime2(divisor = 1000)
```

## 8.9 末尾再帰

Javaでは繰り返しの処理をwhileループで書くかもしれないが、関数型では再帰を使って書くスタイルが一般的である。
最後の処理として自分自身を呼び出す再帰関数を末尾再帰と呼ぶ。Scalaコンパイラーは末尾再帰を検出するとオーバーヘッドのかからない形に関数を最適化してくれる。

### 8.9.1 末尾再帰関数をトレースする

末尾再帰関数は呼び出しのたびに新しいスタックフレームを作ったりはしない。

```scala
def boom(x: Int): Int =
  if (x == 0) throw new Exception("boom!")
  else boom(x - 1) + 1
```

この関数は末尾再帰になっていない（再帰呼出しの後でインクリメント処理を実行している）ので、スタックが呼び出しごとに深くなってしまっている。
boomが末尾再帰になるように書き換えると次のようになる

```scala
def boom(x: Int): Int =
  if (x == 0) throw new Exception("boom!")
  else boom(x - 1)
```

### 8.9.2 末尾再帰の限界

実行環境にJVMを使っている関係上、Scalaの末尾再帰はかなり限定されたものになっている。
例えば次のような再帰関数は最適化できない。

* 再帰が間接的な場合
```scala
def isEven(x: Int): Boolean = if (x == 0) true else isOdd(x - 1)
def isOdd(x: Int): Boolean = if (x == 0) false else isEven(x - 1)
```

* 値としての関数を使っている場合

```scala
def nestedFun(x: Int): Unit = {
  val funValue = nestedFun _
  if(x != 0) { println(x); funValue(x - 1)}
}
```




