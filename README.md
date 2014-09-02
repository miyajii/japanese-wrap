# Japanese Wrap Package

Word wrap for Japanese text.

The following text is written in Japanese, because this package is made for Japanese users. If you do not understand Japanese, this package is worthless.

# 日本語用ワードラップパッケージ
このパッケージは日本語文章での、画面幅(または指定文字数)での改行処理行うAtomのパッケージです。標準のSoft Wrap機能を上書きする形となり、改行の際には下記処理を行います。

* 英文のみの場合は標準のSoft Wrapとほぼ同じ処理をします。
* 2倍幅の文字(主にShfit_JISにおける2バイト文字)を2倍幅で計算します。
* 日本語における禁則処理を行います。

文字コードに関してはUnicode 7.0を参照しています。Unicodeのバージョンが上がった場合はその都度対応予定です。

## 使い方
1. Atom標準のSoft Wrapを有効にします。関連のある設定は下記になります。
    * Preferred Line Length
    * Soft Wrap
    * Soft Wrap At Preferred Line Length
2. パッケージをインストールします。
3. Atomの表示を再読み込みします。うまく反映されない場合は再起動してみてください。

無効にする場合は、パッケージをDisabledにするか、Uninstallしてください。
なお、標準のSoft Wrapが無効の場合は、ラップ機能は動作しません。

## 動作仕様
### 英文(ASCII文字)のみの場合でも異なる動作
下記の点が標準のSoft Wrapと異なります。

* 行幅を超えるwordの場合は強制的に行幅で改行します。

### 禁則処理
JIS X 4051を参考に作成されたW3C技術ノート「日本語組版処理の要件」2012年4月3日版に準じます。
http://www.w3.org/TR/jlreq/ja/

JIS X 4051そのものでは無いことに注意してください。また、縦組みのみ対象となる表記に関しては基本的に無視しています。

#### ぶら下げ
##### 空白文字のぶら下げ
標準のSoft Wrapと同じく、後に続く文字が空白文字のみの場合は行幅を越えてぶら下げを行います。ただし、下記動作が異なります。

* 和文間隔(U+3000)をぶら下げ対象の空白文字に含めません。直前で改行されて行頭に来る場合があります。
  ※標準のSoft Wrapは/\s/で比較しており、CoffeeScript(JavaScript)の\sには和文間隔(U+3000)が含まれます。\sが含む文字はMozillaの資料を見てください。
  https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/RegExp

##### 句読点のぶら下げ
句読点についてぶら下げを行います。対象は設定で変更可能です。

* 全角句読点 デフォルト有効
* 半角句読点 デフォルト有効
* 全角ピリオド/コンマ デフォルト無効
* 半角ピリオド/コンマ デフォルト無効

#### 禁則処理の例外
W3Cの資料と異なる点を記載します。

##### 基本ラテン文字領域にある記号の扱い
基本ラテン文字領域の記号に対して行頭禁則と行末禁則は行わず、代わりに半角・全角形の全角文字を対象とします。

##### 半角文字
半角・全角形の半角文字(JIS X 0201 片仮名図形文字集合)は設定で禁則処理の対象にするかを選択できます。デフォルトは有効です。

##### 縦組みの「く」の字記号
組み合わせでの禁止がうまくできていません。縦組みにしか使われないため、常に横組み表示のAtomでは無視すべきかもしれません。
* 〳 U+3033
* 〴 U+3034
* 〵 U+3035

##### 処理が難しい文字
小書き片仮名フと半濁点付きの組み合わせは二つで1文字扱いとするのですが、処理が複雑になるため実装していません。また、本来は「プ」の小文字表記になるべきですが、ほとんどの処理系では正しく表示されません。
* ㇷ゚ U+31F7, U+309A

### 文字幅
Atomでは"x"の文字幅を基準として1行に入る文字数を計算しています。このパッケージでも同様に"x"の何倍の幅であるかどうかを各文字に割り当てて、計算していきます。

Shift_JISでのいわゆる2バイト文字は2倍幅とします。Shift_JISにマッピングされていない文字は「ＭＳ　ゴシック」(MS Gothic)での文字幅を参考にして規則を作成しています。「ＭＳ　ゴシック」に含まれない文字は似たようなカテゴリーは似たように処理することを原則とします。下記説明では一部重なる文字があります。計算の順序として、0倍幅、1倍幅、2倍幅と確認していくため、前の定義が優先となります。

なお、なるべく文字のカテゴリーで処理を行うため、厳密ではありません。また、Shfit_JIS自体についても、Windowsの実装であるCP932のみではなく、Mac OS XのShift_JIS拡張も参考にしています。

#### 0倍幅で計算する文字
サロゲートペアは上位で文字幅を決めるため、下位部分は常に0倍幅で計算します。

* 下位サロゲート(U+DC00-DFFF)

ゼロ幅文字およびLRMとRLMを0倍幅で計算します。

* ゼロ幅文字およびLRM/RLM(U+200B-200F)
* ゼロ幅非改行空白文字(U+FEFF)
  ※BOMとして使用されている場合も同様です。

前の文字と結合させる文字(/^COMBINING /という名称)を0倍幅で計算します。

* ダイアクリティカルマーク(U+0300-036F)
* ダイアクリティカルマーク補助
* 記号用ダイアクリティカルマーク
* 結合濁点(U+3099)
* 結合半濁点(U+309A)

#### 1倍幅で計算する文字
Shfit_JISにおける1バイト文字(JIS X 0201)である半角文字は1倍幅で計算します。これには記号や拡張や変形を含めたラテン文字が含まれます。

* 基本ラテン文字(U+0000-007F)
* ラテン補助1(U+0080-00FF)
* ラテン文字拡張A/B(U+00100-024F)
* IPA拡張
* 前進を伴う修飾文字
* 和字間隔(U+3000)を除く空白文字(U+2000-200A)

半角片仮名等も1倍幅で計算します。なお、Shift_JISに含まれない"HALFWIDTH *"というUnicodeも1倍幅にします。これにはハングルの半角文字が含まれています。

* 半角・全角形の半角文字 U+FF61-FFDC U+FFE8-FFEE

Mac OS XにおけるShfit_JISで1バイトになる文字も1倍幅で計算します。なお、いくつかは上とかぶります。

* バックスラッシュ \ U+005C (Shfit_JIS:80, CP932:5C)
* ノーブレークスペース U+00A0 (Shfit_JIS:A0)
* 半角円記号 ¥ U+00A5 (Shift_JIS:5C)
* 著作権表示記号 © U+00A9 (Shift_JIS:FD)
* トレードマーク記号 ™ U+2122 (Shift_JIS:FE)

そのほか、CJK互換文字以外はなるべく1倍幅にしていますが、厳密ではありません。

#### 2倍幅で計算する文字
Shift_JISにおける2バイト文字(JIS X 0208)は基本的に2倍幅で計算します。

* 記号
* CJK互換文字
* 半角・全角形の全角文字 U+FF01-FF60 U+FFE0-FFE6

次の文字はデフォルトでは2倍幅で計算しますが、オプションで選択できるようにする予定です。これはShfit_JISでは2バイト文字であり、MS Gothicでは全角幅になっていますが、フォントによってはラテン文字とほぼ同じ幅の場合があるためです。使用するフォントによって変えてください。

* ギリシャ文字及びコプト文字
* キリル文字

上の文字で注意が必要なのは、Shift_JISに含まれない基本形以外の文字も2倍幅で計算することです。多くの日本語フォントではこれらの文字はラテン文字と同幅になりますので、正しくありません。しかし、処理を単純化するために全ての同じ幅としています。この動作は将来変更する可能性があります。

#### その他の文字
上記に含まれない文字は範囲指定として、下記の通りとします。

* U+0000-1FFF 1
* U+2000-FEFF 2
* U+FF00-FFFF 1
* U+10000-1FFFF 1
* U+20000-2FFFF 2
* U+30000-DFFFF 1 現在未割り当て
* U+E0000-EFFFF 1
* U+F0000-FFFFF 2 私用領域
* U+100000-10FFFF 2 私用領域

なお、U+10000以降はサロゲートペアで上位で幅を計算し、下位は0倍幅としています。

## 既知の問題点
* パッケージの有効無効をしても、既に開いているファイルに対してうまく動作しません。
* サロゲートペアが正しくないときの動作は未定義です。
  たぶん、おかしくなります。
* specのテスト用例文がEngrishです。

## TODO
* ギリシャ文字及びコプト文字とキリル文字の扱いに関するオプション機能。
* 使用フォントから文字幅を割り出して計算する機能。
  重くなりすぎて使い物にならない可能性があるので、実装するかは未定です。
* ぶら下げの実装。
* "COMBINING *"な文字の処理。でも、そもそもくっついて表示されない場合が多そうだから、意味なし？
