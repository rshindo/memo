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
      processLine(filename, with, line)
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
  for(line <- source.getLine()) {
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

