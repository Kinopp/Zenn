---
title: "集合演算子（全3問）"
free: false
---

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
これまでの「結合（JOIN）」がテーブルを**横**に繋げる技術だったのに対し、集合演算はクエリの結果同士を**縦**に重ね合わせる技術です。
今回のテーマである「積集合（INTERSECT）」は、マーケティング分析で「リピーター」や「特定の条件をすべて満たす優良顧客」を抽出する際の最強の武器になります。

### 1. 核心：積集合（INTERSECT）のイメージ
`INTERSECT` は、2つの `SELECT` 文の結果を比較し、**「両方に存在する行」だけ**を抜き出します。

#### ① 集合演算のルール
集合演算（INTERSECT, UNION, MINUS）を使うには、重ね合わせる2つの `SELECT` 文の間で以下のルールを守る必要があります。
* **列の数**が同じであること
* **列のデータ型**が一致していること

> **自動的な重複排除**
> `INTERSECT` は、結果の中に重複する行があっても、最終的には1つのユニークな行としてまとめてくれます（`DISTINCT` をかけたような状態になります）。


### 2. 3つの解答例、どれを選ぶべき？
実務において、同じ結果を得るための3つのアプローチを比較してみましょう。

| アプローチ | 特徴 | 現場での評価 |
| :--- | :--- | :--- |
| **例1：INTERSECT** | 数学の集合論そのものの書き方。 | **◎ 可読性最高。** 「2019年かつ2020年」という意図が最も伝わりやすい。 |
| **例2：EXISTS** | 1行ずつ存在をチェックする。 | **○ パフォーマンス重視。** データ量が膨大な場合、インデックスの効き方次第で最速になる。 |
| **例3：INNER JOIN** | 2019年リストと2020年リストをぶつける。 | **△ やや冗長。** 重複を防ぐために `DISTINCT` を意識的に書く必要がある。 |

### 3. ビジネス視点での「ロイヤル顧客」分析
今回のクエリが面白いのは、ただ抽出するだけでなく、最後に `CUSTOMERS` テーブルと結合して「属性（都市や限度額）」でさらに絞り込んでいる点です。

* **ステップ1**: 2019年にも2020年にも買ってくれた「継続性」を特定。
* **ステップ2**: その中から「横浜在住」かつ「クレジットカード限度額が高い」という「経済力」を特定。

このように、**「行動（継続購入）」×「属性（居住地・資産）」** を掛け合わせることで、プロモーションのターゲットを極限まで絞り込むことができるわけです。


### 4. 期待する結果の確認
結果に並んだ Roxanne Crocker さんたちは、まさに「横浜に住み、高い限度額を持ち、なおかつ2年も続けて買ってくれている」という、企業にとって喉から手が出るほど欲しいロイヤルカスタマーです。


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
前問の「リピーター（積集合）」とは真逆の、**「離脱客（差集合）」** を特定するテクニックですね。
ビジネスにおいて「去ってしまったお客様」を特定し、「なぜ離れてしまったのか？」「もう一度戻ってきてもらうにはどうすればいいか？」を分析するのは、売上を伸ばすために極めて重要なステップです。


### 1. 核心：差集合（MINUS / EXCEPT）のイメージ
`MINUS`（Oracle独自の呼称。標準SQLでは `EXCEPT`）は、1つ目の結果セットから、2つ目の結果セットに含まれる要素を**ガバッと引き算**します。

#### ① 動作のロジック
1.  **集合A**: 2019年に購入した全顧客リストを作成。
2.  **集合B**: 2020年に購入した全顧客リストを作成。
3.  **引き算**: AからBに名前がある人をすべて削除します。残ったのは **「2019年にはいたが、2020年にはいなくなった人」** だけです。


### 2. 文法のポイント：MINUS の特徴
集合演算子を使う際の共通ルールに加え、`MINUS` 特有の性質があります。

* **暗黙の重複排除**: `INTERSECT` と同様、結果は自動的に `DISTINCT` されます。
* **順序が重要**: `A MINUS B` と `B MINUS A` では結果が全く異なります（後者は「2020年に新しく入ってきた新規客」になります）。
* **NULLの扱い**: `MINUS` は NULL 同士を「一致」とみなして引き算してくれます。

### 3. 解答例の比較：MINUS vs NOT EXISTS
実務でどちらを使うべきか、判断基準を整理しましょう。

| 特徴 | `MINUS` (例1) | `NOT EXISTS` (例2) |
| :--- | :--- | :--- |
| **可読性** | **◎ 直感的。** 数学的な「A - B」そのもの。 | △ 少し複雑。相関サブクエリの理解が必要。 |
| **拡張性** | △ 列を増やすと、その列すべてが「引き算」の対象になる。 | **◎ 高い。** 特定のIDだけで比較しつつ、他の列も保持しやすい。 |
| **パフォーマンス** | ○ 小・中規模なら高速。 | **◎ 大規模データに強い。** 特にインデックスがある場合。 |

> **プロの使い分け**
> 「とにかくIDのリストだけをパッと引き算したい」時は **`MINUS`**。
> 「複雑な条件を組み合わせたり、大量のデータを高速に処理したい」時は **`NOT EXISTS`**。
> 現場ではこのように使い分けることが多いです。

### 4. 期待する結果の確認
結果に表示された Bud Smyth さんは、「横浜在住で15,000円以上の限度額を持つ優良顧客」でありながら、2020年にはパタリと購入が止まってしまった方です。
マーケティング担当者なら、このリストを見て「限定のカムバッククーポン」をメールで送る、といった施策を考えるはずです。


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
集合演算の締めくくりは、複数のクエリ結果を**縦にガッチャンコ**する **「和集合（UNION / UNION ALL）」** ですね。
実務では「現在のデータ」と「過去の履歴データ」が別々のテーブルに保存されていることがよくあります。それらを一つのリストとしてまとめて見たい時に、この技術は欠かせません。


### 1. 核心：和集合（UNION ALL）のイメージ
`UNION ALL` は、2つの `SELECT` 文の結果を単純に縦に積み上げます。結合（JOIN）が「横に広げる」のに対し、集合演算は「縦に伸ばす」イメージです。


#### ① UNION と UNION ALL の違い
ここが現場で最も重要な使い分けポイントです。

* **`UNION`**: 重複する行があった場合、それを1行にまとめて（排除して）から表示します。
* **`UNION ALL`**: 重複を気にせず、すべての行をそのまま表示します。

> **実務では UNION ALL が基本！**
> `UNION` は重複チェックの処理が入るため、データ量が多いと動作が重くなります。今回のように「履歴」と「現在」でデータが被る可能性が低い（あるいは被っていても出したい）場合は、**高速な `UNION ALL`** を選ぶのがプロの定石です。


### 2. 集合演算の「鉄の掟」
和集合を使うには、上下の `SELECT` 文で以下の2点を完璧に揃える必要があります。

1.  **列の数**: 上が4列なら、下も必ず4列。
2.  **データ型**: 1列目が数値なら下も数値、2列目が日付なら下も日付（あるいは変換後の文字列）でなければなりません。


### 3. テクニック解説：データ型を揃える「職人芸」
今回の解答例で、非常に賢い処理が行われている箇所があります。

### ① 日付と文字列の「型合わせ」
下の `SELECT` 文で、`END_DATE` に相当する箇所に **`'継続中'`** という文字列を入れています。
もし上の `SELECT` 文で `END_DATE` をそのまま「日付型」で出してしまうと、「下は文字列、上は日付」となり、型が合わずにエラーになります。

そのため、上のクエリでも **`TO_CHAR` を使ってあえて文字列に変換**することで、上下の型を「文字列」で統一しているのです。これがLv2のテクニックですね！

### ② 最後の ORDER BY
`ORDER BY` は、クエリの最後に一度だけ書きます。これにより、積み上げられた「過去」と「現在」の全データが混ざり合い、時系列順にきれいに整列されます。


### 4. 期待する結果の確認
101番（Neena Yangさん）のデータを見てください。
* 2007年〜
* 2011年〜
* 2015年〜（現在：継続中）
3行のデータが並んでいます。上の2行は `JOB_HISTORY` から、最後の1行は `EMPLOYEES` から来たものです。これが1つのリストになることで、Neenaさんの**キャリアの歩み**がひと目で分かるようになっています。

### 5. 集合演算子まとめ
Oracle 23aiで利用できる主な演算子は以下の4つです。

| 演算子 | 意味 | 重複の扱い | 特徴 |
| :--- | :--- | :--- | :--- |
| **`UNION`** | 和集合 | **排除する** | 重複を消すため、内部で並べ替え（ソート）が発生し、少し重い。 |
| **`UNION ALL`** | 和集合 | **保持する** | そのまま繋げるだけなので**最も高速**。実務で一番使う。 |
| **`INTERSECT`** | 積集合 | - | 両方の結果に含まれる共通の行だけを抽出する。 |
| **`MINUS`** | 差集合 | - | 1つ目の結果から、2つ目の結果に含まれるものを差し引く。 |

> **Oracle 23ai の進化ポイント:**
> 長らくOracleでは `MINUS` という独自キーワードを使ってきましたが、23aiからは標準SQLである **`EXCEPT`** も使えるようになりました。他DB（PostgreSQLやSQL Server）からの移行がよりスムーズになっています。

![](https://static.zenn.studio/user-upload/45cac6b3fc4b-20260504.png)
