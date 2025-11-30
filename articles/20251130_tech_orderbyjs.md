---
title: "SBI証券（Web版）で注文を高速化する：IFDOCO自動入力ブックマークレット公開"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["SBI証券", "ブックマークレット", "java script"]
published: true
---
## はじめに
株のトレードでは、エントリーと同時に利確と損切をセットで発注することが基本です。しかし、Web版の注文画面では入力項目が多く、次のような悩みを持つ人も少なくありません。

- HyperSBIアプリのワンクリック注文ほど簡易化したくない（誤発注が怖い）
- けれど、Web画面で毎回すべてを手入力するのは遅い
- 寄り付きなど短い勝負時間では、入力の遅れがチャンスロスになる

そこで本記事では、**ブックマークレット（JavaScriptを登録したブックマーク）を使って、IFDOCO注文の入力を半自動化**し、比較的短時間で発注する方法を紹介します。なお、**本記事では取引パスワードの自動入力は扱いません**。以下の通り危険なためです。安全性を保ちながら入力作業だけ高速化したい人向けの記事です。
- ブックマークに平文で残る
ブックマークを開くだけで誰でもJSコードが読めます。
- ブラウザ同期で他端末にもコピーされる
スマホ・自宅PC・仕事PCにそのまま同期され、漏えいリスクが上がります。
- バックアップ・共有で簡単に漏れる
ブックマークをHTMLで保存したときも、パスワードがそのまま書き出されます。

## 1. 自動入力で注文を時短する方法
手順は以下の通りです。
### 1-1.ブックマークレット（bookmarks.html）をブラウザにインポート
以下をhtmlとして保存し、ブラウザにインポートします。
:::details bookmarks.html
```html:bookmarks.html
<!DOCTYPE NETSCAPE-Bookmark-file-1>
<!-- This is an automatically generated file.
     It will be read and overwritten.
     DO NOT EDIT! -->
<META HTTP-EQUIV="Content-Type" CONTENT="text/html; charset=UTF-8">
<TITLE>Bookmarks</TITLE>
<H1>Bookmarks</H1>
<DL><p>
    <DT><H3 ADD_DATE="1723822370" LAST_MODIFIED="1764491349" PERSONAL_TOOLBAR_FOLDER="true">ブックマーク バー</H3>
    <DL><p>
        <DT><A HREF="javascript:(function(){    const v1 = Number(document.getElementById('ifoco_input_price')?.value);%20%20%20if%20(!isNaN(v1))%20{%20%20%20%20%20const%20v2%20=%20v1%20+%2010;%20%20%20%20%20const%20v3%20=%20v1%20+%2015;%20%20%20%20%20if%20(document.getElementById(%27doneoco2_input_trigger_price%27))%20document.getElementById(%27doneoco2_input_trigger_price%27).value%20=%20v2;%20%20%20%20%20if%20(document.getElementById(%27doneoco2_gsn_input_price%27))%20document.getElementById(%27doneoco2_gsn_input_price%27).value%20=%20v3;%20%20%20}%20%20%20%20const%20skipCb%20=%20document.getElementById(%27shouryaku%27);%20%20%20if%20(skipCb)%20{%20%20%20%20%20skipCb.checked%20=%20true;%20%20%20%20%20if%20(typeof%20bottonChange%20===%20%22function%22)%20bottonChange();%20%20%20}%20%20%20%20%20%20%20const%20btn2%20=%20document.getElementById(%27botton2%27);%20%20%20if%20(btn2)%20btn2.click();%20%20})();" ADD_DATE="1752505105">【売り】IFDOCO_損切ライン自動入力～注文</A>
        <DT><A HREF="javascript:(function(){    const v1 = Number(document.getElementById('ifoco_input_price')?.value);%20%20%20if%20(!isNaN(v1))%20{%20%20%20%20%20const%20v2%20=%20v1%20-%2010;%20%20%20%20%20const%20v3%20=%20v1%20-%2015;%20%20%20%20%20if%20(document.getElementById(%27doneoco2_input_trigger_price%27))%20document.getElementById(%27doneoco2_input_trigger_price%27).value%20=%20v2;%20%20%20%20%20if%20(document.getElementById(%27doneoco2_gsn_input_price%27))%20document.getElementById(%27doneoco2_gsn_input_price%27).value%20=%20v3;%20%20%20}%20%20%20%20(function%20syncLimit()%20{%20%20%20%20%20%20const%20r1%20=%20document.querySelector(%27input[name=%22ifoco_selected_limit_in%22]:checked%27);%20%20%20%20%20if%20(r1)%20{%20%20%20%20%20%20%20const%20targetRadio%20=%20document.querySelector(`input[name=%22doneoco_selected_limit_in%22][value=%22${r1.value}%22]`);%20%20%20%20%20%20%20if%20(targetRadio)%20targetRadio.checked%20=%20true;%20%20%20%20%20}%20%20%20%20%20%20const%20sel1%20=%20document.querySelector(%27select[name=%22ifoco_limit_in%22]%27);%20%20%20%20%20const%20sel2%20=%20document.querySelector(%27select[name=%22doneoco_limit_in%22]%27);%20%20%20%20%20if%20(sel1%20&&%20sel2)%20{%20%20%20%20%20%20%20sel2.value%20=%20sel1.value;%20%20%20%20%20}%20%20%20})();%20%20%20%20%20%20const%20skipCb%20=%20document.getElementById(%27shouryaku%27);%20%20%20if%20(skipCb)%20{%20%20%20%20%20skipCb.checked%20=%20true;%20%20%20%20%20if%20(typeof%20bottonChange%20===%20%22function%22)%20bottonChange();%20%20%20}%20%20%20%20%20%20%20const%20btn2%20=%20document.getElementById(%27botton2%27);%20%20%20if%20(btn2)%20btn2.click();%20%20})();" ADD_DATE="1764490625">【買い】IFDOCO_損切ライン自動入力～注文</A>
    </DL><p>
</DL><p>
```
:::
:::details ブックマークをインポートする手順（Chromeの場合）
1. 「ブックマークとリスト」＞「ブックマーク マネージャ」を開きます。
![](/images/20251130_tech_orderbyjs/bookmark1.png =500x)
2. 「ブックマークをインポート」からbookmarks.htmlを選択し、インポートします。
![](/images/20251130_tech_orderbyjs/bookmark2.png =500x)
3. 必要に応じて、ブックマークが保存される場所を変更します。
![](/images/20251130_tech_orderbyjs/bookmark3.png =500x)
:::
### 1-2.IFDOCO画面で 株数・指値・利確ライン・預かり区分 を入力
図の赤枠部分は手入力します。
![](/images/20251130_tech_orderbyjs/enter_value.png =600x)

### 1-3.ブックマークを押下する
ブックマークを押下すると、上図の青枠部分が自動入力されて以下が一気に実行されます。
- 損切価格を入力
- IFD1とIFD2の「期間」を同期
- 「確認画面を省略」にチェック
- 発注ボタンを押下


## 2.各ブックマークレットの処理内容
ブックマークレットでは、通常URLを入れる箇所にJavaScriptコードを登録しています。ここでは「買い」「売り」それぞれの処理内容を解説します。

### 買い注文用の処理
買い注文では、IFD1で指定した指値価格から損切ライン（OCO2）を自動計算します。
- 損切トリガー価格 = 指値 − 10 円
- 損切執行価格 = 指値 − 15 円

また、IFD1とIFD2の期間を同期させる処理を入れています。
さらに、「注文確認画面を省略」の自動チェック・「発注」ボタンの自動クリックまで行われます。
```js:【買い】IFDOCO_損切ライン自動入力～注文
javascript:(function(){

  /* --- 指値から損切ポイント 自動計算 --- */
  const v1 = Number(document.getElementById('ifoco_input_price')?.value);
  if (!isNaN(v1)) {
    const v2 = v1 - 10;
    const v3 = v1 - 15;
    if (document.getElementById('doneoco2_input_trigger_price')) document.getElementById('doneoco2_input_trigger_price').value = v2;
    if (document.getElementById('doneoco2_gsn_input_price')) document.getElementById('doneoco2_gsn_input_price').value = v3;
  }

  /* --- IFD1とIFD2の期間の同期 --- */
  (function syncLimit() {

    /* --- ラジオ（this_day / WEEKLY / kikan）同期 --- */
    const r1 = document.querySelector('input[name="ifoco_selected_limit_in"]:checked');
    if (r1) {
      const targetRadio = document.querySelector(`input[name="doneoco_selected_limit_in"][value="${r1.value}"]`);
      if (targetRadio) targetRadio.checked = true;
    }

    /* --- 期間指定の場合：日付セレクト値を同期 --- */
    const sel1 = document.querySelector('select[name="ifoco_limit_in"]');
    const sel2 = document.querySelector('select[name="doneoco_limit_in"]');
    if (sel1 && sel2) {
      sel2.value = sel1.value;
    }
  })();
  
  /* --- 「注文確認画面を省略」にチェックを入れる --- */
  const skipCb = document.getElementById('shouryaku');
  if (skipCb) {
    skipCb.checked = true;
    if (typeof bottonChange === "function") bottonChange();
  }
  
  /* --- 「注文発注」ボタンを押下する --- */
  const btn2 = document.getElementById('botton2');
  if (btn2) btn2.click();

})();
```


### 売り注文用の処理
売り注文も同様に、指値から損切ラインを自動計算します。
- 損切トリガー価格 = 指値 + 10 円
- 損切執行価格 = 指値 + 15 円

ただし、売り注文は期間をデフォルトの「当日中」とすることが多いため、買い注文のような期間同期処理は含めていません。
```js:【売り】IFDOCO_損切ライン自動入力～注文
javascript:(function(){

  /* --- 指値から損切ポイント 自動計算 --- */
  const v1 = Number(document.getElementById('ifoco_input_price')?.value);
  if (!isNaN(v1)) {
    const v2 = v1 + 10;
    const v3 = v1 + 15;
    if (document.getElementById('doneoco2_input_trigger_price')) document.getElementById('doneoco2_input_trigger_price').value = v2;
    if (document.getElementById('doneoco2_gsn_input_price')) document.getElementById('doneoco2_gsn_input_price').value = v3;
  }

  /* --- 「注文確認画面を省略」にチェックを入れる --- */
  const skipCb = document.getElementById('shouryaku');
  if (skipCb) {
    skipCb.checked = true;
    if (typeof bottonChange === "function") bottonChange();
  }
  
  /* --- 「注文発注」ボタンを押下する --- */
  const btn2 = document.getElementById('botton2');
  if (btn2) btn2.click();

})();
```



## おわりに
IFDOCO注文は便利ですが、入力に時間がかかるのが難点です。今回紹介したブックマークレットを使えば、お決まりの作業を自動化でき、発注までの流れを素早く整えられます。

寄り付き前後の忙しい場面でも、入力の負担が減ることで判断に集中しやすくなります。自分のスタイルに合わせて調整しながら、より快適な注文環境づくりに役立ててみてください。
