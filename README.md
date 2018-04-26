# 発生率の作成方法について

## 目的(なぜ書いたか)
下記の色々な集計方式がある。
* A. 期中の発生count / 期初のcount
* B. 期中の発生count / 長さ(対象eventは期間の終わりまでextend)
* C. 期中の発生count / 長さ(対象eventはextendしない)
* D. Cを解約力の最尤推定量とみなして、絶対発生率に変換。1 - exp(-C)
どれが正しい？なぜ使い分けているか？どう使い分けるべきか？本当にあってる？などいつも迷うので整理。

## 発生率について
### 前提
* EA teamが作成しているのは絶対発生率である。(大前提)
* 基本的に一致推定量になっているか、という観点からしかみません。基本的に分散の観点からの話はしない。
* Count baseで考えていくが、Amount baseへ拡張しても一致性は変わらないはず。分散は大きくなるはず。
* event発生までの待ち時間が指数分布に従うモデルを考える。
ベルヌーイ分布ではなく指数分布を使う理由は、脱退までの時間に情報が含まれていると考えた方が自然で、複数の脱退要因がある場合にも分析しやすいため。  
Aだけはベルヌーイ分布を仮定していると考えられる。Aは基本的にlapseの分析でその他の脱退が無視出来るため指数分布を使う必要がない。  
特に長さという単語を使っている場合は指数分布モデルを仮定している。

### 各手法について
#### A
一番基本的。と思いきやそうでもない。これで求められるのは相対発生率であるため、これが使えるのは分析対象以外の脱退要因が存在しないかほぼ無視出来る時のみである事に注意。  
要はlapseには使えるが、mortality, morbidityには使うべきではない。  
* merit
  - lapseのように期間内で強度にスキューがあると考えられる場合でも、正しく推定出来る。A以外は基本的に期間内では強度一定を仮定しており、そうでない場合は分析が歪んでしまう。
  - (一見)シンプル。説明がしやすい。
* demerit
  - 複数の脱退要因がある場合は使用できない。（しづらい？）
  - 各ノードの期間の長さを揃える必要がある。
    - 速報性が失われる。Fully observedで見ないと分析が歪む。
    - Calendar別での分析に使用できない。

#### B, C, D
mortality, morbidityの分析の際はこちらを使用する。
* merit
  - 各ノードの期間の長さを揃える必要がない。
    - Calendar別での分析が可能。速報性がある。
  - 複数脱退要因があっても歪まない。
* demerit
  - 期間内で強度にスキューがある場合は歪む。(強度一定の仮定が必要)

#### B, C, Dの差異について
Dは一致推定量になっておりこれが一番正確。(Cが強度の最尤推定量になっている事は指数分布を仮定すれば簡単に証明できる)  
B, CはDのある種の近似と考えられる。近似精度についてはBの方が圧倒的によい。  
なぜ近似になっているかは
* 脱退要因が１つの時、観測期間の長さ×絶対発生率 ≒ 発生確率となっている。
* 別の脱退要因がある場合、eventが観測されたノードに関しては別の脱退要因が起きた(るであろう)ところまで伸ばした長さを使うのが適切。
詳細はTBD  
近似精度についての詳細は別資料参照。

## A/Eについて(編集中)
ひとつは発生率を経由する方式である。Actual lapse rate/ Expected rapse rate。これはsimpleである。  
しかしこれだと集計区分をマージする事ができないため、基本的には以下の予定発生数を計算して分母とする計算方法が使用されている。  

* A'. 期中の発生count / (期初のcount × 予定絶対発生率)
* B'. 期中の発生count / (長さ(対象eventは期間の終わりまでextend) * 予定絶対発生率)
* C'. 期中の発生count / (長さ(対象eventはextendしない) * 予定絶対発生率)

A'は脱退要因が一つしかない場合(lapseのような)は問題がない事は明らか。  
B', C'は発生率を議論した際と同じideaで考えれば分母が予定発生数の近似になっていることがわかる。  
注意すべきは,Expectedと言いながら期中の分析対象以外の要因による脱退の情報は使ってしまっていることであるが、  
未来の情報でも分析対象以外の脱退の情報であれば使うのがEAチームの分析としては自然。  
(Valuationで使用しているExpectedとは違うかもしれない)  

## QA
* 1件中1件(期央で)起きたら発生率100%と推定すべき？  
→ 二項分布を仮定しているならYES。指数分布を仮定してやっている場合はNO。  
長さ方式でやっていて、伸ばして分母１にしたら100%になるからok、というのはちょっとおかしい。  

* monthly persistency product方式について
TBD  
