---
title: "第14章 階層構造の取り扱い（全4問）"
free: false
---

# 問題14-1：階層問い合わせ（SYS_CONNECT_BY_PATH）

### 難易度：★★★☆☆ (Lv.3)

## 問題
HRスキーマの`EMPLOYEES`テーブルを参照し、従業員とその上司の関係をツリー構造のパスとして取得してください。

**【抽出・編集ルール】**

* **階層の表現**：最上位の上司から現在の従業員までを「 <-- 」で繋いで表示してください。
* 例：`最上位上司 <-- 直属上司 <-- 従業員`


* **開始地点**：社長（`MANAGER_ID` が `NULL` の人物）をルート（根）として開始してください。
* **リレーション**：ある行の `EMPLOYEE_ID` が、次の行の `MANAGER_ID` と一致する関係で繋いでください。

## 期待する結果
| 	EMPLOYEE_TREE	| 
| 	----	| 
| 	100	| 
| 	100 <-- 101	| 
| 	100 <-- 101 <-- 108	| 
| 	100 <-- 101 <-- 108 <-- 109	| 
| 	100 <-- 101 <-- 108 <-- 110	| 
| 	100 <-- 101 <-- 108 <-- 111	| 
| 	100 <-- 101 <-- 108 <-- 112	| 
| 	100 <-- 101 <-- 108 <-- 113	| 
| 	100 <-- 101 <-- 200	| 
| 	100 <-- 101 <-- 203	| 
| 	100 <-- 101 <-- 204	| 
| 	100 <-- 101 <-- 205	| 
| 	100 <-- 101 <-- 205 <-- 206	| 
| 	100 <-- 102	| 
| 	100 <-- 102 <-- 103	| 
| 	100 <-- 102 <-- 103 <-- 104	| 
| 	100 <-- 102 <-- 103 <-- 105	| 
| 	100 <-- 102 <-- 103 <-- 106	| 
| 	100 <-- 102 <-- 103 <-- 107	| 
| 	100 <-- 114	| 
| 	100 <-- 114 <-- 115	| 
| 	100 <-- 114 <-- 116	| 
| 	100 <-- 114 <-- 117	| 
| 	100 <-- 114 <-- 118	| 
| 	100 <-- 114 <-- 119	| 
| 	100 <-- 120	| 
| 	100 <-- 120 <-- 125	| 
| 	100 <-- 120 <-- 126	| 
| 	100 <-- 120 <-- 127	| 
| 	100 <-- 120 <-- 128	| 
| 	100 <-- 120 <-- 180	| 
| 	100 <-- 120 <-- 181	| 
| 	100 <-- 120 <-- 182	| 
| 	100 <-- 120 <-- 183	| 
| 	100 <-- 121	| 
| 	100 <-- 121 <-- 129	| 
| 	100 <-- 121 <-- 130	| 
| 	100 <-- 121 <-- 131	| 
| 	100 <-- 121 <-- 132	| 
| 	100 <-- 121 <-- 184	| 
| 	100 <-- 121 <-- 185	| 
| 	100 <-- 121 <-- 186	| 
| 	100 <-- 121 <-- 187	| 
| 	100 <-- 122	| 
| 	100 <-- 122 <-- 133	| 
| 	100 <-- 122 <-- 134	| 
| 	100 <-- 122 <-- 135	| 
| 	100 <-- 122 <-- 136	| 
| 	100 <-- 122 <-- 188	| 
| 	100 <-- 122 <-- 189	| 
| 	100 <-- 122 <-- 190	| 
| 	100 <-- 122 <-- 191	| 
| 	100 <-- 123	| 
| 	100 <-- 123 <-- 137	| 
| 	100 <-- 123 <-- 138	| 
| 	100 <-- 123 <-- 139	| 
| 	100 <-- 123 <-- 140	| 
| 	100 <-- 123 <-- 192	| 
| 	100 <-- 123 <-- 193	| 
| 	100 <-- 123 <-- 194	| 
| 	100 <-- 123 <-- 195	| 
| 	100 <-- 124	| 
| 	100 <-- 124 <-- 141	| 
| 	100 <-- 124 <-- 142	| 
| 	100 <-- 124 <-- 143	| 
| 	100 <-- 124 <-- 144	| 
| 	100 <-- 124 <-- 196	| 
| 	100 <-- 124 <-- 197	| 
| 	100 <-- 124 <-- 198	| 
| 	100 <-- 124 <-- 199	| 
| 	100 <-- 145	| 
| 	100 <-- 145 <-- 150	| 
| 	100 <-- 145 <-- 151	| 
| 	100 <-- 145 <-- 152	| 
| 	100 <-- 145 <-- 153	| 
| 	100 <-- 145 <-- 154	| 
| 	100 <-- 145 <-- 155	| 
| 	100 <-- 146	| 
| 	100 <-- 146 <-- 156	| 
| 	100 <-- 146 <-- 157	| 
| 	100 <-- 146 <-- 158	| 
| 	100 <-- 146 <-- 159	| 
| 	100 <-- 146 <-- 160	| 
| 	100 <-- 146 <-- 161	| 
| 	100 <-- 147	| 
| 	100 <-- 147 <-- 162	| 
| 	100 <-- 147 <-- 163	| 
| 	100 <-- 147 <-- 164	| 
| 	100 <-- 147 <-- 165	| 
| 	100 <-- 147 <-- 166	| 
| 	100 <-- 147 <-- 167	| 
| 	100 <-- 148	| 
| 	100 <-- 148 <-- 168	| 
| 	100 <-- 148 <-- 169	| 
| 	100 <-- 148 <-- 170	| 
| 	100 <-- 148 <-- 171	| 
| 	100 <-- 148 <-- 172	| 
| 	100 <-- 148 <-- 173	| 
| 	100 <-- 149	| 
| 	100 <-- 149 <-- 174	| 
| 	100 <-- 149 <-- 175	| 
| 	100 <-- 149 <-- 176	| 
| 	100 <-- 149 <-- 177	| 
| 	100 <-- 149 <-- 178	| 
| 	100 <-- 149 <-- 179	| 
| 	100 <-- 201	| 
| 	100 <-- 201 <-- 202	| 

## 解答例
```sql
SELECT
    LTRIM(
        SYS_CONNECT_BY_PATH(employee_id, ' <-- '),
        ' <-- '
    ) AS employee_tree
FROM
    hr.employees
START WITH
    manager_id IS NULL
CONNECT BY
    PRIOR employee_id = manager_id
```

## 解説
**「階層問い合わせ（START WITH / CONNECT BY）」** は、組織図や部品表（BOM）など、親子関係を持つデータを扱う際の必須テクニックです。
初心者には「魔法」のように見えますが、仕組みを正しく理解しないと「無限ループ」や「意図しないデータの欠落」を招きます。

### 階層問い合わせ：データの「家系図」を解き明かす

#### 1. 階層問い合わせの基本構造
「どこから始めて（START WITH）」「どう繋ぐか（CONNECT BY）」を指定します。

```sql
SELECT 
    LEVEL,              -- 階層の深さ (1, 2, 3...)
    LPAD(' ', (LEVEL-1)*2) || ENAME AS TREE, -- 見た目を整える
    EMPNO, MGR
FROM EMP
START WITH MGR IS NULL  -- ルート（社長）から開始
CONNECT BY PRIOR EMPNO = MGR; -- 「前の行の社員番号」が「今の行の上司番号」
```

##### 💡 最重要キーワード：PRIOR（さっきの）
初心者が一番混乱するのが `PRIOR` をどちらに付けるかです。
* **`PRIOR` ＝ 「一階層上の（親の）」** と読み替えましょう。
* `CONNECT BY PRIOR EMPNO = MGR`
    * （親の）社員番号 ＝ （今の）上司番号 $\rightarrow$ **トップダウン（上から下へ）**
* `CONNECT BY EMPNO = PRIOR MGR`
    * （今の）社員番号 ＝ （親の）上司番号 $\rightarrow$ **ボトムアップ（下から上へ）**


#### 2. 階層問い合わせで使える便利な「擬似列・関数」

| 項目 | 説明 |
| :--- | :--- |
| **LEVEL** | 階層の深さを表す数値。1から始まります。 |
| **CONNECT_BY_ISLEAF** | 最下層（末端）なら 1、そうでなければ 0 を返します。 |
| **SYS_CONNECT_BY_PATH** | ルートからの全経路を文字列で連結して表示します。 |
| **ORDER SIBLINGS BY** | **重要！** 階層構造を壊さずに、同じ親を持つ兄弟間だけでソートします。 |


#### 3. 初心者が陥りやすい「落とし穴」

##### ① WHERE句とCONNECT BY句の順番
`WHERE` 句は「階層構造を作った**後**」に効きます。
* もし「特定の部署を除外したい」時に `WHERE` を使うと、その部署の**部下たちは残ってしまい、ツリーが分断**されます。
* 部下ごと消したい場合は、`CONNECT BY` 句の中に条件（`AND DEPTNO != 10` など）を書く必要があります。

##### ② 無限ループ（NOCYCLE）
データに「Aの上司はB、Bの上司はA」のような循環（サイクル）があると、Oracleはエラーを吐いて停止します。
* **対策：** `CONNECT BY NOCYCLE` と記述することで、ループを検知して停止させることができます。

##### ③ パフォーマンス問題
巨大なツリー（数万件以上の親子関係）に対して複雑な結合を行いながら `CONNECT BY` を行うと、メモリを大量に消費します。実行計画を確認し、結合キー（`EMPNO` や `MGR`）にインデックスがあるか必ず確認しましょう。

#### 4. アドバイス
Oracle 23ai（および近年のバージョン）では、標準SQLである **「再帰CTE（WITH句による再帰）」** も利用可能です。

* **CONNECT BY:** Oracle独自。短く書けて直感的。
* **再帰CTE:** ANSI標準。複雑な計算や、Oracle以外（PostgreSQLやSQL Server）への移行を考えるならこちら。

> **上級者への一言：**
> 「23aiでは再帰CTEのデバッグもしやすくなっていますが、パッと階層を出したい時は依然として `CONNECT BY` の方が記述量が少なく圧倒的に楽です。適材適所で使い分けましょう！」

----

<br><br>


# 【完全版】問題14-2：階層の視覚化（インデントと深さ優先探索）

### 難易度：★★★★☆ (Lv.4)

## 問題
HRスキーマの`EMPLOYEES`テーブルを参照し、社長から末端の従業員までの関係をツリー形式で表示してください。

**【表示・整形ルール】**

* **レベル番号**：階層の深さを表す「1. 」「2. 」といった番号を氏名の先頭に付与してください。
* **インデント**：階層が1つ下がるごとに、氏名の前に**半角スペースを2つ**付与してください。
* **氏名**：`FIRST_NAME` と `LAST_NAME` を半角スペースで結合してください。
* **ソート順**：深さ優先探索（Depth First Search）の順序で表示してください。
* ※深さ優先探索とは、ある上司の下にいる部下たちを、そのさらに下の階層まで含めて先に全て辿る方式です。


## 期待する結果
| 	MANAGER_ID	| 	EMPLOYEE_ID	| 	EMPLOYEE_NAME	| 
| 	----	| 	----	| 	----	| 
| 	 - 	| 	100	| 	1. Steven King	| 
| 	100	| 	101	| 	&nbsp;&nbsp;2. Neena Kochhar	| 
| 	101	| 	108	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Nancy Greenberg	| 
| 	108	| 	109	| 	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. Daniel Faviet	| 
| 	108	| 	110	| 	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. John Chen	| 
| 	108	| 	111	| 	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. Ismael Sciarra	| 
| 	108	| 	112	| 	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. Jose Manuel Urman	| 
| 	108	| 	113	| 	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. Luis Popp	| 
| 	101	| 	200	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Jennifer Whalen	| 
| 	101	| 	203	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Susan Mavris	| 
| 	101	| 	204	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Hermann Baer	| 
| 	101	| 	205	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Shelley Higgins	| 
| 	205	| 	206	| 	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. William Gietz	| 
| 	100	| 	102	| 	&nbsp;&nbsp;2. Lex De Haan	| 
| 	102	| 	103	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Alexander Hunold	| 
| 	103	| 	104	| 	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. Bruce Ernst	| 
| 	103	| 	105	| 	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. David Austin	| 
| 	103	| 	106	| 	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. Valli Pataballa	| 
| 	103	| 	107	| 	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4. Diana Lorentz	| 
| 	100	| 	114	| 	&nbsp;&nbsp;2. Den Raphaely	| 
| 	114	| 	115	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Alexander Khoo	| 
| 	114	| 	116	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Shelli Baida	| 
| 	114	| 	117	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Sigal Tobias	| 
| 	114	| 	118	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Guy Himuro	| 
| 	114	| 	119	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Karen Colmenares	| 
| 	100	| 	120	| 	&nbsp;&nbsp;2. Matthew Weiss	| 
| 	120	| 	125	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Julia Nayer	| 
| 	120	| 	126	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Irene Mikkilineni	| 
| 	120	| 	127	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. James Landry	| 
| 	120	| 	128	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Steven Markle	| 
| 	120	| 	180	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Winston Taylor	| 
| 	120	| 	181	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Jean Fleaur	| 
| 	120	| 	182	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Martha Sullivan	| 
| 	120	| 	183	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Girard Geoni	| 
| 	100	| 	121	| 	&nbsp;&nbsp;2. Adam Fripp	| 
| 	121	| 	129	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Laura Bissot	| 
| 	121	| 	130	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Mozhe Atkinson	| 
| 	121	| 	131	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. James Marlow	| 
| 	121	| 	132	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. TJ Olson	| 
| 	121	| 	184	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Nandita Sarchand	| 
| 	121	| 	185	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Alexis Bull	| 
| 	121	| 	186	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Julia Dellinger	| 
| 	121	| 	187	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Anthony Cabrio	| 
| 	100	| 	122	| 	&nbsp;&nbsp;2. Payam Kaufling	| 
| 	122	| 	133	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Jason Mallin	| 
| 	122	| 	134	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Michael Rogers	| 
| 	122	| 	135	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Ki Gee	| 
| 	122	| 	136	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Hazel Philtanker	| 
| 	122	| 	188	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Kelly Chung	| 
| 	122	| 	189	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Jennifer Dilly	| 
| 	122	| 	190	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Timothy Gates	| 
| 	122	| 	191	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Randall Perkins	| 
| 	100	| 	123	| 	&nbsp;&nbsp;2. Shanta Vollman	| 
| 	123	| 	137	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Renske Ladwig	| 
| 	123	| 	138	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Stephen Stiles	| 
| 	123	| 	139	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. John Seo	| 
| 	123	| 	140	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Joshua Patel	| 
| 	123	| 	192	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Sarah Bell	| 
| 	123	| 	193	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Britney Everett	| 
| 	123	| 	194	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Samuel McCain	| 
| 	123	| 	195	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Vance Jones	| 
| 	100	| 	124	| 	&nbsp;&nbsp;2. Kevin Mourgos	| 
| 	124	| 	141	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Trenna Rajs	| 
| 	124	| 	142	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Curtis Davies	| 
| 	124	| 	143	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Randall Matos	| 
| 	124	| 	144	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Peter Vargas	| 
| 	124	| 	196	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Alana Walsh	| 
| 	124	| 	197	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Kevin Feeney	| 
| 	124	| 	198	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Donald OConnell	| 
| 	124	| 	199	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Douglas Grant	| 
| 	100	| 	145	| 	&nbsp;&nbsp;2. John Russell	| 
| 	145	| 	150	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Peter Tucker	| 
| 	145	| 	151	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. David Bernstein	| 
| 	145	| 	152	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Peter Hall	| 
| 	145	| 	153	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Christopher Olsen	| 
| 	145	| 	154	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Nanette Cambrault	| 
| 	145	| 	155	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Oliver Tuvault	| 
| 	100	| 	146	| 	&nbsp;&nbsp;2. Karen Partners	| 
| 	146	| 	156	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Janette King	| 
| 	146	| 	157	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Patrick Sully	| 
| 	146	| 	158	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Allan McEwen	| 
| 	146	| 	159	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Lindsey Smith	| 
| 	146	| 	160	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Louise Doran	| 
| 	146	| 	161	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Sarath Sewall	| 
| 	100	| 	147	| 	&nbsp;&nbsp;2. Alberto Errazuriz	| 
| 	147	| 	162	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Clara Vishney	| 
| 	147	| 	163	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Danielle Greene	| 
| 	147	| 	164	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Mattea Marvins	| 
| 	147	| 	165	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. David Lee	| 
| 	147	| 	166	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Sundar Ande	| 
| 	147	| 	167	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Amit Banda	| 
| 	100	| 	148	| 	&nbsp;&nbsp;2. Gerald Cambrault	| 
| 	148	| 	168	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Lisa Ozer	| 
| 	148	| 	169	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Harrison Bloom	| 
| 	148	| 	170	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Tayler Fox	| 
| 	148	| 	171	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. William Smith	| 
| 	148	| 	172	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Elizabeth Bates	| 
| 	148	| 	173	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Sundita Kumar	| 
| 	100	| 	149	| 	&nbsp;&nbsp;2. Eleni Zlotkey	| 
| 	149	| 	174	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Ellen Abel	| 
| 	149	| 	175	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Alyssa Hutton	| 
| 	149	| 	176	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Jonathon Taylor	| 
| 	149	| 	177	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Jack Livingston	| 
| 	149	| 	178	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Kimberely Grant	| 
| 	149	| 	179	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Charles Johnson	| 
| 	100	| 	201	| 	&nbsp;&nbsp;2. Michael Hartstein	| 
| 	201	| 	202	| 	&nbsp;&nbsp;&nbsp;&nbsp;3. Pat Fay	|


## 解答例
```sql:例1：再帰的with句を使用
WITH recursive_pr (
    employee_id,
    manager_id,
    first_name,
    last_name,
    lvl
) AS (
    SELECT
        e1.employee_id,
        e1.manager_id,
        e1.first_name,
        e1.last_name,
        1 AS lvl
    FROM
        hr.employees e1
    WHERE
        e1.manager_id IS NULL
    UNION ALL
    SELECT
        e2.employee_id,
        e2.manager_id,
        e2.first_name,
        e2.last_name,
        rpr.lvl + 1
    FROM
             recursive_pr rpr
        INNER JOIN hr.employees e2 ON e2.manager_id = rpr.employee_id
)
    SEARCH DEPTH FIRST BY manager_id SET rpr_order
SELECT
    manager_id,
    employee_id,
    lpad(' ', 2 *(lvl - 1))
    || lvl
    || '. '
    || first_name
    || ' '
    || last_name AS employee_name
FROM
    recursive_pr
ORDER BY
    rpr_order
```

```sql:例2：階層問合せ演算子を使用
SELECT
    manager_id,
    employee_id,
    LPAD(' ', 2 *(LEVEL - 1))
    || LEVEL
    || '. '
    || first_name
    || ' '
    || last_name AS employee_name
FROM
    hr.employees
START WITH
    manager_id IS NULL
CONNECT BY
    PRIOR employee_id = manager_id
```

## 解説
SQLにおける「階層問合せ（Hierarchical Query）」の集大成です。会社組織や部品構成表（BOM）のように、親子関係があるデータを扱う際の必須テクニックですね。
今回のポイントは、単に繋げるだけでなく「見た目を美しく整える（インデントとレベル番号）」ところにあります。

### 1. 核心：ツリー構造を支える「インデントの魔法」
期待する結果のような「美しいツリー」を作る最大のポイントは、解答例のどちらにも登場する **`LPAD` 関数** です。

#### ① `LPAD` と `LEVEL` のコンビネーション

```sql
LPAD(' ', 2 * (lvl - 1)) || lvl || '. ' || first_name
```

* **`lvl` (または `LEVEL`)**: 階層の深さを表す数値です（社長=1, 副社長=2...）。
* **`LPAD(' ', 2 * (lvl - 1))`**: 「階層が1つ深くなるごとに、半角スペースを2つずつ追加する」という命令です。

これにより、SQLの実行結果がテキストベースであっても、一目で組織のレポートライン（誰が誰の下か）が分かるようになります。


### 2. 解法の深掘り：2つのアプローチ
実務でどちらを使うべきか、その「キャラの違い」を整理しておきましょう。

#### 例1：再帰的WITH句（標準SQLスタイル）
モダンな開発現場で好まれる、非常に論理的な書き方です。

1. **アンカー部（上側）**: まず「社長（マネージャーがいない人）」を特定します。
2. **再帰部（下側）**: 特定した人に紐づく「部下」を次々に繋げていきます。
3. **SEARCH DEPTH FIRST**: ここが重要です！「一人の上司の部下をすべて掘り下げてから、次の上司へ行く」という**深さ優先探索**を指定することで、組織図として自然な並び順を実現しています。

#### 例2：CONNECT BY（Oracleの伝統芸）
Oracleを使うなら避けては通れない、非常に強力かつ簡潔な独自構文です。

* **`START WITH`**: 探索を開始する「根（ルート）」を指定。
* **`CONNECT BY PRIOR`**: 「今の行のID（親）と、次の行のManagerID（子）を繋ぐ」という親子関係を一行で定義。
* **デフォルトの挙動**: `CONNECT BY` は標準で深さ優先探索を行うため、並び替えの指定を最小限に抑えられます。

### 3. 実務での使い分け：どっちが最強？

| 特徴 | 再帰的WITH句 (例1) | CONNECT BY (例2) |
| --- | --- | --- |
| **可読性** | 手続きを追えるので理解しやすい。 | 非常に短く、慣れると爆速で書ける。 |
| **汎用性** | **◎** PostgreSQLやSQL Serverでも動く。 | **△** Oracle専用（一部他DBで互換あり）。 |
| **柔軟性** | 複雑な計算を再帰中に組み込みやすい。 | 基本的なツリー構造には最適。 |

> **プロの視点：DFS vs BFS**
> 今回の「上司のすぐ下にその部下を並べる」のが **深さ優先探索 (DFS)**。
> 一方で、「まず全社員のレベル1を出し、次に全員のレベル2を出す……」という並びは **幅優先探索 (BFS)** と呼ばれます。用途によって使い分けられるようになると、Lv4卒業ですね！


## 参考リンク
https://www.shift-the-oracle.com/sql/with.html#with-recursive


----
<br><br>


# 【完全版】問題14-3：組織ツリーの完全走査（配下の集計分析）

### 難易度：★★★★☆ (Lv.4)

## 問題
HRスキーマの`EMPLOYEES`テーブルを参照し、「自分自身」および「その配下にいる全従業員（間接的な部下も含む）」を一つの組織ユニットとして、以下の指標を算出してください。

**【対象者】**

* 自分以外に1人以上の部下（直接・間接を問わず）を持っている従業員。

**【算出項目】**
1. **TOTAL_ORG_SALARY**：自分自身を含めた配下全員の給与（`SALARY`）の合計。
2. **TOTAL_SUBORDINATES**：自分自身を除いた配下全員の人数。
3. **PCT_OF_TOTAL_ORG_SALARY**：組織全体の総人件費に対する、自分自身の給与が占める割合（％）。
4. **INDENTED_EMP_NAME**：階層レベルに応じて、氏名の前にスペースを付与して階層化して表示。

**【表示順】**
* 組織の人件費総額（`TOTAL_ORG_SALARY`）の降順。

![](https://static.zenn.studio/user-upload/a8759092a862-20260512.png =400x)


## 期待する結果
| MANAGER_NODE | INDENTED_EMP_NAME | OWN_SALARY | TOTAL_ORG_SALARY | TOTAL_SUBORDINATES | PCT_OF_TOTAL_ORG_SALARY | 
| ------------ | ----------------- | ---------- | ---------------- | ------------------ | ----------------------- | 
| 100          | Steven King       | 24000      | 691416           | 106                | 3.47                    | 
| 101          | Neena Yang        | 17000      | 109816           | 11                 | 15.48                   | 
| 145          | John Singh        | 14000      | 65000            | 6                  | 21.54                   | 
| 146          | Karen Partners    | 13500      | 64500            | 6                  | 20.93                   | 
| 148          | Gerald Cambrault  | 11000      | 62900            | 6                  | 17.49                   | 
| 149          | Eleni Zlotkey     | 10500      | 60500            | 6                  | 17.36                   | 
| 147          | Alberto Errazuriz | 12000      | 58600            | 6                  | 20.48                   | 
| 108          | Nancy Gruenberg   | 12008      | 51608            | 5                  | 23.27                   | 
| 102          | Lex Garcia        | 17000      | 45800            | 5                  | 37.12                   | 
| 121          | Adam Fripp        | 8200       | 33600            | 8                  | 24.4                    | 
| 123          | Shanta Vollman    | 6500       | 32400            | 8                  | 20.06                   | 
| 122          | Payam Kaufling    | 7900       | 31500            | 8                  | 25.08                   | 
| 120          | Matthew Weiss     | 8000       | 30100            | 8                  | 26.58                   | 
| 124          | Kevin Mourgos     | 5800       | 28800            | 8                  | 20.14                   | 
| 103          | Alexander James   | 9000       | 28800            | 4                  | 31.25                   | 
| 114          | Den Li            | 11000      | 24900            | 5                  | 44.18                   | 
| 205          | Shelley Higgins   | 12008      | 20308            | 1                  | 59.13                   | 
| 201          | Michael Martinez  | 13000      | 19000            | 1                  | 68.42                   | 

## 解答例
```sql
WITH orgtree (
    employee_id,
    manager_id,
    emp_name,
    salary,
    org_level,
    hierarchy_path
) AS (
    -- 1. ベースクエリ：トップマネジメント（社長など、マネージャーがいない従業員）
    SELECT
        employee_id,
        manager_id,
        first_name
        || ' '
        || last_name                                 AS emp_name,
        salary,
        1                                            AS org_level,
        CAST(TO_CHAR(employee_id) AS VARCHAR2(1000)) AS hierarchy_path
    FROM
        hr.employees
    WHERE
        manager_id IS NULL

    UNION ALL
    
    -- 2. 再帰クエリ：直属の部下をツリーに結合していく
    SELECT
        e.employee_id,
        e.manager_id,
        e.first_name
        || ' '
        || e.last_name,
        e.salary,
        t.org_level + 1,
        t.hierarchy_path
        || '->'
        || TO_CHAR(e.employee_id)
    FROM
             hr.employees e
        JOIN orgtree t ON e.manager_id = t.employee_id
), orgrollup AS (
    -- 3. 階層パス（hierarchy_path）を利用して、自身の配下（間接的な部下も含む）を特定し集計
    SELECT
        t1.employee_id            AS manager_node,
        t1.emp_name,
        t1.org_level,
        t1.salary                 AS own_salary,
        SUM(t2.salary)            AS total_org_salary,
        COUNT(t2.employee_id) - 1 AS total_subordinates
    FROM
             orgtree t1
        JOIN orgtree t2 ON t2.hierarchy_path LIKE t1.hierarchy_path || '%'
    GROUP BY
        t1.employee_id,
        t1.emp_name,
        t1.org_level,
        t1.salary
)
-- 4. 最終結果の整形（組織全体の給与コストが高い順）
SELECT
    manager_node,
    LPAD(' ',(org_level - 1) * 4)
    || emp_name                                   AS indented_emp_name,
    own_salary,
    total_org_salary,
    total_subordinates,
    ROUND(own_salary / total_org_salary * 100, 2) AS pct_of_total_org_salary
FROM
    orgrollup
WHERE
    total_subordinates > 0
ORDER BY
    total_org_salary DESC
```

## 解説
これまでの階層問い合わせは「上司と部下の関係を繋ぐ」だけでしたが、今回はその先の **「配下全員の集計（ロールアップ）」** という、実務のBIレポートや人件費シミュレーションで最も重宝される、かつ最も難易度の高いテクニックです。

### 1. 核心：なぜこの問題が「Lv4」なのか？
通常の `GROUP BY` は、同じ階層にあるデータをまとめるのは得意ですが、組織図のような「枝分かれした先のデータ」を全て足し合わせることはできません。

この問題を解くには、単にツリーを作るだけでなく、**「誰が誰の傘下にいるか」という家系図的な全履歴**を管理し、それを元に再集計するという2段構えのロジックが必要になります。

### 2. 解法のロジック解剖
解答例のクエリは、4つのフェーズで「組織の全貌」を解き明かしています。

#### ① `orgtree`：再帰によるパス（道筋）の構築
ここで最も重要なのは `hierarchy_path` です。
* **社長（100）**: パスは `"100"`
* **副社長（101）**: パスは `"100->101"`
* **マネージャー（108）**: パスは `"100->101->108"`

このように、**「自分がどの家系に属しているか」を文字列として保持**します。


#### ② `orgrollup`：全配下の「傘下」検索
ここが魔法のポイントです。
```sql
JOIN orgtree t2 ON t2.hierarchy_path LIKE t1.hierarchy_path || '%'
```
自分（`t1`）のパスが、相手（`t2`）のパスの「前方一致」であるかを確認しています。
例えば、`t1` が「101」の場合、パスが 「101...」 で始まる人（101自身と、その部下全員）を全てヒットさせ、その給与を合計しています。

#### ③ 数値計算と割合の算出
自身の報酬が組織全体でどれくらいのインパクトを持っているかを計算します。

$$\text{PCT\_OF\_TOTAL\_ORG\_SALARY} = \frac{\text{OWN\_SALARY}}{\text{TOTAL\_ORG\_SALARY}} \times 100$$


### 3. 実務での活用シーン：人件費の「深掘り」
このクエリが書けると、経営層に対して非常に付加価値の高いデータを提示できます。

* **部門別コストの可視化**: 特定の役員が率いる「一派」全員で、いくらのコストがかかっているか。
* **スパン・オブ・コントロール分析**: `TOTAL_SUBORDINATES` を見ることで、一人のマネージャーが何人の部下（間接含む）を抱え、管理コストが適正かを判断できます。

### 4. 期待する結果のポイント

結果のトップに君臨する **Steven King** さんを見てください。
* **TOTAL_ORG_SALARY**: 691,416（全従業員の給与総額）
* **TOTAL_SUBORDINATES**: 106（自分以外の全社員数）
* **PCT**: 3.47%（自身の給与は、組織全体の約3.5%を占める）

これが、階下の **Neena Yang** さんになると、彼女の「傘下」だけの集計に切り替わります。このように、**「視点を変えるだけで、その下の宇宙が全て再集計される」** のが、再帰SQLの真骨頂です。

> **パフォーマンスの注意点**
> 今回使用した `LIKE` による結合は直感的で分かりやすいですが、データが数万件を超えると非常に重くなります。その場合は、階層モデルを「入れ子集合モデル」などで管理する手法もありますが、数千件程度の組織図なら、この再帰パス方式が最もスマートです。


# 【完全版】問題14-4：【総合演習】カテゴリ別成長率と市場シェア分析

### 難易度：★★★★☆ (Lv.4)

## 問題
SH（売上履歴）スキーマのデータを用い、以下の4つの高度なステップを1つのストーリーとして完結させてください。

1. **データの土台作り**: カテゴリ×年別の売上を集計。
2. **形状変換（PIVOT）**: 昨対比較をしやすくするため、年を列に展開。
3. **高度な指標算出**:
   * **成長率**: 2020年から2021年にかけてどれだけ伸びたか。
   * **全体シェア**: 2021年の総売上に対して、各カテゴリがどれだけ貢献しているか。


4. **順位付けとフィルタリング**: 有益なデータ（成長しており、かつ一定のシェアがあるもの）のみを抽出し、ランキングを付与。


## 期待する結果
| CATEGORY          | S2020        | S2021        | GROWTH_RATE | SHARE_IN_TOTAL | GROWTH_RANK | 
| ----------------- | ------------ | ------------ | ----------- | -------------- | ----------- | 
| Tennis            | 2,916,369.92 | 5,200,605.88 | 78.32%      | 21.88%         | 1           | 
| Golf              | 3,781,243.27 | 4,336,456.40 | 14.68%      | 18.25%         | 2           | 
| Soccer / Football | 3,884,664.22 | 4,329,248.89 | 11.44%      | 18.22%         | 3           | 

## 解答例
```sql
WITH category_sales_raw AS (
    -- 【STEP 1】 基礎集計
    -- まずは「年」と「カテゴリ」という最小単位で売上を合計します。
    SELECT
        p.prod_category                  AS category,
        TO_CHAR(s.time_id, 'YYYY')       AS sales_year,
        SUM(s.amount_sold)               AS total_amount
    FROM
        sh.sales s
        INNER JOIN sh.products p ON s.prod_id = p.prod_id
    WHERE
        TO_CHAR(s.time_id, 'YYYY') IN ('2020', '2021')
    GROUP BY
        p.prod_category,
        TO_CHAR(s.time_id, 'YYYY')
),
category_pivoted AS (
    -- 【STEP 2】 PIVOT（形状変換）
    -- 2020年と2021年の値を「同じ行」に並べます。
    -- これで「Sales2021 - Sales2020」といった計算が簡単になります。
    SELECT *
    FROM category_sales_raw
    PIVOT (
        SUM(total_amount)
        FOR sales_year IN ('2020' AS sales_2020, '2021' AS sales_2021)
    )
),
analyzed_data AS (
    -- 【STEP 3】 数値分析
    -- ここで成長率とシェアを一気に計算します。
    SELECT
        category,
        sales_2020,
        sales_2021,
        -- 成長率の計算式：
        ROUND(
            (sales_2021 - sales_2020) / NULLIF(sales_2020, 0) * 100, 
            2
        ) AS growth_rate,
        -- 全体に対する2021年シェア（分析関数）
        ROUND(
            RATIO_TO_REPORT(sales_2021) OVER() * 100, 
            2
        ) AS share_in_total
    FROM
        category_pivoted
)
-- 【STEP 4】 最終フォーマットとランキング
-- 最後に、ビジネスで見やすい形に整え、フィルタリングをかけます。
SELECT
    category,
    TO_CHAR(sales_2020, '9,999,999.99') AS s2020,
    TO_CHAR(sales_2021, '9,999,999.99') AS s2021,
    growth_rate || '%'                  AS growth_rate,
    share_in_total || '%'               AS share_in_total,
    RANK() OVER(ORDER BY growth_rate DESC) AS growth_rank
FROM
    analyzed_data
WHERE
    growth_rate > 0           -- 成長しているカテゴリ
    AND share_in_total >= 0.1 -- 意味のあるシェア（0.1%以上）を持っている
ORDER BY
    growth_rank
```

## 解説
このクエリは、単なるデータの抽出を超えて、**「生の販売記録から、経営の意思決定に直結するインサイトを生成する」** という、データアナリストの真髄が詰まったクエリーです。

### 1. データの「種」をまく (`category_sales_raw`)
まずは、膨大な `SALES` テーブルから、必要な情報だけを削り出します。

* **役割**: 分析対象の「年（2020/2021）」と「カテゴリ」を特定し、最小単位の集計を行います。
* **ポイント**: `TO_CHAR(time_id, 'YYYY')` で日付から年を切り出し、グループ化の軸にしています。

### 2. 比較のために「形」を変える (`category_pivoted`)
ここが一つ目の山場、**PIVOT（横持ち変換）** です。

* **なぜ必要か？**: 元のデータは「2020年の行」と「2021年の行」が縦に並んでいます。しかし、成長率を計算するには「同じ行の隣同士」に数値がある必要があります。
* **魔法の瞬間**: 縦に並んでいた年度を列名に変換し、1カテゴリ1行の形にまとめました。

### 3. 「価値」を算出する (`analyzed_data`)
形が整ったら、分析関数の出番です。ここでは「物差し」を2つ作っています。

#### ① 成長率 (GROWTH_RATE)
比較可能な形になったおかげで、`(今期 - 前期) / 前期` というシンプルな計算が可能になりました。
> **💡 プロのこだわり**: `NULLIF(sales_2020, 0)` を使うことで、もし去年が売上ゼロだった場合に発生する「0除算エラー」をエレガントに回避しています。

#### ② 全体シェア (SHARE_IN_TOTAL)
ここで登場するのが、第12章の奥義 **`RATIO_TO_REPORT`** です。
* **役割**: 「2021年の全売上」という分母を瞬時に作り、各カテゴリの貢献度（％）を算出します。


### 4. 宝石だけを「選別」して磨く (最終SELECT)
最後に、経営陣に提出するための「最終フォーマット」に整えます。

* **絞り込み**: 「成長率 > 0（伸びている）」かつ「シェア >= 0.1%（無視できない規模）」というビジネスフィルタを適用します。
* **ランキング**: 成長率が高い順に `RANK()` を振ります。このランクは、**WHERE句で絞り込まれた後の精鋭たち**の中での順位になります。
* **装飾**: `TO_CHAR` でカンマ区切りにし、`|| '%'` で単位を付ける。この「見た目の配慮」が、レポートの信頼性を高めます。

### 期待する結果から見えるストーリー
結果を見ると、**Tennis** カテゴリが約78%という驚異的な成長率で1位に輝いています。

* **アナリストの視点**: 「成長率1位（Tennis）」だけでなく、2位の **Golf** や3位の **Soccer** もシェアが18%を超えており、これらは **「売上の柱でありながら、さらに伸びている超優良カテゴリ」** であることが一目でわかります。
