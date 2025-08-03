# 私の質問：  
昔の話で申し訳ないが、MS-DOSの時代では、DisplayBIOS（INT 10H)を直接叩いて、キー入出力や画面のクリアなどを行った記憶があります。現代のLinuxPC（今回は３２ビットの環境で行います）で同じことを行おうとすると、どのような方法が考えられますか？  
  
  
# ChatGPT:  
ご質問のように、MS-DOS 時代に INT 10h（BIOS割り込み）で画面制御やキー入力を直接操作していた頃と同じようなことを、現代の Linux（32bit環境）で再現するには、アプローチが大きく異なります。以下に、目的別にその代替手段を体系的にまとめます。  
  
✅ 結論：Linux では BIOS 直叩きは不可。代替は主に以下の3つ  

|目的	|MS-DOS の方法|	Linux での代表的代替方法|
|-----|------------|------------------------|
|画面出力	|INT 10h AH=0Eh	|ncurses / ANSI escape sequences|  
|画面クリア	|INT 10h AH=06h	|ANSI "\033[2J" や ncurses::clear()|  
|カーソル移動	|INT 10h AH=02h	|ANSI "\033[<row>;<col>H"|  
|キー入力	|INT 16h AH=00h	|termios + read() / ncurses::getch()|  
|キャラクタ取得	|直接 VRAM アクセス	|仮想端末(TTY)から read() / termios|  
|ランダム生成	|DOS API か RTC	|rand() / /dev/urandom / getrandom()  
  
🔧 方式①：ANSI エスケープシーケンス（最も簡易）  
特徴：  
どの Linux 端末（tty, xterm, gnome-terminal）でも動く  
  
標準出力で printf() するだけで画面制御が可能  
  
例：  
c  
コピーする  
編集する  
printf("\033[2J");            // 画面クリア  
printf("\033[5;10H");         // 5行10列へカーソル移動  
printf("★");                 // 表示  
これは ATXY + DRAW-CHAR に相当します。  
  
🔧 方式②：ncurses ライブラリ（より堅牢）  
特徴：  
curses（Unix系の画面制御ライブラリ）の現代版  
  
入出力・色・ウィンドウなど多数の機能あり  
  
Forthから呼び出すにはCレベルでバインディングが必要  
  
Cコード例：  
c  
コピーする  
編集する  
#include <ncurses.h>

initscr();            // ncurses初期化
clear();              // 画面クリア
move(5, 10);          // カーソル移動
addch('*');           // キャラ表示
refresh();            // 表示更新
endwin();             // 終了
Linuxでの Forth 実装（例えば Gforth）では libc 経由で ncurses 呼び出し可能です。

🔧 方式③：termios + read() によるキー入力制御
c
コピーする
編集する
#include <termios.h>
#include <unistd.h>

struct termios oldt, newt;
tcgetattr(STDIN_FILENO, &oldt);
newt = oldt;
newt.c_lflag &= ~(ICANON | ECHO);
tcsetattr(STDIN_FILENO, TCSANOW, &newt);

char c;
read(STDIN_FILENO, &c, 1); // ノンエコー・ノンブロッキング入力

tcsetattr(STDIN_FILENO, TCSANOW, &oldt);
この方法は KEY?, GETCHAR, LEFT?, RIGHT? のような Forth ワードにマッピングできます。  
  
🚫 なぜ BIOS 割り込みは使えないのか？  
保護モード以降（386以降）は、OSがBIOSを直接呼び出さない  
→ Linuxはすべてユーザーモードのアプリケーションに I/O を制限しているため。  
  
仮想端末（TTY）や X 上の端末は BIOS と無関係  
→ CPUレベルの割り込み呼び出し (INT) は特権モードでしか許可されない。  
  
Linuxカーネルがすでに抽象化された I/O 層を提供している  
→ write(), read(), ioctl() を通して行う。  
  
🧠 もし「BIOS 的なこと」がしたければ？  
DOSEMU2 などで DOS プログラムを実行すれば、 INT 10h も利用可。  
  
もしくは、LinuxのForth環境に libc や ncurses をバインドして、再構成する。  
  
🧩 実用例：Forthで ANSI 制御ワードを定義する  
forth  
コピーする  
編集する  
: cls ( -- )  27 emit [char] [ emit [char] 2 emit [char] J emit ;
: atxy ( x y -- ) 27 emit [char] [ emit . [char] ; emit . [char] H emit ;
: draw-char ( x y c -- )
   rot rot atxy emit ;
これは非常に DOS 的な操作を ANSI で再現する形です。  
  
✅ 結論まとめ  
実現手段	特徴	適用例  
ANSI 制御文字	手軽・互換性高い	テキストForth, bash, printf  
ncurses	画面制御とキー入力一括対応	ゲーム・GUI的なForth  
termios + read()	低レベル入出力制御、BIOS的な挙動に近い	入力処理カスタム時  
dosemu/Bochs	BIOS INTを本当に使う場合	過去の16bitコード再利用  
  
必要であれば：  
  
ncurses を C で呼び出すコード例  
  
forth-dwc に EMIT, ATXY, KEY? を追加する方法  
  
termios をForthから呼び出す方法  
  
などもすぐ提示できます。どの方向性で進めたいかお聞かせください。  






# 私の質問：  
いろいろな実現手段を教えてくれてありがとう。勉強のために、「termios をForthから呼び出す方法」を教えてもらえますか？  
  
  
# ChatGPT:  
こちらこそ、前向きなご質問をありがとうございます。  
termios を Forth から呼び出す方法は、  
Forth の特性（C言語との連携のしやすさ）を活かす非常に良い学びの題材です。  
  
✅ 全体像  
Forth から termios.h の関数を使いたい場合、以下のような方法が考えられます：  
  
主な方法  
方法	内容	難易度	備考  
① Gforth の libcc を使う	Forth から C 関数を直接呼び出す	中	Linux/Gforth 向け  
② 外部 Cライブラリを作る	termios 関数を C でまとめて dlopen()	中上	複数関数をまとめやすい  
③ Forth 処理系を自作/拡張	Forth 系統の処理系に Cコードを埋め込む	上	forth-dwc の改造など  
  
ここでは、**Gforth + libcc（libc 経由）で termios を呼び出す最も実践的な方法（①）**を解説します。  
  
🛠 Gforth で termios を使う方法（libcc）  
Gforth には libcc.fs という標準機能があり、C言語の関数を直接呼び出すことができます。  
  
🔽 準備：libcc の読み込み  
forth
コピーする
編集する
\ libcc.fs をインクルード
require libcc.fs
🔽 構造体 termios を使うには
C構造体のポインタ操作は基本的に Gforth では以下のように行います：

構造体サイズを自分で確保

C 関数を呼び出す

たとえば、以下のような最小構成が可能です。  

🔽 最小例：非カノニカルモードを設定して1文字入力  
forth
コピーする
編集する
\ 必要な libc 関数を読み込む
c-library termios

s" termios.h"    add-header

\ 定義する関数群（termios版）
c-function tcgetattr tcgetattr a n -- n  ( termios構造体取得 )
c-function tcsetattr tcsetattr n a n -- n  ( termios構造体設定 )

c-function getchar getchar -- n

end-c-library
🔽 termios を用いて raw 入力に切り替える
forth
コピーする
編集する
\ 構造体 termios 用にメモリを確保
\ 必要サイズは大体 60〜64バイト
create oldt 64 allot
create newt 64 allot

\ 読み取りモード設定
: enable-raw ( -- )
  0 oldt tcgetattr drop         \ 現在の設定を取得
  oldt 64 newt swap move        \ コピー
  newt c@ 0x10 invert and newt c!   \ ICANON を無効化
  newt c@ 0x08 invert and newt c!   \ ECHO を無効化
  0 newt 0 tcsetattr drop       \ 変更適用
;

: disable-raw ( -- )
  0 oldt 0 tcsetattr drop
;

: get-one-char ( -- c )
  getchar
;

\ 使用例：  
enable-raw  
get-one-char .  \ 入力文字のASCIIコード表示  
disable-raw  
🔍 解説ポイント  
項目	解説  
tcgetattr / tcsetattr	Cの端末I/O制御関数。stdin（FD=0）を対象  
create oldt 64 allot	Cのstruct termiosをエミュレートするためにバイト配列を確保  
ICANON / ECHO 無効化	c@ でフラグ取得 → and / invert でビットマスク処理  
getchar	ノンバッファで1文字読み込みする libc 関数  
  
✅ 結果的にできること  
これにより：  
  
Forth プログラム中で「1文字ずつ即時入力」を実現  
  
KEY?, LEFT?, RIGHT? などの自作が可能  
  
ANSI escape 等と組み合わせてゲームなどのリアルタイム処理が可能  
  
✅ 応用可能なワード  
たとえば：  
  
forth
コピーする
編集する
: key-left? ( c -- f )  27 = ;
: key-right? ( c -- f )  108 = ; \ lキーなど
とすれば、Forth の update-input 部分に活用できます。

✅ 補足：実行の注意点
必ず最後に disable-raw を呼び出すこと（さもないと端末壊れます）

getchar などは libc に依存するので、libcc 経由で安全に呼び出すこと

デバッグしながら一つずつ確かめると安心です

📦 提供可能なもの
ご希望あれば：

上記を forth-dwc に組み込むためのヘッダ／Cラッパー

key-left? などの判定ワード一覧

DRAW-CHAR, SLEEP, RANDOM との統合例

も提示できます。

✍️ まとめ
termios を Forth から使うには libcc.fs（Gforth）が非常に有効。

tcgetattr / tcsetattr を Cのように扱えるので、MS-DOS時代のような「直接端末制御」が可能。

Forth のワード設計と組み合わせることで、「リアルタイム・キャラ入力・画面描画」が一通り作れます。

続きを進めたい場合：

「このコードを実際にテストしたい」

「この機能を forth-dwc に組み込む方法を知りたい」

など、気軽に聞いてください！






# 私の質問：  
どちらも魅力的な提案です。  
まずは実際に動かしてみたいのですが、準備や手順など必要な作業を教えてもらえますか？  


# ChatGPT:  
　　＜保存されたメモリを更新しました＞  
素晴らしいです！  
ここでは、Debian/Linux (32bit) 環境で Gforth を使い、termios をForthから呼び出すためのステップを、初心者にもわかりやすく整理します。  
  
✅ 目標  
Gforth を使って termios による「1文字入力モード」に切り替え  
  
Forth で getchar を直接呼び出して、キー入力をリアルタイムに取得  
  
Forth プログラムとして実行可能な形で準備  
  
🧰 必要なもの  
項目	内容
OS	Debian系 Linux（32bitでOK）
Forth処理系	Gforth（aptで入手可能）
エディタ	任意（vim, nano, etc）
ターミナル	標準の端末（xterm, gnome-terminal, etc）

🧾 ステップバイステップ手順
✅ ステップ1：Gforth をインストール
bash
コピーする
編集する
sudo apt update
sudo apt install gforth
確認：

bash
コピーする
編集する
gforth --version
✅ ステップ2：Forth ファイルを作成（例：rawinput.fs）
以下の内容を rawinput.fs として保存：

forth
コピーする
編集する
\ rawinput.fs -- termios を使った1文字入力モード

require libcc.fs

\ Cの termios 関数読み込み
c-library termios
s" termios.h" add-header
c-function tcgetattr tcgetattr a n -- n
c-function tcsetattr tcsetattr n a n -- n
c-function getchar getchar -- n
end-c-library

\ termios 構造体用バッファ
create oldt 64 allot
create newt 64 allot

\ ICANON=0x02, ECHO=0x08 を削除
: enable-raw ( -- )
  0 oldt tcgetattr drop
  oldt 64 newt swap move
  newt c@ 16 invert and newt c!
  newt c@ 8 invert and newt c!
  0 newt 0 tcsetattr drop
;

: disable-raw ( -- )
  0 oldt 0 tcsetattr drop
;

: get-one-char ( -- c )
  getchar
;

: test-raw-input ( -- )
  enable-raw
  ." Press any key (q to quit)..." cr
  begin
    get-one-char dup emit
    dup [char] q = if drop disable-raw exit then
    cr
  again
;
✅ ステップ3：実行してみる
bash
コピーする
編集する
gforth rawinput.fs
Gforth の中で：

forth
コピーする
編集する
test-raw-input
と打ち込むと、非カノニカルモードになり、キーを1文字打つたびに表示されます。
q を打つと disable-raw が呼ばれて終了します。

📌 注意点と安全性
注意点	内容
disable-raw を忘れず呼ぶ	端末が壊れて入力できなくなる可能性あり
Ctrl+C が使えなくなる可能性も	ICANONとECHOを無効にするため
デバッグは少しずつ	1つずつワードを試していくのがおすすめ

🎁 発展例（次にやってみたいこと）
key-left?, key-right? を定義してゲーム入力対応

atxy, draw-char, clear-screen を ANSI 制御で実装

RANDOM, SLEEP を C経由で追加（必要なら手順も教えます）

✅ まとめ  
ステップ	内容
1. Gforthインストール	sudo apt install gforth
2. Forthファイル作成	rawinput.fs にtermios連携コードを書く
3. 実行テスト	gforth rawinput.fs → test-raw-input 実行

次に進む候補：  
  
ANSIコードで画面制御 (atxy, draw-char) の定義  
  
キー識別のための key-left?, key-right?  
  
これらを統合して Forth版「川下り」ミニゲームに仕上げる  
  
ご希望があればどれからでも着手できます。どうされますか？  
  
  
  
[ トップに戻る](README.md)  




