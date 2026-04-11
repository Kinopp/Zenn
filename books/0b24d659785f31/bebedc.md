---
title: "集計関数（全5問）"
free: false
---

# 前提知識
## 1. 集計関数とは？
集計関数（Aggregate Functions）は、**「複数の行をひとまとめにして、1つの値を導き出す」** ための関数です。
通常、SQLでデータを取得すると「1行につき1つの結果」が返ってきますが、集計関数を使うと「100行を1行（合計や平均など）」に凝縮できます。

![](https://storage.googleapis.com/zenn-user-upload/0b8b696c35f4-20260322.png)

#### なぜ重要か？
ビジネスの現場で「生データ」をそのまま見ることは稀です。「先月の売上合計は？」「部署ごとの平均給与は？」といった**意思決定に必要な「情報」** に変えるのが、この関数の役割です。


## 2. 押さえておくべき基本の5大関数
Oracle 23aiでも変わらず多用される、基本の5つです。

| 関数名 | 役割 | 活用例（HR/SHスキーマ） |
| :--- | :--- | :--- |
| **`COUNT`** | 行数を数える | 全従業員数は何人か？ |
| **`SUM`** | 数値の合計を出す | 全社の給与支払い総額は？ |
| **`AVG`** | 数値の平均を出す | 製品の平均販売価格は？ |
| **`MAX`** | 最大値を見つける | 最も高い売上を記録した日はいつか？ |
| **`MIN`** | 最小値を見つける | 最も勤続年数が長い人の入社日は？ |




----
<br><br>


# 問題8-1（基本的な集計関数） *Lv1*
### **HRスキーマの`EMPLOYEES`テーブルについて、以下の値を算出してください。**
- **`SALARY`の合計**
- **`SALARY`の最大値**
- **`SALARY`の最小値**
- **`SALARY`の平均値**
- **`SALARY`の件数**

## 期待する結果
| SUM    | MAX   | MIN  | AVG                | CNT | 
| ------ | ----- | ---- | ------------------ | --- | 
| 691416 | 24000 | 2100 | 6461.8317757009300 | 107 | 

## 解答例
```sql
SELECT
    SUM(salary) AS "SUM",
    MAX(salary) AS "MAX",
    MIN(salary) AS "MIN",
    AVG(salary) AS "AVG",
    COUNT(*)    AS "CNT"
FROM
    hr.employees
```

## 解説
SQLの集計関数（Aggregate Functions）は、**「大量にある行（データ）をひとまとめにして、一つの値を計算する」** ための非常に強力なツールです。
例えば、「1000件の売上データ」を「1つの合計金額」に圧縮するのが集計関数の役割です。

### 1. 主要な5つの集計関数
まずは、どのデータベースでも必ず使う「基本の5つ」を紹介します。

| 関数名 | 説明 | 備考 |
| --- | --- | --- |
| **`COUNT`** | 行数やデータ数を数える | `COUNT(*)` は全行、`COUNT(列名)` は非NULL行を数える |
| **`SUM`** | 数値の合計を算出する | 数値型のみ。NULLは無視される |
| **`AVG`** | 数値の平均を算出する | NULLを無視して計算（後述の注意点あり） |
| **`MIN`** | 最小値を返す | 数値、日付、文字列（辞書順）に使える |
| **`MAX`** | 最大値を返す | 同上 |

### 2. NULLに関する重要な注意点
集計関数は **「原則としてNULLを無視」** します。

#### 例：テストの平均点（100点, 50点, NULL）の場合

* **`SUM`**: $100 + 50 = 150$
* **`COUNT(点数)`**: NULLは数えないので「2」
* **`AVG`**: $150 \div 2 = 75$

> **落とし穴：**
> もしNULLを「0点」として平均に含めたい場合は、`AVG(COALESCE(点数, 0))` のように書く必要があります。これなら $150 \div 3 = 50$ となります。


## 参考リンク
https://www.shift-the-oracle.com/sql/aggregate-functions/count.html
https://www.shift-the-oracle.com/sql/aggregate-functions/sum.html
https://www.shift-the-oracle.com/sql/aggregate-functions/avg.html
https://www.shift-the-oracle.com/sql/aggregate-functions/max.html
https://www.shift-the-oracle.com/sql/aggregate-functions/min.html


----
<br><br>


# 問題8-2（グループ化） *Lv1*
### **COスキーマの`ORDERS`テーブルから年月ごとの注文数を取得してください。ただし、キャンセルされた注文（`ORDER_STATUS`が「CANCELLED」のもの）は、集計対象から外すようにしてください。（`ORDER_TMS`の昇順）**

## 期待する結果
| ORDER_TMS | CNT | 
| --------- | --- | 
| 2021-02   | 17  | 
| 2021-03   | 72  | 
| 2021-04   | 110 | 
| 2021-05   | 110 | 
| 2021-06   | 123 | 
| 2021-07   | 121 | 
| 2021-08   | 141 | 
| 2021-09   | 157 | 
| 2021-10   | 172 | 
| 2021-11   | 151 | 
| 2021-12   | 189 | 
| 2022-01   | 214 | 
| 2022-02   | 154 | 
| 2022-03   | 152 | 
| 2022-04   | 32  | 


## 解答例
```sql
SELECT
    TO_CHAR(order_tms, 'YYYY-MM') AS order_tms,
    COUNT(*)                      AS cnt
FROM
    co.orders
WHERE
    order_status <> 'CANCELLED'
GROUP BY
    TO_CHAR(order_tms, 'YYYY-MM')
ORDER BY
    order_tms
```

## 解説
SQLにおいて、**`GROUP BY`** はデータを特定のカテゴリーごとに「バケツ分け」する非常に重要な命令です。これを使うことで、膨大な生のデータを「意味のある統計情報」へと要約することができます。

「全社員の平均給与」ではなく「**部署ごとの**平均給与」を出したいとき、その「〜ごとの」を指定するのが `GROUP BY` の役割です。
![](https://storage.googleapis.com/zenn-user-upload/65838e1c81ce-20260322.png)

### 1. GROUP BYのイメージ：データの「バケツ分け」

`GROUP BY` を実行すると、指定したカラムの値が同じ行が、1つのグループ（バケツ）にまとめられます。

例えば、社員テーブルを「部署ID」で `GROUP BY` すると：

1. 部署ID「10」の行が1つのバケツに集まる
2. 部署ID「20」の行が別のバケツに集まる
3. それぞれのバケツに対して、人数を数えたり（`COUNT`）、合計を出したり（`SUM`）する計算が行われる

### 2. 基本的な構文

```sql
SELECT 
    department_id,      -- グループ化の基準
    COUNT(*),           -- そのグループ内の行数
    SUM(salary)         -- そのグループ内の給与合計
FROM 
    employees
GROUP BY 
    department_id;      -- ここでバケツを指定
```

### 3. 複数カラムでのグループ化
「部署ごと」かつ「職種ごと」のように、より細かくバケツを分けることもできます。

```sql
SELECT department_id, job_id, COUNT(*)
FROM employees
GROUP BY department_id, job_id;
```

この場合、「10番部署のプログラマー」「10番部署のマネージャー」といった単位で集計されます。

<br>

:::message
### **ORA-00979: GROUP BYの式ではありません。（not a GROUP BY expression）**

SQL初心者が必ずといっていいほど遭遇する「ORA-00979」エラーについて説明します。
<br>
### 1. エラーの本質：何が起きているのか？
このエラーは、`GROUP BY`句を使用したSQLにおいて、**「集計（グループ化）したい単位」と「SELECTしたい項目」の整合性が取れていない**ときに発生します。

SQLの原則として、`GROUP BY`を使う場合、`SELECT`句に書けるのは以下の2種類だけです。

1. **GROUP BY句に指定した列**
2. **集計関数（SUM, AVG, MAX, MIN, COUNTなど）を通した値**

これ以外の「生の列」をSELECTに混ぜようとすると、Oracleは「グループ化した結果、どの行のデータを出せばいいか判断できない」ため、このエラーを投げます。
<br>
### 2. 具体的な発生パターンと修正例

#### ダメな例（NG）

例えば、部署（DEPT_ID）ごとに給与（SALARY）の合計を出したいが、個人の名前（NAME）も表示したい場合：

```sql
SELECT DEPT_ID, NAME, SUM(SALARY)
FROM EMPLOYEES
GROUP BY DEPT_ID;
```

> **エラーの理由:** `DEPT_ID`でグループ化すると、1つの部署に複数の`NAME`が存在します。Oracleは「1つの部署に対して、どの`NAME`を表示すればいいのか？」が分からないためエラーになります。

<br>
#### 修正案A：GROUP BYに列を追加する
名前もグループ化の基準に含める（＝部署内の個人ごとに集計する）場合です。

```sql
SELECT DEPT_ID, NAME, SUM(SALARY)
FROM EMPLOYEES
GROUP BY DEPT_ID, NAME; -- NAMEを追加
```

<br>
#### 修正案B：集計関数を使用する
「その部署の中で誰でもいいから1人の名前を出したい」といった場合は、集計関数を使います。

```sql
SELECT DEPT_ID, MAX(NAME), SUM(SALARY)
FROM EMPLOYEES
GROUP BY DEPT_ID; -- NAMEにMAX等を適用
```
<br>
### 3. 注意点とハマりやすいポイント

#### ① SELECT句に書いた計算式

SELECT句で計算や加工を行っている場合、その**加工後の形**、あるいは**加工前の列**がGROUP BYに不足しているとエラーになります。

* `SELECT TO_CHAR(HIRE_DATE, 'YYYY') ... GROUP BY TO_CHAR(HIRE_DATE, 'YYYY')` と記述を合わせる必要があります。
<br>
#### ② 定数やリテラルはOK
SELECT句にリテラル（'2024年度' など）を書く場合は、GROUP BYに含める必要はありません。
<br>
#### ③ サブクエリやJOINとの組み合わせ
複雑な結合を行っている際、どのテーブルのどの列がGROUP BYに欠けているか見失いやすくなります。エラーが発生したら、まず**「集計関数で囲まれていない列」をすべてリストアップし、それがGROUP BY句に漏れなく入っているか**を確認してください。
<br>

### 4. 実行順序をイメージする

SQLの内部的な実行順序を理解すると、このエラーが納得しやすくなります。

1. **FROM / JOIN**: データを集める
2. **WHERE**: 行を絞り込む
3. **GROUP BY**: **ここでデータが「塊（バケツ）」にまとめられる** 4.  **HAVING**: まとまったバケツを絞り込む
4. **SELECT**: **表示する（この時点ではもう「個々の行」にはアクセスできず、バケツごとの値しか見えない）**
:::

## 参考リンク
https://www.shift-the-oracle.com/sql/group-by-having.html

----
<br><br>


# 問題8-3（集計後の条件）  *Lv2*
### COスキーマの`ORDERS`テーブルから年月ごとの注文数のうち、月の注文数が150以上の年月を取得してください。ただし、キャンセルされた注文（`ORDER_STATUS`が「CANCELLED」のもの）は、集計対象から外すようにしてください。（`ORDER_TMS`の昇順）

## 期待する結果
| ORDER_TMS | CNT | 
| --------- | --- | 
| 2021-09   | 157 | 
| 2021-10   | 172 | 
| 2021-11   | 151 | 
| 2021-12   | 189 | 
| 2022-01   | 214 | 
| 2022-02   | 154 | 
| 2022-03   | 152 | 


## 解答例
```sql
SELECT
    to_char(order_tms, 'YYYY-MM') AS order_tms,
    COUNT(*)                      AS cnt
FROM
    co.orders
WHERE
    order_status <> 'CANCELLED'
GROUP BY
    to_char(order_tms, 'YYYY-MM')
HAVING
    COUNT(*) >= 150
ORDER BY
    order_tms
```

## 解説
**`HAVING`** は、一言でいうと **「グループ化した後の結果に対するフィルター」** です。

`WHERE` 句が「行（データ）」を絞り込むのに対し、`HAVING` 句は「グループ（集計結果）」を絞り込みます。この違いを理解することが、集計クエリをマスターする最大の鍵です。

### 1. `WHERE` と `HAVING` の決定的な違い

もっとも分かりやすい例えは、**「個人の選別」** か **「チームの選別」** か、という違いです。

| 句 | 絞り込む対象 | タイミング | 使えるもの |
| --- | --- | --- | --- |
| **`WHERE`** | 生のデータ（行） | 集計する**前** | カラムの値そのもの |
| **`HAVING`** | 集計された結果 | 集計した**後** | 集計関数（`SUM`, `AVG`, `COUNT`など） |

### 2. なぜ `HAVING` が必要なのか？

例えば、次のような「各部署の平均給与」を出すクエリを考えてみましょう。

> **「平均給与が 10,000円 を超える部署だけを知りたい」**

これを `WHERE` で書こうとするとエラーになります。

```sql
-- ❌ エラーになる例
SELECT department_id, AVG(salary)
FROM employees
WHERE AVG(salary) > 10000 -- WHERE句に集計関数は書けない！
GROUP BY department_id;
```

なぜなら、`WHERE` が動く時点では、まだ計算（`AVG`）が終わっていないからです。ここで `HAVING` の出番です。

```sql
-- ✅ 正解
SELECT department_id, AVG(salary)
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 10000; -- 集計が終わった後の結果で絞り込む
```


<br>

:::message
### **SQLの評価順**
#### **SQLの評価順序（実行される順番）**
- SQLクエリを作成する際、私たちは `SELECT` から書き始めますが、データベースの内部（実行エンジン）では、実は全く異なる順番で処理されています。

- この **「評価順（実行順序）」** を理解しておくと、「なぜ `SELECT` 句で定義した別名（エイリアス）が `ORDER BY` では使えるのに、`HAVING` や `GROUP BY` では使えないのか？」といった疑問やエラーの原因がスッキリと解決します。
<br>
#### **SQLの評価順序（一般的な流れ）**
- 標準的なSQL（Oracle, MySQL, PostgreSQLなど）では、以下の順番でデータの絞り込みや加工が行われます。

| 実行順 | 句 (Clause) | 役割 |
| --- | --- | --- |
| **1** | **FROM** | どのテーブルからデータを持ってくるか決める（JOINもここ） |
| **2** | **WHERE** | 行を絞り込む（フィルタリング） |
| **3** | **GROUP BY** | データをグループ化する |
| **4** | **HAVING** | グループ化した後の集計結果でさらに絞り込む |
| **5** | **SELECT** | **どの列を表示するか選ぶ（ここで別名が定義される）** |
| **6** | **DISTINCT** | 重複を除去する |
| **7** | **ORDER BY** | **最後にデータを並べ替える** |
| **8** | **LIMIT / FETCH** | 最終的な表示件数を制限する |

<br>

#### **なぜ「別名」が使える場所と使えない場所があるのか？**
- 解答例には、`SELECT` 句で定義した「order_tms」と「cnt」という2つの別名が登場します。ここで注目したいのは、**`ORDER BY` では別名を使っていますが、`HAVING` では使っていない** という点です。

- その理由は、先ほどの「評価順」にあります。
  - **HAVINGで別名が使えない理由**
評価順を見ると、`HAVING`（4番目）は `SELECT`（5番目）よりも**前**に処理されます。そのため、`HAVING` が動いている時点では、まだ `SELECT` で名付けた「cnt」という名前はデータベースの中に存在しておらず、使うことができないのです。
  - **ORDER BYで別名が使える理由**
一方で、`ORDER BY`（7番目）は `SELECT`（5番目）よりも**後**に処理されます。そのため、`SELECT` で定義した「order_tms」という名前を、データベースがすでに認識しており、そのまま利用できるということになります。

- イラストにまとめてみました！
![](https://storage.googleapis.com/zenn-user-upload/e088e4f5308d-20260309.png)

:::

----
<br><br>

# 問題8-4（特定の条件での集計）   *Lv3*
### **OEスキーマの`ORDERS`テーブルについて`ORDER_STATUS`を元に集計をしてください。**
* **集計単位 : 下記表の「グループ」ごと**
* **集計対象 : 件数およびORDER_TOTALの合計**

#### ORDER_STATUS の値とグループ分け
| 値 | 意味 (英語) | 説明 (日本語) | グループ |
| :--- | :--- | :--- | :--- |
| **0** | **Pending** | 保留中（入力が完了していない状態） | **待ち** |
| **1** | **Order entered** | 受付済み（注文が正常に登録された） | **正常** |
| **2** | **Order canceled** | キャンセル済み | **中止** |
| **3** | **Shipped - cash** | 出荷済み（現金払い） | **正常** |
| **4** | **Shipped - credit** | 出荷済み（クレジット払い） | **正常** |
| **5** | **Backordered** | 入荷待ち（在庫不足による出荷待ち） | **待ち** |
| **6** | **Order canceled - incorrect** | キャンセル済み（誤入力・不備など） | **中止** |
| **7** | **Order shipped - partially** | 一部出荷済み | **待ち** |
| **8** | **Order shipped - full** | すべて出荷済み | **正常** |
| **9** | **Order replaced** | 交換済み | **その他** |
| **10** | **Order closed** | 完了（すべての処理が終了） | **正常** |




## 期待する結果
| GROUP  | CNT | SUM       | 
| ------ | --- | --------- | 
| 待ち   | 29  | 578572.5  | 
| 正常   | 50  | 2350361.9 | 
| 中止   | 16  | 476383.7  | 
| その他 | 10  | 262736.6  | 

## 解答例
```sql:例1：GROUP BYにCASE文を使用
SELECT
    CASE
        WHEN order_status IN ( 0, 5, 7 ) THEN
            '待ち'
        WHEN order_status IN ( 1, 3, 4, 8, 10 ) THEN
            '正常'
        WHEN order_status IN ( 2, 6 ) THEN
            '中止'
        ELSE
            'その他'
    END              AS "GROUP",
    COUNT(*)         AS "CNT",
    SUM(order_total) AS "SUM"
FROM
    oe.orders
GROUP BY
    CASE
        WHEN order_status IN ( 0, 5, 7 ) THEN
            '待ち'
        WHEN order_status IN ( 1, 3, 4, 8, 10 ) THEN
            '正常'
        WHEN order_status IN ( 2, 6 ) THEN
            '中止'
        ELSE
            'その他'
    END
```
```sql:例2：GROUP BYに別名を使用（23c以降）
SELECT
    CASE
        WHEN order_status IN ( 0, 5, 7 ) THEN
            '待ち'
        WHEN order_status IN ( 1, 3, 4, 8, 10 ) THEN
            '正常'
        WHEN order_status IN ( 2, 6 ) THEN
            '中止'
        ELSE
            'その他'
    END              AS "GROUP",
    COUNT(*)         AS "CNT",
    SUM(order_total) AS "SUM"
FROM
    oe.orders
GROUP BY
    "GROUP"
```
```sql:例3：サブクエリーを使用
SELECT
    "GROUP",
    COUNT(*)         AS "CNT",
    SUM(order_total) AS "SUM"
FROM
    (
        SELECT
            order_total,
            CASE
                WHEN order_status IN ( 0, 5, 7 ) THEN
                    '待ち'
                WHEN order_status IN ( 1, 3, 4, 8, 10 ) THEN
                    '正常'
                WHEN order_status IN ( 2, 6 ) THEN
                    '中止'
                ELSE
                    'その他'
            END AS "GROUP"
        FROM
            oe.orders
    )
GROUP BY
    "GROUP"
```

## 解説
`GROUP BY` 句に `CASE` 文を使うと、テーブルにある生の値をそのまま集計するのではなく、**「特定の条件でグループ分けした結果」** で集計できるようになります。

### 1. GROUP BY で CASE 文を使う（例1）
多くのデータベース（以前のOracleを含む）では、`SELECT` 句で書いた `CASE` 文を、**`GROUP BY` 句にも全く同じように書く**必要があります。

> **なぜ2回書く必要があるのか？**
> SQLの評価順序では、`GROUP BY`は `SELECT`よりも先に実行されます。そのため、`SELECT` で付けた「GROUP」という名前を `GROUP BY` はまだ知らないため、中身の式をもう一度書く必要があるのです。

### 2. バージョンによる「別名（エイリアス）」の使用（例2）
「同じ `CASE` 文を2回書くのは面倒だし、修正漏れが怖い…」という不満は昔からありました。これに対し、**23c (23ai)以降**では **`SELECT` で付けた別名を `GROUP BY` でそのまま使える**ようになっています。

### 3. 古いバージョンで「2回書く」のを避ける裏技（例3）
Oracle 19c以前など、別名が使えない環境でコードをスッキリさせたい場合は、**インラインビュー（サブクエリ）** を使うのが定石です。



## 参考リンク
https://www.youtube.com/watch?v=SsJluwMpM2o&list=PLb1qVSx1k1VovCRavgMQSS_cvwJoNpAGc&index=6

----
<br><br>

# 問題8-5（１つの文字列への集計）   *Lv2*
### HRスキーマ`COUNTRIES`テーブルを`REGION_ID`でグルーピングし、COUNTRY_LISTを取得してください。COUNTRY_LISTは同一`REGION_ID`に含まれる`COUNTRY_ID`をアルファベット順にカンマ区切りで列挙したものです。

## 期待する結果
| REGION_ID | COUNTRY_LIST                   | 
| --------- | ------------------------------ | 
| 10        | BE, CH, DE, DK, FR, GB, IT, NL | 
| 20        | AR, BR, CA, MX, US             | 
| 30        | CN, IL, IN, JP, KW, ML, SG     | 
| 40        | AU                             | 
| 50        | EG, NG, ZM, ZW                 | 

## 解答例
```sql
SELECT
    region_id,
    LISTAGG(country_id, ', ') WITHIN GROUP(
    ORDER BY
        country_id
    ) AS country_list
FROM
    hr.countries
GROUP BY
    region_id;
```

## 解説

### 1. LISTAGG関数とは？
**`LISTAGG`** は、複数行にまたがるデータを**特定の区切り文字でつなげて、1つの文字列に集約する**関数です。
「部署ごとの社員名リストをカンマ区切りで作る」といった、レポート作成やデータ整形に欠かせない機能です。

### 2. 基本の使い方
まずは、標準的なサンプルスキーマ（`HR` または `SCOTT`）をイメージした例を見てみましょう。

#### 基本構文
```sql
LISTAGG(列名, ', ') WITHIN GROUP (ORDER BY ...) 
```

### ポイント
* **第1引数**: 連結したい列名
* **第2引数**: 区切り文字（`', '`）
* **WITHIN GROUP (ORDER BY ...)**: 連結する順番を指定します。これがないとエラーになります。

### 3. 重複の排除 (Oracle 19c / 23ai)
古いバージョンのOracleでは、連結するリストの中に重複がある場合（例：部署ごとの役職リスト）、一度サブクエリで `DISTINCT` する必要がありました。
しかし、最新のバージョンでは **`DISTINCT` キーワード** が直接使えます。

### 重複を除いたリスト作成
```sql:例
SELECT 
    department_id,
    LISTAGG(DISTINCT job_id, ' / ') WITHIN GROUP (ORDER BY job_id) AS job_roles
FROM 
    hr.employees
GROUP BY 
    department_id;
```
> **注意点**: 重複排除は、大規模なデータセットではソート負荷がかかるため、必要な場合のみ使用しましょう。

### 4. オーバーフロー処理
`LISTAGG` の最大の弱点は、連結後の文字列が **4,000バイト（または32,767バイト）を超えるとエラーになる** ことです。
実務で「データが多すぎてSQLが止まった！」という事故を防ぐため、**`ON OVERFLOW`句** を活用しましょう。

### 安全なリスト連結
```sql
SELECT 
    department_id,
    LISTAGG(last_name, ', ' 
        ON OVERFLOW TRUNCATE '...' WITH COUNT) 
        WITHIN GROUP (ORDER BY last_name) AS employee_list
FROM 
    hr.employees
GROUP BY 
    department_id;
```

* **TRUNCATE '...'**: 上限を超えたら末尾を `...` で省略する。
* **WITH COUNT**: 「... (残り15件)」のように、省略された件数を表示する。



## 参考リンク
https://www.shift-the-oracle.com/sql/aggregate-functions/listagg.html

https://www.youtube.com/watch?v=QSTvwwq5HBg



----
<br><br>

