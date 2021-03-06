Performance measure on Protoractor
====

Protoractor上でのE2Eテストにおいて、
正確なユーザ体感速度計測を提供します。

# Description

Protractorを用いたE2Eテストにおいて、以下を用いて正確なユーザ体感速度を計測します。
1. 特定要素の表示非表示までを計測する。
1. 処理時間はリモート操作するテスト端末側にスクリプトを仕込んで計測する。

# Requirement

|No  |ライブラリ名  |概要  |
|---|---|---|
|1|Protoractor|本プログラムはProtoractor上で動作します。|

# Usage

## install

```js
npm install protoractor-perf-sonar --save-dev
```

## method usage

### 特定要素表示まで計測
```js
perfSonar.untilShow(showTagSelector);
```

### 特定要素非表示まで計測

```js
perfSonar.untilHide(hideTagSelector);
```

### 計測値の取得

```js
var promise = perfSonar.end();
promise.then(function(measureTime) {
  // measureTimeが測定結果となります。
});
```

### 各変数について

|No  |変数名  |概要  |
|---|---|---|
|1|showTagSelector/|計測完了とする表示対象をcssセレクタを用いて指定します。<br>指定セレクタで一つでも要素が見つかれば計測終了となります。<br>一度も要素が非表示とならない場合は計測できません。<br>ng-ifによって、隠れている要素が表示する場合は、子要素を指定することが可能です。<br>ng-showやng-hideで表示を切り替える場合は、ng-showやng-hideが記載されているタグを指定しないと計測できません。|
|2|hideTagSelector|計測完了とする非表示対象をcssセレクタを用いて指定します。<br>指定セレクタが一つも見つからなくなった時点で計測終了となります。<br>一度も要素が表示とならない場合は計測できません。<br>ng-ifによって、表示している要素が隠れるまで計測したい場合は、子要素を指定することが可能です。<br>ng-showやng-hideで表示を切り替える場合は、ng-showやng-hideが記載されているタグを指定しないと計測できません。|

## example
### 特定要素表示まで計測

```js
const PerfSonar = require('protoractor-perf-sonar');
var perfSonar = new PerfSonar.sonar(browser);

// 計測終了条件を指定する(対象要素が表示になるまで)。
perfSonar.untilShow('.show-target');

// テスト実行
element(by.id('gobutton')).click();

// .show-target要素が表示されるまでの時間を取得
perfSonar.end().then(function(measureTime) {
  console.log('性能計測結果:' + measureTime);
});
```

### 特定要素非表示まで計測

```js
const PerfSonar = require('protoractor-perf-sonar');
var perfSonar = new PerfSonar.sonar(browser);

// 計測終了条件を指定する(対象要素が非表示になるまで)。
perfSonar.untilHide('.hide-target');

// テスト実行
element(by.id('gobutton')).click();

// .hide-target要素が非表示になるまでの測定を実施
perfSonar.end().then(function(measureTime) {
  console.log('性能計測結果:' + measureTime);
});
```

# Detail Explanation

## 性能計測における課題

### 画面遷移までの時間ではユーザ体感速度は評価できない

画面遷移完了までの時間計測ではユーザ体感速度計測にならない場合があります。

例えば、画面遷移後に遅延ロードで購入商品を表示する画面の場合には、
ユーザは商品がなければ操作できません。
画面遷移がいくら早くても、商品表示が遅ければユーザがこの画面は処理が遅いと判断されます。

そのため、画面遷移が完了したタイミングではなく、
画面遷移後、特定の要素が表示されるまでを計測して評価する必要があります。

### E2E性能計測にてテスト端末との通信遅延により正確な速度計測が困難

E2Eテストにて、Protractor実行側のマシンにて以下のように計測すると、
テスト端末側にて処理を実行した後のタイムラグが発生し、秒単位で計測結果がずれてしまいます。

```js
var startTime;
element(by.id('before-operation')).click().then(function(){
  startTime = new Date();
});
var duration;
element(by.id('gobutton')).click().then(function(){
  var endTime = new Date();
  duration = endTime.getTime() - startTime.getTime();
});
// Protractor実行側のマシンとテスト端末では通信による遅延などがあるため、
// ボタン押下直後に時間を計測できない。
// 結果秒単位でずれた結果が取得されてしまう。
```

## protractor-perf-sonorを使った課題解決

### 特定要素の表示非表示までを計測
ユーザ体感速度を計測するために、
指定した画面要素が表示/非表示となるまでの時間を簡単に計測することができます。

### 処理時間計測はリモート操作するテスト端末側で実施

処理時間の計測用のScriptをリモートで操作するテスト端末側に仕込み、計測します。
これにより、実機での処理時間を計測することができ、より正確なユーザ体感速度を得られます。

# Author
s-nakagawa

# License
[Apache Version 2.0](https://github.com/serive/es-ml-alert/blob/master/LICENSE)