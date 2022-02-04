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

