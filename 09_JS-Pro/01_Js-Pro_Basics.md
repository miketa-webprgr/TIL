## JS-Pro 基本
---

### 変数を扱う３つの理由
1. 繰り返すことができる
2. 意味が分かりやすくなる
3. 変数として定義した一連の処理を微修正して活用することができる

### 変数(let)
- 再度代入して、上書きすることができる
- Js-Proには出て来ていないが、varというものがある

### 定数(const)
- 上書きできない

### テンプレートリテラル
- Rubyでいうところの #{} に該当する
- `${}`とバッククオートで囲む

### if と switch
- 分岐するような場合に使える
- switchではあまり、演算子を使うような書き方はしないとのこと  

### if と switch の使い分け  
- 一般的には、ifが使われることが多いようだ（ざっと調べた限り）
- switchには、分岐の数が多い場合、メンテナンスしやすいという点がある
  - そのような場合に限って、switchを使うということが一般的らし
  - 不等号などを使うような場合、switchは使わないことが多いらしい

ここで、詳細な解説があったので、流し読みをした。  
[switch\(true\) イディオム考察 \- Qiita](https://qiita.com/t_uda/items/1969e09a970d71e4cfd6)  

### 演算子
なぜJSは === と３回も使うのか疑問だったが、  
実は === は厳密等価演算子といって、普通の等価演算子である == と違うらしい。  

== の場合、型が一致するかどうかまでは判断しない。  
5 == '5' の場合は true となるが、5 === '5' は false となる。  

[JavaScript 忘れがちな === と == の違い \- Qiita](https://qiita.com/PianoScoreJP/items/e43d70ec188c6fed73ed)  


```js
let age = 30;
if(age > 40){
  console.log("おじさんです");
}else if (age >= 20){
  console.log("成人です");
}
else {
  console.log("未成年です")
}
```

```js
let man = "社長";
switch(man){
  case "社長":
    console.log("社長、おはようございます！！！");
    break;
  case "上司":
    console.log("おはようございます！");
    break;
  default:
    console.log("おっはー");
    break;
}
```

```js
let age = 30;
switch(true){
  case age > 40:
    console.log("おじさんです");
    break;
  case age >= 20:
    console.log("成人です");
    break;
  default:
    console.log("未成年です");
    break;
}
```

### 反復処理(whileとfor)

whileはコードが長くなるが、直感的に分かりやすい。  

```js
let number = 1;
while (number <= 20 ){
  console.log(number);
  // number += 1 と同じ
  number ++;
}
```

forはコードが短く書けるが、慣れが必要。  

```js
for(let number = 1; number <= 20; number ++){
  console.log(number);
}
```

forの構造を頭に叩き込むこと。  

```js
for(宣言; ループが続く条件式; 次のループに移る時の処理){
  各ループでの処理
}
```

### 配列
- []で囲むやつ

```js
fruits = ["apple", "orange", "banana"]
fruits[0] = "apple"

// Rubyと一緒で、最初の要素は0で取り出せる。
```

- 反復処理を活用して、数字を一つずつ取り出す
- lengthという「メソッド」（rubyの言い方なのでおそらく違う）を使うとよい

```js

const fruits = ["apple", "orange", "banana"];
for(let f = 0; f < fruits.length; f ++){
  console.log(fruits[f]); 
}
```

### オブジェクト
- 要素は、オブジェクトである
- オブジェクトには、プロパティと値がある
  - Ruby でいうところの属性と値のようなもの？

```js
// name, price, color は プロパティ
/// "apple", 140, "red" は値

const fruits = [
  { name: "apple", price: 140, color: "red" };
  { name: "orange", price: 180, color: "orange" };
  { name: "banana", price: 200, color: "yellow" };
]
```

オブジェクトごとではなく、その要素を取り出したいときは、以下のように取り出す。  
console.log(オブジェクト名[インデックス番号].プロパティ名)

```js
// fruitsオブジェクトの最初の要素からプロパティを取り出す場合

console.log(fruits[0].color) // red
```

### 関数
- まとめた処理を関数として扱うことができる
- 関数を定数と定義することで、呼び出すことが可能になる

```js
// 定数fruitsとして、以下の３つの処理を１つの関数として扱える。
const fruits = function(){
// アロー関数だと、const fruits = () => { という表記になる。 
  console.log("I like apples");
  console.log("I don't like oranges");
  console.log("I like bananas the best);
}

// 以下のコードだけで３つの処理を実行できる
fruits();
```


### 引数

```js
const drink = (name, size) => {
  console.log( `${name}の${size}を飲みたい` );
};

drink("抹茶ミルク", "L"); // 抹茶ミルクのLを飲みたい
```

### 戻り値(return)

関数内の処理で得た結果（値）を再利用したい場合、  
return を使うとよい。

```js
const addConsumptionTax = ( price, taxRate ) => {
  return price * ( 1 + taxRate );
}

const totalPrice = addConsumptionTax( 1000, 0.1 );
console.log(税込価格は、${totalPrice}円です);

// 出力内容：税込価格は、1,100円です
```

なお、returnを使わないとどうなるか。  
結果は以下のとおりである。  

```js
const addConsumptionTax = ( price, taxRate ) => {
  price * ( 1 + taxRate );
}

const totalPrice = addConsumptionTax( 1000, 0.1 );
console.log(`税込価格は、${totalPrice}円です`);

addConsumptionTax( 1000, 0.1 );

// 出力内容：税込価格は、undefinedです
```

単純に同じように出力したいだけなら、returnを使わずとも、以下のように書けなくもない。  
ただ、応用が効きづらくなる。  

```js
const addConsumptionTax = ( price, taxRate ) => {
  console.log(`税込価格は、${price * ( 1 + taxRate )}円です`);
}

addConsumptionTax( 1000, 0.1);

// 出力内容：税込価格は、1,100円です
```

### クラス

- クラスは設計書、インスタンスはその設計書に基づいて作られたオブジェクト
- クラスとインスタンスは対概念（Rubyと同じ）

### constructor
- クラスには、必ずconstructorがある
  - そのクラスを生成する際に必ず実行されるメソッドは、この中に記述する

### this
- contructor{} の中で使う
- これは、Rubyでいうところのinitializeメソッド
- なお、JSpreierをざっと見た限り、他にもthisには使い方があるもよう

```js
class Car {
  constructor() {
    this.vehicleType = "EV車";
  }
}

const car = new Car();
console.log(car.vehicleType); // EV車
```

### メソッド
- function(){} で行動を追加することができる
- JSの場合、以下に注意（メソッドには関係ない）
  - Car.newではなく、new Car()
  - 変数を定義するときは、letもしくはconstを失念しないこと

```js
class Car {
  constructor() {
    
  }
    run(){
        console.log("平均燃費は7km/kWhです");
    }  
}
  
const car = new Car();
car.run();
```

### クラス内の別のメソッドの呼び出し
- thisを使うと、クラス内で別のメソッドを呼び出せる

```js
class Car {
  constructor() {
  }
  run() {
    console.log("平均燃費は7km/kWhです");
  }
  info() {
    // infoメソッド内でrunメソッドを呼び出す
    this.run();
  }
  
}
```

### クラスの継承
extendsを使うと、親クラスのプロパティやメソッドを継承できる。  

```js
class Bus extends Car {
}

const bus = new Bus();

// Carクラスにあるinfoメソッドを引き継いでいるため、Busクラス内での定義付けは不要  
bus.info ();
```

なお、親クラス内のメソッドの戻り値を使うことができる。  

```js
class Car{
  constructor(gasoline, efficiency){
    this.gasoline = gasoline;
    this.efficiency = efficiency;
  }
}

class EV extends Car{
  getDriveDistance(){
  return this.gasoline * this.efficiency;
  }
}

const ev = new EV(20,10);

const driveDistance = ev.getDriveDistance();

console.log(`走行距離は${driveDistance}kmです`) // 走行距離は200kmです 
```

また、引数の理解が怪しかったので、以下のように書けることをテストした。  

```js
class Car{
  getDriveDistance(gasoline, efficiency){
  return gasoline * efficiency;
  }
}

class EV extends Car{
}

const ev = new EV( ); // ここで引数を記入しても、constructorがないので反映されない

const driveDistance = ev.getDriveDistance(20,10);

console.log(`走行距離は${driveDistance}kmです`)// 走行距離は200kmです
```

正解不正解はなく、クラスの性質を理解して、どちらに書くのが適切か判断することが重要。  
なお、今回の場合、EV（電気自動車）特有のことを書いているわけではないので、  
下の書き方の方が適切であるように思う。  

### オーバーライド
- 親クラスのメソッドを小クラスの同名メソッドで上書きすることができる。
- 親クラスのコンストラクタを呼び出し、上書きすることができる。
  - 下記のとおり、superを使う

```js
class Sample {
  constructor(ex1, ex2) {
    this.ex1 = ex1;
    this.ex2 = ex2;
  }
}

class Sample2 extends Sample {
  constructor(ex1, ex2, ex3) {
    super(ex1, ex2);// 親クラスの this.ex1 = ex1; this.ex2=ex2 を引っ張ってこれる
    this.ex3 = ex3;
  }
```

### import と export
- プログラムをモジュールに分けて管理することができる
- モジュールを関連付ける場合、エクスポートとインポートを組み合わせる
  - 出力元のファイルにエクスポートと書き、出力先のファイルにインポートと書く
  - クラス、関数、数値、文字列などをエクスポートできる
  - 複数の関数などをエクスポートする場合、{}で囲む
  - export defaultは、１つのファイルにつき１回しか使えない

```js
import add from "./export";

const sum = add( 2 , 6 );
console.log(sum); // 8
```

```js
// export.jsファイル
function add (a, b) {
  return a + b;
}
export default add;
```

### メソッド全般

### コールバック関数

### pushメソッド

### popメソッド

### reduceメソッド

### everyメソッド

```js
const numbers = [54, 42, 39, 13, 8];

/* 定数「flag」に、15以上の数値を「false」が返るまで出力してください
　 また、出力した数値が15以上かを「true/false」で表示してください */
const flag = numbers.every(num => {
    console.log(num, num >= 15);
    return num > 15;
}) ; 

// 定数「flag」を出力してください
console.log(flag);
```

