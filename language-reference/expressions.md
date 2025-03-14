# 式\(Expressions\)

最終更新日: 2023/9/17
原文: https://docs.swift.org/swift-book/ReferenceManual/Expressions.html

型、演算子、変数、およびその他の名前と構造を紹介する。

Swift では、前置式、バイナリ式、基本式、後置式の 4 種類の式があります。式が評価されると、値を返すか、副作用を起こすか、またはその両方を引き起こします。

前置式とバイナリ式を使用すると、演算子をより小さな式に適用できます。基本式は概念的には最もシンプルな種類の式で、値にアクセスする方法を提供します。後置式は、前置式やバイナリ式と同様に、関数呼び出しやメンバアクセスなど、後置式を使用してより複雑な式を構築できます。各式は、下記のセクションで詳しく説明されています。

> Grammar of an expression:
>
> *expression* → *try-operator*_?_ *await-operator*_?_ *prefix-expression* *infix-expressions*_?_ \
> *expression-list* → *expression* | *expression* **`,`** *expression-list*

## 前置式\(Prefix Expressions\)

前置式は、式と任意の前置演算子を組み合わせます。前置演算子は 1 つの引数を受け取り、その後に式が続きます。

これらの演算子の動作については、[Basic Operators\(基本演算子\)](../language-guide/basic-operators.md)と[Advanced Operators\(高度な演算子\)](../language-guide/advanced-operators.md)を参照ください。

Swift 標準ライブラリによって提供されている演算子については、[Operator Declarations\(演算子宣言\)](https://developer.apple.com/documentation/swift/swift_standard_library/operator_declarations)を参照ください。

> Grammar of a prefix expression:
>
> *prefix-expression* → *prefix-operator*_?_ *postfix-expression* \
> *prefix-expression* → *in-out-expression*

### In-Out 式\(In-Out Expression\)

in-out 式は、関数呼び出し式に in-out パラメータとして渡された変数にマークをします。

```swift
&<#expression#>
```

in-out パラメータの詳細については、[In-Out Parameters\(In-Out パラメータ\)](../language-reference/declarations.md#declarations-in-out-parameters)を参照ください。

in-out 式は、[Implicit Conversion to a Pointer Type\(ポインタ型への暗黙変換\)](expressions.md#implicit-conversion-to-a-pointer-type)で説明されているように、ポインタが必要なコンテキストに非ポインタ引数を指定するときにも使用されます。

> Grammar of an in-out expression:
>
> *in-out-expression* → **`&`** *identifier*

### <a id="try-operator">Try 演算子\(Try Operator\)</a>

_Try 演算子_は、`try` 演算子の後にエラーをスローできる式が続く形で構成されます。形式は次のとおりです:

```swift
try <#expression#>
```

`try` 式の値は _expression_ の値です。

_オプショナル try 式_は `try?` 演算子の後にエラーをスローできる式が続く形で構成されます。形式は次のとおりです:

```swift
try? <#expression#>
```

式がエラーをスローしない場合、`try?` の値は式の値を含むオプショナルです。それ以外の場合、`try?` の値は `nil` です。

_強制 try 式_は `try!` 演算子の後にエラーをスローできる式が続く形で構成されます。形式は次のとおりです:

```swift
try! <#expression#>
```

`try!` の値は _experssion_ の値です。式がエラーをスローすると、実行時エラーが発生します。

バイナリ演算子の左側の式に `try`、`try?` または `try!`、がマークされている場合、その演算子はバイナリ式全体に適用されます。一方で、括弧\(`()`\)を使用して、演算子の適用範囲を明示することもできます。

```swift
//両方の関数呼び出しに適用されます
sum = try someThrowingFunction() + anotherThrowingFunction()

//両方の関数呼び出しに適用されます
sum = try (someThrowingFunction() + anotherThrowingFunction())

//エラー：最初の関数呼び出しにのみ適用されます
sum = (try someThrowingFunction()) + anotherThrowingFunction()
```

バイナリ演算子が代入演算子の場合、または `try` 式が括弧で囲まれていない限り、`try` 式はバイナリ演算子の右側には使用できません。

`try` と `await` 演算子の両方を含む場合は、最初に `try` が来なければなりません。

`try`、`try?` と `try!` の使用方法についての詳細は[Error Handling\(エラーハンドリング\)](../language-guide/error-handling.md)を参照ください。

> Grammar of a try expression:
>
> *try-operator* → **`try`** | **`try`** **`?`** | **`try`** **`!`**

### <a id="await-operator">Await 演算子\(Await Operator\)</a>

_await 式_は、`await` 演算子の後に非同期関数の結果を返す式が続けて構成されます。形式は次のとおりです:

```swift
await <#expression#>
```

`await` 式の値は _experssion_ の値です。

`await` でマークされた式を_潜在的中断ポイント_と呼びます。非同期関数の実行は、`await` でマークされている箇所で中断することができます。また、同時並行コードの実行は他の点で中断されることはありません。つまり、潜在的中断ポイント間で、次の潜在的中断ポイントに行く前に状態の更新が完了する条件で、一時的に破壊された不変式を必要とする状態を、安全に更新することができます。

`await` 式は、`async(priority:operation:)` 関数に渡される末尾クロージャのように、非同期コンテキスト内でのみ使用することができます。`defer` 文、または同期関数型の自動クロージャでは使用できません。

バイナリ演算子の左側の式に `await` 演算子がマークされている場合、その演算子はバイナリ式全体に適用されます。ただし、括弧を使用して、演算子の適用範囲について明示することができます。

```swift
//両方の関数呼び出しに適用されます
sum = await someAsyncFunction() + anotherAsyncFunction()

//両方の関数呼び出しに適用されます
sum = await (someAsyncFunction() + anotherAsyncFunction())

//エラー：最初の関数呼び出しにのみ適用されます
sum = (await someAsyncFunction()) + anotherAsyncFunction()
```

バイナリ演算子が代入演算子の場合、または `await` 式が括弧内に囲まれていない限り、`await` 式はバイナリ演算子の右側に使用できません。

式が `await` と `try` 演算子の両方を含む場合、最初に `try` 演算子が来なければなりません。

> Grammar of an await expression:
>
> *await-operator* → **`await`**

## 中置式\(Infix Expressions\)

_中置式_は、左右の引数を受け取る式と中置バイナリ演算子を組み合わせます。形式は次のとおりです:

```swift
<#left-hand argument#> <#operator#> <#right-hand argument#>
```

これらの演算子の動作については、[Basic Operators\(基本演算子\)](../language-guide/basic-operators.md) と [Advanced Operators\(高度な演算子\)](../language-guide/advanced-operators.md)を参照ください。

標準ライブラリによって提供されている演算子については、[Operator Declarations\(演算子宣言\)](https://developer.apple.com/documentation/swift/operator_declarations)を参照ください。

> NOTE
> 構文解析時には、式はバイナリ演算子のフラットなリストを構成します。このリストは、演算子の優先順位を適用することによってツリーに変換されます。例えば、式 `2 + 3 * 5` は、最初は5つの項目、`2`、`+`、`3`、`*`、および `5` として解釈され、その後 `(2 + (3 * 5))` のツリーに変換します

> Grammar of an infix expression:
>
> *infix-expression* → *infix-operator* *prefix-expression* \
> *infix-expression* → *assignment-operator* *try-operator*_?_ *prefix-expression* \
> *infix-expression* → *conditional-operator* *try-operator*_?_ *prefix-expression* \
> *infix-expression* → *type-casting-operator* \
> *infix-expressions* → *infix-expression* *infix-expressions*_?_

### 代入演算子\(Assignment Operator\)

代入演算子は特定の式に新しい値を設定します。形式は次のとおりです:

```swift
<#expression#> = <#value#>
```

value を評価した結果得られた値が expression に設定されます。式がタプルの場合、値は同じ数の要素を持つタプルでなければなりません。\(タプルはネストすることもできます\)。代入は、値の各部分から expression の中の対応する部分に対して行われます。例えば:

```swift
(a, _, (b, c)) = ("test", 9.45, (12, 3))
// a は"test"、 b は 12、 c は 3、 9.45 は無視されます
```

代入演算子は任意の値を返しません。

> Grammar of an assignment operator:
>
> *assignment-operator* → **`=`**

### 三項条件演算子\(Ternary Conditional Operator\)

三項条件演算子は、条件の値に基づいて、2 つの値のうちの 1 つに評価されます。形式は次のとおりです:

```swift
<#condition#> ? <#expression used if true#> : <#expression used if false#>
```

条件が `true` と評価された場合、条件演算子は最初の式を評価し、その値を返します。それ以外の場合は、2 番目の式を評価してその値を返します。未使用の式は評価されません。

三項条件演算子を使用する例については、[Ternary Conditional Operator\(三項条件演算子\)](../language-guide/basic-operators.md#basic-operator-ternary-conditional-operator)を参照ください。

> Grammar of a conditional operator:
>
> *conditional-operator* → **`?`** *expression* **`:`**

### <a id="type-casting-operators">Type-Casting Operators\(型キャスト演算子\)</a>

4 つの型キャスト演算子があります: `is` 演算子、`as` 演算子、`as?` 演算子、そして `as!` 演算子。

それらは次の形式を持っています:

```swift
<#expression#> is <#type#>
<#expression#> as <#type#>
<#expression#> as? <#type#>
<#expression#> as! <#type#>
```

`is` 演算子は実行時に式が指定された型にキャストできるかどうかを確認します。キャストできる場合は `true` を返します。それ以外の場合は、`false` を返します。

`as` 演算子は、コンパイル時にキャストが常に成功するとわかっている場合にキャストを実行します。アップキャストは、中間変数を使用せずに型のスーパー型のインスタンスとして式を使用できます。下記のアプローチはどれも同等です。

```swift
func f(_ any: Any) { print("Function for Any") }
func f(_ int: Int) { print("Function for Int") }
let x = 10
f(x)
// Function for Int

let y: Any = x
f(y)
// Function for Any

f(x as Any)
// Function for Any
```

ブリッジングを使用して、新しいインスタンスを作成せずに、`String` などの Swift 標準ライブラリ型の式を、それに相応する `NSString` などの Foudation 型で使用できるようにしています。ブリッジングの詳細については、[Working with Foundation Types](https://developer.apple.com/documentation/swift/imported_c_and_objective_c_apis/working_with_foundation_types)を参照ください。

`as?` 演算子は、式の指定された_型_への条件付きキャストを実行します。`as?` 演算子は指定された_型_のオプショナルを返します。実行時に、キャストが成功した場合、_式_の値がオプショナルで返されます。それ以外の場合、返される値は `nil` です。指定された_型_へのキャストが失敗するか、成功することが明らかな場合は、コンパイルエラーが発生します。

`as!` 演算子は、指定された型に強制キャストを実行します。`as!` 演算子は、オプショナル型ではなく、指定された型の値を返します。キャストが失敗した場合は、実行時エラーが発生します。`x as! T` は `(x as? T)!` の挙動と同じです。

型キャストの詳細や型キャスト演算子を使用する例については、[Type Casting\(型キャスト\)](../language-guide/type-casting.md)を参照ください。

> Grammar of a type-casting operator:
>
> *type-casting-operator* → **`is`** *type* \
> *type-casting-operator* → **`as`** *type* \
> *type-casting-operator* → **`as`** **`?`** *type* \
> *type-casting-operator* → **`as`** **`!`** *type*

## 基本式\(Primary Expressions\)

基本式は最も基本的な種類の式です。それらは自身を式として使用したり、他のトークンと組み合わせたり、前置式、バイナリ式、および後置式を作成することができます。

> Grammar of a primary expression:
>
> *primary-expression* → *identifier* *generic-argument-clause*_?_ \
> *primary-expression* → *literal-expression* \
> *primary-expression* → *self-expression* \
> *primary-expression* → *superclass-expression* \
> *primary-expression* → *conditional-expression* \
> *primary-expression* → *closure-expression* \
> *primary-expression* → *parenthesized-expression* \
> *primary-expression* → *tuple-expression* \
> *primary-expression* → *implicit-member-expression* \
> *primary-expression* → *wildcard-expression* \
> *primary-expression* → *macro-expansion-expression* \
> *primary-expression* → *key-path-expression* \
> *primary-expression* → *selector-expression* \
> *primary-expression* → *key-path-string-expression

### <a id="literal-expression">リテラル式\(Literal Expression\)</a>

リテラル式は、通常のリテラル\(文字列や数など\)、配列または辞書リテラル、playground リテラルで構成されます。

> NOTE
> Swift 5.9以前は、以下の特殊リテラルが認識されていました:
> `#column`, `#dsohandle` , `#fileID`, `#filePath`, `#file`, `#function`, `#line`。
> 現在はSwift標準ライブラリのマクロとして実装されています:
> [`column`](https://developer.apple.com/documentation/swift/column()),
> [`dsohandle`](https://developer.apple.com/documentation/swift/dsohandle()),
> [`fileID`](https://developer.apple.com/documentation/swift/fileID()),
> [`filePath`](https://developer.apple.com/documentation/swift/filePath()),
> [`file`](https://developer.apple.com/documentation/swift/file()),
> [`function`](https://developer.apple.com/documentation/swift/function()),
> [`line`](https://developer.apple.com/documentation/swift/line())。

_配列リテラル_は、順序付けられた値の集合です。形式は次のとおりです:

```swift
[<#value 1#>, <#value 2#>, <#...#>]
```

配列内の最後の式の後にカンマ\(`,`\)を続けることもできます。配列リテラルの値は `[T]` 型で、`T` はその内部の式の型です。複数の型の式がある場合、`T` はそれらに最も近い共通のスーパー型になります。空の配列リテラルは、空の角括弧\(`[]`\)を使用し、指定された型の空の配列を作成するためにも使用できます。

```swift
var emptyArray: [Double] = []
```

_辞書リテラル_は、順序のないキーバリューペアのコレクションです。形式は次のとおりです:

```swift
[<#key 1#>: <#value 1#>, <#key 2#>: <#value 2#>, <#...#>]
```

辞書内の最後の式の後にカンマ\(`,`\)を続けることができます。辞書リテラルの値は `[Key：Value]` 型で、`Key` はそのキー式の型、`Value` はその値式の型です。複数の型の式がある場合、キーとバリューはそれぞれの値に最も近い共通のスーパー型になります。空の辞書リテラルは、空の配列リテラルと区別するために、一対の括弧内にコロンを書きます\(`[:]`\)。空の辞書リテラルを使用して、指定されたキーとバリュー型の空の辞書リテラルを作成できます。

```swift
var emptyDictionary: [String: Double] = [:]
```

_playground リテラル_は、プログラムエディタ内の色、ファイル、または画像の対話型な表現を作成するために Xcode によって使用されます。Xcode の外側のプレーンテキストの `playground` リテラルには、特別なリテラル構文を使用します。

Xcode の playground リテラルの使用方法については、Xcode ヘルプ内の[Add a color, file, or image literal](https://help.apple.com/xcode/mac/current/#/dev4c60242fc)を参照ください。

> Grammar of a literal expression:
>
> *literal-expression* → *literal* \
> *literal-expression* → *array-literal* | *dictionary-literal* | *playground-literal*
>
> *array-literal* → **`[`** *array-literal-items*_?_ **`]`** \
> *array-literal-items* → *array-literal-item* **`,`**_?_ | *array-literal-item* **`,`** *array-literal-items* \
> *array-literal-item* → *expression*
>
> *dictionary-literal* → **`[`** *dictionary-literal-items* **`]`** | **`[`** **`:`** **`]`** \
> *dictionary-literal-items* → *dictionary-literal-item* **`,`**_?_ | *dictionary-literal-item* **`,`** *dictionary-literal-items* \
> *dictionary-literal-item* → *expression* **`:`** *expression*
>
> *playground-literal* → **`#colorLiteral`** **`(`** **`red`** **`:`** *expression* **`,`** **`green`** **`:`** *expression* **`,`** **`blue`** **`:`** *expression* **`,`** **`alpha`** **`:`** *expression* **`)`** \
> *playground-literal* → **`#fileLiteral`** **`(`** **`resourceName`** **`:`** *expression* **`)`** \
> *playground-literal* → **`#imageLiteral`** **`(`** **`resourceName`** **`:`** *expression* **`)`**

### self 式\(Self Expression\)

`self` 式は、それが使用さえている現在の型またはインスタンスへの明示的な参照です。形式は次のとおりです:

```swift
self
self.<#member name#>
self[<#subscript index#>]
self(<#initializer arguments#>)
self.init(<#initializer arguments#>)
```

イニシャライザ、サブスクリプト、またはインスタンスメソッドでは、`self` は、それが出現する現在の型のインスタンスを表します。型メソッドでは、`self` はそれが登場する現在の型を表します。

`self` 式は、関数パラメータなどスコープ内に同じ名前の別の変数があり、何を指すのかが曖昧な場合に、メンバへアクセスするときに指定します。例えば:

```swift
class SomeClass {
    var greeting: String
    init(greeting: String) {
        self.greeting = greeting
    }
}
```

値型の mutating メソッドでは、その値型の新しいインスタンスを `self` に代入できます。例えば:

```swift
struct Point {
    var x = 0.0, y = 0.0
    mutating func moveBy(x deltaX: Double, y deltaY: Double) {
        self = Point(x: x + deltaX, y: y + deltaY)
    }
}
```

> Grammar of a self expression:
>
> *self-expression* → **`self`** | *self-method-expression* | *self-subscript-expression* | *self-initializer-expression*
>
> *self-method-expression* → **`self`** **`.`** *identifier* \
> *self-subscript-expression* → **`self`** **`[`** *function-call-argument-list* **`]`** \
> *self-initializer-expression* → **`self`** **`.`** **`init`**

### スーパークラス式\(Superclass Expression\)

_スーパークラス式_は、クラスがスーパークラスとやり取りすることを可能にします。次のいずれかの形式があります:

```swift
super.<#member name#>
super[<#subscript index#>]
super.init(<#initializer arguments#>)
```

最初の形式はスーパークラスのメンバにアクセスするために使用されます。2 番目の形式は、スーパークラスのサブスクリプトの実装にアクセスするために使用されます。3 番目の形式は、スーパークラスのイニシャライザにアクセスするために使用されます。

サブクラスは、スーパークラスの実装を利用するために、メンバ、サブスクリプト、およびイニシャライザの実装でスーパークラス式を使用できます。

> Grammar of a superclass expression:
>
> *superclass-expression* → *superclass-method-expression* | *superclass-subscript-expression* | *superclass-initializer-expression*
>
> *superclass-method-expression* → **`super`** **`.`** *identifier* \
> *superclass-subscript-expression* → **`super`** **`[`** *function-call-argument-list* **`]`** \
> *superclass-initializer-expression* → **`super`** **`.`** **`init`**

### 条件式\(Conditional Expression\)

_条件式_は、条件の値に基づいて、与えられたいくつかの値のうちの 1 つに評価されます。

形式は次の通りです:

```swift
if <#condition 1#> {
   <#expression used if condition 1 is true#>
} else if <#condition 2#> {
   <#expression used if condition 2 is true#>
} else {
   <#expression used if both conditions are false#>
}
switch <#expression#> {
case <#pattern 1#>:
    <#expression 1#>
case <#pattern 2#> where <#condition#>:
    <#expression 2#>
default:
    <#expression 3#>
}
```

条件式は、`if` 文や `switch` 文と同じ動作と構文ですが、以下の段落で説明する違いがあります。

条件式は、以下の状況でのみ使用できます:

- 変数に代入される値として
- 変数または定数宣言の初期値として
- `throw` 式が投げるエラーとして
- 関数、クロージャ、プロパティの `get` が返す値として
- 条件式の分岐内の値として

条件式の分岐は網羅的であり、条件に関係なく常に値を生成することを保証します。つまり、各 `if` 分岐には対応する `else` 分岐が必要です。

各分岐には、その分岐の条件が真である場合に条件式の値として使用される単一式、`throw` 文、または戻り値を返さない関数への呼び出しが含まれます。

各分岐は、同じ型の値を生成する必要があります。各分岐の型チェックは独立しているので、分岐に異なる種類のリテラルを含む場合や、分岐の値が `nil` である場合など、値の型を明示的に指定する必要がある場合があります。このような情報を提供する必要がある場合は、結果が代入される変数に型注釈を追加するか、分岐の値に `as` キャストを追加してください。

```swift
let number: Double = if someCondition { 10 } else { 12.34 }
let number = if someCondition { 10 as Double } else { 12.34 }
```

リザルトビルビルダの内部では、条件式は変数や定数の初期値としてのみ使用することができます。つまり、変数や定数の宣言のないリザルトビルダ内で `if` や `switch` を記述すると、そのコードは分岐文として理解され、リザルトビルダのメソッドの 1 つが、そのコードを変換することになります。

条件式の分岐の 1 つがエラーをスローする場合でも、条件式を `try` 式の中に入れてはいけません。


> Grammar of a conditional expression:
>
> *conditional-expression* → *if-expression* | *switch-expression*
>
> *if-expression* → **`if`** *condition-list* **`{`** *statement* **`}`** *if-expression-tail* \
> *if-expression-tail* → **`else`** *if-expression* \
> *if-expression-tail* → **`else`** **`{`** *statement* **`}`**
>
> *switch-expression* → **`switch`** *expression* **`{`** *switch-expression-cases* **`}`** \
> *switch-expression-cases* → *switch-expression-case* *switch-expression-cases*_?_ \
> *switch-expression-case* → *case-label* *statement* \
> *switch-expression-case* → *default-label* *statement*

### クロージャ式\(Closure Expression\)

_クロージャ式_は、他のプログラミング言語では、_ラムダ_または_匿名関数_とも呼ばれているクロージャを作成します。関数宣言のように、クロージャには文が含まれており、その囲まれている範囲から定数と変数をキャプチャします。形式は次のとおりです:

```swift
{ (<#parameters#>) -> <#return type#> in
   <#statements#>
}
```

[Function Declaration\(関数宣言\)](../language-reference/declarations.md#function-declaration)で説明されているように、_parameters_は関数宣言内のパラメータと同じ形式です。

クロージャ式で `throw` または `async` を記述すると、クロージャはエラーをスローする、または非同期であることを明示します。

```swift
{ (<#parameters#>) async throws -> <#return type#> in
   <#statements#>
}
```

クロージャの本文に `try` 式が含まれている場合、クロージャはエラーをスローすると見なします。同様に、`await` 式が含まれている場合は、非同期であると見なします。

クロージャをより簡潔に書くことができるいくつかの特別な形式があります:

* クロージャは、そのパラメータ、戻り値の型、またはその両方の型を省略できます。パラメータ名と型の両方を省略する場合は、文の前の `in` キーワードを省略してください。省略された型を推論できない場合は、コンパイルエラーが発生します
* クロージャはそのパラメータ名を省略することができます。その際は暗黙的に `$0`、`$1`、 `$2` などのように `$` の後ろにパラメータの位置を続けた名前が与えられます
* 単一式からなるクロージャは、その式の値を返すことが明らかです。この式の内容は、囲まれている式の型を推論するときにも使用されます

次のクロージャ式は同等です:

```swift
myFunction { (x: Int, y: Int) -> Int in
    return x + y
}

myFunction { x, y in
    return x + y
}

myFunction { return $0 + $1 }

myFunction { $0 + $1 }
```

関数の引数としてクロージャを渡す方法については、[Function Call Expression\(関数呼び出し式\)](expressions.md#function-call-expression)を参照ください。

クロージャ式は、関数呼び出しの一部としてすぐにクロージャを使用するときなど、可変または定数に格納されることなく使用できます。上記のコードの `myFunction` に渡されたクロージャ式は、即時に使用される例です。その結果、クロージャ式がエスケープか非エスケープかは、式の周囲のコンテキストによって異なります。クロージャ式は、即時に呼ばれるか、非エスケープ関数の引数として渡されると、非エスケープです。それ以外の場合、クロージャ式はエスケープです。

クロージャのエスケープの詳細については、<a href="../language-guide/closures.md#escaping-closures" target="_self">Escaping Closures(エスケープクロージャ)</a>を参照ください。

#### <a id="capture-lists">キャプチャリスト\(Capture Lists\)</a>

デフォルトでは、クロージャ式は、、周囲のスコープの定数と変数を強い参照を持ってキャプチャします。_キャプチャリスト_を使用して、クロージャ内で値をキャプチャする方法を明示的に制御できます。

キャプチャリストは、パラメータのリストの前に、角括弧\(`[]`\)で囲まれた式のカンマ\(`,`\)区切りのリストとして書かれます。キャプチャリストを使用する場合は、パラメータ名、パラメータ型、および戻り値の型を省略しても、`in` キーワードを使用する必要があります。

キャプチャリストへの各エントリは、クロージャが作成されたときに初期化されます。キャプチャリスト内の各エントリに対して、定数は周囲のスコープの同じ名前を持つ定数または変数の値で初期化できます。例えば、下記のコードでは、`a` はキャプチャリストに含まれていますが、`b` は含まれていません。

```swift
var a = 0
var b = 0
let closure = { [a] in
    print(a, b)
}

a = 10
b = 10
closure()

// 0 10
```

クロージャの範囲内の定数と周囲の範囲内の変数に `a` という同じ名前の異なる変数がありますが、`b` という名前の変数は 1 つだけです。内部スコープ内の `a` は、クロージャが作成されたときに外側の `a` 値で初期化されますが、それらの値は繋がっていません。つまり、これは、外側の範囲内の `a` の値の変化が内側の範囲内の `a` の値に影響を与えず、クロージャの内側の値の変化も外側の `a` に影響を与えません。対照的に、`b` という名前の変数は外側の範囲内に 1 つしかなく、クロージャの内側または外側からの変化は両方に影響を与えます。

キャプチャされた変数の型に参照セマンティクスがある場合、この区別はありません。例えば、下のコードに `x` という 2 つの変数がありますが、外部スコープの変数と内部スコープの定数は、両方とも参照セマンティクスのために同じオブジェクトを参照します。

```swift
class SimpleClass {
    var value: Int = 0
}
var x = SimpleClass()
var y = SimpleClass()
let closure = { [x] in
    print(x.value, y.value)
}

x.value = 10
y.value = 10
closure()
// 10 10
```

式の値の型がクラスの場合は、式の値へ弱参照または非所有参照で取り込むために、キャプチャリスト内の式に `weak` または `unowned` をマークすることができます。

```swift
myFunction { print(self.title) }                    // 暗黙的な強参照
myFunction { [self] in print(self.title) }          // 明示的な強参照
myFunction { [weak self] in print(self!.title) }    // 弱参照
myFunction { [unowned self] in print(self.title) }  // 非所有参照
```

任意の式をキャプチャリスト内の名前付きの値にバインドすることもできます。クロージャが作成されたときに式が評価され、値は指定された強度でキャプチャされます。例えば:

```swift
// parent として self.parent を弱参照する
myFunction { [weak parent = self.parent] in print(parent!.title) }
```

クロージャ式の詳細と例については、[Closure Expressions\(クロージャ式\)](../language-guide/closures.md#closure-expressions)を参照ください。キャプチャリストの詳細および例については、[Resolving Strong Reference Cycles for Closures\(クロージャの強循環参照の解消\)](../language-guide/automatic-reference-counting.md#resolving-strong-reference-cycles-for-closures)を参照ください。

> Grammar of a closure expression:
>
> *closure-expression* → **`{`** *attributes*_?_ *closure-signature*_?_ *statements*_?_ **`}`**
>
> *closure-signature* → *capture-list*_?_ *closure-parameter-clause* **`async`**_?_ **`throws`**_?_ *function-result*_?_ **`in`** \
> *closure-signature* → *capture-list* **`in`**
>
> *closure-parameter-clause* → **`(`** **`)`** | **`(`** *closure-parameter-list* **`)`** | *identifier-list* \
> *closure-parameter-list* → *closure-parameter* | *closure-parameter* **`,`** *closure-parameter-list* \
> *closure-parameter* → *closure-parameter-name* *type-annotation*_?_ \
> *closure-parameter* → *closure-parameter-name* *type-annotation* **`...`** \
> *closure-parameter-name* → *identifier*
>
> *capture-list* → **`[`** *capture-list-items* **`]`** \
> *capture-list-items* → *capture-list-item* | *capture-list-item* **`,`** *capture-list-items* \
> *capture-list-item* → *capture-specifier*_?_ *identifier* \
> *capture-list-item* → *capture-specifier*_?_ *identifier* **`=`** *expression* \
> *capture-list-item* → *capture-specifier*_?_ *self-expression* \
> *capture-specifier* → **`weak`** | **`unowned`** | **`unowned(safe)`** | **`unowned(unsafe)`**

### <a id="implicit-member-expression">暗黙メンバ式\(Implicit Member Expression\)</a>

_暗黙メンバ式_は、型推論によって暗黙的に型を決定できるコンテキストにおいて、列挙ケースや型メソッドなどの型のメンバにアクセスするための省略記法です。形式は次のとおりです:

```swift
.<#member name#>
```

例えば:

```swift
var x = MyEnumeration.someValue
x = .anotherValue
```

推論された型がオプショナルの場合は、暗黙メンバ式にオプショナルでない型のメンバを使用することもできます。

```swift
var someOptional: MyEnumeration? = .someValue
```

暗黙メンバ式の後に[Postfix Expressions\(後置式\)](../language-reference/expressions.md#postfix-expressions)でリストされている後置演算子またはその他の後置構文を続けることができます。これは_暗黙メンバ式チェーン_と呼ばれます。全ての後置式チェーンで同じ型を持つことが一般的ですが、最低限の要件として、暗黙メンバ式チェーン全体がそのコンテキストで暗黙的に推論される型と互換性がある必要があります。具体的には、暗黙的に推論される型がオプショナルの場合は、オプショナル以外の型の値を使用でき、クラス型の場合、そのサブクラスを使用できます。例えば:

```swift
class SomeClass {
    static var shared = SomeClass()
    static var sharedSubclass = SomeSubclass()
    var a = AnotherClass()
}
class SomeSubclass: SomeClass { }
class AnotherClass {
    static var s = SomeClass()
    func f() -> SomeClass { return AnotherClass.s }
}
let x: SomeClass = .shared.a.f()
let y: SomeClass? = .shared
let z: SomeClass = .sharedSubclass
```

上記のコードでは、`x` の型はそのコンテキストから暗黙的に推論された型と正確に一致し、`y` の型は `SomeClass` から `SomeClass?` に変換され、`z` の型は `SomeSubclass` から `SomeClass` に変換されます。

> Grammar of a implicit member expression:
>
> *implicit-member-expression* → **`.`** *identifier* \
> *implicit-member-expression* → **`.`** *identifier* **`.`** *postfix-expression*

### 括弧で囲まれた式\(Parenthesized Expression\)

_括弧で囲まれた式_は、括弧で囲まれた式で構成されます。式を明示的にグループ化することで、括弧を使用して操作の優先順位を指定できます。括弧のグループ化は式の型を変更しません\(例：`(1)` はただの `Int` です。

> Grammar of a parenthesized expression:
>
> *parenthesized-expression* → **`(`** *expression* **`)`**

### タプル式\(Tuple Expression\)

タプル式は、括弧で囲まれた式のカンマ区切りのリストで構成されています。各式は、コロン\(`:`\)で区切られ、その前に識別子を指定することもできます。形式は次のとおりです:

```swift
(<#identifier 1#>: <#expression 1#>, <#identifier 2#>: <#expression 2#>, <#...#>)
```

_タプル式_の各識別子は、タプル式の範囲内で一意な必要があります。ネストしたタプル式では、同じレベルでネスト識別子を一意にする必要があります。例えば、`(a: 10, a: 20)` はラベル `a` が同じレベルで 2 回使用されているため無効です。ただし、`(a: 10, b: (a: 1, x: 2))` は有効です。`a` は 2 回使用されていますが、外側のタプルに 1 回、内側のタプルに 1 回使用されています。

タプル式には、式を全く含めなくても、2 つ以上の式を含めることもできます。括弧内の単一式は括弧で囲まれた式です。

> NOTE
> 空のタプル式と空のタプル型はいずれもSwiftでは `()` で書きます。`Void` は `()` のタイプエイリアスのため、空のタプル型を書くために使用できます。ただし、全てのタイプエイリアスと同様に、`Void` は常に型で、空のタプル式を書くためには使用できません。

> Grammar of a tuple expression:
>
> *tuple-expression* → **`(`** **`)`** | **`(`** *tuple-element* **`,`** *tuple-element-list* **`)`** \
> *tuple-element-list* → *tuple-element* | *tuple-element* **`,`** *tuple-element-list* \
> *tuple-element* → *expression* | *identifier* **`:`** *expression*

### ワイルドカード式\(Wildcard Expression\)

ワイルドカード式は、代入中に値を明示的に無視するために使用されます。例えば、次の代入式 `10` は `x` に代入されますが、`20` は無視されています:

```swift
(x, _) = (10, 20)
// x は 10 で 20 は 無視されます
```

> Grammar of a wildcard expression:
>
> *wildcard-expression* → **`_`**

### マクロ展開式\(Macro-Expansion Expression\)

*マクロ展開式*は、マクロ名と、その後ろに括弧で囲んだカンマ区切りのマクロ引数のリストで構成されます。
マクロはコンパイル時に展開されます。
マクロ展開式は次のような形式をとります:

```swift
<#macro name#>(<#macro argument 1#>, <#macro argument 2#>)
```

マクロ展開式では、引数がない場合は括弧を省略します。

マクロ式は、Swift 標準ライブラリの[`file`](http://developer.apple.com/documentation/swift/documentation/swift/file)と[`line`](http://developer.apple.com/documentation/swift/documentation/swift/line)マクロを除いて、パラメータのデフォルト値に使用することはできません。

関数やメソッドのパラメータのデフォルト値として使用される場合、これらのマクロの値はデフォルト値式が呼び出し先で評価されたときに決定されます。

> Grammar of a macro-expansion expression:
>
> *macro-expansion-expression* → **`#`** *identifier* *generic-argument-clause*_?_ *function-call-argument-clause*_?_ *trailing-closures*_?_

### <a id="keypath-expression">KeyPath 式\(Key-Path Expression\)</a>

_KeyPath 式_は、型のプロパティまたはサブスクリプトを参照します。key-value observing などのような、動的プログラミングのタスクで KeyPath 式を使用します。次の形式があります:

```swift
\<#type name#>.<#path#>
```

_type name_ は、`String`、`[Int]`、や `Set<Int>` などのジェネリックなパラメータを含めた、具体的な型の名前です。

_path_ は、プロパティ名、サブスクリプト、オプショナルチェーン式、および強制アンラップ式で構成されます。これらの KeyPath コンポーネントのそれぞれは、必要に応じて任意の順序で繰り返すことができます。

コンパイル時には、KeyPath 式は [KeyPath](https://developer.apple.com/documentation/swift/keypath)クラスのインスタンスに置き換えられます。

KeyPath を使用して値にアクセスするには、KeyPath を `subscript(keyPath:)` に渡します。これは全ての型で利用可能です。例えば:

```swift
struct SomeStructure {
    var someValue: Int
}

let s = SomeStructure(someValue: 12)
let pathToProperty = \SomeStructure.someValue

let value = s[keyPath: pathToProperty]
// value は 12
```

_type name_ は、型推論で暗黙的に型を決定できるコンテキストでは省略できます。次のコードは、`\SomeClass.someProperty` の代わりに `\.someProperty` を使用しています:

```swift
class SomeClass: NSObject {
    @objc dynamic var someProperty: Int
    init(someProperty: Int) {
        self.someProperty = someProperty
    }
}

let c = SomeClass(someProperty: 10)
c.observe(\.someProperty) { object, change in
    // ...
}
```

_path_ は、self KeyPath `(\.self)` を作成するために `self` を参照できます。self KeyPath は、インスタンス全体を参照しているので、それを使用して、変数に格納されている全てのデータを単一のステップでアクセスして変更できます。例えば:

```swift
var compoundValue = (a: 1, b: 2)
// compoundValue = (a: 10, b: 20) と同じ
compoundValue[keyPath: \.self] = (a: 10, b: 20)
```

_path_ には、プロパティのプロパティを参照するために、ピリオドで区切って複数のプロパティ名を含めることができます。このコードは、KeyPath 式 `\OuterStructure.outer.someValue` を使用して、`OuterStructure` 型の `outer` プロパティの `someValue` プロパティにアクセスしています:

```swift
struct OuterStructure {
    var outer: SomeStructure
    init(someValue: Int) {
        self.outer = SomeStructure(someValue: someValue)
    }
}

let nested = OuterStructure(someValue: 24)
let nestedKeyPath = \OuterStructure.outer.someValue

let nestedValue = nested[keyPath: nestedKeyPath]
// nestedValue は 24
```

_path_ は、サブスクリプトのパラメータ型が `Hashable` プロトコルに準拠している限り角括弧\(`[]`\)を使用してサブスクリプトを含めることができます。この例では、KeyPath のサブスクリプトを使用して、配列の 2 番目の要素にアクセスしています。

```swift
let greetings = ["hello", "hola", "bonjour", "안녕"]
let myGreeting = greetings[keyPath: \[String].[1]]
// myGreeting は 'hola'
```

サブスクリプトで使用される値は、名前付きの値またはリテラルです。値は Value セマンティクスを使用して KeyPath 内にキャプチャされます。次のコードは、KeyPath 式と `greetings` 配列の 3 番目の要素の両方にアクセスするために、可変の `index` を使用しています。`index` が変更されると、KeyPath 式は依然として 3 番目の要素を参照する一方、クロージャは新しいインデックスを使用しています。

```swift
var index = 2
let path = \[String].[index]
let fn: ([String]) -> String = { strings in strings[index] }

print(greetings[keyPath: path])
// bonjour
print(fn(greetings))
// bonjour

// index に新しい値を設定しても、path には影響しません
index += 1
print(greetings[keyPath: path])
// bonjour

// fn が index を参照するので、新しい値を使用しています
print(fn(greetings))
// 안녕
```

_path_ はオプショナルチェーンと強制アンラップを使用できます。このコードは、オプショナルの文字列のプロパティにアクセスするための KeyPath でオプショナルチェーンを使用しています。

```swift
let firstGreeting: String? = greetings.first
print(firstGreeting?.count as Any)
// Optional(5)

// KeyPath を使用して同じことをしています
let count = greetings[keyPath: \[String].first?.count]
print(count as Any)
// Optional(5)
```

KeyPath のコンポーネントを、型内に深くネストされている値にアクセスために組み合わせることができます。次のコードは、これらのコンポーネントを組み合わせた KeyPath 式を使用して、配列内の辞書のプロパティの様々な値にアクセスしています:

```swift
let interestingNumbers = ["prime": [2, 3, 5, 7, 11, 13, 17],
                          "triangular": [1, 3, 6, 10, 15, 21, 28],
                          "hexagonal": [1, 6, 15, 28, 45, 66, 91]]
print(interestingNumbers[keyPath: \[String: [Int]].["prime"]] as Any)
// Optional([2, 3, 5, 7, 11, 13, 17])
print(interestingNumbers[keyPath: \[String: [Int]].["prime"]![0]])
// 2
print(interestingNumbers[keyPath: \[String: [Int]].["hexagonal"]!.count])
// 7
print(interestingNumbers[keyPath: \[String: [Int]].["hexagonal"]!.count.bitWidth])
// 64
```

関数またはクロージャを使用できるコンテキストでは、KeyPath 式を使用できます。具体的には、`(SomeType) -> Value` 型の関数やクロージャの代わりに、基の型が `SomeType` で、そのパスが `Value` 型の値を生成することができます。

```swift
struct Task {
    var description: String
    var completed: Bool
}
var toDoList = [
    Task(description: "Practice ping-pong.", completed: false),
    Task(description: "Buy a pirate costume.", completed: true),
    Task(description: "Visit Boston in the Fall.", completed: false),
]

// 以下の両方のアプローチは同等です
let descriptions = toDoList.filter(\.completed).map(\.description)
let descriptions2 = toDoList.filter { $0.completed }.map { $0.description }
```

KeyPath 式の副作用は、式が評価される時点でのみ評価されます。例えば、KeyPath 式でサブスクリプトの内側の関数呼び出しを行うと、関数は、KeyPath が使用される度にではなく、式を評価する際に 1 回だけ呼び出されます。

```swift
func makeIndex() -> Int {
    print("Made an index")
    return 0
}
// 下の行は makeIndex() を呼び出します
let taskKeyPath = \[Task][makeIndex()]
// Made an index

// taskKeyPath を使用すると makeIndex() は再び呼び出されません。
let someTask = toDoList[keyPath: taskKeyPath]
```

Objective-C API とやり取りするコード内の KeyPath の使用方法の詳細については、[Using Objective-C Runtime Features in Swift](https://developer.apple.com/documentation/swift/using_objective_c_runtime_features_in_swift)を参照ください。Key-Value Coding や Key-Value Observing については、[Key-Value Coding Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueCoding/index.html#//apple_ref/doc/uid/10000107i)と[Key-Value Observing Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html#//apple_ref/doc/uid/10000177i)を参照ください。

> Grammar of a key-path expression:
>
> *key-path-expression* → **`\`** *type*_?_ **`.`** *key-path-components* \
> *key-path-components* → *key-path-component* | *key-path-component* **`.`** *key-path-components* \
> *key-path-component* → *identifier* *key-path-postfixes*_?_ | *key-path-postfixes*
>
> *key-path-postfixes* → *key-path-postfix* *key-path-postfixes*_?_ \
> *key-path-postfix* → **`?`** | **`!`** | **`self`** | **`[`** *function-call-argument-list* **`]`**

### <a id="selector-expression">Selector 式\(Selector Expression\)</a>

セレクタ式を使用すると、Objective-C のメソッドまたはプロパティの get や set を参照するために使用されるセレクタにアクセスできます。形式は次のとおりです:

```swift
#selector(<#method name#>)
#selector(getter: <#property name#>)
#selector(setter: <#property name#>)
```

メソッド名とプロパティ名は、Objective-C ランタイムで使用可能なメソッドまたはプロパティを参照する必要があります。セレクタ式の値は `Selector` 型のインスタンスです。例えば:

```swift
class SomeClass: NSObject {
    @objc let property: String

    @objc(doSomethingWithInt:)
    func doSomething(_ x: Int) { }

    init(property: String) {
        self.property = property
    }
}
let selectorForMethod = #selector(SomeClass.doSomething(_:))
let selectorForPropertyGetter = #selector(getter: SomeClass.property)
```

プロパティの get のセレクタを作成すると、プロパティ名は変数または定数プロパティを参照できます。対照的に、set のセレクタを作成すると、プロパティ名は変数プロパティのみ参照しなければなりません。

_method name_ は、同じ名前でシグネチャが異なるメソッド間の曖昧さを軽減するために `as` 演算子と一緒にグループ化するための括弧を含めることができます。例えば:

```swift
extension SomeClass {
    @objc(doSomethingWithString:)
    func doSomething(_ x: String) { }
}
let anotherSelector = #selector(SomeClass.doSomething(_:) as (SomeClass) -> (String) -> Void)
```

セレクタが実行時ではなくコンパイル時に作成されるため、コンパイラはメソッドまたはプロパティが存在すること、およびそれらが Objective-C ランタイムに公開されていることを確認できます。

> NOTE
> メソッド名とプロパティ名は式ですが、それらは決して評価されません。

Objective-C API とやり取りする Swift コードでセレクタを使用する方法の詳細については、[Using Objective-C Runtime Features in Swift](https://developer.apple.com/documentation/swift/using_objective_c_runtime_features_in_swift)を参照ください。

> Grammar of a selector expression:
>
> *selector-expression* → **`#selector`** **`(`** *expression* **`)`** \
> *selector-expression* → **`#selector`** **`(`** **`getter:`** *expression* **`)`** \
> *selector-expression* → **`#selector`** **`(`** **`setter:`** *expression* **`)`**

### KeyPath 文字列式\(Key-Path String Expression\)

KeyPath 文字列式を使用すると、Key-Value Coding や Key-Value Observing API で使用するために、Objective-C のプロパティを参照するための文字列にアクセスできます。形式は次のとおりです:

```swift
#keyPath(<#property name#>)
```

_property name_ は、Objective-C ランタイムで使用可能なプロパティを参照する必要があります。コンパイル時には、KeyPath 文字列式は文字列リテラルに置き換えられます。例えば:

```swift
class SomeClass: NSObject {
    @objc var someProperty: Int
    init(someProperty: Int) {
        self.someProperty = someProperty
    }
}

let c = SomeClass(someProperty: 12)
let keyPath = #keyPath(SomeClass.someProperty)

if let value = c.value(forKey: keyPath) {
    print(value)
}
// 12
```

クラス内で KeyPath 文字列式を使用すると、クラス名なしでプロパティ名だけを書くことでそのクラスのプロパティを参照できます。

```swift
extension SomeClass {
    func getSomeKeyPath() -> String {
        return #keyPath(someProperty)
    }
}
print(keyPath == c.getSomeKeyPath())
// true
```

KeyPath 文字列は、実行時ではなくコンパイル時に作成されているため、コンパイラはプロパティが存在すること、およびそのプロパティが Objective-C ランタイムに公開されていることを確認できます。

Objective-C API とやり取りする Swift コードで KeyPath を使用する方法の詳細については、[Using Objective-C Runtime Features in Swift](https://developer.apple.com/documentation/swift/using_objective_c_runtime_features_in_swift)を参照ください。Key-Value Coding と Key-Value Observing については、[Key-Value Coding Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueCoding/index.html#//apple_ref/doc/uid/10000107i)と[Key-Value Observing Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueObserving/KeyValueObserving.html#//apple_ref/doc/uid/10000177i)を参照ください。

> NOTE
> プロパティ名は式ですが、それらは決して評価されません。

> Grammar of a key-path string expression:
>
> *key-path-string-expression* → **`#keyPath`** **`(`** *expression* **`)`**

## <a id="postfix-expressions">後置式\(Postfix Expressions\)</a>

_後置式_は、後置演算子またはその他の後置構文を式に適用することによって形成されます。構文的には、全ての基本式も後置式です。

これらの演算子の動作については、[Basic Operators\(基本演算子\)](../language-guide/basic-operators.md)と[Advanced Operators\(高度な演算子\)](../language-guide/advanced-operators.md)を参照ください。

Swift 標準ライブラリによって提供されている演算子については、[Operator Declarations\(演算子宣言\)](https://developer.apple.com/documentation/swift/swift_standard_library/operator_declarations)を参照ください。

> Grammar of a postfix expression:
>
> *postfix-expression* → *primary-expression* \
> *postfix-expression* → *postfix-expression* *postfix-operator* \
> *postfix-expression* → *function-call-expression* \
> *postfix-expression* → *initializer-expression* \
> *postfix-expression* → *explicit-member-expression* \
> *postfix-expression* → *postfix-self-expression* \
> *postfix-expression* → *subscript-expression* \
> *postfix-expression* → *forced-value-expression* \
> *postfix-expression* → *optional-chaining-expression*

### <a id="function-call-expression">関数呼び出し式\(Function Call Expression\)</a>

_関数呼び出し式_は、関数名とそれに続く関数の引数のカンマ区切りのリストからなる関数名で構成されています。関数呼び出し式は形式は次のとおりです:

```swift
<#function name#>(<#argument value 1#>, <#argument value 2#>)
```

_function name_ は、関数型の任意の式です。

関数定義にパラメータ名が含まれている場合、関数呼び出しは、コロン\(`:`\)で区切られた引数値の前に名前を含める必要があります。この種の関数呼び出し式は形式は次のとおりです:

```swift
<#function name#>(<#argument name 1#>: <#argument value 1#>, <#argument name 2#>: <#argument value 2#>)
```

関数呼び出し式は、閉じ括弧\(`}`\)の直後にクロージャ式の形で末尾クロージャを含めることができます。末尾クロージャは、最後の括弧内の引数の後の関数型の引数と解釈されます。最初のクロージャ式に引数ラベルは付けません。次のクロージャ式の前には引数ラベルを付けます。下記の例は、末尾クロージャの構文を使用して、 末尾クロージャを使用しない関数呼び出しバージョンと同等だということを示しています:

```swift
// someFunction 関数は引数として整数とクロージャを受け取ります
someFunction(x: x, f: { $0 == 13 })
someFunction(x: x) { $0 == 13 }

// anotherFunction 関数は引数として整数と 2 つのクロージャを受け取ります
anotherFunction(x: x, f: { $0 == 13 }, g: { print(99) })
anotherFunction(x: x) { $0 == 13 } g: { print(99) }
```

末尾クロージャが関数の唯一の引数の場合は、括弧を省略できます。

```swift
// someMethod は唯一の引数としてクロージャを受け取ります
myData.someMethod() { $0 == 13 }
myData.someMethod { $0 == 13 }
```

引数に末尾クロージャを含めるために、コンパイラは次のように左から右へ関数のパラメータを調べます:

| 末尾クロージャ | パラメータ | アクション |
| :---: | :---: | :---: |
| ラベルあり | ラベルあり | ラベルが同じ場合、クロージャはパラメータと一致します。それ以外の場合は、スキップされます |
| ラベルあり | ラベルなし | パラメータはスキップされます |
| ラベルなし | ラベルあり/なし | 下記に定義されているように、パラメータが関数型と見なされる場合、クロージャはパラメータと一致します。それ以外の場合は、スキップされます。 |

末尾クロージャは、それが一致する関数のパラメータに渡されます。スキャンプロセス中にスキップされたパラメータには、値が渡されません。例えば、デフォルトのパラメータを使用できます。一致するパラメータを見つけた後、スキャンは次の末尾クロージャと次のパラメータに続きます。マッチングプロセスの最後に、全ての末尾クロージャが一致している必要があります。

_構造上_、パラメータが in-out パラメータではなく、次のいずれかの場合、パラメータは関数型と見なされます:

* `(Bool) -> Int` のようにパラメータの型が関数型
* `@autoclosure () -> ((Bool) -> Int)` のように、ラップされた式の型が関数型の自動クロージャパラメータ
* `((Bool) -> Int)...` のように、配列要素の型が関数型の可変長パラメータ
* `Optional<(Bool) -> Int>` のように、型がオプショナルの 1 つ以上の層にラップされているパラメータ
* `(Optional<(Bool) -> Int>)...` のように、上記の許可された型を組み合わせたパラメータ

末尾クロージャが機能的には関数型のように見えるが関数ではないパラメータと一致する場合、クロージャは必要に応じてラップされます。例えば、パラメータの型がオプショナルの型の場合、クロージャは自動的に `Optional` でラップされます

これは右から左にマッチングを実行していた Swift 5.3 以前のコードから移行を簡単にするために、スキャン方向で異なる結果を生成する場合は、古い右から左へ順序付けされ、コンパイラは警告を生成します。それ以降の Swift のバージョンでは常に左から右へ正しく順序付けします。

```swift
typealias Callback = (Int) -> Int
func someFunction(firstClosure: Callback? = nil,
                  secondClosure: Callback? = nil) {
    let first = firstClosure?(10)
    let second = secondClosure?(20)
    print(first ?? "-", second ?? "-")
}

someFunction()  // - -
someFunction { return $0 + 100 }  // Ambiguous
someFunction { return $0 } secondClosure: { return $0 }  // 10 20
```

上記の例では、"Ambiguous"とマークされている関数の呼び出しは"- 120"が出力され、Swift 5.3 ではコンパイラが警告を生成します。それ以降の Swift のバージョンでは "110 -"が出力されます。

クラス、構造体、または列挙型は、<a href="../language-reference/declarations.md#methods-with-special-names" target="_self">Methods with Special Names(特別な名前のメソッド)</a>で説明されているような、いくつかのメソッドの 1 つを宣言することで、関数呼び出しの糖衣構文(シンタックスシュガー)を使うことができます。

#### <a id="implicit-conversion-to-a-pointer-type">ポインタ型への暗黙変換\(Implicit Conversion to a Pointer Type\)</a>

関数呼び出し式で、引数とパラメータが異なる場合、コンパイラは次のリストの暗黙的な変換の 1 つを適用することによって、その型が一致するようにします。

* `inout SomeType` は、`UnsafePointer<SomeType>` または `UnsafeMutablePointer<SomeType>` になる可能性があります
* `inout Array<SomeType>` は、`UnsafePointer<SomeType>` または `UnsafeMutablePointer<SomeType>` になる可能性があります
* `Array<SomeType>` は、`UnsafePointer<SomeType>` になる可能性があります
* `String` は `UnsafePointer<CChar>` になる可能性があります

次の 2 つの関数呼び出しは同等です:

```swift
func unsafeFunction(pointer: UnsafePointer<Int>) {
    // ...
}
var myNumber = 1234

unsafeFunction(pointer: &myNumber)
withUnsafePointer(to: myNumber) { unsafeFunction(pointer: $0) }
```

これらの暗黙の変換によって作成されたポインタは、関数呼び出しの間だけ有効です。未定義の動作を避けるために、関数呼び出しが終了した後までポインタを保持しないようにしてください。

> NOTE
> 配列を暗黙的に安全でないポインタに変換すると、Swift は、配列のストレージが必要に応じて配列を変換またはコピーすることによって連続していることを保証します。例えば、この構文は、そのストレージに関する API の契約がない\(動作が定義されているか定かではない\) `NSArray` のサブクラスから `Array` にブリッジされた配列でこの構文を使用できます。配列のストレージがすでに連続していることを保証する必要がある場合、暗黙の変換を行わないようにするために、`Array` の代わりに `ContigureArray` を使用します

`withUnsafePointer(to:)` のような明示的な機能の代わりに、`&` を使うことで、低レベルの C 言語の関数を呼び出しやすくするのに役立ちます。ただし、他の Swift コードから関数を呼び出すときは、安全でない API を明示的に使用する代わりとして `&` を使用しないでください。

> Grammar of a function call expression:
>
> *function-call-expression* → *postfix-expression* *function-call-argument-clause* \
> *function-call-expression* → *postfix-expression* *function-call-argument-clause*_?_ *trailing-closures*
>
> *function-call-argument-clause* → **`(`** **`)`** | **`(`** *function-call-argument-list* **`)`** \
> *function-call-argument-list* → *function-call-argument* | *function-call-argument* **`,`** *function-call-argument-list* \
> *function-call-argument* → *expression* | *identifier* **`:`** *expression* \
> *function-call-argument* → *operator* | *identifier* **`:`** *operator*
>
> *trailing-closures* → *closure-expression* *labeled-trailing-closures*_?_ \
> *labeled-trailing-closures* → *labeled-trailing-closure* *labeled-trailing-closures*_?_ \
> *labeled-trailing-closure* → *identifier* **`:`** *closure-expression*

### <a id="initializer-expression">イニシャライザ式\(Initializer Expression\)</a>

_イニシャライザ式_は型のイニシャライザへアクセスします。形式は次のとおりです:

```swift
<#expression#>.init(<#initializer arguments#>)
```

イニシャライザ式を使用して、型の新しいインスタンスを初期化します。スーパークラスのイニシャライザに委譲するイニシャライザ式を使用することもできます。

```swift
class SomeSubClass: SomeSuperClass {
    override init() {
        // subclass の初期化処理をここに
        super.init()
    }
}
```

関数のように、イニシャライザを値として使用することができます。例えば:

```swift
// 型注釈は、String に複数のイニシャイザがあるため必要です
let initializer: (Int) -> String = String.init
let oneTwoThree = [1, 2, 3].map(initializer).reduce("", +)
print(oneTwoThree)
// 123
```

名前で型を指定した場合は、イニシャライザ式を使用せずに型のイニシャライザにアクセスできます。他の場合では、イニシャライザ式を使用する必要があります。

```swift
let s1 = SomeType.init(data: 3)  // 有効
let s2 = SomeType(data: 1)       // これも有効

let s3 = type(of: someValue).init(data: 7)  // 有効
let s4 = type(of: someValue)(data: 5)       // エラー
```

> Grammar of an initializer expression:
>
> *initializer-expression* → *postfix-expression* **`.`** **`init`** \
> *initializer-expression* → *postfix-expression* **`.`** **`init`** **`(`** *argument-names* **`)`**

### <a id="explicit-member-expression">明示的メンバ式\(Explicit Member Expression\)</a>

_明示的メンバ式_では、名前付き型、タプル、またはモジュールのメンバへアクセスできます。アイテムとそのメンバの識別子の間のピリオド\(`.`\)で構成されています。

```swift
<#expression#>.<#member name#>
```

名前付き型のメンバは、型の宣言または extension の一部で指定されます。例えば:

```swift
class SomeClass {
    var someProperty = 42
}
let c = SomeClass()
let y = c.someProperty  // メンバへのアクセス
```

タプルのメンバは、0 から始まる整数が順番に暗黙的に指定されており、使用することができます。例えば:

```swift
var t = (10, 20, 30)
t.0 = t.1
// t は今 (20, 20, 30)
```

モジュールのメンバはそのモジュールの最上位の宣言にアクセスします。

`dynamicMemberLookup` 属性で宣言された型には、[Attributes\(属性\)](attributes.md)で説明されているように、実行時に検索できるメンバが含まれています。

パラメータ名だけが異なるメソッドまたはイニシャライザを区別するには、パラメータ名を括弧内に入れ、パラメータ名の後にコロン\(`:`\)を書きます。名前のない引数にはアンダースコア\(`_`\)を書きます。オーバーロードされたメソッドを区別するには、型注釈を使用してください。例えば:

```swift
class SomeClass {
    func someMethod(x: Int, y: Int) {}
    func someMethod(x: Int, z: Int) {}
    func overloadedMethod(x: Int, y: Int) {}
    func overloadedMethod(x: Int, y: Bool) {}
}
let instance = SomeClass()

let a = instance.someMethod              // あいまい
let b = instance.someMethod(x:y:)        // 明確

let d = instance.overloadedMethod        // あいまい
let d = instance.overloadedMethod(x:y:)  // まだあいまい
let d: (Int, Bool) -> Void  = instance.overloadedMethod(x:y:)  // 明確
```

ピリオドが行の先頭に示されている場合は、暗黙メンバ式としてではなく、明示的メンバ式の一部として解釈されます。例えば、次のリストはメソッドチェーンで呼び出しが複数行にわたって分割された呼び出しを示しています:

```swift
let x = [10, 3, 20, 15, 4]
    .sorted()
    .filter { $0 > 5 }
    .map { $0 * 100 }
```

それぞれのメソッドが呼ばれた時に制御するための条件付きコンパイルブロックを複数行のチェーン構文と組み合わせることもできます。例えば、次のコードは、iOS では異なるフィルタ条件が使用されます:

```swift
let numbers = [10, 20, 33, 43, 50]
#if os(iOS)
.filter { $0 < 40 }
#else
.filter { $0 > 25 }
#endif
```

`#if` と `#endif`、その他のコンパイルディレクティブの間に、条件付きコンパイルブロックに、暗黙メンバ式とそれに続く 0 個以上の接尾辞を含めて、後置式を形成することができます。また、他の条件付きコンパイルブロック、またはこれらの式とブロックの組み合わせも含めることができます。

この構文は明示的メンバ式を記載できるところんまらどこでも使用可能ですが、トップレベルのコードでは使用できません。

条件付きコンパイルブロックの中では、`#if` コンパイルディレクティブの条件分岐は、少なくとも 1 つの式を含めなければなりません。その他の分岐は空でも構いません。

> Grammar of an explicit member expression:
>
> *explicit-member-expression* → *postfix-expression* **`.`** *decimal-digits* \
> *explicit-member-expression* → *postfix-expression* **`.`** *identifier* *generic-argument-clause*_?_ \
> *explicit-member-expression* → *postfix-expression* **`.`** *identifier* **`(`** *argument-names* **`)`** \
> *explicit-member-expression* → *postfix-expression* *conditional-compilation-block*
>
> *argument-names* → *argument-name* *argument-names*_?_ \
> *argument-name* → *identifier* **`:`**

### 後置 self 式\(Postfix Self Expression\)

後置 `self` 式は、型や式の直後に `.self` を付けて構成します。次の形式があります:

```swift
<#expression#>.self
<#type#>.self
```

最初の形式は _expression_ の値に評価されます。例えば、`x.self` は `x` と評価されます。

2 番目の形式は _type_ の値に評価されます。この形式を使用して、型に値としてアクセスできます。例えば、`SomeClass.self` は `SomeClass` 型自体に評価されるため、型レベルの引数を受け取る関数またはメソッドに渡すことができます。

> Grammar of a postfix self expression:
>
> *postfix-self-expression* → *postfix-expression* **`.`** **`self`**

### サブスクリプト式\(Subscript Expression\)

_サブスクリプト式_は、対応するサブスクリプト宣言の get と set を使用してサブスクリプトへのアクセスすることができます。形式は次のとおりです:

```swift
<#expression#>[<#index expressions#>]
```

サブスクリプト式の値を評価するには、_expression_ 型のサブスクリプトの get をサブスクリプトのパラメータとして_インデックス式_を渡して呼び出します。値を設定するために、サブスクリプトの set を同じ方法で呼び出します。

サブスクリプト宣言については、[Protocol Subscript Declaration\(プロトコルサブスクリプト宣言\)](declarations.md#protocol-subscript-declaration)を参照ください。

> Grammar of a subscript expression:
>
> *subscript-expression* → *postfix-expression* **`[`** *function-call-argument-list* **`]`**

### 強制アンラップ式\(Forced-Value Expression\)

_強制アンラップ式_は、特定の値が `nil` ではないオプショナル値を表します。形式は次のとおりです:

```swift
<#expression#>!
```

_expression_ の値が `nil` でない場合、オプショナル値はアンラップされ、対応するオプショナルの非オプショナルの型で返されます。それ以外の場合は、実行時エラーが発生します。

強制アンラップされた値は、値自体を変化させる、またはその値のメンバの 1 つに割り当てることによって、変更できます。例えば:

```swift
var x: Int? = 0
x! += 1
// x は 1

var someDictionary = ["a": [1, 2, 3], "b": [10, 20]]
someDictionary["a"]![0] = 100
// someDictionary は ["a": [100, 2, 3], "b": [10, 20]]
```

> Grammar of a forced-value expression:
>
> *forced-value-expression* → *postfix-expression* **`!`**

### <a id="optional-chaining-expression">オプショナルチェーン式\(Optional-Chaining Expression\)</a>

_オプショナルチェーン式_は後置式で、オプショナル値を使用するための簡単な構文を提供します。形式は次のとおりです:

```swift
<#expression#>?
```

後置 `?` 演算子は式の値を変更せずに式からオプショナルチェーン式を作成します。

オプショナルチェーン式は、後置式で使用しなければならず、後置式を特別な方法で評価します。オプショナルチェーン式の値が `nil` の場合、後置式の他の全ての操作は無視され、後置式全体が `nil` に評価されます。`nil` ではない場合、値はアンラップされ、後置式の残りの部分を評価するために使用されます。どちらの場合も、後置式の値は依然としてオプショナル型です。

オプショナルチェーン式を含む後置式が他の後置式の内側にネストされている場合は、最も外側の式だけがオプショナル型を返します。下記の例では、`c` が `nil` ではない場合、その値はアンラップされ、その値は `.performAction()` を評価するために使用される `.property` を評価するために使用されます。全体の式 `c？.property.performAction()` はオプショナルの型の値を持ちます。

```swift
var c: SomeClass?
var result: Bool? = c?.property.performAction()
```

次の例は、オプショナルチェーンを使用せずに上記の例の動作を表現しています。

```swift
var result: Bool?
if let unwrappedC = c {
    result = unwrappedC.property.performAction()
}
```

オプショナルチェーン式のアンラップ値は、その値自体を変える、またはその値のメンバに代入することで変更できます。オプショナルチェーン式の値が `nil` の場合、代入演算子の右側の式は評価されません。例えば:

```swift
func someFunctionWithSideEffects() -> Int {
    return 42  // 実際の副作用はありません
}
var someDictionary = ["a": [1, 2, 3], "b": [10, 20]]

someDictionary["not here"]?[0] = someFunctionWithSideEffects()
// someFunctionWithSideEffects は評価されません
// someDictionary はまだ ["a": [1, 2, 3], "b": [10, 20]]

someDictionary["a"]?[0] = someFunctionWithSideEffects()
// someFunctionWithSideEffects は評価され、42 を返します
// someDictionary は今 ["a": [42, 2, 3], "b": [10, 20]]
```

> Grammar of an optional-chaining expression:
>
> *optional-chaining-expression* → *postfix-expression* **`?`**
