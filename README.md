# 4d-tips-print-using-svg
SVGで帳票を出力するサンプル

###概要

SVGは宣言型のドキュメントであり，容易に編集できる標準テキストファイルです。

帳票類のテンプレートをSVGで作成すれば，簡単にデータの「差し込み印刷」ができます。

###手順
1. ピクチャ型の変数オブジェクト1個からなるフォームを作成します。

サイズと位置は何でも構いません。

変数名は「SVG」で良いでしょう。

ピクチャフォーマットは「トランケート（中央合わせしない）」つまり「そのまま」を選択します。

2. A4サイズ（210mm x 297mm）のSVGテンプレートを作成します。

変数やフィールドなど，計算値を挿入したい箇所には4Dタグを置きます。

```xml
<?xml version="1.0" encoding="utf-8" standalone="no" ?>
<svg
  xmlns="http://www.w3.org/2000/svg"
  xmlns:xlink="http://www.w3.org/1999/xlink"
  version="1.1"
  width="210mm"
  height="297mm">
<defs>
  <style type="text/css">
  textArea.label_1 {
    font-size:99.12;
    kerning:auto;
    display-align:center;
    text-align:center;
    font-weight:normal;
    fill;black;
  }
  .rect_1 {
    stroke:black;
    stroke-width:0.1mm;
    fill:red;
    fill-opacity:0.3;
  }
  </style>
</defs>
<g>
<rect x="0.4in" y="5pc" width="5in" height="20pc" class="rect_1" />
<textArea x="0.4in" y="5pc" width="5in" height="20pc" class="label_1"><!--#4dtext $1--></textArea>
</g>
</svg>
```

3. SVGテンプレートをPROCESS 4D TAGSにかけます。

```
  //テンプレートファイル
$path:=Get 4D folder(Current resources folder)+"sample.4dtag"
$template:=Document to text($path;"utf-8")

  //Unicodeなのでテキストで処理
C_TEXT($xml)
PROCESS 4D TAGS($template;$xml;"ふぉーでいー")

  //テキストをピクチャに変換！
C_BLOB($svg)
CONVERT FROM TEXT($xml;"utf-8";$svg)
BLOB TO PICTURE($svg;SVG)  //SVGはフォームオブジェクト

  //A4サイズいっぱい
$mm_in:=25.4
$DPI:=72
$height:=297*($DPI/$mm_in)
$width:=210*($DPI/$mm_in)

  //用紙ぎりぎりで
C_REAL($marginLeft;$marginTop;$marginRight;$marginBottom)
GET PRINTABLE MARGIN($marginLeft;$marginTop;$marginRight;$marginBottom)
SET PRINTABLE MARGIN(0;0;0;0)

SET PRINT PREVIEW(True)

OPEN PRINTING JOB
FORM LOAD("A4")

  //背景色（印刷領域確認用）を透明に
  //OBJECT SET RGB COLORS(*;"SVG";Foreground color;Background color none)

$printed:=Print object(*;"SVG";0;0;$width;$height)
FORM UNLOAD
CLOSE PRINTING JOB

  //元どおりに
SET PRINTABLE MARGIN($marginLeft;$marginTop;$marginRight;$marginBottom)
```

これだけです！

ポイント

* SVGなので，ピクセル以外の単位（インチ・ミリメートル・パイカ）で正確にレイアウトが指定できる。
* 小数点以下の正確さで位置・サイズ・フォント・線の幅などが指定できる。
* カーニング・透明度・影付けなど，フォームでは無理なレンダリング指定ができる。
* CSSでスタイルが制御できる。
* 外部ファイルで管理できる。

など。

###気づいたこと

15R5（64ビット/Cocoa版）だと用紙の淵いっぱいまで印刷ができました。

15.x（32ビット/Carbon版）だと，超えられないマージンの壁があります。

![sample](https://cloud.githubusercontent.com/assets/10509075/21215808/c37fb426-c2e6-11e6-8460-19d8816882b3.png)
