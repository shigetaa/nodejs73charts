# 移動平均を利用した予測とグラフの描画
需要予測とは物の需要を予測することです。
需要予測には様々な手法があり、状況に適した方法を選択したり
複数の手法を試したりすることができます。

## 需要予測について
ビジネスにおいて、需要をよそくする事で在庫削減や欠品削減に大きな効果が得られます。
需要予測は実際の業務に役に立つノウハウです。
計算もそれほど難しくないので、一度理解してしまえば、様々なところで利用することができるでしょう。
膨大なデータの中からデータマイニングを行う際にもこの手法が約二立ちます。

しかし、需要予測をはじめ、為替や株の値動きの予測を行う際によく言われるのが「予測はあくまでも予測なので現実にはその通りにならない」
と言うことです。現実の社会は複雑で完全な予測は出来ないものです。
それは、どんなに複雑なアルゴリズムを利用してもおなじです。
ですから、利用する際には、予測の誤差を考慮する必要があります。
それでも、企業は需要予測を行うことで生産計画や販売キャンペーンを策定したり、設備投資や採用計画を行うことになります。

需要予測には色々な手法がありますが、最も広く利用されているのは、移動平均(Moving Average) と 指数平滑法(Exponential Moving Average; EMA)です。

## 単純移動平均について
単純移動平均(Simple Movieng Average; SMA) とは、時系列デーｔあを平滑化する手法です。
金融や気象などの計測分野で使われています。
直近の n個のデータに対して単純な平均を取る方法です。

例えば、定期的な英語のテストを行ったとして、点数が今日は60点、前回は46点、その前は53点だったとします。
すると、その平均点は、(60+46+53)/5=53 という式で求めることができます。

直近の n個の各データを、Pm, Pm-1, Pm-2, Pm-3... で表すと次のようになります。

<img src="https://latex.codecogs.com/svg.image?SMA&space;=&space;\frac{Pm&plus;Pm-1&plus;Pm-2&plus;Pm-3...}{n}" title="SMA = \frac{Pm+Pm-1+Pm-2+Pm-3...}{n}" />

しかし、直近100個のデータ総計を、データの個数100で割って1つの値を得るならば、データ全体が増加傾向にあるのか、減少傾向にあるのかを知ることができません。
平均はあくまでも平均であり、貴重な情報が失われていることになります。
「単純移動平均」はグラフに描画した時に、上下するギザギザしたデータを、なめらかなグラフになる様にデータを変換する手法です。

### グラフを描画してみよう
ここで、移動平均を計算する前に、普通に折れ線グラフ(ラインチャート) を描画する方法を紹介します。
JavaScriptでグラフを描画する場合には、HTMLとWebブラウザー、そしてビュジュアライゼーションの為のライブラリを使用します。
ここでは、Google Charts を利用してみます。

[Google Charts<br>https://developers.google.com/chart](https://developers.google.com/chart)

簡単に折れ線グラフを描画するHTMLファイルを`line-chart.html`と言う名前で作成して表示出来るか見てみます。
```html
<html>

<head>
	<!-- Google Chartsの取り込み -->
	<script type="text/javascript" src="https://www.google.com/jsapi"></script>
	<!-- 気温データの読込 -->
	<script type="text/javascript">
		// Charts APIの初期化
		google.load('visualization', '1.1', { packages: ['line'] });
		google.setOnLoadCallback(drawChart);
		// 実際の描画
		function drawChart() {
			// データオブジェクトを作成
			var data = new google.visualization.DataTable();
			// データのカラムを指定
			data.addColumn('number', 'テストの回数');
			data.addColumn('number', '国語の点');
			// データの値を指定 [テストの回数, 国語の点]
			data.addRows([
				[1, 30], [2, 50], [3, 43], [4, 60], [5, 77],
				[8, 70], [9, 80], [10, 87], [11, 68], [12, 60]
			]);
			// 描画オプション
			var options = {
				width: 800, height: 400
			};
			// 描画
			var chart = new google.charts.Line(document.getElementById('chart'));
			chart.draw(data, options);
		}
	</script>
</head>

<body>
	<div id="chart"></div>
</body>

</html>
```
HTML5に対応したWebブラウザーで上記のHTMLを開くと折れ線グラフが描画されるのが確認できます。

### 過去の気象データのダウンロード
平均気温をグラフに描画するプログラム`csv2kion.js`を作成していきます。
気象庁の過去の気象データをダウンロードページから平均気温をダウンロードしてください。
CSV形式でダウンロードできます。

[気象庁・過去の気象データ<br>https://www.data.jma.go.jp/obd/stats/etrn/](https://www.data.jma.go.jp/obd/stats/etrn/)

CSV形式でダウンロード出来るとは言え、そのまま利用できる形式ではないので、Excelなどの表計算ソフトで読み込んで、「日付」「平均気温」だけの情報に形成しましょう。
通常CSVファイルの文字コードは、Shift_JISですが、Javascript で扱いやすいようにUTF-8でファイル名`2022_osaka.csv`として保存します。

その上で、CSVファイルをGoogle Charts 用に二次元配列データに変換して、グラフで表示してみます。
ここでは、CoffeeScript を利用して、CSVファイルをJSON形式に変換するプログラムを`csv2js.coffee`ファイル名として作成します。

```coffee
FILE_CSV = './2022_osaka.csv';
FILE_JS = './2022_osaka.js';

fs = require 'fs'

txt = fs.readFileSync FILE_CSV, "utf-8"
lines = txt.split "\r\n"

result = [];
for v in lines
  cells = v.split ','
  date_s = cells[0].split("/").splice(1, 2).join("/");
  temp = parseFloat cells[1]
  result.push([date_s, temp])

json = JSON.stringify result
js = "var kion_data = " + json;
fs.writeFileSync FILE_JS, js, "utf-8"
console.log "ok"
```
以下のコマンドを実行して、JSON形式`2022_osaka.js`ファイルを作成します。
```bash
coffee csv2js.coffee
```

データができたら、これをチャート上に表示してみます。
チャートを表示するHTMLファイル`2022_osaka.html`を作成します。

```html
<html>

<head>
	<!-- Google Chartsの取り込み -->
	<script type="text/javascript" src="https://www.google.com/jsapi"></script>
	<!-- 気温データの読込 -->
	<script type="text/javascript" src="2022_osaka.js"></script>
	<script type="text/javascript">
		// Charts APIの初期化
		google.load('visualization', '1.1', { packages: ['line'] });
		google.setOnLoadCallback(drawChart);
		// 実際の描画
		function drawChart() {
			// データオブジェクトを作成
			var data = new google.visualization.DataTable();
			// データのカラムを指定 ----- (*1)
			data.addColumn('string', '日付');
			data.addColumn('number', '平均気温');
			// 値を設定 ---- (*2)
			data.addRows(kion_data);
			// 描画オプション
			var options = {
				width: 800, height: 400
			};
			// 描画
			var chart = new google.charts.Line(document.getElementById('chart'));
			chart.draw(data, options);
		}
	</script>
</head>

<body>
	<div id="chart"></div>
</body>

</html>
```
HTMLファイルをWebブラウザーで表示して、チャートが表示出来ているか確認してみましょう。

### 気温の移動平均を計算してグラフに描画
それでは、毎日の移動平均を計算してグラフに描画してみます。
前項でダウンロードしたCSVファイルを元にして、移動平均を加えたJavaScriptのデータを計算し挿入してみます。
まずは、JavaScriptのデータを作成するCoffeeScriptのプログラム`2022_osaka_sma.js`を作成します。

```coffee
FILE_CSV = './2022_osaka.csv'
FILE_JS  = './2022_osaka_sma.js'
MA_RANGE = 7

fs = require 'fs'

# ファイルからCSVファイルを読む
loadCSV = (filename) ->
  txt = fs.readFileSync filename, "utf-8"
  lines = txt.split "\r\n"
  # CSVテキストを二次元配列変数に変換
  list = []
  for v in lines
    cells = v.split ','
    date_s = cells[0].split("/").slice(1,3).join("/")
    temp   = parseFloat cells[1]
    list.push([date_s, temp])
  return list

# 移動平均を計算
calcMovingAverage = (i, list, range) ->
  # 期間を決定
  m_from = i - range
  m_to   = m_from + range - 1
  # 期間が不完全ならば0を返す
  if m_from < 0 then return NaN
  # 合計を計算
  sum = 0
  for j in [m_from .. m_to]
    sum += list[j][1]
  # 平均を計算
  return sum / range

# メイン処理
main = ->
  # CSVを読み込む
  list = loadCSV(FILE_CSV)
  # 各行について移動平均を求めリストに追加
  for val, index in list
    av = calcMovingAverage(index, list, MA_RANGE)
    list[index].push(av)
  # JavaScriptを出力
  js = "var kion_data = " + JSON.stringify(list)
  fs.writeFileSync FILE_JS, js, "utf-8"

main()
console.log "ok"
```
以下のコマンドを実行して、JSON形式`2022_osaka_sma.js`ファイルを作成します。
```bash
coffee csv2js_sma.coffee
```

データができたら、これをチャート上に表示してみます。
チャートを表示するHTMLファイル`2022_osaka_sma.html`を作成します。

```html
<html>

<head>
	<!-- Google Chartsの取り込み -->
	<script type="text/javascript" src="https://www.google.com/jsapi"></script>
	<!-- 気温データの読込 -->
	<script type="text/javascript" src="2022_osaka_sma.js"></script>
	<script type="text/javascript">
		// Charts APIの初期化
		google.load('visualization', '1.1', { packages: ['line'] });
		google.setOnLoadCallback(drawChart);
		// 実際の描画
		function drawChart() {
			// データオブジェクトを作成
			var data = new google.visualization.DataTable();
			// データのカラムを指定 ----- (*1)
			data.addColumn('string', '日付');
			data.addColumn('number', '平均気温');
			data.addColumn('number', '移動平均');
			// 値を設定
			data.addRows(kion_data);
			// 描画オプション
			var options = { width: 800, height: 400 };
			// 描画
			var chart = new google.charts.Line(
				document.getElementById('chart'));
			chart.draw(data, options);
		}
	</script>
</head>

<body>
	<div id="chart"></div>
</body>

</html>
```
HTMLファイルをWebブラウザーで表示して、移動平均線を含んだチャートが表示出来ているか確認してみましょう。

### 指数平滑法について
指数平滑法(指数移動平均法 Exponential Average; EMA) は、移動平均法と並んで広く使われている分析手法です。
得られた過去データのうち、より新しいデータに大きなウェイトを置き、過去になるほどウェイトが小さくなる(指数関数的に減少する)というものです。

移動平均法と比べて、今回の出来事が直前の出来事に強く影響される場合、出来事の変動に出来るだけ追従させたい場合などに用いられ、短期的な予測に適しています。
財務上の時系列予測や株価変動分析などにも使用されます。

下記のような式で表されます。なお、a の値は 0 < a < 1 とします。

> 予測値 = a x 前回の実績値 + (1 - a) x 前回予測値 = 前回予測値 + a x (前回実績値 - 前回予測値)

`a` は **平滑化定数**になちます、平滑化定数は、0以上1以下で定義される係数。
`a`が1に近いほど直近のデータが重視され、`a`が0に近いほど過去のデータが重視される。
需要が不安定な場合は`a`を1に近づけ、安定的である場合は`a`を0に近づけます。

前回の実績値が予測値からどれほど外れたかを算出し、それに一定の係数aをかけて得た修正値を、前回予測値に加減することで、今回の予測値を算出します。

それでは、前回同様に、CoffeeScript で CSV を JavaScript に変換する`csv2js_ema.coffee`ファイルを作成しますが、今回は、指数平滑法を利用して予測値を計算します。

```coffee
FILE_CSV = './2022_osaka.csv'
FILE_JS  = './2022_osaka_ema.js'

EMA_RANGE = 10
EMA_ALPHA = 2 / (EMA_RANGE + 1)

fs = require 'fs'

# 指数移動平均を計算する関数
calcExpMovingAverage = (i, list, alpha) ->
  # 今回の値
  value = list[i][1]
  # 前回の値
  if i == 0
    last_value = value
  else
    last_value = list[i - 1][2]
  # 予測値を計算
  new_value = last_value + alpha * (value - last_value)
  # 予測値を記録
  list[i][2] = new_value
  return new_value


# ファイルからCSVファイルを読む
loadCSV = (filename) ->
  txt = fs.readFileSync filename, "utf-8"
  lines = txt.split "\r\n"
  # CSVテキストを二次元配列変数に変換
  list = []
  for v in lines
    cells = v.split ','
    date_s = cells[0].split("/").slice(1,3).join("/");
    temp   = parseFloat cells[1]
    list.push([date_s, temp])
  return list

# メイン処理
main = ->
  # CSVを読み込む
  list = loadCSV(FILE_CSV)
  # 各行についてEMAを求める
  for v, index in list
    av = calcExpMovingAverage(index, list, EMA_ALPHA)
    console.log list[index][0], list[index][1], av
  # JavaScriptを出力
  js = "var kion_data = " + JSON.stringify(list)
  fs.writeFileSync FILE_JS, js, "utf-8"

main()
console.log "ok"
```
予測値は`new_value = last_value + alpha * (value - last_value)`の様に求めています。

以下のコマンドを実行して、JSON形式`2022_osaka_ema.js`ファイルを作成します。
```bash
coffee csv2js_ema.coffee
```

データができたら、これをチャート上に表示してみます。
チャートを表示するHTMLファイル`2022_osaka_ema.html`を作成します。

```html
<html>

<head>
	<!-- Google Chartsの取り込み -->
	<script type="text/javascript" src="https://www.google.com/jsapi"></script>
	<!-- 気温データの読込 -->
	<script type="text/javascript" src="2022_osaka_ema.js"></script>
	<script type="text/javascript">
		// Charts APIの初期化
		google.load('visualization', '1.1', { packages: ['line'] });
		google.setOnLoadCallback(drawChart);
		// 実際の描画
		function drawChart() {
			// データオブジェクトを作成
			var data = new google.visualization.DataTable();
			// データのカラムを指定
			data.addColumn('string', '日付');
			data.addColumn('number', '平均気温');
			data.addColumn('number', '指数平均');
			// 値を設定
			data.addRows(kion_data);
			// 描画オプション
			var options = { width: 800, height: 400 };
			// 描画
			var chart = new google.charts.Line(
				document.getElementById('chart'));
			chart.draw(data, options);
		}
	</script>
</head>

<body>
	<div id="chart"></div>
</body>

</html>
```
HTMLファイルをWebブラウザーで表示して、移動平均線を含んだチャートが表示出来ているか確認してみましょう。

## 明日の平均気温を予測
それでは、ダウンロードした気温データで翌日の気温を、単純移動平均(SMA)、指数移動平均(EMA)で予測するプログラム`2022_osaka_yosoku.coffee`を作成してみましょう。

```coffee
# EMAとSMAを利用して翌日の気温を予測する

# 予測データ
FILE_CSV = './2022_osaka.csv'
# 定数
SMA_RANGE = 3
EMA_ALPHA = 2 / (SMA_RANGE + 1)

# モジュールの取り込み
fs = require 'fs'

# 指数移動平均を計算する関数
calcExpMovingAverage = (i, list, alpha) ->
  # 今回の値
  value = list[i][1]
  # 前回の値
  if i == 0
    last_value = value
  else
    last_value = list[i - 1][2]
  # 予測値を計算
  new_value = last_value + alpha * (value - last_value)
  # 予測値を記録
  list[i][2] = new_value
  return new_value

# 移動平均を計算する関数
calcSimpleMovingAverage = (i, list, range) ->
  # 期間を選択
  m_from = i - range + 1
  m_to   = i
  cnt = 0
  sum = 0
  for j in [m_from .. m_to]
    if j >= 0
      sum += list[j][1]
      cnt++
  value = sum / cnt
  # 予測値を記録
  list[i][3] = value
  return value

# ファイルからCSVファイルを読む
loadCSV = (filename) ->
  txt = fs.readFileSync filename, "utf-8"
  lines = txt.split "\r\n"
  # CSVテキストを二次元配列変数に変換
  list = []
  for v in lines
    cells = v.split ','
    date_s = cells[0].split("/").slice(1,3).join("/");
    temp   = parseFloat cells[1]
    list.push([date_s, temp])
  return list

# 少数点以下を揃えて出力
fmt = (v) ->
  v = "" + (Math.round(v * 100) / 100)
  if v.indexOf(".") < 0 then v += ".00"
  v = v.replace /\.?(\d+)$/, (ma, n) ->
    n += "00"
    n = n.substr(0, 2)
    return "." + n

# メイン処理
main = ->
  # CSVを読み込む
  list = loadCSV(FILE_CSV)
  # 各行について予測値を求める
  sum_ema = sum_sma = cnt = 0
  for v, index in list
    if index + 1 >= list.length then continue
    date_s = list[index + 1][0]
    real_v = list[index + 1][1]
    ema = calcExpMovingAverage(index, list, EMA_ALPHA)
    ema_err = ema - real_v
    sum_ema += Math.abs(ema_err)
    sma = calcSimpleMovingAverage(index, list, SMA_RANGE)
    sma_err = sma - real_v
    sum_sma += Math.abs(sma_err)
    cnt++
    console.log "+ #{date_s} 実際 #{real_v}"
    console.log "| - ema = #{fmt(ema)} : 誤差#{fmt(ema_err)}"
    console.log "| - sma = #{fmt(sma)} : 誤差#{fmt(sma_err)}"
  # 誤差を表示
  console.log "---"
  ave_ema = sum_ema / cnt
  ave_sma = sum_sma / cnt
  console.log "誤差EMA=#{fmt(sum_ema)} 平均=#{fmt(ave_ema)}"
  console.log "誤差SMA=#{fmt(sum_sma)} 平均=#{fmt(ave_sma)}"

main()
```

実行するには、以下のコマンドを実行します。
```bash
coffee 2022_osaka_yosoku.coffee
```
```bash
+ 2/1 実際 7.9
| - ema = 6.76 : 誤差-1.14
| - sma = 5.43 : 誤差-2.47
+ 2/2 実際 9.5
| - ema = 7.33 : 誤差-2.17
| - sma = 6.80 : 誤差-2.70
+ 2/3 実際 5.7
| - ema = 8.41 : 誤差2.71
| - sma = 8.37 : 誤差2.67
+ 2/4 実際 6.7
| - ema = 7.06 : 誤差0.36
| - sma = 7.70 : 誤差1.00
+ 2/5 実際 7.3
| - ema = 6.88 : 誤差-0.42
| - sma = 7.30 : 誤差0.00
...省略...
```
実際には気象は複雑な条件があるので一概に言えませんが、簡単な予測をする際に、単純移動平均(SMA)、指数移動平均(EMA)の利用法方法として活用してみてください。