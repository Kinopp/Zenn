---
title: "分析関数（全9問）"
free: false
---

# 問題12-1：全体の順位（RANK関数）

### 難易度：★★☆☆☆ (Lv.2)

## 問題
HRスキーマの`EMPLOYEES`テーブルより、職種（`JOB_ID`）が「**IT_PROG**」の従業員を対象として、**給与（`SALARY`）が高い順**にランク付けを行ってください。

**【条件およびルール】**

* 給与が同じ場合は同じ順位を割り当ててください。
* 同じ順位が複数存在した後は、その人数分だけ順位を飛ばしてください（例：3位が2人いたら次は5位）。
* 結果はランクの昇順で表示してください。

## 期待する結果
| FIRST_NAME | SALARY | RANK | 
| ---------- | ------ | ---- | 
| Alexander  | 9000   | 1    | 
| Bruce      | 6000   | 2    | 
| David      | 4800   | 3    | 
| Valli      | 4800   | 3    | 
| Diana      | 4200   | 5    | 

## 解答例
```sql:例1：RANK関数を使用
SELECT
    e1.first_name,
    e1.salary,
    RANK() OVER(
        ORDER BY
            salary DESC
    ) AS rank
FROM
    hr.employees e1
WHERE
    e1.job_id = 'IT_PROG'
ORDER BY
    rank
```
```sql:例2：自己結合で解決
SELECT
    e1.first_name,
    e1.salary,
    COUNT(e2.salary) + 1 AS rank
FROM
    hr.employees e1
    LEFT JOIN hr.employees e2 
      ON e1.salary < e2.salary
     AND e2.job_id = 'IT_PROG'
WHERE
    e1.job_id = 'IT_PROG'
GROUP BY
    e1.employee_id,
    e1.first_name,
    e1.salary
ORDER BY
    rank
```

## 解説
SQLの中級テクニックの中でも「華」と言える **分析関数（Window関数）** のセクションに突入です。
「給与が高い順に並べる」だけでなく、そこに「1位、2位…」という**順位の数字**を直接持たせることができるようになると、上位N人の抽出や、グループ内の比較が驚くほど簡単になります。

### 1. 核心：RANK() 分析関数の仕組み
分析関数の最大の特徴は、**「行をまとめずに（集約せずに）、集計結果を各行に添える」** 点にあります。



#### ① OVER(ORDER BY ...) の役割
```sql
RANK() OVER(ORDER BY salary DESC)
```
* **`RANK()`**: 順位を計算せよ、という命令。
* **`OVER`**: どの範囲に対して計算するかを指定。
* **`ORDER BY salary DESC`**: 「給与の高い順」というルールで並べた時の番号を振る、という意味です。

### 2. 徹底比較：3つのランキング関数
実務では、**「同着（同じ給与）」** をどう扱うかによって、関数を使い分ける必要があります。ここがLv2の試験や現場で最も問われるポイントです。

| 関数 | 特徴 | 例（4800が2人の場合） |
| :--- | :--- | :--- |
| **`RANK`** | 同着がいると、次の順位を**飛ばす** | 1位 → 2位 → **3位 → 3位 → 5位** |
| **`DENSE_RANK`** | 同着がいても、次の順位を**飛ばさない** | 1位 → 2位 → **3位 → 3位 → 4位** |
| **`ROW_NUMBER`** | 同着でも、**一意な連番**を振る | 1位 → 2位 → **3位 → 4位** → 5位 |



今回の期待する結果では、DavidさんとValliさんが共に「3位」で、次のDianaさんが「5位」になっているので、**`RANK`関数**が正解となります。


### 3. 解答例2：自己結合による「人力ランキング」
分析関数が使えなかった時代の、非常に賢いロジックです。

#### ① ロジックの考え方
「自分の順位」とは、**「自分より給与が高い人が何人いるか ＋ 1」** である、という数学的な定義に基づいています。

1.  **左側(e1)**: 自分のデータ。
2.  **右側(e2)**: 自分より高い給与(`e1.salary < e2.salary`)の人のリスト。
3.  **集計**: 右側にヒットした人数を `COUNT` し、そこに `+1` することで順位を算出します。

> **パフォーマンスの差**
> 自己結合によるランキングは、データが1万件を超えると計算量が膨大になり、非常に重くなります。現代のシステムでは、例1の**分析関数**を使うのが圧倒的に高速で、プロの書き方と言えます。

### 4. 実行順序のポイント
解答例1の最後にある `ORDER BY rank` に注目してください。
SQLの実行順序では、`SELECT` 句で計算された `rank` という別名は、最後の `ORDER BY` でしか使えません。

もし「3位以上の人だけ出したい」と思って `WHERE rank <= 3` と書いても、**WHERE句の時点ではまだ順位が計算されていない**ためエラーになります。その場合は、前章で学んだ「副問い合わせ」を組み合わせる必要があります。

----
<br><br>

# 問題12-2：グループごとのナンバーワン（Top-N分析）

### 難易度：★★☆☆☆ (Lv.2)

## 問題

HRスキーマの`EMPLOYEES`テーブルより、**各職種（`JOB_ID`）における最高給与者**を特定してください。

**【条件およびルール】**

* 職種（`JOB_ID`）ごとにグループ分けを行い、その中で給与（`SALARY`）が高い順にランク付けを行ってください。
* 各グループで**1位**になった従業員のみを表示してください。
* 給与が同額で1位が複数人いる場合は、その全員を表示してください。
* 結果は、給与（`SALARY`）の昇順、次に職種（`JOB_ID`）の昇順で並べてください。

## 期待する結果
| FIRST_NAME | JOB_ID     | SALARY | 
| ---------- | ---------- | ------ | 
| Alexander  | PU_CLERK   | 3100   | 
| Renske     | ST_CLERK   | 3600   | 
| Nandita    | SH_CLERK   | 4200   | 
| Jennifer   | AD_ASST    | 4400   | 
| Pat        | MK_REP     | 6000   | 
| Susan      | HR_REP     | 6500   | 
| Adam       | ST_MAN     | 8200   | 
| William    | AC_ACCOUNT | 8300   | 
| Daniel     | FI_ACCOUNT | 9000   | 
| Alexander  | IT_PROG    | 9000   | 
| Hermann    | PR_REP     | 10000  | 
| Den        | PU_MAN     | 11000  | 
| Lisa       | SA_REP     | 11500  | 
| Shelley    | AC_MGR     | 12008  | 
| Nancy      | FI_MGR     | 12008  | 
| Michael    | MK_MAN     | 13000  | 
| John       | SA_MAN     | 14000  | 
| Neena      | AD_VP      | 17000  | 
| Lex        | AD_VP      | 17000  | 
| Steven     | AD_PRES    | 24000  | 

## 解答例
```sql
SELECT
    first_name,
    job_id,
    salary
FROM
    (
        SELECT
            first_name,
            job_id,
            salary,
            RANK()
            OVER(PARTITION BY job_id
                 ORDER BY
                     salary DESC
            ) AS rank
        FROM
            hr.employees
    )
WHERE
    rank = 1
ORDER BY
    salary,
    job_id
```

## 解説
「全体の1位」を出すのは簡単ですが、「職種ごと」「部署ごと」といった**グループの中での1位**を特定するこのテクニックは、実務のレポート作成において最も需要が高いスキルの一つです。

### 1. 核心：PARTITION BY の「小部屋」効果
今回の主役は `OVER` 句の中に登場した **`PARTITION BY`** です。


#### ① 動作の仕組み
通常の `GROUP BY` は行を一つにまとめて（集計して）しまいますが、分析関数の `PARTITION BY` は **「行をまとめるのではなく、一時的に小部屋（パーティション）に分ける」** という動きをします。

1. **分ける**: `job_id` ごとに従業員を別々の部屋に分ける。
2. **並べる**: 各部屋の中で `salary` の高い順に並べる。
3. **振る**: 部屋ごとに「1位、2位…」と番号を振る。

このおかげで、IT部門の1位と、営業部門の1位を同時に、かつ独立して計算できるわけです。

### 2. なぜ「副問い合わせ」が必要なのか？
解答例が `FROM ( SELECT ... )` という二段構えになっているのには、SQLの絶対的なルールが関係しています。

> **分析関数は「最後の方」に計算される**
> SQLの処理順序では、`WHERE` 句のフィルタリングが終わった**後**に分析関数が計算されます。
> そのため、`WHERE rank = 1` と書こうとしても、その時点ではまだ `rank` という列（順位）がこの世に存在していないため、エラーになってしまうのです。
> 
> **解決策**: 一度内側のクエリで順位を確定させ、それを一つの「テーブル」として外側から参照することで、はじめて順位による絞り込みが可能になります。

### 3. 実務での「タイ（同着）」の扱い
期待する結果の `AD_VP` 職種を見てください。NeenaさんとLexさんが二人とも17,000円で「1位」として表示されています。

* **`RANK()` を使うメリット**: 今回のように「1位が二人いたら、二人とも出す」という要件に最適です。
* **`ROW_NUMBER()` を使う場合**: もし「誰でもいいから各職種から**一人だけ**抽出したい」という場合は、同額でも強制的に1位と2位に分ける `ROW_NUMBER()` を使います。

### 4. 期待する結果の確認
結果を見ると、高給取りの `AD_PRES`（社長）から、事務職の `PU_CLERK` まで、それぞれの職種における「頂点」がずらりと並んでいます。
このように、異なるカテゴリのデータを横並びで比較できるのが分析関数の強みです。

----
<br><br>

# 問題12-3：合計・平均（分析関数の応用）

### 難易度：★★☆☆☆ (Lv.2)

## 問題
SHスキーマの`PRODUCTS`テーブルを使用して、商品ID（`PROD_ID`）が「**23**」の商品について、その価格（`PROD_LIST_PRICE`）を多角的に比較するための参考値を算出してください。

**【算出する項目】**

* **CAT_MAX**：その商品と同じカテゴリ（`PROD_CATEGORY_ID`）内での最高価格。
* **ALL_AVG**：全商品の平均価格（小数点第3位を四捨五入して第2位まで表示）。
* **CAT_AVG**：その商品と同じカテゴリ内での平均価格（同上）。
* **CAT_RANK**：その商品と同じカテゴリ内での価格が高い順のランキング。


## 期待する結果
| PROD_ID | PROD_NAME           | PROD_LIST_PRICE | CAT_MAX | ALL_AVG | CAT_AVG | CAT_RANK | 
| ------- | ------------------- | --------------- | ------- | ------- | ------- | -------- | 
| 23      | Plastic Cricket Bat | 21.99           | 199.99  | 139.55  | 31.41   | 12       | 

## 解答例
```sql:例1：一般的な書き方
WITH analyzed_view AS (
    SELECT
        prod_id,
        prod_name,
        prod_list_price,
        MAX(prod_list_price)
        OVER(PARTITION BY prod_category_id) AS "CAT_MAX",
        ROUND(AVG(prod_list_price) OVER(), 2) AS "ALL_AVG",
        ROUND(AVG(prod_list_price)
              OVER(PARTITION BY prod_category_id), 2) AS "CAT_AVG",
        RANK() OVER(PARTITION BY prod_category_id
                        ORDER BY prod_list_price DESC) AS "CAT_RANK"
    FROM
        sh.products
)
SELECT
    *
FROM
    analyzed_view
WHERE
    prod_id = 23
```
```sql:例2：WINDOW句を使用（23ai以降）
WITH analyzed_view AS (
    SELECT
        prod_id,
        prod_name,
        prod_list_price,
        MAX(prod_list_price)
        OVER w AS "CAT_MAX",
        ROUND(AVG(prod_list_price) OVER(), 2) AS "ALL_AVG",
        ROUND(AVG(prod_list_price)
              OVER w, 2) AS "CAT_AVG",
        RANK() OVER(PARTITION BY prod_category_id
                        ORDER BY prod_list_price DESC) AS "CAT_RANK"
    FROM
        sh.products
    WINDOW w AS (PARTITION BY prod_category_id)
)
SELECT
    *
FROM
    analyzed_view
WHERE
    prod_id = 23
```

## 解説
今回は分析関数の真骨頂とも言える、**「個別のデータと、グループ統計値を横並びにして比較する」** という非常に高度なテクニックですね。

通常、`GROUP BY` を使うと行が一つにまとまってしまいますが、分析関数を使うことで「自分のデータ」を維持したまま、「カテゴリ内の最大値」や「全体の平均値」といった**物差し**を横に並べることができます。


### 1. 核心：OVER句の使い分け
今回の解答例では、`OVER` 句の中身を変えることで、異なる範囲の統計値を一度に取得しています。

#### ① カテゴリごとの統計 (`PARTITION BY prod_category_id`)
```sql
MAX(prod_list_price) OVER(PARTITION BY prod_category_id)
```
「商品カテゴリという小部屋」に分けてから計算します。これにより、同じカテゴリ内での最大値や平均値が、そのカテゴリに属する全商品の横に表示されます。

#### ② 全体の統計 (空の `OVER()`)
```sql
AVG(prod_list_price) OVER()
```
`OVER` の中身を空にすると、**「テーブル全体のすべての行」** を一つの範囲として計算します。これにより、どの商品を見ても「全商品の平均」という同じ値が横に並ぶことになります。

### 2. 知っておくと得する最新機能：WINDOW句（例2）
Oracle Database 23ai（最新版）から導入された **`WINDOW` 句**は、SQLの可読性を劇的に向上させます。

```sql
WINDOW w AS (PARTITION BY prod_category_id)
```
「カテゴリごとに分ける」という定義に `w` という名前を付け、それを `OVER w` として何度も使い回しています。
同じ `PARTITION BY` を何度も書く手間が省けるだけでなく、一箇所直せば全ての計算範囲が変わるため、メンテナンスが非常に楽になります。

### 3. なぜ「分析関数」が実務で好まれるのか？
もし分析関数を使わずに同じ結果を出そうとすると、以下の手順が必要になります。
1. `GROUP BY` でカテゴリごとの最大値を出すクエリを作る。
2. `AVG` で全体の平均を出すクエリを作る。
3. それらを元のテーブルに `JOIN` でくっつける。

これではクエリが非常に長く、処理も重くなってしまいます。**分析関数なら、一度テーブルを読み込むだけでこれら全ての計算を完了できる**ため、圧倒的に高速でスマートです。

### 4. 期待する結果の分析（ビジネス視点）
結果を見ると、ID 23番の「Plastic Cricket Bat」について以下のことが一目で分かります。
* **価格**: 21.99
* **カテゴリ内での立ち位置**: カテゴリ平均（31.41）よりも安く、カテゴリ内では12位。
* **全体での立ち位置**: 全商品平均（139.55）に比べると、かなり安価な商品。

このように、**「一つの数値から、多角的な比較分析を瞬時に行う」** のが、このクエリの目的です。

> **実行順序の再確認**
> 前問同様、ここでも「分析関数を計算した後に `WHERE` で絞り込む」ためにサブクエリ（CTE）を使っていますね。この「入れ子構造」の感覚が、上級者への鍵となります。


----
<br><br>


# 【完全版】問題12-4：ランニング集計（在庫引き当てロジック）

### 難易度：★★★★☆ (Lv.4)

## 問題
商品の注文が「Boy's Shirt (White)」および「Boy's Socks (Grey)」それぞれについて **30個** ずつ入りました。
COスキーマの`INVENTORY`テーブルを参照し、以下のルールに従って、どの店舗から何個ずつピックアップ（引き当て）すればよいかを算出してください。

**【引き当てルール】**

* **優先順位**：`STORE_ID` の昇順に店舗を回ります。
* **引き当て方法**：
* その店舗の在庫（`PRODUCT_INVENTORY`）を使い、注文数（30個）に達するまで順次合算していきます。
* すでに前の店舗までの累計で注文が満たされている場合は、その店舗からはピックアップしません。
* 最後の店舗で注文数に達した場合、その店舗からは「残りの必要分」だけをピックアップします。


* **対象商品**：`PRODUCT_ID` が 1 および 6。

## 期待する結果
| PRODUCT_ID | PRODUCT_NAME        | STORE_ID | STORE_NAME    | PICK_NUM | 
| ---------- | ------------------- | -------- | ------------- | -------- | 
| 1          | Boy's Shirt (White) | 1        | Online        | 3        | 
| 1          | Boy's Shirt (White) | 2        | San Francisco | 9        | 
| 1          | Boy's Shirt (White) | 3        | Seattle       | 1        | 
| 1          | Boy's Shirt (White) | 4        | New York City | 3        | 
| 1          | Boy's Shirt (White) | 5        | Chicago       | 4        | 
| 1          | Boy's Shirt (White) | 6        | London        | 10       | 
| 6          | Boy's Socks (Grey)  | 1        | Online        | 9        | 
| 6          | Boy's Socks (Grey)  | 4        | New York City | 3        | 
| 6          | Boy's Socks (Grey)  | 5        | Chicago       | 1        | 
| 6          | Boy's Socks (Grey)  | 7        | Bucharest     | 6        | 
| 6          | Boy's Socks (Grey)  | 8        | Berlin        | 2        | 
| 6          | Boy's Socks (Grey)  | 9        | Utrecht       | 9        | 

## 解答例

```sql
WITH sum_product_inventory AS (
    SELECT
        product_id,
        store_id,
        product_inventory,
        NVL(SUM(product_inventory)
            OVER(PARTITION BY product_id
                 ORDER BY store_id
                 ROWS BETWEEN UNBOUNDED PRECEDING 
                          AND 1 PRECEDING), 0) AS acc_inv
    FROM
        co.inventory
    WHERE
        product_id IN ( 1, 6 )
    ORDER BY
        product_id,
        store_id
)
SELECT
    p.product_id,
    p.product_name,
    s.store_id,
    s.store_name,
    least(spi.product_inventory, 30 - spi.acc_inv) AS pick_num
FROM
    sum_product_inventory spi
    INNER JOIN co.products p 
      ON spi.product_id = p.product_id
    INNER JOIN co.stores   s 
      ON spi.store_id = s.store_id
WHERE
    spi.acc_inv < 30
```

## 解説
今回は実務の「在庫引き当て」「銀行口座の残高推移」「FIFO（先入先出）の計算」などで必須となる、**ランニング集計（累計計算）** の極致です。
単なる合計ではなく、「今までの累計を見て、今回の取り分を決める」という、極めてプログラミング的な思考をSQL一行で実現しています。

### 1. 核心：ウィンドウフレームの制御
このクエリの心臓部は、`SUM` 関数の中にある **`ROWS BETWEEN`** という記述です。

#### ① 「一つ手前までの累計」を出す理由
```sql
SUM(product_inventory) OVER(
    PARTITION BY product_id
    ORDER BY store_id
    ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING
) AS acc_inv
```
普通に `SUM` を取ると「自分を含めた累計」になりますが、あえて **`1 PRECEDING`（一行前）** までに絞ることで、**「現在の店舗にたどり着くまでに、他で何個確保できているか」** を算出しています。

* **UNBOUNDED PRECEDING**: 一番最初の行から
* **1 PRECEDING**: 一つ前の行まで

これにより、最初の店舗では `acc_inv`（既確保分）が `NULL` になるため、`NVL(..., 0)` で 0個として扱い、2店舗目以降から徐々に数字が積み上がっていく仕掛けです。


### 2. ロジックの魔法：LEAST 関数の使い方
メインクエリにある `least(product_inventory, 30 - acc_inv)` が、在庫引き当てのロジックを完璧に処理しています。

#### なぜ LEAST（最小値）なのか？
注文数30個に対し、残り必要数は `30 - acc_inv` で計算できます。

1.  **在庫がたっぷりある場合**:
    「残り必要数」の方が小さくなるため、`LEAST` により「必要分だけ」をピックアップします。
2.  **在庫が足りない場合**:
    「その店舗の在庫数」の方が小さくなるため、`LEAST` により「ある分だけ全部」をピックアップします。

このシンプルな関数一つで、**「足りなければ次へ、余ればそこで止める」** という複雑な分岐をこなしているのです。


### 3. 実行順序とフィルタリング
最後に、`WHERE spi.acc_inv < 30` という条件が効いています。

累計が30に達した瞬間に、次の店舗からは `acc_inv` が 30（以上）になります。この `WHERE` 句によって、**「もう注文を満たした後の、余計な店舗」** を結果からバッサリ切り捨てているわけです。

### 4. 期待する結果のシミュレーション
例えば商品1番を見てみましょう。
* 店舗1：既確保 0個 → 在庫3個あるので、3個ゲット（累計3）
* 店舗2：既確保 3個 → 在庫9個あるので、9個ゲット（累計12）
* ...
* 店舗6：既確保 20個 → 残り10個必要。在庫はたっぷりあるけど、`LEAST` により **10個だけ** ピックして完了！


----
<br><br>

# 【完全版】問題12-5：ランク関数と別解の比較（Top-N分析）

### 難易度：★★★☆☆ (Lv.3)

## 問題
HRスキーマの`EMPLOYEES`テーブルと`DEPARTMENTS`テーブルを使用して、**部署（`DEPARTMENT_NAME`）ごとに最も給与（`SALARY`）が高い従業員**を抽出してください。

**【条件およびルール】**

* 部署ごとにグループ化し、その中で給与が1位の人物を表示します。
* 1位が複数人いる場合（同額タイ）は、その全員を表示してください。
* 解答例1（RANK関数）と解答例2（LATERAL結合）の両方の考え方を理解しましょう。
* 結果の表示順は問いません。


## 期待する結果
| 	DEPARTMENT_NAME	| 	EMPLOYEE_ID	| 	FIRST_NAME	| 	LAST_NAME	| 	SALARY	| 
| 	----	| 	----	| 	----	| 	----	| 	----	| 
| 	Administration	| 	200	| 	Jennifer	| 	Whalen	| 	4400	| 
| 	Marketing	| 	201	| 	Michael	| 	Hartstein	| 	13000	| 
| 	Purchasing	| 	114	| 	Den	| 	Raphaely	| 	11000	| 
| 	Human Resources	| 	203	| 	Susan	| 	Mavris	| 	6500	| 
| 	Shipping	| 	121	| 	Adam	| 	Fripp	| 	8200	| 
| 	IT	| 	103	| 	Alexander	| 	Hunold	| 	9000	| 
| 	Public Relations	| 	204	| 	Hermann	| 	Baer	| 	10000	| 
| 	Sales	| 	145	| 	John	| 	Russell	| 	14000	| 
| 	Executive	| 	100	| 	Steven	| 	King	| 	24000	| 
| 	Finance	| 	108	| 	Nancy	| 	Greenberg	| 	12008	| 
| 	Accounting	| 	205	| 	Shelley	| 	Higgins	| 	12008	|


## 解答例
```sql:例1：RANK関数を使用
WITH re AS (
    SELECT
        employee_id,
        first_name,
        last_name,
        salary,
        department_id,
        RANK() OVER(
             PARTITION BY department_id
             ORDER BY salary DESC
        ) AS rank
    FROM
        hr.employees
)
SELECT
    dp.department_name,
    re.employee_id,
    re.first_name,
    re.last_name,
    re.salary
FROM re
    INNER JOIN hr.departments dp 
      ON re.department_id = dp.department_id
WHERE
    re.rank = 1
```

```sql:例2：LATERAL結合を使用して相関副問い合わせする
SELECT
    dp.department_name,
    re.employee_id,
    re.first_name,
    re.last_name,
    re.salary
FROM  hr.departments dp
    CROSS JOIN LATERAL (
        SELECT
            employee_id,
            first_name,
            last_name,
            salary,
            department_id
        FROM  hr.employees em
        WHERE
            em.department_id = dp.department_id
        ORDER BY salary DESC
        FETCH FIRST ROW WITH TIES
    ) re
```

## 解説
今回のテーマは、これまでの「Top-N分析」をさらに一歩進めた、**「分析関数 vs 最新SQL構文（LATERAL結合）」** の比較です。
実務において「各カテゴリの最大値を持つ行をまるごと持ってくる」という処理は非常に頻出しますが、書き方によってパフォーマンスや可読性が大きく変わる、エンジニアの腕の見せ所でもあります。

### 1. 例1：分析関数（RANK）による王道アプローチ
前問（12-2）でも登場した、最も一般的で汎用性の高い書き方です。

1.  **内側（CTE: re）**: 全従業員に、自分の部署内での順位をスタンプ（`RANK()`）していきます。
2.  **外側**: 部署テーブルと結合し、スタンプが「1位」の人だけをフィルタリングします。

> **なぜ結合を外側でするのか？**
> 先に従業員テーブルだけで順位を確定させてから部署名をくっつけることで、無駄な結合処理を減らすことができます。この「絞り込んでから繋げる」のが、大規模データでの鉄則です。


### 2. 例2：LATERAL結合による「次世代」アプローチ
Oracle 12cから導入された比較的新しい構文で、プログラミングの **「For Each ループ」** に近い感覚で書けるのが特徴です。

#### ① 動作の仕組み
1.  **親（dp）**: まず「部署リスト」を1行ずつ読み込みます。
2.  **子（LATERAL内）**: 読み込んだ部署IDを「引数」として渡し、その部署の従業員を給与順に並べ替えます。
3.  **抽出**: `FETCH FIRST ROW WITH TIES` で、その部署の1位の人（同着含む）を1人（あるいは複数人）だけ取ってきます。

#### ③ FETCH FIRST ROW WITH TIES の凄さ
これまで `RANK() OVER...` と書いてから外側で `WHERE rank = 1` と書かなければならなかった処理を、**たった一行で、しかも同着（TIES）を含めて**記述できます。

### 3. 実務での使い分け：どっちが最強？

| 特徴 | RANK関数（例1） | LATERAL結合（例2） |
| :--- | :--- | :--- |
| **可読性** | CTEを使うため、構造が分かりやすい。 | 手続き型（ループ）っぽく直感的に書ける。 |
| **得意なデータ** | 従業員全員をまんべんなく見る場合。 | **部署数は少ないが、各部署の人数が膨大**な場合。 |
| **インデックス** | テーブルフルスキャンになりがち。 | 部署IDにインデックスがあれば、**爆速**で動く。 |


### 4. 期待する結果のポイント
結果を見ると、Finance部門のNancyさん（12,008）や、Sales部門のJohnさん（14,000）など、各部門の「最高権力者（最高給与者）」が抽出されています。

例1の `RANK()` と例2の `WITH TIES` はどちらも「同着1位」を漏らさず出力するため、もし給与が全く同じ1位が2人いれば、その部署は2行表示されます。これがビジネスデータとしての「誠実さ」です。

<br>

:::message
### LATERAL結合

#### 1. 一言でいうと「1行ずつ注文を聞いてくれる結合」
通常の結合（JOIN）は、2つの大きな表をドカン！と一気にくっつけるイメージです。
それに対してLATERAL結合は、**「左側のテーブルの1行ごとに、右側のサブクエリを実行する」** という動きをします。

##### 具体的なイメージ（部署と従業員の例）
1. **部署テーブル（左）** から「営業部」を取り出す。
2. 「営業部」という情報を**右側のサブクエリ**に渡す。
3. サブクエリが「営業部の中で給料が高い人」を1人探し出す。
4. 次に、部署テーブルから「総務部」を取り出す……（以下繰り返し）

このように、左側のデータを使って右側のクエリを「その都度」動かせるのが最大の特徴です。
<br>
#### 2. なぜ「LATERAL」が必要なの？（これまでの限界）
実は、通常のサブクエリ（副問い合わせ）では、外側のテーブルの列を自由に使えないというルールがありました。

```sql
-- ❌ これはエラーになる（昔のSQLの限界）
SELECT dp.department_name, re.first_name
FROM hr.departments dp
INNER JOIN (
    SELECT * FROM hr.employees em 
    WHERE em.department_id = dp.department_id -- ここで「dpって誰？」となる
) re ON 1=1;
```

通常のJOINでは、カッコの中（サブクエリ）は**外の世界（dp）を知ることができません。**
ここで `LATERAL` というキーワードを添えると、**「外側の `dp` の値を中に入れてもいいよ！」** という許可が出るのです。
<br>

#### 3. クエリの解剖
今回の例2をもう一度見てみましょう。

```sql
SELECT
    dp.department_name,
    re.first_name,
    re.salary
FROM hr.departments dp  -- ① まず「部署」を1つ選ぶ
CROSS JOIN LATERAL (    -- ② その部署の情報を「中」に持ち込む
    SELECT first_name, salary
    FROM hr.employees em
    WHERE em.department_id = dp.department_id -- ③ 外の dp.department_id を参照！
    ORDER BY salary DESC
    FETCH FIRST ROW WITH TIES -- ④ その部署内でのトップを特定
) re
```
<br>
##### 💡 ポイント
* **「部署ごとにトップ1」** という処理が、カッコの中に完結して書けるので、ロジックが非常にスッキリします。
* `WITH TIES` を使えば、「給料が同額の1位」が複数いても、LATERALが自動的に全員分を「1位のセット」として返してくれます。
<br>
#### 4. LATERALを使うメリット
1.  **「各グループのトップN件」が書きやすい**
    「各部署のトップ3を出したい」といった場合、通常のGROUP BYでは非常に複雑になりますが、LATERALならサブクエリに `FETCH FIRST 3 ROWS` と書くだけです。
2.  **計算結果を使い回せる**
    例えば「消費税を計算した結果（税込額）」を使って、さらに別の計算をしたい場合、LATERAL内で計算した値を外側のSELECTでそのまま使えます。
3.  **パフォーマンスの最適化**
    「部署数は少ないけれど、全従業員数は膨大」という場合、全社員をランク付け（RANK関数）するよりも、部署ごとにピンポイントでトップを探しに行くLATERALの方が速いことがあります。
:::

----
<br><br>

# 問題12-6：構成比の算出（RATIO_TO_REPORT）

### 難易度：★★★☆☆ (Lv.3)

## 問題
SHスキーマの`SUPPLEMENTARY_DEMOGRAPHICS`テーブルを利用して、顧客の属性に基づいた独自のセグメントを作成し、それぞれのセグメントが全体の中でどの程度の割合を占めているかを調査してください。

**【顧客セグメント（CUSTOMER_SEGMENT）の定義】**
1. **Established Large Family**：居住年数（`YRS_RESIDENCE`）が 10年より長く、かつ世帯人数（`HOUSEHOLD_SIZE`）が「4-8人」または「9人以上」。
2. **Long-term Resident**：上記以外で、居住年数が 10年より長い。
3. **Newcomer**：居住年数が 2年以下。
4. **Other**：上記いずれにも当てはまらない。

**【集計ルール】**
* セグメント別のレコード数（`TOTAL_COUNT`）を出す。
* 全体に対する割合（`PERCENTAGE`）を算出し、「XX.XX%」の形式で表示する。
* 結果は、レコード数が多い順に並べる。


## 期待する結果
| CUSTOMER_SEGMENT         | TOTAL_COUNT | PERCENTAGE | 
| ------------------------ | ----------- | ---------- | 
| Other                    | 3534        | 78.53%     | 
| Newcomer                 | 949         | 21.09%     | 
| Long-term Resident       | 14          | 0.31%      | 
| Established Large Family | 3           | 0.07%      |

## 解答例
```sql
SELECT
    CASE
        WHEN yrs_residence > 10
             AND household_size IN ( '4-8', '9+' ) THEN
            'ESTABLISHED LARGE FAMILY'
        WHEN yrs_residence > 10 THEN
            'LONG-TERM RESIDENT'
        WHEN yrs_residence <= 2 THEN
            'NEWCOMER'
        ELSE
            'OTHER'
    END      AS customer_segment, -- ここで名前を付ける
    COUNT(*) AS total_count,
    ROUND(RATIO_TO_REPORT(COUNT(*))
          OVER() * 100,
          2)
    || '%'   AS percentage
FROM
    sh.supplementary_demographics
GROUP BY
    customer_segment  -- ★23aiの場合、エイリアスがそのまま使える！
ORDER BY
    total_count DESC;
```

## 解説
分析関数の真髄、「構成比（シェア）」の計算ですね。マーケティング分析の現場では、単に「数」を出すだけでなく、「それは全体の中で何％を占めるのか？」を瞬時に提示するスキルが求められます。
今回の主役は、Oracle独自の強力な関数 **`RATIO_TO_REPORT`** です。

### 1. 核心：RATIO_TO_REPORT の凄さ
通常、「全体に対する割合」を計算するには、以下の計算式が必要です。
$$\text{PERCENTAGE} = \frac{\text{セグメントの件数}}{\text{テーブル全体の総件数}} \times 100$$

これを普通のSQLで書こうとすると、「セグメントごとのカウント」と「全体のカウント」を出すためにサブクエリを組み合わせる必要があり、非常に面倒です。

#### ① 動作の仕組み
```sql
RATIO_TO_REPORT(COUNT(*)) OVER()
```
* **`COUNT(*)`**: 各セグメントの集計結果。
* **`OVER()`**: 範囲を「全体」に指定。
* **役割**: 「自分のセグメントの数」を「分母（全体の数）」で割った値を、0〜1の範囲で自動的に算出してくれます。これに100を掛ければ、即座にパーセンテージの完成です！


### 2. CASE式の「優先順位」の鉄則

セグメント定義の `CASE` 文において、条件を書く順番は非常に重要です。



* `Established Large Family`（条件2つ）
* `Long-term Resident`（条件1つ）

この2つは「居住年数 > 10」という条件が重複しています。`CASE` 文は **「上から評価して、最初に一致したところで抜ける」** という性質があるため、より条件が厳しい（細かい）方を先に書くのが鉄則です。もし順番を逆にすると、大家族の人も全員ただの `Long-term Resident` に分類されてしまいます。


### 3. 実務の知恵：Oracle 23ai の「エイリアス参照」
解答例の `GROUP BY customer_segment` に注目してください。
従来のSQL（19c以前）では、`SELECT` 句で付けた名前（エイリアス）を `GROUP BY` で使うことはできず、長い `CASE` 文をもう一度書く必要がありました。

> **23ai ならコードがスッキリ！**
> 最新の Oracle 23ai では、`SELECT` で定義した名前をそのまま `GROUP BY` や `HAVING` で使えるようになりました。これにより、コードの重複が減り、ミスも劇的に少なくなります。実務で古いバージョンを使っている場合は、インラインビュー（サブクエリ）で囲ってからグループ化するのが一般的です。


### 4. 期待する結果の分析（ビジネス視点）

結果を見ると、`Other` が約78.5%と圧倒的多数を占め、`Established Large Family` はわずか0.07%しかいません。

* **分析のヒント**: 「大家族かつ定住層」は非常に希少なセグメントであることがわかります。もしあなたが不動産会社のマーケターなら、このわずかな0.07%に向けて「広々としたリノベーション物件」のDMを送るよりも、21%を占める `Newcomer` に向けて「地域密着のライフスタイル提案」をする方が効率的かもしれません。


## 参考リンク
https://docs.oracle.com/cd/E16338_01/server.112/b56299/functions142.htm
https://atmarkit.itmedia.co.jp/ait/articles/0510/29/news012_2.html


----
<br><br>



# 問題12-7：項目ごとのシェア（PARTITION BY の活用）

### 難易度：★★★☆☆ (Lv.3)

## 問題
SHスキーマの`SUPPLEMENTARY_DEMOGRAPHICS`テーブルを利用して、特定の学歴（`EDUCATION`）グループの中で、それぞれの職業（`OCCUPATION`）が占める構成比を算出してください。

**【抽出・集計ルール】**

* **対象データ**：学歴（`EDUCATION`）が **'PhD'** または **'1st-4th'** の顧客。
* **CUST_COUNT**：学歴×職業ごとの顧客数。
* **OCC_SHARE_IN_EDU**：その学歴グループ内（100%）における各職業の割合を算出し、「XX.XX%」の形式で表示する。
* **表示順**：学歴（`EDUCATION`）の昇順、次に顧客数（`CUST_COUNT`）の降順。


## 期待する結果
| EDUCATION | OCCUPATION | CUST_COUNT | OCC_SHARE_IN_EDU | 
| --------- | ---------- | ---------- | ---------------- | 
| 1st-4th   | Other      | 8          | 47.06%           | 
| 1st-4th   | Crafts     | 2          | 11.76%           | 
| 1st-4th   | Exec.      | 2          | 11.76%           | 
| 1st-4th   | Farming    | 2          | 11.76%           | 
| 1st-4th   | Transp.    | 2          | 11.76%           | 
| 1st-4th   | Prof.      | 1          | 5.88%            | 
| PhD       | Prof.      | 37         | 75.51%           | 
| PhD       | Exec.      | 5          | 10.2%            | 
| PhD       | ?          | 2          | 4.08%            | 
| PhD       | Crafts     | 2          | 4.08%            | 
| PhD       | TechSup    | 1          | 2.04%            | 
| PhD       | Sales      | 1          | 2.04%            | 
| PhD       | Cleric.    | 1          | 2.04%            | 

## 解答例
```sql
SELECT
    d.education,
    d.occupation,
    COUNT(*) AS cust_count,
    ROUND(ratio_to_report(COUNT(*))
          OVER(PARTITION BY d.education) * 100, 2)
        || '%'   AS occ_share_in_edu
FROM
    sh.supplementary_demographics d
WHERE
    d.education IN ('PhD','1st-4th')
GROUP BY
    d.education,
    d.occupation
ORDER BY
    education ASC,
    cust_count DESC
```

## 解説
分析関数セクションのクライマックス、**「グループ内シェア（部分構成比）」** の算出です。
前問（12-6）では「全体に対する割合」を出しましたが、今回は「学歴というグループの中での割合」を出すという、より実戦的なデータ分析のテクニックを学びます。


### 1. 核心：PARTITION BY で「分母」を切り替える
今回の主役も `RATIO_TO_REPORT` ですが、`OVER` 句の中に **`PARTITION BY`** が加わったことが最大のポイントです。

#### ① 動作の仕組み
* **前問（OVER()）**: テーブル全体の総計を分母にして計算。
* **今回（OVER(PARTITION BY education)）**: 「PhDならPhDの合計」「1st-4thなら1st-4thの合計」という具合に、**学歴ごとに分母をリセット**して計算します。

これによって、「PhD保持者のうち、専門職（Prof.）は何％か？」という、特定の属性内部でのパワーバランスを可視化できるようになります。

### 2. 実行順序：集計（GROUP BY）の後に計算
このクエリがどう動いているか、頭の中の処理順序を整理しましょう。

1.  **絞り込み（WHERE）**: まず、学歴を 'PhD' と '1st-4th' に限定します。
2.  **集計（GROUP BY）**: 「学歴 × 職業」の組み合わせで件数を数えます（`CUST_COUNT`）。
3.  **シェア計算（分析関数）**: **集計された後の数字**を使って、グループ内での割合を計算します。
    * PhDグループの合計（37 + 5 + 2... = 49人）を出し、それぞれの職業をその 49 で割る。
    * 1st-4thグループの合計（8 + 2 + 2... = 17人）を出し、それぞれの職業をその 17 で割る。

### 3. ビジネス視点での分析：データの裏側を読む
期待する結果を見ると、非常に興味深い傾向が読み取れます。

* **PhD（博士号）層**: `Prof.`（専門職）が **75.51%** と圧倒的です。高学歴が特定の職種に強く結びついていることがわかります。
* **1st-4th（初等教育）層**: 最も多い `Other` でも **47.06%** です。特定の職種に偏らず、多様な職業に分散している（あるいは専門職以外の選択肢が多い）傾向が見て取れます。

このように、母数（分母）が全く違うグループ同士であっても、**「％」という共通の尺度**に変換することで、その内部構造をフェアに比較できるのがこのスキルの凄みです。


### 4. プロのテクニック：NULLや不明データの扱い
結果の中に `?` という職業が含まれています。実務データには必ずと言っていいほど、このような「不明値」や「ノイズ」が含まれます。
分析関数を使う際は、これらも含めて 100% とするのか、あるいはあらかじめ `WHERE` 句で除外して「有効回答のみのシェア」を出すのか。ここを判断できるようになると、データサイエンティストへの道が見えてきます。

----
<br><br>

# 問題12-8：前後比較（LAG関数）

### 難易度：★★★☆☆ (Lv.3)

## 問題

HRスキーマの`EMPLOYEES`テーブルより、**同一部門内において、自分の1つ前に採用された人**の情報を取得してください。

**【抽出・編集ルール】**

* **対象データ**：`EMPLOYEE_ID` が 190 から 210 までの従業員。
* **表示項目**：
* 自身の情報：`EMPLOYEE_ID`, `DEPARTMENT_ID`, `FIRST_NAME`, `HIRE_DATE`
* 1つ前に採用された人の情報：`PREV_FIRST_NAME`, `PREV_HIRE_DATE`


* **ソート順**：部門ID（`DEPARTMENT_ID`）の昇順、次に採用日（`HIRE_DATE`）の昇順。



## 期待する結果
| EMPLOYEE_ID | DEPARTMENT_ID | FIRST_NAME | HIRE_DATE            | PREV_FIRST_NAME | PREV_HIRE_DATE       | 
| ----------- | ------------- | ---------- | -------------------- | --------------- | -------------------- | 
| 200         | 10            | Jennifer   | 2013-09-17T00:00:00Z |                 |                      | 
| 201         | 20            | Michael    | 2014-02-17T00:00:00Z |                 |                      | 
| 202         | 20            | Pat        | 2015-08-17T00:00:00Z | Michael         | 2014-02-17T00:00:00Z | 
| 203         | 40            | Susan      | 2012-06-07T00:00:00Z |                 |                      | 
| 192         | 50            | Sarah      | 2014-02-04T00:00:00Z |                 |                      | 
| 193         | 50            | Britney    | 2015-03-03T00:00:00Z | Sarah           | 2014-02-04T00:00:00Z | 
| 196         | 50            | Alana      | 2016-04-24T00:00:00Z | Britney         | 2015-03-03T00:00:00Z | 
| 197         | 50            | Kevin      | 2016-05-23T00:00:00Z | Alana           | 2016-04-24T00:00:00Z | 
| 194         | 50            | Samuel     | 2016-07-01T00:00:00Z | Kevin           | 2016-05-23T00:00:00Z | 
| 190         | 50            | Timothy    | 2016-07-11T00:00:00Z | Samuel          | 2016-07-01T00:00:00Z | 
| 195         | 50            | Vance      | 2017-03-17T00:00:00Z | Timothy         | 2016-07-11T00:00:00Z | 
| 198         | 50            | Donald     | 2017-06-21T00:00:00Z | Vance           | 2017-03-17T00:00:00Z | 
| 191         | 50            | Randall    | 2017-12-19T00:00:00Z | Donald          | 2017-06-21T00:00:00Z | 
| 199         | 50            | Douglas    | 2018-01-13T00:00:00Z | Randall         | 2017-12-19T00:00:00Z | 
| 204         | 70            | Hermann    | 2012-06-07T00:00:00Z |                 |                      | 
| 206         | 110           | William    | 2012-06-07T00:00:00Z |                 |                      | 

## 解答例
```sql
SELECT
    employee_id,
    department_id,
    first_name,
    hire_date,
    -- 同じ部署内で、採用日順に並べた1つ前の行の値を取得
    LAG(first_name) OVER(PARTITION BY department_id
         ORDER BY
             hire_date
    ) AS prev_first_name,
    LAG(hire_date) OVER(PARTITION BY department_id
         ORDER BY
             hire_date
    ) AS prev_hire_date
FROM
    hr.employees
WHERE
    employee_id BETWEEN 190 AND 210
ORDER BY
    department_id,
    hire_date
```

## 解説
分析関数の真骨頂とも言える、**「行と行の値を比較する」** テクニックに到達しました。
通常、SQLは「現在の行」のデータしか見ることができませんが、**`LAG`関数**や **`LEAD`関数** を使うと、あたかもタイムマシンのように **「1つ前の行」や「1つ後の行」の値**を現在の行に持ってくることができます。

### 1. 核心：LAG関数の「のぞき見」ロジック
`LAG`は「遅れる」という意味で、指定した順序に従って「前の行」にあるデータを取得します。

#### ① PARTITION BY department_id
「部署」という小部屋にデータを分けます。これにより、**別の部署の人が混ざることなく**、同じ部署内だけで「誰が先に採用されたか」を比較できます。

#### ② ORDER BY hire_date
小部屋（部署）の中で、採用日の古い順に整列させます。この「並び順」が確定して初めて、「1つ前（＝自分より1回前に採用された人）」が誰なのかが決まります。


### 2. 実行結果の読み解き：なぜ「空欄（NULL）」があるのか？
期待する結果の以下の行に注目してください。

* **Jennifer (ID 200)**: 部署10の中で一番最初に採用されたため、彼女の「前」には誰もいません。そのため、`PREV_` カラムは **NULL（空欄）** になります。
* **Michael (ID 201)**: 部署20のトップバッターなので同様にNULLです。
* **Pat (ID 202)**: 同じ部署20でMichaelの次に採用されたので、`PREV_` カラムに **Michael** の情報が表示されます。

このように、**「グループの先頭行は必ずNULLになる」** という性質を理解しておくことが、実務でこの関数を使いこなすポイントです。

### 3. 実務での活用シーン：成長率の計算
この `LAG` 関数は、ビジネス分析において「前月比」や「前年比」を出す際に最も威力を発揮します。

* **売上の前月比**: `(売上 - LAG(売上) OVER(...)) / LAG(売上) OVER(...)`
* **株価の変動率**: 1日前の終値と今日の値を比較する。
* **製造ラインの工程間隔**: 前の製品が通過してから何分後に次の製品が来たかを測る。

> **LEAD関数（未来をのぞく）**
> `LAG`（過去）とは逆に、1つ先の行を見るのが **`LEAD`関数** です。「次の採用者は誰か？」「次回の注文はいつか？」といった分析をしたいときは、これを使います。

### 4. 自己結合との違い
もし `LAG` を使わずにこの問題を解こうとすると、非常に複雑な「自己結合（SELF JOIN）」が必要になります。

* **自己結合**: テーブルを2回読み込み、複雑な結合条件を書く必要があるため、処理が重くなりがち。
* **分析関数（LAG）**: テーブルを1回スキャンするだけで前後を特定できるため、**圧倒的に高速で、コードもスッキリ**します。

----
<br><br>


# 【完全版】問題12-9：同一年月の複数注文（分析関数による解＜問題10-5の別解＞）

### 難易度：★★★☆☆ (Lv.3)

## 問題
OEスキーマの`ORDERS`テーブルを使用して、**同じ年月の間に2回以上の注文を行った顧客**とその注文詳細を特定してください。

**【条件およびルール】**

* **分析関数**を使用して、「顧客ごと・年月ごと」の注文回数を計算してください。
* 注文が2回以上（複数）ある行のみを抽出してください。
* 顧客名（`CUST_NAME`）は `CUSTOMERS` テーブルより取得し、姓名を半角スペースで結合してください。
* 結果の表示順は問いません。


## 期待する結果
| CUSTOMER_ID | CUST_NAME          | ORDER_ID | ORDER_DATE                  | 
| ----------- | ------------------ | -------- | --------------------------- | 
| 105         | Matthias MacGraw   | 2358     | 2008-01-08T17:03:12.654278Z | 
| 105         | Matthias MacGraw   | 2356     | 2008-01-26T09:22:41.934562Z | 
| 148         | Gustav Steenburgen | 2451     | 2007-12-17T17:03:52.562632Z | 
| 148         | Gustav Steenburgen | 2386     | 2007-12-06T12:22:34.225609Z | 


## 解答例
```sql:例1：分析関数（COUNT）を使用
SELECT
    customer_id,
    cust_name,
    order_id,
    order_date
FROM
    (
        SELECT
            o.customer_id,
            c.cust_first_name || ' ' || c.cust_last_name  AS cust_name,
            o.order_id,
            o.order_date,
            COUNT(*) OVER(PARTITION BY o.customer_id,
                  TRUNC(o.order_date, 'mm')) AS monthly_order_count
        FROM
                 oe.orders o
            INNER JOIN oe.customers c 
               ON o.customer_id = c.customer_id
    )
WHERE
    monthly_order_count > 1; -- 2回以上注文した人だけ抽出
```

```sql:例2：分析関数（MIN,MAX）を使用
SELECT
    customer_id,
    cust_name,
    order_id,
    order_date
FROM
    (
        SELECT
            o.customer_id,
            c.cust_first_name || ' ' || c.cust_last_name  AS cust_name,
            o.order_id,
            o.order_date,
            MIN(o.order_id) OVER w AS monthly_min,
            MAX(o.order_id) OVER w AS monthly_max
        FROM
                 oe.orders o
            INNER JOIN oe.customers c 
               ON o.customer_id = c.customer_id
        WINDOW w as (PARTITION BY o.customer_id,TRUNC(o.order_date, 'mm'))
    )
WHERE
    monthly_min <> monthly_max
```

## 解説
以前、問題10-5で「自己結合」を使って解いたこの課題。今回は **「分析関数（ウィンドウ関数）」** という現代の強力な武器を使ってスマートに解決しています。
「同じ人が同じ月に何回注文したか」を特定する際、自己結合だとテーブルを2回読み込む必要がありましたが、分析関数なら**一度テーブルをスキャンするだけで答えが出る**ため、実務でもこちらが推奨されるパターンです。

### 1. 核心：PARTITION BY に複数項目を指定する
今回の解法のキモは、`OVER` 句の中にある `PARTITION BY` です。

#### ① 動作の仕組み
```sql
PARTITION BY o.customer_id, TRUNC(o.order_tms, 'mm')
```
通常は1つの項目で分けますが、このようにカンマで区切ることで **「顧客ID 且つ 注文月」** という、より細かい「小部屋」を作ることができます。

1. **分ける**: 「田中さんの2021年1月」「田中さんの2021年2月」「佐藤さんの2021年1月」……といった具合に部屋を分けます。
2. **数える**: その各部屋の中で、注文（行）がいくつあるかを数えます。

### 2. 解答例のロジック比較

#### 例1：COUNT(*) を使う（王道）
最も直感的で分かりやすいアプローチです。
* **ロジック**: 「その部屋の行数が2以上（`> 1`）」であれば、その月は複数回注文したことになります。
* **メリット**: コードを読んだ瞬間に「月間の注文回数を数えているんだな」と意図が伝わります。

#### 例2：MIN と MAX を比較する
「最大と最小が違う ＝ 2種類以上のデータがある」という性質を利用した面白い解法です。
* **ロジック**: 同一月内の「最小の注文ID」と「最大の注文ID」を取得します。もし注文が1回しかなければ両者は一致しますが、**2回以上あれば必ず「最小 ≠ 最大」** になります。
* **メリット**: ユニークキー（ID）の性質を活かした非常にテクニカルな手法です。


### 3. なぜ「自己結合」より「分析関数」なのか？

| 比較項目 | 自己結合 (10-5) | 分析関数 (12-9) |
| :--- | :--- | :--- |
| **可読性** | JOIN条件が複雑になりがち。 | 1行の定義で完結。 |
| **パフォーマンス** | **△** テーブルを複数回読むため重い。 | **◎** テーブルスキャン1回で済む。 |
| **拡張性** | 3回、4回注文した人の特定が大変。 | `> 2` や `> 3` と変えるだけで即対応。 |

> **TRUNC(date, 'mm') の便利さ**
> 日付から「年月」だけを取り出して比較する際、`TRUNC(order_date, 'mm')` は「その月の1日」に日付を切り捨ててくれます。`EXTRACT` を2回書くよりもスマートで、インデックス効率が良い場合も多いですよ！

### 4. 期待する結果の確認
結果を見ると、Matthias MacGrawさん（105番）は、2008年1月に「8日」と「26日」の2回注文していることが、分析関数によって1行ずつ正確にマークされています。

分析関数を使えば、**「行のディテール（具体的な注文日など）を保持したまま、グループ統計（その月の回数）でフィルタリングする」** という、高度なデータ抽出がこんなにも美しく書けるわけです。
