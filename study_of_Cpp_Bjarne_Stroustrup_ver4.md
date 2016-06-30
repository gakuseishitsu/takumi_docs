# ビャーネ・ストラウストラップ プログラミング言語C++ [第4版]

## abstract
 C++のお勉強のために「ビャーネ・ストラウストラップ プログラミング言語C\+\+[第4版]」を読んで, ためになった部分をここにまとめていく.

## Index
* [第1章 本書の読み進め方](#第1章-本書の読み進め方)
* [第2章 C\+\+を探検しよう:基本](#第2章-C\+\+を探検しよう:基本)
* [第3章 C\+\+を探検しよう:抽象化のメカニズム](#第3章-C\+\+を探検しよう:抽象化のメカニズム)

## 第1章 本書の読み進め方
- **1.3.2 C++11の新機能について**
 1.　コンストラクタによって不変条件を確立する(メンバの範囲等を明らかにする). (§2.4.3.2)
 2.　コンストラクタとデストラクタの組み合わせによって, 資源管理を単純化する. (§5.2)
 3.　裸のnewとdelete演算子を避ける. (§3.2.1.2, §11.2.1)
 4.　大規模オブジェクトのコピーを避けるために, ムーブセマンティクスを使用する. (§3.2.2, §17.5.2)
 5.　多層型のオブジェクトの参照に, unique\_ptrを使用する. (§5.2.1)
 6.　解体の責任を持つ所有権を一意に特定できない共有オブジェクトの参照には, shared\_ptrを使用する. (§5.2.1)
 7.　C\+\+98とC\+\+11の詳しい差は (§44.2)

## 第2章 C\+\+を探検しよう:基本
- **2.2.3 定数**
 -　const : この値を変更しないことをコンパイラに対して約束する.
 -　constexpr : この値はコンパイル時に評価される.
```cpp
const int dmv = 17; //dmvは名前の付いた定数
int val = 17; //varは定数でない
constexpr double max1 = 1.4*square(dmv); //square(17)が定数であればOK
constexpr double max2 = 1.4*square(var); //エラー : valは定数でない
const double max3 = 1.4*square(var); //OK : 実行時に評価される
double sum(const vector<double>&); //sumは実引数を変更しない
vector<double> v {1.2, 3.4, 4.5};
const double s1 = sum(v); //OK : 実行時に評価される
constexpr double s2 = sum(v); //エラー : sum(v)は定数式でない
```
 -　constexprは例えば, 単に演算結果をreturnするi行だけのようなできるだけ単純なものがいい.
```cpp
constexpr double square(double x) {return x*x;}
```
- **2.3.2 構造体**
 -　構造体/クラス関数への値渡し, ポインタ渡し, 参照渡しの違い
 　-　値渡し : 渡された変数の値が別のアドレスにコピーされる. だから関数内で変更しても, 呼び出しもとの変数の値は変化しない.
 　-　ポインタ渡し : 渡された変数のアドレスを指すポインタが別のアドレスに割り当てられてそれを通じて呼び出し元の変数の値を操作できる, だから関数内で変更すると, 呼び出し元も変更される.
 　-　参照渡し : 渡された変数は呼び出し元の変数と値, アドレス含め全く同じになる. これはコピーではないので当然関数ないで変更すると呼び出し元の値も変化する.
```cpp
void f(Vector v, Vector&rv, Vector* pv){
	int i1 = v.sz;
	int i2 = rv.sz;
	int i3 = pv->sz;
}
```
- **2.3.3 列挙体**
 -　クラス以外のユーザ定義型で比較的小さな整数値の集合を表すのに好都合.
 -　列挙型はユーザ定義型なので, 演算子も定義できる.
```cpp
enum class Traffic_light{green, yellow, red};
Traffic_light light = Traffic_light::red;
　
Traffic_light& operator++(Traffic_light& t){
	switch(t){
		case Traffic_light::green: return t=Traffic_light::yellos;
		case Traffic_light::yellow: return t=Traffic_light::red;
		case Traffic_light::red: return t=Traffic_light::green;
	}
}
　
Traffic_light next = ++light;
```

## 第3章 C\+\+を探検しよう:抽象化のメカニズム
- **2.2.3 定数**