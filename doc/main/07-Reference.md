# 参照
クラスの新しい機能を説明する前に、ちょっとコーヒーブレイク。とは言っても結構重要な機能です。

## なぜ配列以外でもポインタを使うのか
メンバ変数がクソみたいに多い構造体のお話。例えば、

```cpp
struct STRUCT {
	char c0001;
	char c0002;
	char c0003;
	// ... 省略 ...
	char c1022;
	char c1023;
	char c1024;
};
```

こんな感じの構造体があるじゃろ？`char`型のサイズは1バイトだから、この構造体のサイズは1024バイト、つまり1KBになるわけだ。

これを関数の引数として渡す。こんな感じで。

```cpp
void test(STRUCT s) {
	// なにかする
}

int main() {
	STRUCT s;
	test(s);

	return 0;
}
```

こうすると、以下のようなことが起こる。

1. 1KB分の変数`s`が`main()`関数内で確保される
2. `test()`関数の呼び出し。`main()`関数内の`s`が **コピーされて** `test()`関数の方に渡る
3. `test()`関数終了。コピーされた`s`が解放される(スコープを抜ける)
4. `main()`関数終了。変数`s`が解放される(スコープを抜ける)

細かいことを抜きにすると、このプログラムは最大で約 **2KB** のメモリを食う。`test()`関数の引数は値渡しで、変数が **まるごとコピー** されるようになっているからだ。

とてもヤダね。

ポインタ使おう。

```cpp
void test(STRUCT* ps) {
	// なにかする
}

int main() {
	STRUCT s;
	test(&s);

	return 0;
}
```

これだと、変数`s`で1KB。その他はもはや誤差程度のメモリしか食わない。`test()`関数の引数が参照渡しになったことで、変数`s`がまるごとコピーされずに済んだ。

ポインタを使う意味は、「配列を扱える」以外にもこうした使い方があると思う。

## でもさ、ポインタってぶっちゃけめんどくね？？？
わかる:clap: アドレス操作とかしない時でも`*`とかつけたり`&`とかつけたり面倒臭い。もっと簡単なの無いのか？？？

あります。「参照」という機能です。

### 構文

```cpp
型名& 参照名 = 変数;
```

### 例

```cpp
#include <iostream>
using namespace std;

int main() {
	int number = 810;      // 普通の変数
	int& rNumber = number; // 参照

	cout << rNumber << '\n';

	return 0;
}
```

### 実行結果
> 810

### 解説
なんと、`rNumber`を出力したら`number`が出力されました！

簡単に言うと、変数に別名をつけることができます。この例だと、`number`の別名として`rNumber`という名前をつけました。

別名をつけただけなので、`number == rNumber`は当然真になるし、`&number == &rNumber`も真になります。

参照は関数の引数にも使えます(むしろこっちがメイン)。例えば、2つの変数を入れ替える`swap()`関数。C言語ではこのように作っていました。

```c
#include <stdio.h>

void swap(int* n1, int* n2) {
	int tmp = *n1;
	*n1 = *n2;
	*n2 = tmp;
}

int main() {
	int n1 = 810;
	int n2 = 893;

	printf("%d %d\n", n1, n2);
	swap(&n1, &n2);
	printf("%d %d\n", n1, n2);

	return 0;
}
```

> 810 893  
> 893 810

これをC++の参照を使って書くとこうなります。

```cpp
#include <iostream>
using namespace std;

void swap(int& n1, int& n2) {
	int tmp = n1;
	n1 = n2;
	n2 = tmp;
}

int main() {
	int n1 = 810;
	int n2 = 893;

	cout << n1 << ' ' << n2 << '\n';
	swap(n1, n2);
	cout << n1 << ' ' << n2 << '\n';

	return 0;
}
```

> 810 893  
> 893 810

参照を関数の引数で使うと、自動で参照渡しをしてくれるようになります。アドレスをほとんど意識すること無く、参照渡しで処理ができていることが分かります。

・・・ということは、前節の例、1つ`&`をつけるだけでメモリを食いつぶす問題が解決します。

```cpp
void test(STRUCT& s) {
	//          ^ ここに & をつけるだけ！
	// なにかする
}

int main() {
	STRUCT s;
	test(s);

	return 0;
}
```

メチャメチャ楽に解決。このように、C++で関数に構造体やクラス変数を渡すときは、大抵は参照を使って渡します。

<[前に戻る](06-DynamicAllocation.md) - [目次へ](../../README.md) - [次へ進む](08-StaticMember.md)>
