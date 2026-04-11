---
title: "分析関数（全8問）"
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
### 1. 推奨される手法：ウィンドウ関数
SQLでデータをランキング付けする際、非常に便利なのがランク関数です。主に使われるのは **`ROW_NUMBER`**、**`RANK`**、**`DENSE_RANK`** の3つですが、これらは「同順位（同じ値）をどう扱うか」という点に違いがあります。

#### ① 主要な3つのランク関数

| 関数名 | 同順位の扱い | 次の順位の飛び | 特徴 |
| --- | --- | --- | --- |
| **`ROW_NUMBER()`** | 一意の連番を振る | 飛ばない | 同点でも必ず 1, 2, 3... と連番になる |
| **`RANK()`** | 同順位は同じ値 | 飛ぶ | 1位が2人いたら、次は3位になる（隙間ができる） |
| **`DENSE_RANK()`** | 同順位は同じ値 | **飛ばない** | 1位が2人いても、次は2位になる（隙間がない） |

#### ② 基本的な構文
ランク関数は常に `OVER` 句と一緒に使用します。

```sql
RANK() OVER (ORDER BY 列名 [ASC|DESC])
```

* **`ORDER BY`**: 何を基準に順位を決めるか指定します（`DESC`で降順、`ASC`で昇順）。


  
### 2. 自己結合による順位取得の考え方
SQLで順位を求める手法の一つに、自己結合を使った論理があります。その基本的な考え方は以下の通りです。

**「自分よりも高い報酬を得ている人が何人いるかを数え、その人数に1を足したものが自分の順位である」**

* 自分より高い報酬の人が **0人** の場合： $0 + 1 = 1$位
* 自分より高い報酬の人が **2人** の場合： $2 + 1 = 3$位

#### パフォーマンスに関する注意点
自己結合による手法は論理的には非常に明快ですが、実務においては以下の理由から**非推奨**とされています。

* **1行ごとの集計処理**: データが1行増えるごとに、その1行に対して「自分より大きいデータ」を探してカウントする処理が発生します。
* **計算量の増大**: データ量が2倍になると計算量は4倍（ $O(n^2)$ ）に近い形で増えていくため、大量のデータを扱う際に処理時間が大幅に遅くなる原因となります。
* **データベースへの高負荷**: プログラミングで言うところの「二重ループ」に近い動きをデータベース内部で行うため、システム全体のパフォーマンスを低下させる恐れがあります。

<br><br>

----

# 問題12-2（項目ごとの順位）  *Lv2*
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
### 1. 職種（JOB_ID）ごとのランキング手法
特定のグループごとに順位を付けたい場合、ウィンドウ関数の `PARTITION BY` を使用します。
今回のケースでは、**「JOB_ID」ごとにグループ化**し、その中で報酬などの条件に基づいたランキングを算出しています。

#### 基本的な構文
```sql
RANK() OVER (PARTITION BY 列名1 ORDER BY 列名2 [ASC|DESC])
```

* **`ORDER BY`**: 何を基準に順位を決めるか指定します（`DESC`で降順、`ASC`で昇順）。
* **`PARTITION BY`**（オプション）: グループごとに順位をリセットしたい場合に使います（例：クラスごとに順位を振る）。


### 2. 「1位のみ」を抽出する際の注意点
ランキング1位のデータだけを表示したい場合、条件として `WHERE 順位 = 1` を指定する必要がありますが、SQLが命令を処理する **「評価順序（実行順序）」** の関係から、 **「ランキングを計算したSELECT文を、さらに外側から別のSELECT文で囲む（副問合せにする）」** という手順が必要になります。
ランキングを算出した同じ階層の `WHERE` 句に、その順位を直接指定して絞り込むことはできません。


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
SUMやAVGは集計関数としてだけでなく、分析関数としても使用できます。通常の`GROUP BY`では一行にまとめられてしまいますが、分析関数として使用すると、データを集計しつつ **「行を減らさない」** ということが可能になります。


### 1. 分析関数版 SUM / AVG の基本構造
通常の集計関数と異なり、`OVER`句を後ろに付けます。

```sql
SUM(列名1) OVER([PARTITION BY] 列名2 )
AVG(列名1) OVER([PARTITION BY] 列名2 )
```


### 2. 累計と移動平均
ここでは取り上げませんが、分析関数が真価を発揮するのは、`ORDER BY`を組み合わせた時です。**累計や移動平均**など算出可能になります。


### 3. 初心者が絶対注意すべき「3つの落とし穴」

#### ① `ORDER BY` を書くと「範囲」が変わる
SUMやAVGに対し、安易に`ORDER BY` を入れてはいけません。
* `SUM(val) OVER()`：**全行**の合計。
* `SUM(val) OVER(ORDER BY id)`：**最初から現在の行まで**の累計。

※`ORDER BY`の使い方は、以降の問題で解説します。

#### ② パフォーマンスとメモリ
分析関数は非常に便利ですが、内部で **「ソート（並び替え）」** が発生します。大量のデータに対して複雑な `PARTITION BY` や `ORDER BY` を多用すると、一時表領域（TEMP）を食いつぶしたり、実行速度が低下したりすることがあります。
* インデックスが効くかどうかを確認する
* 必要な列だけに絞ってから計算する

といった配慮が上級者への一歩です。


### 4. Oracle 23ai の新機能：WINDOW句
例2のように、23ai（および標準SQL）では、同じ `OVER` 句の内容を何度も書くのを回避したい場合、最後に定義をまとめることができます。



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




## 参考リンク



----

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
初心者はよく `GROUP BY department_id` と `MAX(salary)` を使おうとしますが、それだと「その給与を誰がもらっているか（名前）」を出すのが非常に面倒になります。今回の2つの例は、その問題をスマートに解決する「現代的なSQL」の代表格です。


### 例1：RANK関数（ウィンドウ関数）を使用


#### 解説：全体を仕切って順位をつける
このクエリは、**「一度クラス全員に順位表を配ってから、1位の人だけを呼び出す」** というイメージです。

1.  **`PARTITION BY`**: 部署ごとに「仕切り」を作ります。
2.  **`ORDER BY salary DESC`**: その仕切りの中で給与が高い順に並べます。
3.  **`RANK()`**: 同率1位がいれば、どちらも「1位」とします。
4.  **`WITH`句 (CTE)**: 順位がついた一時的な表（re）を作り、外側のメインクエリで `rank = 1` の人を抽出します。

#### ⚠️ 注意点：同率順位の扱い
* **RANK()**: 給与が同額の人が2人いたら、両方とも1位になります（1位, 1位, 3位...）。
* **DENSE_RANK()**: 同率がいても次を飛ばしません（1位, 1位, 2位...）。
* **ROW_NUMBER()**: 同率でも必ずユニークな番号を振ります（1位, 2位...）。
> **アドバイス:** 「最高給与者が2人いたら2人とも出す」という要件なら `RANK()` で正解です。


### 例2：LATERAL結合を使用して相関副問い合わせ

#### 解説：部署ごとに「最高の人」を探しに行く
こちらは、**「部署リストを1枚ずつめくりながら、その都度、その部署の最高給与者を1名（または同率全員）スカウトしてくる」** という動きをします。

1.  **`LATERAL`**: 左側にある `hr.departments` の値を、右側のカッコ内（副問い合わせ）で使えるようにする魔法のキーワードです。
2.  **`ORDER BY ... FETCH FIRST ROW`**: 23aiでも推奨される標準的なTop-N構文です。
3.  **`WITH TIES`**: ここがポイントです！これをつけることで、例1の `RANK()` と同様に **「同率1位」をすべて含める**ことができます。

#### ⚠️ 注意点：パフォーマンスと結合
* **インデックスの重要性**: `employees` テーブルの `department_id` と `salary` にインデックスがない場合、部署の数だけフルスキャンに近い動きになり、遅くなる可能性があります。
* **`CROSS JOIN` の罠**: 従業員が一人もいない部署がある場合、`CROSS JOIN` だとその部署自体が結果から消えてしまいます。もし「従業員0人の部署」も表示したいなら、`LEFT JOIN LATERAL (...) ON 1=1` と書く必要があります。

### 比較表

| 比較項目 | 例1：RANK関数 | 例2：LATERAL結合 |
| :--- | :--- | :--- |
| **対象レベル** | 初級〜中級（必須知識） | 中級〜上級（モダンな書き方） |
| **考え方** | 全データを処理してフィルター | 1行ずつ必要な分だけ取ってくる |
| **23aiの利点** | 従来通りの安定性 | `FETCH` 句との相性が抜群 |
| **可読性** | 直感的で分かりやすい | 少し複雑だが、結合ロジックが明確 |

<br>


----
<br><br>

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
##### 💡 ここが「やさしい」ポイント
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
`RATIO_TO_REPORT` は、一言で言うと **「全体（またはグループ内）におけるその行の割合（シェア）」** を一発で計算してくれる、Oracle独自の非常に便利な分析関数です。「パーセント計算」を劇的にラクにしてくれます。


### 1. 直感的なイメージ


例えば、1枚のピザを4人で分けるとします。
* Aさん：3切れ
* Bさん：2切れ
* Cさん：4切れ
* Dさん：1切れ
* **合計：10切れ**

このとき、Bさんの割合は $2 / 10 = 0.2$ です。
`RATIO_TO_REPORT` は、この「自分の値 ÷ 合計」という計算を、裏側で自動的にやってくれる関数です。

### 2. 基本的な書き方

```sql
RATIO_TO_REPORT( 値 ) OVER()
```

これだけで、「全行の合計」に対する「その行の値」の割合が **0〜1の間**で返ってきます。

#### よく使う「お作法」
実務では 0.25 ではなく 25% と表示したいことが多いので、以下のように組み合わせて使います。

```sql
-- 100を掛けてパーセントにし、小数点第2位で丸める
ROUND(RATIO_TO_REPORT(COUNT(*)) OVER() * 100, 2) || '%'
```


### 3. なぜこの関数が優れているのか？
「各部署の給与が全体に占める割合」を出したい場合を考えてみましょう。

#### 昔ながらの書き方（大変...）
```sql
SELECT 
    department_id, 
    sum_sal / total_sal AS ratio
FROM (
    -- 部署ごとの合計を出す
    SELECT department_id, SUM(salary) as sum_sal FROM hr.employees GROUP BY department_id
) a,
(
    -- 全体の合計を出すためだけに別のクエリを走らせる
    SELECT SUM(salary) as total_sal FROM hr.employees
) b;
```
このように、**「個別の集計」と「全体の合計」を別々に計算してくっつける（結合する）** 必要がありました。

#### RATIO_TO_REPORT を使った書き方（スマート！）
```sql
SELECT 
    department_id,
    SUM(salary) AS dept_sal,
    RATIO_TO_REPORT(SUM(salary)) OVER() AS ratio
FROM hr.employees
GROUP BY department_id;
```
`OVER()` と書くだけで、SQLが勝手に全行の `SUM(salary)` を計算して分母にしてくれます。



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

### 2. ここがポイント！「分析の質」を高める工夫

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
