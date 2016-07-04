# ビャーネ・ストラウストラップ プログラミング言語C++ [第4版]

## abstract
- C++の勉強のために「ビャーネ・ストラウストラップ プログラミング言語C\+\+[第4版]」を読んで, ためになった部分をここにまとめていく.
- 使用しているmarkdown用のエディタはHaroopadで必ずしもgithubでキレイに表示されるとは限らない.

## Index
* [第1章 本書の読み進め方](#chap1)
* [第2章 C\+\+を探検しよう:基本](#chap2)
* [第3章 C\+\+を探検しよう:抽象化のメカニズム](#chap3)
* [第4章 C\+\+を探検しよう:コンテナとアルゴリズム](#chap4)
* [第5章 C\+\+を探検しよう:並行処理とユーティリティ](#chap5)
* [第6章 型と宣言](#chap6)
* [第7章 ポインタと配列と参照](#chap7)
* [第8章 構造体と共用体と列挙体](#chap8)
* [第9章 文](#chap9)
* [第10章 式](#chap10)
* [第11章 主要な演算子](#chap11)
* [第12章 関数](#chap12)
* [第13章 例外処理](#chap13)

<h2 id="chap1">第1章 本書の読み進め方</h2>
- **1.3.2 C++11の新機能について**  
 1. コンストラクタによって不変条件を確立する(メンバの範囲等を明らかにする). (§2.4.3.2)  
 2. コンストラクタとデストラクタの組み合わせによって, 資源管理を単純化する. (§5.2)  
 3. 裸のnewとdelete演算子を避ける. (§3.2.1.2, §11.2.1)  
 4. 大規模オブジェクトのコピーを避けるために, ムーブセマンティクスを使用する. (§3.2.2, §17.5.2)  
 5. 多層型のオブジェクトの参照に, unique\_ptrを使用する. (§5.2.1)  
 6. 解体の責任を持つ所有権を一意に特定できない共有オブジェクトの参照には, shared\_ptrを使用する. (§5.2.1)  
 7. C\+\+98とC\+\+11の詳しい差は (§44.2)  

<h2 id="chap2">第2章 C++を探検しよう:基本</h2>
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
 -　オペレータの前に&は別につけなくてもいいが, 意味合いとしてオペレータは参照を表すのでこの書き方に従ったほうが良い.  
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

<h2 id="chap3">第3章 C++を探検しよう:抽象化のメカニズム</h2>
- **3.2.4 クラス階層**  
  -　空き領域に確保したオブジェクトのポインタを関数が返却することは以下の点で危険である.  
  　-　ユーザは返却されたポインタのdeleteに失敗するかもしれない.  
  　-　返却されたポインタが示すオブジェクトをユーザが認知せずdeleteしないかもしれない.  
  -　そこでポインタを返却する場合にはunique_ptrを用いると解決できる.  
```cpp
enum class Kind {circle, triangle, smiley};
　
unique_ptr<Shape>read_shape(istream& is){
	//... shapeの先頭部をisから読み込んでKind kを判断
	switch (k){
		case Kind::circle:
			// circleのデータ{Point,int}をpとrに読み込む
			return unique_ptr<Shape>{new Circle{p,r}};
		//...
	}
}
　
void user(){
	vector<unique_ptr<Shape>> v;
	while (cin)
		v.push_back(read_shape(cin));
	draw_all(v); // 全要素に対してdraw()を呼び出す
} // すべてのShapeは暗黙裏に解体される
```
- **3.3.1 コンテナのコピー**  
  -　あるクラスがポインタ経由でオブジェクトにアクセスしなければならないクラスであれば, デフォルトのメンバ単位のコピーは, 災難となる場合がほとんど.  
  -　よってコンテナ等のコピーには**コピーコンストラクタ**と**コピー代入**を定義する.  
```cpp
class Vector{
provate:
	double* elem; // elemはsz個のdouble配列へのポインタ
	int sz;
public:
	Vector(int s); // コンストラクタ : 不変条件を確立して資源を獲得
	=Vector(){delete[] elem;} //デストラクタ : 資源を開放
　
	Vector(const Vector& a); //コピーコンストラクタ
	Vector& operator=(const Vector& a); //コピー代入
　
	double& operator[](int i);
	const double& operator[]{int i} const;
　
	int size() const;
};
　
Vector::Vector(const Vector& a)
	:elem{new double[a.sz]},
	 sz {a.sz}
{
	for (int i=0;i!=sz;++i){
		elem[i] = a.elem[i];
	}
}
　
Vector& Vector::operator=(const Vector& a){
	double* p = new double[a.sz];
	for (int i=0;i!=a.sz;++i)
		p[i] = a.elem[i];
	delete[] elem; //古い要素をデリート
	elem = p;
	sz = a.sz;
	return *this;
}
```
- **3.3.2 コンテナのムーブ**  
  -　コピーコンストラクタ等を用いてコンテナのコピーができるが, コンテナが大規模の場合には厄介となる. ムーブコンストラクタによって単に関数から演算結果を取り出すことができる.  
  -　以下のように定義すると, 関数の外に演算結果を転送する際に, コンパイラはムーブコンストラクタを使うようになる. 例えばr=x+y+ZではVectorは一切コピーされることなく, ムーブされる.  
  -　"&&"は”右辺値参照"を意味し, 他の誰も代入を行えない何かを参照するものであり, 安全に値を盗むことができる.  
```cpp
class Vector{
	//...
	Vector(const Vector& a); //コピーコンストラクタ
	Vector& operator=(const Vector& a); //コピー代入
　
	Vector(Vector&& a); //ムーブコンストラクタ
	Vector& operator=(Vector&& a); //ムーブ代入
}
　
Vector::Vector(Vector&& a)
	:elem{a.elem}, //aから要素を盗む
	sz{a.sz}
{
	a.elem = nullptr; // もはやaには要素は全くない
	a.sz = 0;
}
```
- **3.3.4 演算子の抑制**  
  -　階層内のクラスに対して, デフォルトのコピーやムーブを利用すると惨事につながる.  
  -　以下のように書くと, デフォルトのコピー演算, ムーブ演算を削除できる(デフォルト定義の削除).  
```cpp
class Shape{
public:
	Shape(const Shape& a) =delete;
	Shape& operator=(const Shape&) =delete;
　
	Shape(Shape&&) =delete;
	Shape& operator=(Shape&&) =delete;
}
```
- **3.4.4 可変個引数テンプレート**  
  -　任意の型と任意の個数の引数を受け取ることができる.  
  -　先頭のheadに対して何等かの処理を行って, それ以降の引数tailに対してf()を再帰的に呼びだす.  
```cpp
void f(){} //何も行わない
template<typename T>
void g(T x){cout << x << " ";}
　
template<typename T, typename... Tail>
void f(T head, Tail... tail){
	g(head);
	f(tail...);
}
　
int main(){
	f(1, 2.2, "hello"); //output: 1 2.2 hello
	f(0.2, 'c', "yuck!", 0, 1, 2); // output: 0.2 c yuck! 0 1 2
}
```
- **3.4.5 別名**  
  -　usingで型やテンプレートに同義語を与える.  
```cpp
using size_t = unsigned int;
```
  -　例えば標準コンテナは, 自身が保持する値の型の名前としてvalue_typeを提供し, この規約に従うあらゆるコンテナに対して動作するコードが記述できる.  
```cpp
template<typename T>
class Vector{
public:
	using value_type = T;
	// ...
}
　
template<typename C>
using Value_type = typename C::valuetype; // Cの要素の型
　
template<typename Container>
void algo(Container& c){
	Vector<Value_type<Container>> vec; // ここに結果を保存
	// ... vecを利用 ...
}
```
  -　別名は, テンプレートの引数の一部またはすべてをバインドして, 新しいテンプレートを定義できる.  
```cpp
template<typename Key, typename Value>
class Map{
	// ...
};
　
template<typename Value>
using String_map = Map<string, Value>;
　
String_map<int> m; // mはMap<string, int>
```

<h2 id="chap4">第4章 C++を探検しよう:コンテナとアルゴリズム</h2>
- **4.5.1 反復子の利用**  
  -　反復子を使うと, アルゴリズムとコンテナが分離できる. アルゴリズムは反復子を介してデータを処理するが, そのデータが格納されているコンテナについては何も知らないし, コンテナも自身のデータが適用されるアルゴリズムについて何も知らない. ただ要求に応える形でbegin()やend()などの反復子を提供する.  
  -　以下に反復子を使ったコンテナに対するアルゴリズムの例を示す. find_allはC中のvをすべて探すアルゴリズムである.  
```cpp
template<typename T>
using Iterator = typename T::iterator; //Tの反復子
　
template<typename C, typename V>
vector<Iterator<C>> find_all(C& c, V v){
	vector<Iterator<C>> res;
	for(auto p = c.begin();p!c.end();++p)
		if (*p==v)
			res.push_back(p);
	return res;
}
　
void test{
	string m {"Mary had a little lamb"};
	for(auto p : find_all(m,'a')) // pはstring::iterator
		if(*p='a')
			cerr << "string bug!\n";
　
	list<double> ld {1.1, 2.2, 3.3, 1.1};
	for(auto p : find_all(ld, 1.1))
		if (*p!=1.1)
			cerr << "list bug!\n";
}
```
- **4.5.3 ストリーム反復子**  
  -　ストリームにも反復子がある. 通常はistream_iteratorやostream_iteratorを直接利用することはなく, アルゴリズムの引数として与えることが多い.  
  -　以下にファイルを読み取って, 単語をソートして重複するものを除去してその結果をファイルへと書き出すプログラムを示す.  
```cpp
int main(){
	string from, to;
	cin >> from >> to; // 入力元と出力先のファイル名を読み込む
　
	ifstream is {from}; // ファイル"from"の入力ストリーム
	istream_iterator<string> ii {is}; // streamへの入力反復子
	istream_iterator<string> eos {}; // 入力終端のための番兵
　
	ofstream os {to}; // ファイル"to"への出力ストリーム
	ostream_iterator<string> oo {os, "\n"}; //streamへの出力反復子, 第二引数は出力する値の区切り
　
	vector<string> b {ii, eos}; // bは入力で初期化されるvector
	sort(b.begin(), b.end()); // バッファをソート
	unique_copy(b.begin(), b.end(), oo); // 重複なく出力にバッファをコピー
	return !is.eof() || !os; // エラーの状態を返却
}
```

<h2 id="chap5">第5章 C++を探検しよう:並行処理とユーティリティ</h2>
- **5.4.1 時間**  
  -　処理時間を測定するときは以下のようにする.  
```cpp
using namespace std::chrono;
auto t0 = high_resolution_clock::now();
do_work();
auto t1 = high_resolution_clock::now();
cout << duration_cast<milliseconds>(t1-t0).count() << "msec\n";
```

<h2 id="chap6">第6章 型と宣言</h2>
- **6.3.5 初期化**  
  -　初期化子には以下の4つがある. このうちあらゆる局面で利用できるのは先頭のみ. よって筆者は先頭を勧めている.  
```cpp
X a1 {v};
X a2 = {V};
X a3 = v;
X a4(v);
```
  -　autoを用いる場合には=を用いたほうが良い.  
```cpp
auto z1 {99}; // z1はinitializer_list<int>
auto z2 = 99; // z2はint
```
- **6.3.6.1 auto型指定子**  
  -　autoは型を記述したり取得したりするのに手間がかかるときに便利になる.  
  -　特に理由がない場合, 狭いスコープではautoを使うべき.  
```cpp
template<typename T> void fi(vector<T>& arg){
	for (typename vector<T>::iterator p = arg.begin();p!=arg.end();++p)
		*p = 7;
	for (auto p = arg.begin();p!=arg.end();++p)
		*p = 7;
}
```
- **6.3.6.3 decltype型指定子**  
  -　floatでできた行列とdoubleでできた行列を足す関数を作るときにdecltypeで返り値の型を宣言できる.  
```cpp
template<typename T, typename U>
auto operator+(const Matrix<T>& a, const Matrix<U>& b) -> Matrix<decltype(T{}+U{})>;
　
template<typename T, typename U>
auto operator+(const Matrix<T>& a, const Matrix<U>& b) -> Matrix<decltype(T{}+U{})>{
	Matrix<decltype(T{}+U{})> res;
	for (int i=0;i!=a.rows();++i)
		for (int j=0;j!=a.cols();++j)
			res(i,j) += a(i,j) + b(i,j);
	return res;
}
```

<h2 id="chap7">第7章 ポインタと配列と参照</h2>
- **7.3 配列**  
  -　配列は静的にも割り当てられるし, スタック上にも割り当てられるし, 空き容量にも割り当てられる.  
  -　空き容量に割り当てられた配列は使用後にそのポインタを一回だけdelete[]しなければいけない. これを簡単に行うには空き容量上の配列を資源ハンドル(string, vector, unique_ptr等)にまかせる.  
  -　静的やスタック上に割り当てた場合はdelete[]してはいけない.  
```cpp
int a1[10]; // 静的記憶領域に格納される10個のint
　
void f(){
	int a2[20]; // スタック上に格納される20個のint
	int*p = new int[40]; // 空き領域に格納される40個のint
}
```
- **7.4.1 配列の操作**  
  -　以下のコードは実質的な速度差はない, 好きなほうを選べばよい  
```cpp
void f(char v[]){
	for (int i=0;v[i]!=0;++i)
		use(v[i]);
　
	for (char* p=v;*p!=0;++p)
		use(*p);
}
```

<h2 id="chap8">第8章 構造体と共用体と列挙体</h2>
- **8.2.1 structのレイアウト**  
  -　structのオブジェクトはメンバが宣言順番に格納されるが, マシンによって一定の境界にアラインされるため, メンバの合計のバイト数が構造体のバイト数にはならない. よってメモリを最適化したいならば, メンバは大きさ順で宣言したほうがいい.  
- **8.4 列挙体**  
  -　新しく追加されたenum classとの差  
　-　enum class : 列挙子の名前のスコープは, enum内に局所的であり, その値が他の型へと暗黙的に変換されない.  
　-　単なる enum : 列挙子の名前はenumと同じスコープに入って, その値は整数へと暗黙的に変換される.  
  -　予想外の動作を防ぐためには基本的にはenum classを使ったほうがいい.  
```cpp
enum class Traffic_light { red, yellow, green};
enum class Warning { green, yellow, orange, red};
　
Warning a1 = 7; // エラー : int->Wrning変換はない
int a2 = green; // エラー : greenはスコープにない
int a3 = Warning::green; // エラー : Warning->int変換はない
Warning a4 = Warning::green; // OK
　
void f(Traffic_light x){
	if(x){} // エラー : 0との暗黙的な比較はない
	if(x == 9){} // エラー : 9はTraffic_lightではない
	if(x == red){} // エラー：redはスコープ中にない
	if(x == Warning::red){} // エラー : xはWarningではない
	if(x == Traffic_light::red){} //OK
}
```
- **8.4.1 enum class**  
  -　以下のようなenum classやオペレータ|や&を定義することで安全かつ簡潔に利用できる.  
```cpp
enum class Printer_flags{
	none = 0,
	acknowledge = 1,
	paper_empty = 2,
	busy = 4,
	out_of_black = 8,
	out_of_color = 16,
	//
};
　
constexpr Printer_flags operator|(Printer_flags a, Printer_flgs b){
	return static_cast<Printer_flags>((static_cast<int>(a)|(static_cast<int>(b));
}
constexpr Printer_flags operator&(Printer_flags a, Printer_flgs b){
	return static_cast<Printer_flags>((static_cast<int>(a)&(static_cast<int>(b));
}
　
void g(Printer_flgs x){
	switch (x){
	case Printer_flags::acknowledge:
		//...
		break;
	case Printer_flags::busy:
		//...
		break;
	case Printer_flags::out_of_black|Printer_flags::out_of_color:
		//...
		break;
	//...
	}
}
```

<h2 id="chap9">第9章 文</h2>
- **特筆することはない**  

<h2 id="chap10">第10章 式</h2>
- **特筆することはない**  

<h2 id="chap11">第11章 主要な演算子</h2>
- **特筆することはない**  

<h2 id="chap12">第12章 関数</h2>
- **12.2.1 参照引数**  
  -　引数の受け渡しは以下のように選択するのがよい(Vectorとかは例外みたい?)  
　-　1. 小規模オブジェクトには, 値渡しを用いる.  
　-　2. 変更する必要がない大規模オブジェクトにはconst参照渡しを用いる  
　-　3. 引数を通してオブジェクトを変更せずに, returnによって値を返す  
　-　4. ムーブと完全転送の実装では, 右辺値参照を用いる  
　-　5. オブジェクトが存在しないことがありうる場合にはポインタを用いる. このときオブジェクトが存在しないことはnullptrで表現する  
　-　6. どうしても必要な場合のみ参照渡しを用いる  
- **12.2.5 デフォルト引数**  
  -　クラスにおいて本当のコンストラクタを一個だけにして処理を一か所に集中させると, 重複の繰り返しが不要になる.  
```cpp
class complex {
	double re, im;
public:
	complex(double r,double i) :re{r},im{i}{} //二個のスカラからcomplexを構築
	complex(double r) :re{r}, im{0}{} // 一個のスカラからcomplexを構築
	complex() :re{0}, im{0}{} //デフォルトのcomplexは{0,0}
}
　
class complex {
	double re, im;
public:
	complex(double r,double i) :re{r},im{i}{} //二個のスカラからcomplexを構築
	complex(double r) :complex{r,0} {} // 一個のスカラからcomplexを構築
	complex() :complex{0,0} {} //デフォルトのcomplexは{0,0}
}
　
class complex {
	double re, im;
public:
	complex(double r = {},double i = {}) :re{r},im{i} {}
}
```

<h2 id="chap13">第13章 例外処理</h2>
- **13.1.1 a**  