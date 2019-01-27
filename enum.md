# TypeScript handbook Enums

https://www.typescriptlang.org/docs/handbook/enums.html

## Enums

### Enums

enums では、名前付きの定数群を定義します。その定数群の内容を明示したり、それぞれ異なるケースを簡単に作成することができます。TypeScript では、数値ベースと文字列ベースの enums を提供しています。

### Numeric enums

まずは、他の言語でも馴染みのある数値 enums を見ていきます。`enum` キーワードを用いて定義します。

```typescript
enum Direction {
  Up = 1,
  Down,
  Left,
  Right
}
```

ここでは、初期値`1`を持つ`Up`の数値 enum を定義しています。それ以降のメンバーは自動的にインクリメントされるので、`Direction.Up`は`1`、`Down`は`2`、`Left`は`3`、`Right`は`4`となります。

また、初期値は省くこともできます。

```typescript
enum Direction {
  Up,
  Down,
  Left,
  Right
}
```

この場合、`Up`は`0`となり、`Down`は`1`となります。このオートインクリメントは、値自体は何でも良いが、同じ enum 内で値を区別するのに役立ちます。

enum の利用法はとてもシンプルです。プロパティにアクセスするように enum のメンバーにアクセスし、enum の名前を使って型を宣言するだけです。

```typescript
enum Response {
  No = 0,
  Yes = 1
}

function respond(recipient: string, message: Response): void {
  // ...
}

respond("Princess Caroline", Response.Yes);
```

数値 enum は、[計算値と定数](#Computed-and-constant-members)を混在させることができます。つまり、初期値のない enum は最初か、数値定数や他の定数 enum のメンバーの初期値を持つ数値 enum の後ろでなければなりません。したがって次のようには宣言できません。

```typescript
enum E {
  A = getSomeValue(),
  B // エラー! 'A' は定数で初期化されていないため、'B'には初期値が必要
}
```

### String enums

文字列 enum も似たようなコンセプトですが、後述するように[実行時のふるまい](#Enums-at-runtime)に少しだけ違いがあります。文字列 enum では、メンバーはそれぞれ文字列リテラルかあるいは他の文字列 enum メンバーを初期値として持たなければなりません。

```typescript
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT"
}
```

文字列 enum はオートインクリメントしませんが、そのかわりうまく"serialize"することができます。例えばデバッグで、数値 enum の実行中の値を読み取ろうとしても、その値自体には意味はない([リバースマッピング](#Enums-at-runtime)はしばしば役に立ちますが)ので困難ですが、文字列 enum であれば、実行中の値からもメンバーの名前以上の意味を汲み取れます。

### Heterogeneous enums

必要性は低いですが、文字列 enum と数値 enum を混在させることは可能です。

```typescript
enum BooleanLikeHeterogeneousEnum {
  No = 0,
  Yes = "YES"
}
```

JavaScript の実行中のふるまいをうまく利用しようとしているのでない場合は、おすすめしません。

### Computed and constant members

enum メンバーはそれぞれ定数か計算値を持ちます。次のように定義した場合、enum メンバーは定数を持っているとみなされます。

- enum の最初のメンバーが初期値を持たない場合

この場合メンバーの値は 0 となります。

```typescript
// E.X は定数:
enum E {
  X
}
```

- 初期値を持たず、かつ一つ前のメンバーが数値定数である場合。

この場合、メンバーの値は一つ前のメンバーの値に 1 を足した数値定数となります。

```typescript
// 'E1' と 'E2' のすべてのメンバーは定数

enum E1 {
  X,
  Y,
  Z
}

enum E2 {
  A = 1,
  B,
  C
}
```

- enum メンバーの初期値が定数 enum 式である場合

定数 enum 式とは、コンパイル時に評価可能な式のことで、次のような場合に当てはまります。

1. リテラル enum 式 (基本的には文字列リテラルか数値リテラル)
2. すでに定義した enum メンバーへの参照 (他の enum でも可能)
3. 括弧付きの定数 enum 式
4. `+`、`-`、`~`の単項演算子が使われた定数 enum 式
5. `+`、`-`、`*`、`/`、`%`、`<<`、`>>`、`>>>`、`&`、`|`、`^`の二項演算子が使われた定数 enum 式。ただし、式の結果が`NaN`か`Infinity`の場合はコンパイルエラーとなります。

上記以外の場合、enum メンバーはすべて計算値としてみなされます。

```typescript
enum FileAccess {
  // 定数メンバー
  None,
  Read = 1 << 1,
  Write = 1 << 2,
  ReadWrite = Read | Write,
  // 計算値メンバー
  G = "123".length
}
```

### Union enums and enum member types

特殊な enum メンバーのひとつに、リテラル enum メンバーがあります。リテラル enum メンバーは、初期値を持たないか、あるいは、次のような初期値を持ちます。

- 文字列リテラル (`foo`、`bar`、`baz`など)
- 数値リテラル (`1`、`100`など)
- マイナスの数値リテラル (`-1`、`-100`など)

すべてのメンバーがリテラル enum メンバーであるとき、いくつかの特殊な構文が使用できます。

ひとつめは、enum メンバーは型としても使用することができるようになります。たとえば、特定のメンバーは、enum メンバーの値だけを持つことができるというように定義することができます。

```typescript
enum ShapeKind {
  Circle,
  Square
}

interface Circle {
  kind: ShapeKind.Circle;
  radius: number;
}

interface Square {
  kind: ShapeKind.Square;
  sideLength: number;
}

let c: Circle = {
  kind: ShapeKind.Square,
  //    ~~~~~~~~~~~~~~~~ エラー!
  radius: 100
};
```

また、enum 型はそれ自身がメンバーの union 型となります。union 型についてはまだ説明していませんが、union enums では、enum 自身に存在する値の集まりであること利用するということを知っておいてください。これにより、TypeScript は値を比較するときに起こしがちなささいなバグを防ぐことが可能になります。次の例を見てください。

```typescript
enum E {
  Foo,
  Bar
}

function f(x: E) {
  if (x !== E.Foo || x !== E.Bar) {
    //             ~~~~~~~~~~~
    // エラー! '!=='オペレーターは'E.Foo'と'E.Bar'には適用できない
  }
}
```

この例では、`x`が`E.Foo`ではないかどうかをチェックしています。このチェックがパスすれば、`||`はスキップされて、`if`文中が実行されます。しかし、前述のチェックがパスしなかった場合、`x`は`E.Foo`に限定されます。そのため、`x`が`E.Bar`であるかどうかを確認することは無駄ということになります。

### Enums at runtime

enums は実行時にも存在する実オブジェクトです。例えば次の enum は

```typescript
enum E {
  X,
  Y,
  Z
}
```

実際に次の関数に渡すことができます。

```typescript
function f(obj: { X: number }) {
  return obj.X;
}

// 'E'は数値のプロパティである'X'を持つため動作する
f(E);
```

#### Reverse mappings

enum はメンバーの名前と同じプロパティを持つオブジェクトを作成することに加えて、enum の値からその名前に対する reverse mapping も生成します。例えば、次のようなコードは

```typescript
enum Enum {
  A
}
let a = Enum.A;
let nameOfA = Enum[a]; // "A"
```

次のような JavaScript にコンパイルされます。

```typescript
var Enum;
(function(Enum) {
  Enum[(Enum["A"] = 0)] = "A";
})(Enum || (Enum = {}));
var a = Enum.A;
var nameOfA = Enum[a]; // "A"
```

生成されたコードからは、enum は順方向(`name` -> `value`)と逆方向(`value` -> `name`)の両方のマッピングを持つオブジェクトにコンパイルされることがわかります。他の enum のメンバーへの参照は常にプロパティアクセスとして出力され、インラインされることはありません。

また、文字列 enum のメンバーは reverse mapping を生成しないことに注意してください。

#### `const` enums

ほとんどのケースでは、enums は有効な解決策ですが、より厳格な条件下で使用したいこともあるでしょう。余分に生成されるコードや enum の値へアクセスする間接的な参照のコストを避けるために、`const` enums を使用することができます。const enums は`const`修飾子を enums につけて定義します。

```typescript
const enum Enum {
  A = 1,
  B = A * 2
}
```

`const` enums は定数 enum 式にのみ用います。通常の enums とは異なり、コンパイル時に完全に取り除かれます。また、const enums メンバーはインラインされます。このために計算値を持ちません。

```typescript
const enum Directions {
  Up,
  Down,
  Left,
  Right
}

let directions = [
  Directions.Up,
  Directions.Down,
  Directions.Left,
  Directions.Right
];
```

上記コードは、次のようにコンパイルされます。

```typescript
var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */];
```

### Ambient enums

ambient enums はすでにある enum 型を宣言するために使用します。

```typescript
declare enum Enum {
  A = 1,
  B,
  C = 2
}
```

ambient と ambient ではない enums の大きな違いは、通常の enums では前のメンバーが定数であり、かつ、初期値を持たなければそのメンバーも定数とみなされるのに対して、ambient (const ではない) enum メンバーは初期値を持たなければ、常に計算値としてみなされます。
