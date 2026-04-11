---
title: "集合演算子（全3問）"
free: false
---

# 前提知識
集計関数が「縦（行）を凝縮する」ものだったのに対し、集合演算子は **「複数のクエリ結果（表）を、数学の集合のようにガッチャンコする」** 操作です。

## 1. 集合演算子とは？
集合演算子は、2つ以上の`SELECT`文の結果を1つの結果セットに統合します。
よく比較される「結合（JOIN）」との違いを理解するのが上達の近道です。

* **結合（JOIN）：** 横に広げる（社員名に部署名を「付け足す」）
* **集合演算子：** 縦に積み上げる（営業部の名簿と開発部の名簿を「合体させる」）


![](https://storage.googleapis.com/zenn-user-upload/d8ef42682a6f-20260322.png)

## 2. 4つの主要な演算子
Oracle 23aiで利用できる主な演算子は以下の4つです。

| 演算子 | 意味 | 重複の扱い | 特徴 |
| :--- | :--- | :--- | :--- |
| **`UNION`** | 和集合 | **排除する** | 重複を消すため、内部で並べ替え（ソート）が発生し、少し重い。 |
| **`UNION ALL`** | 和集合 | **保持する** | そのまま繋げるだけなので**最も高速**。実務で一番使う。 |
| **`INTERSECT`** | 積集合 | - | 両方の結果に含まれる共通の行だけを抽出する。 |
| **`MINUS`** | 差集合 | - | 1つ目の結果から、2つ目の結果に含まれるものを差し引く。 |

> **💡 Oracle 23ai の進化ポイント:**
> 長らくOracleでは `MINUS` という独自キーワードを使ってきましたが、23aiからは標準SQLである **`EXCEPT`** も使えるようになりました。他DB（PostgreSQLやSQL Server）からの移行がよりスムーズになっています。


## 3. 集合演算子を使うための「3つの絶対条件」
これに違反するとSQLエラー（`ORA-01789`など）になります。

1.  **列の数が一致していること:** 1つ目のSELECTが3列なら、2つ目も3列でなければなりません。
2.  **データ型が対応していること:** 1番目の列が「数値型」なら、もう一方の1番目の列も「数値型」である必要があります。
3.  **列名は1つ目が優先される:** 最終的な結果セットのヘッダー（列名）は、最初のSELECT文で定義した名前になります。




# 問題11-1（積集合） *Lv2*
### 2019年も2020年も「継続して」購入してくれた「ロイヤル顧客」を特定してください。下記条件に従い取得してください。（順不同）
* **購入情報はSHスキーマ`SALES`テーブルを参照してください。**
* **購入日は`TIME_ID`を参照してください。**
* **SHスキーマ`CUSTOMERS`テーブルの顧客都市（`CUST_CITY`）が「Yokohama」、顧客クレジットカード限度額（`CUST_CREDIT_LIMIT`）が15000以上を対象としてください。**


## 期待する結果
| CUST_ID | NAME            | CUST_INCOME_LEVEL    | CUST_CREDIT_LIMIT | 
| ------- | --------------- | -------------------- | ----------------- | 
| 6555    | Rosanna Rill    | J: 190,000 - 249,999 | 15000             | 
| 19010   | Roxanne Crocker | I: 170,000 - 189,999 | 15000             | 
| 24556   | Zylia Hanson    | J: 190,000 - 249,999 | 15000             | 

## 解答例
```sql:例1：INTERSECT（積集合）を使用
WITH both_purchase AS (
    SELECT
        cust_id
    FROM
        sh.sales
    WHERE
        time_id BETWEEN date'2019-01-01' 
                    AND date'2019-12-31'
    INTERSECT
    SELECT
        cust_id
    FROM
        sh.sales
    WHERE
        time_id BETWEEN date'2020-01-01' 
                    AND date'2020-12-31'
)
SELECT
    c.cust_id,
    c.cust_first_name || ' ' || c.cust_last_name AS "NAME",
    c.cust_income_level,
    c.cust_credit_limit
FROM
    both_purchase bp
    INNER JOIN sh.customers c 
       ON bp.cust_id = c.cust_id
WHERE
        cust_city = 'Yokohama'
    AND cust_credit_limit >= 15000
```
```sql:例2：EXISTSを使用して算出
WITH both_purchase AS (
    SELECT DISTINCT
        cust_id
    FROM
        sh.sales s19
    WHERE
        time_id BETWEEN DATE'2019-01-01' 
                    AND DATE'2019-12-31'
        AND EXISTS ( 
            SELECT
                'X'
            FROM
                sh.sales s20
            WHERE
                time_id BETWEEN DATE'2020-01-01' 
                            AND DATE'2020-12-31'
                AND s19.cust_id = s20.cust_id
        )
)
SELECT
    c.cust_id,
    c.cust_first_name || ' ' || c.cust_last_name AS "NAME",
    c.cust_income_level,
    c.cust_credit_limit
FROM
    both_purchase bp
    INNER JOIN sh.customers c 
       ON bp.cust_id = c.cust_id
WHERE
        cust_city = 'Yokohama'
    AND cust_credit_limit >= 15000
```
```sql:例3：結合して算出
WITH purchase2019 AS (
-- 2019年の顧客
    SELECT DISTINCT
        cust_id
    FROM
        sh.sales
    WHERE
        time_id BETWEEN date'2019-01-01' 
                    AND date'2019-12-31'
), purchase2020 AS (
-- 2020年の顧客
    SELECT DISTINCT
        cust_id
    FROM
        sh.sales
    WHERE
        time_id BETWEEN date'2020-01-01' 
                    AND date'2020-12-31'
)
SELECT
    c.cust_id,
    c.cust_first_name || ' ' || c.cust_last_name AS "NAME",
    c.cust_income_level,
    c.cust_credit_limit
FROM
         purchase2019 p19
    INNER JOIN purchase2020 p20 
       ON p19.cust_id = p20.cust_id
    INNER JOIN sh.customers c 
       ON p19.cust_id = c.cust_id
WHERE
        cust_city = 'Yokohama'
    AND cust_credit_limit >= 15000
```
## 解説
`INTERSECT`（積集合）を一言で言うと、**「2つの集合のどちらにも存在する共通のデータ（重なり部分）」** を抽出する演算子です。
数学の集合論で言うところの $A \cap B$ を実現するもので、ベン図で真ん中の重なった部分を取り出すイメージです。

### 1. 概念図と基本動作
例えば、「昨年来店した顧客」と「今年来店した顧客」の両方に名前がある人、つまり **「リピート顧客」** を探すときに非常に直感的です。

#### 基本の書き方
```sql
SELECT cust_id FROM sales_2025
INTERSECT
SELECT cust_id FROM sales_2026;
```

### 2. INTERSECT vs INNER JOIN vs EXISTS
SQLにおいて「共通するデータを抽出する」という目的は同じでも、`INTERSECT`、`INNER JOIN`、`EXISTS` はそのアプローチと結果の性質が大きく異なります。

それぞれの違いを一言で表すと以下のようになります。

* **`INTERSECT`**: 2つの集合の「重なり」を抽出する（**数学的・集合論的**）
* **`INNER JOIN`**: 2つのテーブルを「くっつける」（**リレーショナル・結合的**）
* **`EXISTS`**: 条件に合う行が「あるかないか」でフィルターする（**論理的・条件的**）



#### 比較表

| 比較項目 | `INTERSECT` | `INNER JOIN` | `EXISTS` |
| --- | --- | --- | --- |
| **主な目的** | 全く同じ構造の行の共通部 | 複数の情報を横に並べる | メインの行を絞り込む |
| **重複の扱い** | **自動で排除** (唯一無二) | **増える可能性あり** (多対多など) | **変わらない** (メインのまま) |
| **NULLの比較** | **NULL = NULL** とみなす | 通常の一致では無視される | `JOIN`条件次第 |
| **出力カラム** | 全クエリで共通の列のみ | 両方のテーブルの列を合体 | **メインクエリの列のみ** |
| **動作タイプ** | 集合演算 (Set Op) | 物理結合 (Join) | 半結合 (Semi-Join) |

#### 具体的な違いと実行例

「過去に購入した顧客」と「キャンペーン対象顧客」を比較するシーンで見てみましょう。

##### ① `INTERSECT` (集合の重なり)

数学のベン図で真ん中だけを取り出すイメージです。

```sql
SELECT cust_id FROM sales
INTERSECT
SELECT cust_id FROM campaign;
```

* **特徴**: 結果は必ずユニーク（重複なし）になります。
* **注意**: `cust_id` 以外の「名前」なども出したい場合、両方のSELECTに含める必要があり、少しでもデータが違う（例えば片方の住所が古いなど）と「別物」とみなされて抽出されません。

##### ② `INNER JOIN` (情報の結合)
2枚のカードを重ねて1枚にするイメージです。

```sql
SELECT s.cust_id, s.amount, c.campaign_name
FROM sales s
INNER JOIN campaign c ON s.cust_id = c.cust_id;
```

* **特徴**: 別のテーブルにある情報（キャンペーン名など）を横に並べて取得できます。
* **注意**: もし1人の顧客に2つのキャンペーンが紐付いていた場合、結果の行数が**2行に増えます（ファンアウト現象）**。

##### ③ `EXISTS` (存在チェックによる抽出)
「あっちのリストに名前がある人だけ通して！」という門番のようなイメージです。

```sql
SELECT * FROM sales s
WHERE EXISTS (
    SELECT 1 FROM campaign c 
    WHERE c.cust_id = s.cust_id
);
```

* **特徴**: メインの `sales` テーブルの行が**増えたり減ったり（重複排除）することなく**、純粋にフィルターとして機能します。
* **注意**: サブクエリ側の列（キャンペーン名など）を `SELECT` 句で使うことはできません。

#### 使い分けのガイドライン

#### `INTERSECT` を使うべきとき

* 「2つの期間で共通してアクションした人」をサクッと出したい。
* データのクレンジングが済んでおり、行全体が一致するかを見たい。

#### `INNER JOIN` を使うべきとき

* 「誰が、どのキャンペーンで、何を買ったか」のように、**異なるテーブルの情報をガッチャンコして見せたい。**

#### `EXISTS` を使うべきとき（実務での推奨度：高）

* **「メインテーブルの形を変えたくない」** とき。
* サブクエリ側に重複があっても、メインの件数を維持したいとき。
* パフォーマンスを意識し、インデックスを活用して高速に「存在」だけを確認したいとき。

> **豆知識**：
> 実は `INTERSECT` は、内部的に「ソート」や「ハッシュ」を行うため、コストが高くなることがあります。一方 `EXISTS` は、1件見つかった瞬間に探索を止める（Early Exit）ため、巨大なテーブル同士の比較では `EXISTS` の方が圧倒的に速いことがよくあります。


## 参考リンク
https://www.shift-the-oracle.com/sql/intersect-operator.html

----
<br><br>


# 問題11-2（差集合） *Lv2*
### 2019年には購入があったが、2020年には一度も購入がない「離脱顧客」を特定してください。下記条件に従い取得してください。（順不同）
* **購入情報はSHスキーマ`SALES`テーブルを参照してください。**
* **購入日は`TIME_ID`を参照してください。**
* **SHスキーマ`CUSTOMERS`テーブルの顧客都市（`CUST_CITY`）が「Yokohama」、顧客クレジットカード限度額（`CUST_CREDIT_LIMIT`）が15000以上を対象としてください。**

## 期待する結果
| CUST_ID | NAME      | CUST_INCOME_LEVEL    | CUST_CREDIT_LIMIT | 
| ------- | --------- | -------------------- | ----------------- | 
| 33555   | Bud Smyth | J: 190,000 - 249,999 | 15000             | 

## 解答例
```sql:例1：MINUS/EXCEPT（差集合）を使用
WITH both_purchase AS (
    SELECT
        cust_id
    FROM
        sh.sales
    WHERE
        time_id BETWEEN date'2019-01-01' 
                    AND date'2019-12-31'
    MINUS   -- EXCEPTでも可
    SELECT
        cust_id
    FROM
        sh.sales
    WHERE
        time_id BETWEEN date'2020-01-01' 
                    AND date'2020-12-31'
)
SELECT
    c.cust_id,
    c.cust_first_name || ' ' || c.cust_last_name AS "NAME",
    c.cust_income_level,
    c.cust_credit_limit
FROM
    both_purchase bp
    INNER JOIN sh.customers c 
       ON bp.cust_id = c.cust_id
WHERE
        cust_city = 'Yokohama'
    AND cust_credit_limit >= 15000
```
```sql:例2：NOT EXISTSを使用
WITH both_purchase AS (
    SELECT DISTINCT
        cust_id
    FROM
        sh.sales s19
    WHERE
        time_id BETWEEN DATE'2019-01-01' 
                    AND DATE'2019-12-31'
        AND NOT EXISTS ( 
            SELECT
                'X'
            FROM
                sh.sales s20
            WHERE
                time_id BETWEEN DATE'2020-01-01' 
                            AND DATE'2020-12-31'
                AND s19.cust_id = s20.cust_id
        )
)
SELECT
    c.cust_id,
    c.cust_first_name || ' ' || c.cust_last_name AS "NAME",
    c.cust_income_level,
    c.cust_credit_limit
FROM
    both_purchase bp
    INNER JOIN sh.customers c 
       ON bp.cust_id = c.cust_id
WHERE
        cust_city = 'Yokohama'
    AND cust_credit_limit >= 15000
```
## 解説
SQLにおける **`MINUS`** は、2つの `SELECT` 文の結果を比較し、**「1番目の結果にはあるが、2番目の結果には存在しない行」** を抽出するセット演算子（集合演算子）です。また、SQL標準の **`EXCEPT`** は21c以降で正式にサポートされました。

数学の集合で言うところの **「差集合 ($A - B$)」** に相当します。

### 1. `MINUS` の基本構造と動作

#### 基本構文

```sql
SELECT column1, column2 FROM tableA
MINUS
SELECT column1, column2 FROM tableB;
```

#### 動作のルール
1. **重複の自動排除**: `MINUS` は結果を返す前に、自動的に `DISTINCT` を行います。つまり、1番目のテーブルに同じ行が複数あっても、結果には1行しか残りません。
2. **列の数と型の不一致はNG**: 1つ目と2つ目の `SELECT` 句で、カラムの数とデータ型（または互換性のある型）が完全に一致している必要があります。
3. **ソートの発生**: 内部的に重複排除と比較を行うため、通常は結果セットに対してソート処理が発生します。

### 2. 特筆すべき挙動：NULL の扱い

ここが `MINUS` の最も特徴的な部分です。

通常の `WHERE` 句での比較（`=`, `!=`）では、`NULL` は「不明」として扱われ、比較結果は一致しません。しかし、`MINUS`（および他のセット演算子）において **`NULL` は「値」として扱われ、`NULL` 同士は「等しい」とみなされます。**

* **1つ目の集合**: `(ID: 1, Name: NULL)`
* **2つ目の集合**: `(ID: 1, Name: NULL)`
* **結果**: 空（1つ目の NULL と 2つ目の NULL が「一致」とみなされて差し引かれるため）

### 3. 実務での活用例

今回のテーマである「離脱分析」において、`MINUS` は非常に直感的な武器になります。

#### 例：2024年には購入したが、2025年には一度も購入していない顧客ID

```sql
-- 2024年の顧客リスト
SELECT cust_id FROM sh.sales WHERE time_id BETWEEN '2024-01-01' AND '2024-12-31'
MINUS
-- 2025年の顧客リスト
SELECT cust_id FROM sh.sales WHERE time_id BETWEEN '2025-01-01' AND '2025-12-31';
```

### 4. `NOT EXISTS` との使い分け

「差分を出す」という意味では `NOT EXISTS` も同じことができますが、性質が異なります。

| 観点 | `MINUS` が適している場合 | `NOT EXISTS` が適している場合 |
| --- | --- | --- |
| **情報の維持** | IDリストだけが欲しいとき（重複が消えてもいい） | **購入金額や日付など、重複行を維持したまま**除外したいとき |
| **可読性** | スクリプトが短く、数学的に直感的 | メインテーブルのカラムを多く出力したいとき |
| **パフォーマンス** | 両方の集合が中規模で、インデックスがないとき | **サブクエリ側が巨大で、インデックスが整備されているとき** |
| **NULL** | NULL値を含むカラムで厳密に差をとりたいとき | 通常の結合条件（`=`）でNULLを無視したいとき |


## 参考リンク
https://www.shift-the-oracle.com/sql/minus-operator.html

----
<br><br>







# 問題11-3（和集合） *Lv2*
### 過去、現在全ての職歴情報を取得してください。下記条件に従い取得してください。（順不同）
* **現在の職歴情報はHRスキーマ`EMPLOYEES`テーブルを参照してください。**
* **過去の職歴情報はHRスキーマ`JOB_HISTORY`テーブルを参照してください。**
* **現在の職歴について、`START_DATE`を`HIRE_DATE`、`END_DATE`を'継続中'としてください。**


## 期待する結果
| EMPLOYEE_ID | START_DATE | END_DATE   | JOB_ID     | 
| ----------- | ---------- | ---------- | ---------- | 
| 100         | 2013-06-17 | 継続中     | AD_PRES    | 
| 101         | 2007-09-21 | 2011-10-27 | AC_ACCOUNT | 
| 101         | 2011-10-28 | 2015-03-15 | AC_MGR     | 
| 101         | 2015-09-21 | 継続中     | AD_VP      | 
| 102         | 2011-01-13 | 2016-07-24 | IT_PROG    | 
| 102         | 2011-01-13 | 継続中     | AD_VP      | 
| 103         | 2016-01-03 | 継続中     | IT_PROG    | 
| 104         | 2017-05-21 | 継続中     | IT_PROG    | 
| 105         | 2015-06-25 | 継続中     | IT_PROG    | 
| 106         | 2016-02-05 | 継続中     | IT_PROG    | 
| 107         | 2017-02-07 | 継続中     | IT_PROG    | 
| 108         | 2012-08-17 | 継続中     | FI_MGR     | 
| 109         | 2012-08-16 | 継続中     | FI_ACCOUNT | 

## 解答例
```sql
SELECT
    employee_id,
    TO_CHAR(start_date, 'YYYY-MM-DD') AS start_date,
    TO_CHAR(end_date, 'YYYY-MM-DD')   AS end_date,
    job_id
FROM
    hr.job_history
WHERE
    employee_id < 110
UNION ALL
SELECT
    employee_id,
    TO_CHAR(hire_date, 'YYYY-MM-DD') AS start_date,
    '継続中',
    job_id
FROM
    hr.employees
WHERE
    employee_id < 110
ORDER BY
    employee_id,
    start_date
```

## 解説
SQLにおいて、複数の`SELECT`文の結果を一つに統合する「集合演算子」の代表格が **`UNION`** と **`UNION ALL`** です。
一言でいうと、**「重複を削るのが `UNION`」「全部そのまま出すのが `UNION ALL`」** です。この違いがパフォーマンスに劇的な差を生むことがあります。

### 1. UNION と UNION ALL の違い

| 特徴 | UNION | UNION ALL |
| :--- | :--- | :--- |
| **重複行** | **削除する** (唯一の行だけ残す) | **削除しない** (すべて表示する) |
| **ソート (並び替え)** | 重複チェックのため内部で**行われる** | **行われない** |
| **実行速度** | 遅い (負荷が高い) | **速い** (負荷が低い) |
| **データの順番** | ある程度整列される | 抽出された順（バラバラ） |



### 2. 具体的な使い分け

#### ① UNION ALL を使うべきケース（推奨）
「データが重複していないことが分かっている」または「重複していても問題ない」場合は、**迷わず `UNION ALL` を選択**してください。
* 理由：重複チェック（ソート処理）が発生しないため、メモリ消費が少なく圧倒的に高速です。

```sql
-- 2025年の売上と2026年の売上を単純にくっつける（重複はないはず）
SELECT * FROM sales_2025
UNION ALL
SELECT * FROM sales_2026;
```

#### ② UNION を使うべきケース
「複数のテーブルからデータを集めるが、同じデータが混じっている可能性があり、それを1行にまとめたい」場合に使います。
* 内部的には `UNION ALL` を実行した後に `DISTINCT`（重複排除）をかけているようなイメージです。

```sql
-- 会員登録テーブルとイベント申込テーブルから「ユニークなメールアドレス一覧」を作る
SELECT email FROM members
UNION
SELECT email FROM event_attendees;
```


### 3. 鉄則と注意点

#### ① カラムの数と型を一致させる
結合するすべての `SELECT` 文で、**「カラムの数」と「データ型」が一致**していなければなりません。
* ❌ 1つめが `NUMBER` 型、2つめが `VARCHAR2` 型だとエラーになります。
* 💡 型が違う場合は、`TO_CHAR` などで明示的に型を合わせる必要があります。

#### ② 列名は「最初のSELECT文」が優先される
結果セットのヘッダー（列名）は、1番目の `SELECT` 文で指定したものが採用されます。
```sql
SELECT emp_name AS name FROM staff -- これが列名になる
UNION ALL
SELECT member_name FROM guests;
```

#### ③ ORDER BY は「最後」に1回だけ
各 `SELECT` 文に個別に `ORDER BY` を書くことはできません。並び替えをしたい場合は、クエリの最後に1つだけ記述します。
```sql
SELECT id, name FROM table_a
UNION ALL
SELECT id, name FROM table_b
ORDER BY id; -- 全体の結果に対してソートがかかる
```

#### ④ パフォーマンスの「隠れたコスト」
`UNION` を使うと、データベースは全てのデータをメモリ（または一時領域）に展開し、並び替えてから重複を比較します。数百万件のデータに対して `UNION` を使うと、**一時領域（TEMP）不足でエラーになったり、システム全体が重くなったりする**ため注意が必要です。

### 4. アドバイス
実務では、**「まずは `UNION ALL` で書けないか？」** を検討してください。重複を消す必要がある場合でも、`WHERE` 句などで事前にデータを絞り込んでから結合する方が効率的です。


----
