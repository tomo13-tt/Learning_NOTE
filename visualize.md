# データ可視化

この章では、`matplotlib`、`pandas`、`seaborn` を使って、Python でグラフを描く方法を学びます。

この章の主眼は、データ分析そのものではなく、可視化をコードで実装できるようになることです。同じ目的のグラフを 3 つのライブラリでどう書き分けるかを比較しながら、実務で使いやすい書き方を身につけます。

## 1. この章の目的

- `matplotlib`、`pandas`、`seaborn` の基本的な使い方を理解する
- 代表的なグラフを 3 つのライブラリで描けるようになる
- コード量、可読性、表現力の違いを比較できる
- 用途に応じて、どのライブラリを使うべきか判断できる

## 2. 3 つのライブラリの位置づけ

### `matplotlib`

もっとも基本になる可視化ライブラリです。細かい制御がしやすく、他のライブラリの土台にもなっています。

### `pandas`

`DataFrame` や `Series` から、そのまま簡単にグラフを描けます。内部では `matplotlib` を使っています。

### `seaborn`

`DataFrame` を前提に、統計的な可視化を少ないコードで描きやすくしたライブラリです。内部では `matplotlib` を使っており、見た目も整いやすく、分析用途でよく使われます。

### ざっくりした使い分け

- 基礎から理解したい: `matplotlib`
- すぐに描きたい: `pandas`
- 見やすく簡潔に描きたい: `seaborn`

## 3. 事前準備

この章では `seaborn` に含まれている `mpg` データセットを使います。

`mpg` は、自動車の燃費や性能に関する表形式データです。`mpg` は miles per gallon の略で、1 ガロンの燃料で何マイル走れるかを表します。値が大きいほど燃費がよいと解釈できます。

このデータセットには、燃費を表す `mpg`、出力に関係する `horsepower`、車体の重さを表す `weight`、年式を表す `model_year`、地域を表す `origin` などが含まれています。数値列とカテゴリ列の両方があり、可視化の練習に向いています。

```py
import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns

df = sns.load_dataset("mpg").dropna()
df.head()
```

列名を確認しておきます。

```py
df.columns
```

主に使う列は次の通りです。

- `mpg`
- `horsepower`
- `weight`
- `model_year`
- `origin`

最初に `seaborn` のテーマを設定しておくと見やすくなります。

```py
sns.set_theme(style="whitegrid")
```

---

## 4. `matplotlib` の `Figure` と `Axes`

この先の `matplotlib` のコードを読みやすくするために、先に `Figure` と `Axes` を押さえておきます。

- `Figure`: グラフ全体の入れ物です。画像として保存するときの対象も基本的にはこれです。
- `Axes`: 実際にグラフを描く領域です。タイトル、軸、目盛り、棒や線などはこの中に載ります。

つまり、`Figure` が全体のキャンバスで、その中に 1 つ以上の `Axes` が入る、という関係です。

たとえば、次の 1 行を見てください。

```py
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
```
(memo:axesに書きたいグラフの情報が入る)
このコードでは、1 行 2 列のレイアウトを持つ `Figure` が 1 つ作られ、その中に 2 つの `Axes` が作られます。

- `fig`: 作られた `Figure`
- `axes`: 作られた `Axes` の集合

(memo:大きいキャンバス(Fig)の中に複数のグラフ(axes)を置ける)  

ここでの `axes[0]` は左側の描画領域、`axes[1]` は右側の描画領域です。各 `Axes` に対して `plot()`、`hist()`、`set_title()` などを呼ぶことで、「どこに何を描くか」を明示できます。

```py
fig, axes = plt.subplots(1, 2, figsize=(12, 4))

axes[0].hist(df["mpg"], bins=10)
axes[0].set_title("mpg")

axes[1].hist(df["horsepower"], bins=10)
axes[1].set_title("horsepower")
```

一方で、この章では次のような書き方も多く出てきます。

```py
plt.figure(figsize=(8, 4))
plt.plot(line_df["model_year"], line_df["mpg"])
```

これは `pyplot` を使った簡潔な書き方で、「現在の `Figure` と `Axes` に対して描く」スタイルです。最初の学習ではこの書き方でも十分ですが、複数のグラフを並べたり、保存や細かい調整をしたりするときは `Figure` と `Axes` を意識すると理解しやすくなります。

`pyplot` スタイルの内部イメージを、学習用にかなり単純化した疑似コードで表すと次のようになります。

```py
current_figure = None

def figure(*args, **kwargs):
    fig = Figure(*args, **kwargs)
    current_figure = fig
    return fig

def gcf():
    if current_figure is None:
        return figure()
    return current_figure

def gca():
    fig = gcf()
    if fig has no axes:
        fig.add_subplot(1, 1, 1)  # 1行1列のレイアウトにAxesを1つ追加する
    return fig.current_axes

def plot(*args, **kwargs):
    ax = gca()
    return ax.plot(*args, **kwargs)
```

この疑似コードから分かるポイントは次の通りです。

- `plt.figure()` は `Figure` を作って返す
- 同時に、その `Figure` を `pyplot` 側でも「現在の `Figure`」として覚える
- `plt.plot()` は内部で現在の `Axes` を探し、必要なら作ってから描画する
- 実際の描画処理は最終的に `Axes.plot()` に流れていく

実装そのものはもう少し複雑ですが、大まかな考え方はこの理解で十分です。興味があれば、次の順番で実際のコードや公式ドキュメントを参照すると流れを追いやすくなります。

- `matplotlib.pyplot` の公式概要: https://matplotlib.org/stable/api/pyplot_summary.html

---

## 5. 折れ線グラフ

折れ線グラフは、順序のある値の変化を表現するときに使います。
ここでは、`model_year` ごとの `mpg` と `horsepower` の平均値の変化を可視化します。

### `matplotlib` で書く

```py
line_df = df.groupby("model_year", as_index=False)[["mpg", "horsepower"]].mean()
# as_index=False：インデックスを表示。
plt.figure(figsize=(8, 4))
plt.plot(line_df["model_year"], line_df["mpg"], label="mpg")
#plt.plot:折れ線グラフ, line_df:
plt.plot(line_df["model_year"], line_df["horsepower"], label="horsepower")
plt.xlabel("model_year")
plt.ylabel("average value")
plt.title("Line Plot with matplotlib")
plt.grid()  
# ()内記載なし(False)でグリッド線なし。Trueで記載。
plt.legend()  
# 上のplotコマンド内のlabel=で指定したグラフの情報を記載
plt.show()
```

### `pandas` で書く

```py
line_df = df.groupby("model_year")[["mpg", "horsepower"]].mean()

line_df.plot(figsize=(8, 4), title="Line Plot with pandas")
plt.xlabel("model_year")
plt.ylabel("average value")
plt.show()
```

### `seaborn` で書く

```py
line_df = df.groupby("model_year", as_index=False)[["mpg", "horsepower"]].mean()

plt.figure(figsize=(8, 4))
sns.lineplot(data=line_df, x="model_year", y="mpg", label="mpg")
sns.lineplot(data=line_df, x="model_year", y="horsepower", label="horsepower")
plt.xlabel("model_year")
plt.ylabel("average value")
plt.title("Line Plot with seaborn")
plt.grid()
plt.legend()
plt.show()
```

### 3 者の比較

- `matplotlib`: 仕組みが分かりやすく、制御しやすい
- `pandas`: もっとも短く書ける
- `seaborn`: `DataFrame` ベースで統一感のある書き方ができる

### 第一選択

単純な折れ線グラフなら `pandas` が手早いです。細かく制御したいなら `matplotlib` を使います。

---

## 6. 棒グラフ

棒グラフは、カテゴリごとの件数や値を比較するときに使います。
ここでは、`origin` ごとの自動車の件数を可視化します。

### `matplotlib` で書く

```py
origin_counts = df["origin"].value_counts()

plt.figure(figsize=(6, 4))
plt.bar(origin_counts.index, origin_counts.values)
plt.xlabel("origin")
plt.ylabel("count")
plt.title("Bar Plot with matplotlib")
plt.show()
```

### `pandas` で書く

```py
df["origin"].value_counts().plot(kind="bar", figsize=(6, 4), title="Bar Plot with pandas")
plt.xlabel("origin")
plt.ylabel("count")
plt.show()
```

### `seaborn` で書く

```py
plt.figure(figsize=(6, 4))
sns.countplot(data=df, x="origin")
plt.title("Bar Plot with seaborn")
plt.show()
```

### 3 者の比較

- `matplotlib`: 件数を自分で用意して描く必要がある
- `pandas`: 集計から描画までつなげやすい
- `seaborn`: `countplot` が自然で見やすい

### 第一選択

件数をそのまま描くなら `seaborn` の `countplot` が便利です。集計済みデータなら `pandas` も使いやすいです。

---

## 7. ヒストグラム

ヒストグラムは、1 つの数値列の分布を確認するときに使います。
ここでは、`mpg` の分布を可視化します。

### `matplotlib` で書く

```py
plt.figure(figsize=(7, 4))
plt.hist(df["mpg"], bins=10)
plt.xlabel("mpg")
plt.ylabel("count")
plt.title("Histogram with matplotlib")
plt.show()
```

### `pandas` で書く

```py
df["mpg"].plot(kind="hist", bins=10, figsize=(7, 4), title="Histogram with pandas")
plt.xlabel("mpg")
plt.show()
```

### `seaborn` で書く

```py
sns.displot(df, x="mpg", bins=10)
plt.show()
```

### 3 者の比較

- `matplotlib`: 基本形を理解しやすい
- `pandas`: 1 列だけなら短く書ける
- `seaborn`: 分布系の可視化を拡張しやすい

### 第一選択

単純な確認なら `pandas`、分布系の可視化を広げたいなら `seaborn` が便利です。

---

## 8. 箱ひげ図

箱ひげ図は、カテゴリごとの数値分布を比較するときによく使います。
ここでは、`origin` ごとの `mpg` の分布を比較します。

### `matplotlib` で書く

```py
data = [df[df["origin"] == origin]["mpg"] for origin in df["origin"].unique()]
# originの要素を順番に出してoriginという変数に入れて。、
plt.figure(figsize=(8, 5))
plt.boxplot(data, tick_labels=df["origin"].unique())
plt.ylabel("mpg")
plt.title("Box Plot with matplotlib")
plt.show()
```

### `pandas` で書く

```py
df.boxplot(column="mpg", by="origin", figsize=(8, 5))
plt.title("Box Plot with pandas")
plt.suptitle("") 
# タイトル消す
plt.xlabel("origin")
plt.ylabel("mpg")
plt.show()
```

### `seaborn` で書く

```py
plt.figure(figsize=(8, 5))
sns.boxplot(data=df, x="origin", y="mpg", hue="origin")
plt.title("Box Plot with seaborn")
plt.show()
```

### 3 者の比較

- `matplotlib`: 前処理を自分で書く必要がある
- `pandas`: ある程度短く書けるが少し独特
- `seaborn`: もっとも自然に書ける

### 第一選択

カテゴリと数値列の組み合わせなら `seaborn` が第一候補です。

---

## 9. バイオリンプロット

バイオリンプロットは、カテゴリごとの分布形状もあわせて見たいときに使います。
ここでは、`origin` ごとの `mpg` の分布形状を可視化します。

### `matplotlib` で書く

`matplotlib` にも `violinplot()` はありますが、カテゴリ列との組み合わせを扱うには少し書きづらいため、この章では `seaborn` を中心に扱います。

### `pandas` で書く

`pandas` にはバイオリンプロット専用のメソッドはありません。

### `seaborn` で書く

```py
plt.figure(figsize=(8, 5))
sns.violinplot(data=df, x="origin", y="mpg", hue="origin")
plt.title("Violin Plot with seaborn")
plt.show()
```

### 3 者の比較

- `matplotlib`: 書けるが、この用途では扱いにくい
- `pandas`: 直接は描けない
- `seaborn`: もっとも簡潔

### 第一選択

バイオリンプロットは `seaborn` を使うのが自然です。

---

## 10. 散布図

散布図は、2 つの数値列の関係を見るときに使います。
ここでは、`horsepower` と `mpg` の関係を可視化します。

### `matplotlib` で書く

```py
plt.figure(figsize=(7, 5))
plt.scatter(df["horsepower"], df["mpg"])
plt.xlabel("horsepower")
plt.ylabel("mpg")
plt.title("Scatter Plot with matplotlib")
plt.show()
```

### `pandas` で書く

```py
df.plot(kind="scatter", x="horsepower", y="mpg", figsize=(7, 5), title="Scatter Plot with pandas")
plt.show()
```

### `seaborn` で書く

```py
sns.jointplot(data=df, x="horsepower", y="mpg", kind="scatter", hue="origin")
plt.show()
```

### 3 者の比較

- `matplotlib`: 基本的な散布図を理解しやすい
- `pandas`: 短く書ける
- `seaborn`: 周辺分布や色分けを含めて広げやすい

### 第一選択

単純な散布図なら `matplotlib` や `pandas` で十分です。カテゴリによる色分けまで含めるなら `seaborn` が便利です。

---

## 11. ヒートマップ

ヒートマップは、行列状の値を色で表現したいときに使います。
ここでは、数値列どうしの相関係数をヒートマップで可視化します。

### `matplotlib` で書く

```py
corr = df.select_dtypes(include="number").corr()  
# corr = df.corr(numeric_only=True)でも平気  

plt.figure(figsize=(8, 6))
plt.imshow(corr, cmap="coolwarm", vmin=-1, vmax=1)
plt.xticks(range(len(corr.columns)), corr.columns, rotation=45)
# rotation:軸の名前を見やすくする
plt.yticks(range(len(corr.columns)), corr.columns)
plt.colorbar()
plt.title("Heatmap with matplotlib")
plt.show()
```

### `pandas` で書く

`pandas` だけでヒートマップを自然に描く用途はあまり多くありません。相関行列の作成までは `pandas`、描画は `matplotlib` や `seaborn` を使うことが一般的です。

### `seaborn` で書く

```py
corr = df.select_dtypes(include="number").corr()

plt.figure(figsize=(8, 6))
sns.heatmap(corr, annot=True, cmap="coolwarm", vmin=-1, vmax=1)
plt.title("Heatmap with seaborn")
plt.show()
```
sns.heatmap(corr, vmin=-1, vmax=1, annot=True)  


### 3 者の比較

- `matplotlib`: 書けるが少し手間がかかる
- `pandas`: 行列作成には便利だが描画向きではない
- `seaborn`: もっとも自然に書ける

### 第一選択

ヒートマップは `seaborn` が第一候補です。

---

## 12. ペアプロット

ペアプロットは、複数の数値列の散布図と分布をまとめて確認するグラフです。
ここでは、`mpg`、`horsepower`、`weight`、`acceleration` の関係を `origin` ごとに色分けして可視化します。

### `matplotlib` で書く

`matplotlib` だけでペアプロットを組むこともできますが、かなり記述量が増えるため、この章では扱いません。

### `pandas` で書く

`pandas` にも `scatter_matrix()` はありますが、実務では `seaborn` の `pairplot()` を使うことが多いです。

### `seaborn` で書く

```py
sns.pairplot(
    df[["mpg", "horsepower", "weight", "acceleration", "origin"]],
    hue="origin",
    diag_kind="hist",
)
plt.show()
```  
以下でも可能(hueでの色分けはできない)  
sns.pairplot(
    df.select_dtypes(include='number')
)  

以下も可能(diag_kind='hist'で対角線を指定できる)  
sns.pairplot(
    df, hue='origin', diag_kind='hist'
)
plt.show()

### 3 者の比較

- `matplotlib`: 自前で組むのは重い
- `pandas`: 近い機能はあるが使用頻度はやや低い
- `seaborn`: もっとも簡潔で使いやすい

### 第一選択

ペアプロットは `seaborn` を使います。

---

## 13. 描き分けとサブプロット

カテゴリごとに図を分けたいときや、複数のグラフを並べたいときもあります。
ここでは、`mpg` や `horsepower` を並べて見たり、`origin` ごとに `mpg` のヒストグラムを分けて表示したりします。

### `matplotlib` のサブプロット

`matplotlib` で複数のグラフを並べるときは、事前に見た `plt.subplots()` を使うことが多いです。ここでは 1 行 2 列のレイアウトを作り、左に `mpg`、右に `horsepower` を描きます。

```py
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
# 1行2列のfigを作成。 2,4とかで指定したら...
axes[0].hist(df["mpg"], bins=10)
axes[0].set_title("mpg")
# [0, 1]とかでどこのaxxesに作成するか指定できる
axes[1].hist(df["horsepower"], bins=10)
axes[1].set_title("horsepower")

plt.tight_layout()
plt.show()
```

`plt.tight_layout()` は、タイトルや軸ラベルが重ならないように余白を自動で整えるためのものです。

### `pandas` で書く

`pandas` でも複数列を並べて表示することはできます。

```py
df[["mpg", "horsepower"]].hist(figsize=(10, 4))
plt.show()
```

### `seaborn` の `FacetGrid`

```py
fig = sns.FacetGrid(df, col="origin")
fig.map(sns.histplot, "mpg", bins=10)
plt.show()
```

### 3 者の比較

- `matplotlib`: レイアウトや軸を細かく制御しやすい
- `pandas`: 簡単な並列表示を短く書ける
- `seaborn`: カテゴリごとの描き分けを自然に書ける

### 第一選択

単純に複数のグラフを並べるなら `matplotlib` の `subplots()` が基本です。カテゴリごとに自動で分けたいなら `seaborn` の `FacetGrid` が向いています。`pandas` は手早く確認したいときの選択肢です。

---

## 14. グラフの保存

作成したグラフは画像として保存できます。

### `matplotlib` で書く

```py
plt.figure(figsize=(6, 4))
plt.hist(df["mpg"], bins=10)
plt.title("Histogram with matplotlib")
plt.savefig("matplotlib_hist.png")
plt.show()
```

### `pandas` で書く

`pandas` の `plot()` は、内部では `matplotlib` の `Axes` に描画しています。そのため、戻り値として `Axes` を受け取っておくと、その `Axes` が属する `Figure` を使って保存できます。

```py
ax = df["mpg"].plot(kind="hist", bins=10, figsize=(6, 4), title="Histogram with pandas")
ax.figure.savefig("pandas_hist.png")
plt.show()
```

### `seaborn` で書く

通常の `seaborn` グラフは `matplotlib` と同じように保存できます。

```py
plt.figure(figsize=(6, 4))
sns.histplot(data=df, x="mpg", bins=10)
plt.title("Histogram with seaborn")
plt.savefig("seaborn_hist.png")
plt.show()
```

`pairplot()` や `jointplot()` のように別オブジェクトを返すものは、その戻り値に対して保存メソッドを呼びます。

```py
grid = sns.pairplot(
    df[["mpg", "horsepower", "weight", "acceleration", "origin"]],
    hue="origin",
    diag_kind="hist",
)
grid.savefig("sample.png")
```

### 3 者の比較

- `matplotlib`: `plt.savefig()` でそのまま保存できる
- `pandas`: 戻り値の `Axes` から `figure.savefig()` を呼べる
- `seaborn`: 通常は `matplotlib` と同じで、`pairplot()` などは戻り値側で保存する

### 第一選択

保存自体はどのライブラリでも難しくありません。基本は `plt.show()` より前に保存し、`seaborn` の高水準 API だけ戻り値の型を意識すると安全です。


---

## 16. 演習

### 大問 1: 基本可視化演習

以下では、同じ目的のグラフを複数のライブラリで描いてみてください。

#### 演習 A

`model_year` ごとの `mpg` と `horsepower` の平均を折れ線グラフで、`matplotlib`、`pandas`、`seaborn` の 3 通りで描いてください。

#### 演習 B

`origin` の件数を棒グラフで、`matplotlib`、`pandas`、`seaborn` の 3 通りで描いてください。

#### 演習 C

`mpg` のヒストグラムを、`matplotlib`、`pandas`、`seaborn` の 3 通りで描いてください。

#### 演習 D

`origin` ごとの `mpg` を比較する箱ひげ図を、できる範囲で複数のライブラリで描いてください。

#### 演習 E

`horsepower` と `mpg` の散布図を、`matplotlib`、`pandas`、`seaborn` の 3 通りで描いてください。

#### 演習 F

数値列の相関ヒートマップを `seaborn` で描いてください。

#### 演習 G

`origin` ごとに `mpg` のヒストグラムを分けて表示してください。`matplotlib` の `subplots()` と `seaborn` の `FacetGrid` の両方を試してください。

### 大問 2: 実践演習: PokeAPI を用いた総合 EDA

ここからは、自ら手を動かして EDA を行う総合演習に入ります。実務でのデータ収集プロセスを模倣し、外部の API から最新のデータを直接取得して分析を行います。

#### 事前知識: API

##### API: Application Programming Interface

API は、「自分のプログラムと、外部のデータをつなぐ窓口」です。レストランのウェイターのように、Python からの注文（request）を受け取り、外部サーバーからデータ（response）を運んできてくれます。

#### Step 0: データの準備

以下のコードを実行し、PokeAPI から第 1 世代・第 2 世代にあたる 1〜251 匹目のポケモンデータを取得して、データフレーム `df_poke` を作成してください。通信を行うため、実行には少し時間がかかります。

```py
import requests
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

print("PokeAPIからデータを取得中...")
pokemon_list = []

# 1〜251匹目のポケモンデータを取得
for i in range(1, 252):
    url = f"https://pokeapi.co/api/v2/pokemon/{i}"
    response = requests.get(url)
    data = response.json()

    pokemon_list.append({
        "pokedex_number": i,
        "name": data["name"],
        "hp": data["stats"][0]["base_stat"],
        "attack": data["stats"][1]["base_stat"],
        "defense": data["stats"][2]["base_stat"],
        "type1": data["types"][0]["type"]["name"],
    })

df_poke = pd.DataFrame(pokemon_list)
print("データ取得完了！")
display(df_poke.head())
```

#### 課題 A: データのフィルタリングと相関分析

単なる集計ではなく、「ポケモンの種族値の傾向を分析し、バトルの勝敗を予測するモデルを作るための事前調査」という視点で取り組んでください。

1. `df_poke` を用い、初代環境に絞るため、全国図鑑番号 `pokedex_number` が 1 から 151 までのポケモンにフィルタリングしたデータセット `gen1_df` を作成してください。
2. `gen1_df` の中から、`type1` が `normal` のポケモンのみを抽出したデータセットを作成してください。
3. そのノーマルタイプのデータを用いて、攻撃力 `attack` と防御力 `defense` の関係を、散布図で可視化してください。
4. この 2 変数間の相関係数を算出し、数値として出力してください。


#### 課題 B: カテゴリごとの分布と外れ値の探索

ポケモンのタフさを示す指標 `hp` について、タイプごとの傾向を分析します。別講義で学んだ `pandas` の `groupby` を活用してください。

1. `gen1_df` を用い、各タイプ `type1` ごとに、`hp` の平均値・標準偏差・ポケモン数を集計し、平均値が高い順にソートしたテーブルを作成して表示してください。
2. 同時に、箱ひげ図 `sns.boxplot()` を用いて、各タイプの `hp` の分布を可視化してください。見やすくするために、集計表の平均値順でタイプを並べ替えてプロットしてみましょう。
