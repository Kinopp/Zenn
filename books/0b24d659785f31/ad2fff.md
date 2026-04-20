---
title: "分析関数（全9問）"
free: false
---

# 前提知識
**分析関数（ウィンドウ関数）** は、現代のSQLにおいて「中級者」から「上級者」へステップアップするための最大の壁であり、同時に最強の武器でもあります。
Oracle 23aiを使い倒すなら、この機能を知らずにデータ分析は語れません。

## 1. 分析関数とは？
集計関数（SUMやAVG）がデータを「ギュッと一つにまとめる」のに対し、分析関数は **「行をまとめることなく、各行に集計結果を添える」** 機能です。
イメージとしては、各行の横に **「魔法の鏡」** を置くようなものです。その鏡には、自分自身のデータだけでなく、「同じ部署の平均」や「前後の行の値」などが映し出されます。

### 集計関数との決定的な違い

| 特徴 | 集計関数 (GROUP BY) | 分析関数 (OVER句) |
| :--- | :--- | :--- |
| **結果の行数** | グループの数まで減る | **元の行数のまま** |
| **行同士の比較** | できない | **前後や範囲の比較が得意** |
| **主な用途** | 全体の要約レポート | ランキング、移動平均、前月比 |


## 2. 基本の構文：`OVER` 句の3要素
分析関数の心臓部は `OVER()` です。この中に「どの範囲を」「どう並べて」計算するかを記述します。

```sql
関数名() OVER (
  PARTITION BY 列名  -- 1. どこで区切るか（グループ化）
  ORDER BY 列名      -- 2. どの順で並べるか（並び順）
  ROWS/RANGE ...     -- 3. どこまで含めるか（フレーム）
)
```

1.  **`PARTITION BY`**: データの仕切り板。部署ごと、年度ごとなどに区切ります。
2.  **`ORDER BY`**: 区切りの中での並び順。ランキングや累計を出す時に必須です。
3.  **`ROWS/RANGE`**: 「自分の前後3行」といった、さらに細かい範囲指定です。


![](https://storage.googleapis.com/zenn-user-upload/10993cbd2cba-20260322.png)


----
<br><br>


# 問題12-1（全体の順位） *Lv2*
### **HRスキーマ`EMPLOYEES`テーブルより、`JOB_ID`が「IT_PROG」である従業員について、`SALARY`が大きい順にランク付けを行って下さい。（ランキング順に表示）**

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
### 例1：RANK関数を使用

現在、実務でランキングを作るなら**間違いなくこちら**を選びます。

#### 解説
* **`RANK() OVER(ORDER BY salary DESC)`**: 「給与が高い順に並べて、上から順位を振れ」という命令です。
* **同順位の扱い**: 給与が同じ人がいた場合、同じ順位（例：2位が2人）を割り当て、**次の順位を飛ばします**（例：1位、2位、2位、**4位**...）。

#### 注意点：初心者がハマる「似た関数」との違い
資料では、以下の3つの違いをセットで教えると非常に喜ばれます。

| 関数 | 同順位の挙動 | 次の番号 |
| :--- | :--- | :--- |
| **`RANK`** | 同じ順位にする | 飛ばす（1, 2, 2, **4**） |
| **`DENSE_RANK`** | 同じ順位にする | 飛ばさない（1, 2, 2, **3**） |
| **`ROW_NUMBER`** | 厳格に1行ずつ振る | 飛ばさない（1, 2, 3, 4） |


### 例2：自己結合で解決

分析関数がなかった時代の「力技」ですが、SQLの論理的な思考を養うのに最適です。

#### 解説
* **考え方のヒント**: 「自分の順位 ＝ **自分より給与が高い人の数 + 1**」という論理を利用しています。
* **`LEFT JOIN` の役割**: 1位の人（自分より高い人が0人の人）も消えないように、外部結合を使っています。
* **`COUNT(e2.salary) + 1`**: 自分（e1）より高い給与を持つ人（e2）が何人いるか数え、最後に1を足して自分の順位にしています。

#### 注意点：実務での「危険性」
* **パフォーマンスの劣化（$O(n^2)$）**: 1,000人の社員がいれば、最悪 $1,000 \times 1,000 = 1,000,000$ 回の比較が発生します。データ量が増えると劇的に遅くなるため、**現代の実務でこの書き方をするメリットはほぼありません。**
* **重複データの罠**: `GROUP BY` に `employee_id` を含めているのは、同姓同名の別人がいた場合に集計が混ざるのを防ぐためです。


<br><br>

----

# 問題12-2（グループごとのナンバーワン（Top-N分析））  *Lv2*
### **HRスキーマ`EMPLOYEES`テーブルより、各職種における最高給与者を特定してください。具体的には、`JOB_ID`ごとに、`SALARY`が大きい順にランク付けを行い、１位の人を表示してください。（`SALARY`,`JOB_ID`の昇順で表示）**

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
このクエリは、「全社員の中のトップ」ではなく、**「職種（job_id）というグループごとに、誰が一番高い給与をもらっているか」** を抽出する、スマートな書き方です。

### 何をやっているのか？

このクエリは「2段階」のステップを踏んでいます。

#### 1. インライン・ビュー（内側のSELECT）
ここで全員に **「職種内での背番号（順位）」** を振っています。
* **`PARTITION BY job_id`**: ここが魔法の言葉です。社員を職種ごとに個別の「小部屋」に分けます。
* **`ORDER BY salary DESC`**: 各小部屋の中で、給与が高い順に並べます。
* **`RANK()`**: その並び順に従って、1位、2位…と順位をつけていきます。

#### 2. 外側のSELECT
内側で計算した「順位（rank）」を使って、最後に絞り込みを行います。
* **`WHERE rank = 1`**: 各職種で1位になった人だけを残します。

### 押さえるべきポイント
#### なぜカッコ（副問合せ）が必要なの？
SQLの評価順では、**「WHERE句は、RANK()などの分析関数よりも先に実行されます」** 
そのため、一度カッコの中で計算を終わらせてからでないと、「順位が1位の人」という条件で絞り込むことができないのです。

### 実践的な注意点

#### ① 同率1位がいた場合
`RANK()` 関数を使っているため、もし「IT_PROG」の中に最高給与額（例: 9000）の人が2人いた場合、**どちらも `rank = 1` になり、両方の行が出力されます。**
* **対策:** もし「同率でも1人だけ出したい」という場合は、`ROW_NUMBER()` 関数を使うと、内部的にユニークな順位を振ってくれます。

#### ② ソート（ORDER BY）の意図
外側の最後に `ORDER BY salary, job_id` とあります。
* これは「職種ごとのトップ1」を、さらに「給与が低い職種トップ1」から順に並べていることを意味します。
* 会社の「職種ごとの最高給与格差」を見たい場合に有効な並び順です。


----
<br><br>

# 問題12-3（合計、平均）  　*Lv2*
### SHスキーマ`PRODUCTS`テーブルより、PROD_ID=23の商品について、商品メーカー希望小売価格（`PROD_LIST_PRICE`）と比較するための下記参考値を表示してください。
* **CAT_MAX**：同一商品カテゴリ（`PROD_CATEGORY_ID`）におけるメーカー希望小売価格（`PROD_LIST_PRICE`）の最大値 
* **ALL_AVG**：全商品のメーカー希望小売価格（`PROD_LIST_PRICE`）の平均値（小数点第３位四捨五入） 
* **CAT_AVG**：同一商品カテゴリ（`PROD_CATEGORY_ID`）におけるメーカー希望小売価格（`PROD_LIST_PRICE`）の平均値（小数点第３位四捨五入）
* **CAT_RANK**：同一商品カテゴリ（`PROD_CATEGORY_ID`）におけるメーカー希望小売価格（`PROD_LIST_PRICE`）の降順でのランキング


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
このクエリのポイントは、**「特定の1商品（ID: 23）の情報を見ているのに、その背後にある『カテゴリ内での立ち位置』や『全体平均との比較』が同時にわかる」** という点にあります。

### 1. クエリの解説：何をやっているのか？
このクエリは、「商品ID 23」というミクロな視点に、マーケット全体のマクロな視点をガッチャンコさせています。

#### 計算している4つの指標
* **`CAT_MAX`**: その商品が属するカテゴリの中で、一番高い商品の値段。
* **`ALL_AVG`**: 商品マスター全件の平均価格（全体の相場）。
* **`CAT_AVG`**: 同じカテゴリ内の平均価格（カテゴリの相場）。
* **`CAT_RANK`**: カテゴリ内で価格が高い順に並べた時の順位。



#### ステップ別の動き
1.  **WITH句 (`analyzed_view`)**: 
    まず「分析用ビュー」をメモリ上に展開します。ここで `OVER()` を使って、全商品に対してカテゴリごとの集計やランク付けを計算してしまいます。
2.  **外側のSELECT**: 
    計算済みの大きな表から、`WHERE prod_id = 23` で自分の知りたい商品だけを「スッと」抜き出します。

### 2. 【23aiの新機能】WINDOW句（例2）の凄さ
例1と例2の結果は全く同じですが、例2では **`WINDOW w AS (...)`** という構文が使われています。

#### なぜこれが嬉しいのか？
これまでのSQLでは、同じ `PARTITION BY prod_category_id` を何度も何度も書く必要がありました（例1参照）。
* **DRY原則（Don't Repeat Yourself）**: 同じ定義を `w` という名前にまとめて一箇所で定義できます。
* **保守性が爆上がり**: もし「カテゴリごと」ではなく「メーカーごと」に分析単位を変えたくなっても、末尾の `WINDOW` 句を1箇所書き換えるだけで、`MAX` も `AVG` もすべて一気に変更されます。

> **補足:** > `CAT_RANK` だけが `w` を使っていないのは、`RANK()` には `ORDER BY` が必須だからです。`WINDOW` 句には `PARTITION BY` だけを定義し、個別の関数で `ORDER BY` を追加して組み合わせることも可能です（※23aiの柔軟なWINDOW定義）。

### 3. 注意点
#### ① 実行順序の罠（なぜWITH句を使うのか）
初心者の方はこう思うかもしれません。「最初から `WHERE prod_id = 23` を書けばいいじゃない」と。
* **注意点**: もし先に `WHERE` で1行に絞ってしまうと、`AVG()` や `RANK()` は「その1行だけ」を対象に計算してしまい、平均価格が自分の価格と同じになり、順位も必ず1位になってしまいます。
* **対策**: このクエリのように「全体で計算してから、最後に絞り込む」という2段構えが必要です。

#### ② `OVER()` と `OVER(PARTITION BY ...)` の使い分け
* `OVER()`（空っぽ）: **クラス全員**の平均。
* `OVER(PARTITION BY ...)`: **同じ部活内**での平均。
この「スコープ（範囲）」の違いを意識することが、分析関数マスターへの第一歩です。


----
<br><br>


# 【完全版】問題12-4（ランニング集計）  *Lv4*
### **`PRODUCT_ID`=1（Boy's Shirt (White)）および6（Boy's Socks (Grey)）の商品について、それぞれ30個の注文が来た。下記のルールに従い、商品をピックアップしてください。**
* **COスキーマ`INVENTORY`テーブルを参照する。**
* **`STORE_ID`の昇順でピックアップする。**
* **店舗の在庫数（`PRODUCT_INVENTORY`）より注文数に達するまでピックアップする。注文数に満たない場合は全てピックアップし、不足分は次の店舗よりピックアップする。**

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
今回は実務的な **「在庫の引き当て（ピッキング）ロジック」** です。
このクエリは、単なる集計ではなく「特定の目標数（今回は30個）に達するまで、各店舗からいくつずつ在庫を持ってくるか」という、**物流や在庫管理システムでそのまま使える高度なロジック**をSQLで表現しています。

### 1. 何をやっているのか？
このクエリの目的は、**「商品ID 1と6について、合計30個確保するために、どの店舗から何個出すべきか」** という指示書を作ることです。

#### CTE (`sum_product_inventory`) の役割
ここで一番重要な「ウィンドウ関数（Window Function）」のテクニックが使われています。

* **`ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING`**:
    * これがこのクエリ最大のポイントです。
    * 「現在の行を含まず、**前の行までの累積合計**」を計算しています。
    * これにより、「今からこの店舗で在庫を取る前に、すでに他の店舗で何個確保できているか（`acc_inv`）」がわかります。
* **`NVL(..., 0)`**:
    * 最初の店舗では、前の店舗がないため累計はNULLになります。それを0として扱っています。

#### メインSELECTの役割
ここで実際に「その店舗から何個取るか」を計算しています。

* **`least(product_inventory, 30 - spi.acc_inv)`**:
    * **「その店舗にある在庫すべて」** か **「目標の30個に足りない残りの数」** の、**小さい方**を選択します。
    * これにより、目標の30個を超えて取りすぎることを防いでいます。
* **`WHERE spi.acc_inv < 30`**:
    * すでに前の店舗までで30個確保できてしまっている場合は、その後の店舗をリストに出さないようにしています。

### 2. 押さえるべきポイント
#### 「現在の行を含まない累積」の意味
通常のランニング集計（Running Total）は自分自身を含みますが、このクエリではあえて **「自分より前までの合計」** を出しています。
* **なぜ？**: 「自分が今から何個引き当てるべきか」を決めるためには、「自分の前の店舗までに何個集まったか」を知る必要があるからです。

#### `LEAST` 関数の賢い使い方
`CASE` 文を使っても書けますが、`LEAST` 関数を使うことで「上限を超えないように値を抑える」ロジックを非常にシンプルに記述できています。


----
<br><br>

# 【完全版】問題12-5（ランク関数と別解の比較） *Lv3*
### HRスキーマ`EMPLOYEES`テーブルより、部署（`DEPARTMENT`）ごとに、最も賃金（`SALARY`）の高い人物を抽出してください。（順不同）

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
今回は **「各部署のトップ（最高給与者）」** を抽出する2つの異なるアプローチです。
**「一気に計算して後から絞り込む」** 例1と、**「部署ごとに一人ずつ探しに行く」** 例2の違いを以下に解説します。

### 例1：RANK関数を使用（王道の全体集計型）
実務で最も一般的、かつ「分析関数」の基本を学べる書き方です。

#### 解説
1.  **WITH句（CTE）で下準備**: 全社員に対して、所属部署内（`PARTITION BY department_id`）での給与順位を計算し、`re` という名前の仮想テーブルに保存します。
2.  **JOINとフィルタリング**: その後、部署名（`department_name`）を結合し、最後に `WHERE rank = 1` で各部署のトップだけを残します。

#### ポイント：`RANK` と `WITH TIES` の関係
このクエリでは `RANK()` を使っているため、**もし最高給与額の人が同じ部署に2人いたら、2人とも `rank = 1` になり、2人とも表示されます。** これは「例2」の `WITH TIES` と同じ挙動になります。

### 例2：LATERAL結合（相関副問い合わせ）

少し上級者向けに見えますが、考え方は非常に直感的（プログラミング的）です。

#### 解説
* **LATERALとは？**: 右側のサブクエリが、左側のテーブル（`dp`）の値を参照できるようにする魔法のキーワードです。
* **「部署ごとに実行」**: イメージとしては、**部署一覧を上から1つずつ見て、「この部署の給与1位は誰かな？」とサブクエリを実行して回る**感じです。
* **`FETCH FIRST ROW WITH TIES`**: 給与順に並べて最初の1行を取りますが、`WITH TIES` があることで、同率1位がいる場合に全員分を連れてきてくれます。

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

# 問題12-6（シェアの計算） *Lv3*
### SHスキーマ`SUPPLEMENTARY_DEMOGRAPHICS`テーブルを利用し、「居住年数」と「世帯人数」を組み合わせて、独自の顧客セグメント（例：定住層、新世代層など）を作成して下さい。具体的には以下の条件でデータを取得してください。
* **以下の場合分けで顧客セグメント（CUSTOMER_SEGMENT）を定義してください。**
  * **居住年数（YRS_RESIDENCE）が10より大きく、世帯人数（ESTABLISHED LARGE FAMILY）が 「'4-8'または'9+'」の場合、 「Established Large Family」とする。**
  * **上記以外で、居住年数（YRS_RESIDENCE）が10より大きい場合、「Long-term Resident」とする。**
  * **居住年数（YRS_RESIDENCE）が2以下場合、「Newcomer」とする。**
  * **上記以外、「Other」とする。**

* **顧客セグメント別のレコード数（TOTAL_COUNT）と割合（PERCENTAGE）を算出する**


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
今回はマーケティング分析で頻出する **「顧客セグメンテーション（層別化）」** です。

### 何をやっているのか？
このクエリの目的は、顧客の「居住年数（`yrs_residence`）」と「世帯人数（`household_size`）」から**独自の顧客グループを作り、それぞれのグループが全体に占める割合（％）を算出する**ことです。

### ① セグメント分け（CASE式）
複雑な条件を順番に評価し、顧客を4つのラベルに分類しています。
* **ESTABLISHED LARGE FAMILY**: 居住10年超 ＋ 世帯4人以上の「定着大家族」。
* **LONG-TERM RESIDENT**: 居住10年超（かつ上記以外）の「長期居住者」。
* **NEWCOMER**: 居住2年以下の「新人」。
* **OTHER**: それ以外。

### ② エイリアスでのグループ化
* **`GROUP BY customer_segment`**:
    これまでのSQL（21c以前）では、`GROUP BY` 句にエイリアス（別名）を使うことはできず、上にある長い `CASE WHEN...` をもう一度丸ごとコピーして書く必要がありました。
    23aiからは、**SELECT句で定義した名前をそのまま使える**ようになり、コードが圧倒的にスッキリしました。

### ③ 構成比の算出（RATIO_TO_REPORT）
* **`RATIO_TO_REPORT(COUNT(*)) OVER()`**:
    「現在の行の数 ÷ 全体の合計数」を自動で計算してくれる非常に便利な分析関数です。これを使わないと「全合計を出すサブクエリ」を書く必要がありましたが、1行で解決しています。

## 参考リンク
https://docs.oracle.com/cd/E16338_01/server.112/b56299/functions142.htm
https://atmarkit.itmedia.co.jp/ait/articles/0510/29/news012_2.html


----
<br><br>



# 問題12-7（項目ごとのシェアの計算） *Lv3*
### SHスキーマ`SUPPLEMENTARY_DEMOGRAPHICS`テーブルを利用し、学歴（`EDUCATION`）ごとのグループの中で、各職業（`OCCUPATION`）が占める割合を算出してください。具体的には以下の条件でデータを取得してください。
* **取得対象は、`EDUCATION` が 'PhD' または '1st-4th' である。**
* **`EDUCATION` の昇順、`CUST_COUNT`の降順に表示。**


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
    cust_count DESC;
```

## 解説
### 1. クエリの解説：`PARTITION BY` の魔法

このクエリのポイントは `OVER(PARTITION BY d.EDUCATION)` にあります。

#### 処理のステップ
1.  **データの絞り込み (`WHERE`)**: まず、'PhD' と '1st-4th' のデータだけを抽出します。
2.  **集計 (`GROUP BY`)**: 「学歴 × 職業」の組み合わせごとに、人数（`COUNT(*)`）を数えます。
3.  **グループ内での割合計算 (`RATIO_TO_REPORT`)**:
    * `OVER()` だけだと、PhDと1st-4thを**混ぜた合計**に対する割合になります。
    * `PARTITION BY d.EDUCATION` と書くことで、**「PhDの中での合計」**と**「1st-4thの中での合計」** を別々に計算し、それぞれの枠内でのシェアを算出します。

### 2. ポイント

#### データの対比（PhD vs 1st-4th）
この条件設定により、「PhD層にはどんな職業が多いのか？（教授や研究職？）」と「1st-4th層にはどんな職業が多いのか？」を直接比較できます。
* **PhDグループ**: 合計 100%
* **1st-4thグループ**: 合計 100%
このように分母を揃えることで、母集団の数が違っても（例：PhDが100人、1st-4thが1000人でも）、**構成比の傾向**を正しく比較できるようになります。

#### 並び替え (`ORDER BY`)
`EDUCATION ASC, CUST_COUNT DESC` としていることで、学歴ごとに固まった状態で、どの職業が一番多いのかが上から順に並びます。レポートとして非常に見やすい形式です。

### 3. 注意点とアドバイス

#### ① 実行順序の理解
`RATIO_TO_REPORT` などの分析関数は、**`WHERE` 句でデータが絞り込まれた後に計算されます。**
もし `WHERE` 句を外すと、分母（合計）がテーブル全件になり、パーセンテージが全く異なる値になります。「今、何に対する割合を出しているのか」を常に意識することが大切です。

#### ② NULL の存在
`OCCUPATION`（職業）が `NULL`（空欄）の顧客がいる場合、その `NULL` も一つの行として集計されます。
* もし「職業が判明している人だけ」で 100% を作りたいなら、`WHERE OCCUPATION IS NOT NULL` を追加する必要があります。


----
<br><br>

# 問題12-8（前後比較） *Lv3*
- 3. 前後比較（直前に雇われた人との比較）
「同じ部門内で、自分の1つ前に採用された人は誰か？」といった、時系列の比較を行うケースです。

## 期待する結果
絞る

## 解答例
```sql
-- 同じ部署で、自分の直前に採用された人（HIRE_DATEが最大かつ自分より前）を探す
SELECT 
    E1.FIRST_NAME AS 本人, 
    E1.HIRE_DATE AS 採用日,
    E2.FIRST_NAME AS 前回の採用者, 
    E2.HIRE_DATE AS 前回採用日
FROM 
    HR.EMPLOYEES E1
LEFT JOIN 
    HR.EMPLOYEES E2 ON E1.DEPARTMENT_ID = E2.DEPARTMENT_ID -- 同じ部署
                   AND E1.HIRE_DATE > E2.HIRE_DATE       -- 自分より前
WHERE 
    -- 自分より前の中で、最も採用日が新しい（＝直前）人だけを抽出
    E2.HIRE_DATE = (
        SELECT MAX(HIRE_DATE) 
        FROM HR.EMPLOYEES E3 
        WHERE E3.DEPARTMENT_ID = E1.DEPARTMENT_ID 
          AND E3.HIRE_DATE < E1.HIRE_DATE
    ) OR E2.HIRE_DATE IS NULL -- 最初の採用者も表示
ORDER BY 
    E1.DEPARTMENT_ID, E1.HIRE_DATE;
```

```sql
SELECT 
    FIRST_NAME AS 本人,
    HIRE_DATE AS 採用日,
    -- 同じ部署内で、採用日順に並べた1つ前の行の値を取得
    LAG(FIRST_NAME) OVER (PARTITION BY DEPARTMENT_ID ORDER BY HIRE_DATE) AS 前回の採用者,
    LAG(HIRE_DATE)  OVER (PARTITION BY DEPARTMENT_ID ORDER BY HIRE_DATE) AS 前回採用日
FROM 
    HR.EMPLOYEES
ORDER BY 
    DEPARTMENT_ID, HIRE_DATE;
```

## 解説
このクエリは、**「相関サブクエリ」と「自己結合」を組み合わせて、時系列の連続性を表現する**という、非常に高度で論理的なパズルを解くような構成になっています。

初心者から中級者へのステップアップとして、「なぜこの複雑な手順が必要なのか」を解説するのに最適なサンプルです。

---

## 1. クエリの解説：3つの役割分担

このクエリでは、同じ `EMPLOYEES` テーブルに3つの異なる役割を与えています。

| エイリアス | 役割 | 説明 |
| :--- | :--- | :--- |
| **E1** | **本人** | リストのメインとなる「今の行」の人。 |
| **E2** | **候補者** | 同じ部署で自分より前に採用された「候補」全員。 |
| **E3** | **判定者** | サブクエリ内で「自分より前の中で一番新しいのはいつか？」を特定する。 |

### ロジックの流れ
1.  **結合 (`LEFT JOIN`)**: まず、本人（E1）に対して、同じ部署で先に採用された人（E2）を全員紐づけます。この時点では、1人に対して複数の「前任候補」が紐づき、行が膨らんでいます。
2.  **絞り込み (`WHERE`)**: ここが核心です。サブクエリ（E3）を使って、「自分より前の採用日のうち、最大の（＝一番自分に近い）日付」を特定し、それに合致するE2だけを残します。
3.  **NULLの許容**: `OR E2.HIRE_DATE IS NULL` を入れることで、部署で一番最初に採用された人（＝前回の採用者がいない人）も結果から消さずに表示しています。

---

## 2. 注意点：ここが実務での落とし穴！

このクエリは論理的には正しいですが、実務や大規模データで運用する際には以下の点に注意が必要です。

### ① パフォーマンスの問題（「$N^2$」の罠）
この書き方は **「相関サブクエリ」** と呼ばれ、メインの行（E1）が1行増えるごとに、サブクエリ（E3）がその都度実行されます。
* データが100件なら問題ありませんが、10万件になると処理が指数関数的に重くなり、23aiのパワーを持ってしても「重いクエリ」になってしまいます。

### ② 同じ採用日の人がいる場合の挙動（タイの問題）
もし同じ日に2人が採用されていた場合、`E3.HIRE_DATE < E1.HIRE_DATE` という不等号の影響で、**「同日のもう一人」が「前回採用者」として認識されません。**
* 秒まで記録されていれば問題ありませんが、日付のみの管理だと「1つ前」の定義が曖昧になります。

### ③ 結合の冗長性
実は、`WHERE` 句のサブクエリで日付を特定しているため、`LEFT JOIN` の `ON` 句にある `E1.HIRE_DATE > E2.HIRE_DATE` という条件は、ロジックとしては重複しています（サブクエリがその役割を兼ねているため）。

---

## 3. 【現代の正解】23ai/上級者ならこう書く（LAG関数）

Oracle 8i（！）以降、この種の「前後の行との比較」には **分析関数（ウィンドウ関数）の `LAG`** を使うのが一般的です。こちらの方が圧倒的に高速で、読みやすくなります。

```sql
SELECT 
    FIRST_NAME AS 本人,
    HIRE_DATE AS 採用日,
    -- 同じ部署内で、採用日順に並べた1つ前の行の値を取得
    LAG(FIRST_NAME) OVER (PARTITION BY DEPARTMENT_ID ORDER BY HIRE_DATE) AS 前回の採用者,
    LAG(HIRE_DATE)  OVER (PARTITION BY DEPARTMENT_ID ORDER BY HIRE_DATE) AS 前回採用日
FROM 
    HR.EMPLOYEES
ORDER BY 
    DEPARTMENT_ID, HIRE_DATE;
```

### なぜ `LAG` が推奨されるのか？
* **高速**: テーブルを1回スキャンするだけで済みます（自己結合やサブクエリのように何度も読み直さない）。
* **簡潔**: 「誰が判定者か」を考える必要がなく、「1つ前（LAG）」と書くだけで意図が伝わります。
* **同日対策**: 同じ採用日の人がいても、内部的な並び順に従って確実に「前の行」を拾えます。

----
<br><br>


# 【完全版】問題12-9（問題10-5の別解） *Lv3*
### OEスキーマ`ORDERS`テーブルより同一年月（`ORDER_DATE`の年月）に複数の注文をした人と注文を取得してください。（順不同）

- **`CUST_NAME`は`CUSTOMERS`テーブルより取得してください**
- **分析関数を使用して取得して下さい**


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
### 例1：分析関数（COUNT）を使用
「その月の注文回数が1より大きいか？」という直感的なアプローチです。

#### 解説
* **`COUNT(*) OVER(...)`**: これが分析関数のキモです。通常の `GROUP BY` と違い、**「行をまとめずに集計結果だけを各行に付与」** します。
* **`PARTITION BY o.customer_id, TRUNC(o.order_date, 'mm')`**: 「顧客ごと」かつ「月ごと」にデータを区切っています。`TRUNC` 関数で日付を「月」単位に切り捨てるのが定番のテクニックです。
* **インライン・ビューの使用**: SQLのルール上、`WHERE` 句で分析関数の結果（`monthly_order_count`）を直接判定できないため、一度内側で計算してから外側で絞り込んでいます。


### 例2：分析関数（MIN, MAX）を使用
「最小の注文IDと最大の注文IDが違う ＝ 2つ以上IDが存在する」というトリッキーで賢いアプローチです。

#### 解説
* **`WINDOW w AS (...)`**: **Oracle 21c/23aiで導入された新機能**（SQL標準）です！同じ範囲定義（PARTITION BY...）を何度も書かずに、`w` という名前で使い回せるため、SQLが劇的に読みやすくなります。
* **`monthly_min <> monthly_max`**: IDの最小と最大が異なるということは、そのグループには少なくとも2つの異なる注文があることを意味します。
