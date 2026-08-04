---
title: "第14章 階層構造の取り扱い（全7問）"
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

----
<br><br>

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

----
<br><br>

# 問題14-5：組織階層の逆引き（ボトムアップ探索とCONNECT_BY_ROOT）

### 難易度：★★★☆☆ (Lv.3)

## 問題
コンプライアンス部門および人事制度改革プロジェクトから、「各従業員から見た『直属の上司（親）』だけでなく、組織ラインを一番上まで遡った『最終承認権限者（ルート）』までの承認経路（ワークフロー）を可視化してほしい」との監査依頼がありました。

トップダウンの組織図を作るだけでなく、末端の従業員側を起点として、組織のトップ（社長）に向かって家系図を「遡る」ことで、各従業員が最終的にどの統括ラインに属しているかを一網打尽にするクエリを構築します。

HRスキーマ`employees` テーブルを参照し、**従業員ID（`EMPLOYEE_ID`）が 150 から 155 までのメンバー**を起点として、組織をボトムアップ（下から上）に探索し、以下の条件を満たす結果を取得してください。

**【抽出・編集ルール】**

1. **START_EMP_ID / START_EMP_NAME**：探索の起点となった従業員のIDと氏名（`FIRST_NAME` と `LAST_NAME` を半角スペースで結合）。
2. **CURRENT_LEVEL**：起点となった従業員から数えて、その行の上司が何階層上に位置するか（自分自身を1とする）。
3. **UPWARD_MANAGER_NAME**：その階層にいる上司の氏名。
4. **FINAL_LEADER_NAME**：その組織ラインの最上位に位置するボス（ルートノード：社長）の氏名。
5. **表示順**：起点となった従業員IDの昇順、次いで、組織を遡る順番（`CURRENT_LEVEL` の昇順）で表示してください。

## 期待する結果
| START_EMP_ID | START_EMP_NAME    | CURRENT_LEVEL | UPWARD_MANAGER_NAME | FINAL_LEADER_NAME | 
| ------------ | ----------------- | ------------- | ------------------- | ----------------- | 
| 150          | Sean Tucker       | 1             | Sean Tucker         | Steven King       | 
| 150          | Sean Tucker       | 2             | John Singh          | Steven King       | 
| 150          | Sean Tucker       | 3             | Steven King         | Steven King       | 
| 151          | David Bernstein   | 1             | David Bernstein     | Steven King       | 
| 151          | David Bernstein   | 2             | John Singh          | Steven King       | 
| 151          | David Bernstein   | 3             | Steven King         | Steven King       | 
| 152          | Peter Hall        | 1             | Peter Hall          | Steven King       | 
| 152          | Peter Hall        | 2             | John Singh          | Steven King       | 
| 152          | Peter Hall        | 3             | Steven King         | Steven King       | 
| 153          | Christopher Olsen | 1             | Christopher Olsen   | Steven King       | 
| 153          | Christopher Olsen | 2             | John Singh          | Steven King       | 
| 153          | Christopher Olsen | 3             | Steven King         | Steven King       | 
| 154          | Nanette Cambrault | 1             | Nanette Cambrault   | Steven King       | 
| 154          | Nanette Cambrault | 2             | John Singh          | Steven King       | 
| 154          | Nanette Cambrault | 3             | Steven King         | Steven King       | 
| 155          | Oliver Tuvault    | 1             | Oliver Tuvault      | Steven King       | 
| 155          | Oliver Tuvault    | 2             | John Singh          | Steven King       | 
| 155          | Oliver Tuvault    | 3             | Steven King         | Steven King       | 

## 解答例
```sql
SELECT
    CONNECT_BY_ROOT(employee_id) AS start_emp_id,
    CONNECT_BY_ROOT(first_name || ' ' || last_name) AS start_emp_name,
    LEVEL AS current_level,
    first_name || ' ' || last_name AS upward_manager_name,
    -- 最上位（ルート）の情報を動的に取得
    (
        SELECT first_name || ' ' || last_name 
        FROM hr.employees 
        WHERE manager_id IS NULL
    ) AS final_leader_name
FROM
    hr.employees
START WITH
    employee_id BETWEEN 150 AND 155
CONNECT BY
    employee_id = PRIOR manager_id
ORDER BY
    start_emp_id,
    current_level
```

## 解説
今回は実務で非常に重宝される **「ボトムアップ（逆引き）探索」** です。
通常の組織図は「社長 $\rightarrow$ 部長 $\rightarrow$ 課員」というトップダウン（上から下）で探索しますが、今回のポイントは「末端の社員を起点にして、社長に向かって組織ラインを駆け上がる」点にあります。
この問題をクリアするための構造と、仕込まれたテクニックを詳しく解説します。

### 1. スクリプトの解説：組織を駆け上がる「逆方向」のリンク
このクエリの魔法は、`CONNECT BY` 句の `PRIOR` の位置にあります。ここを理解すると、階層問い合わせがぐっと得意になります。

#### ボトムアップの心臓部

```sql
START WITH employee_id BETWEEN 150 AND 155
CONNECT BY employee_id = PRIOR manager_id
```

通常のトップダウン探索では `PRIOR employee_id = manager_id`（「1行前の社員ID」が「次の行の上司ID」であること）と書きますが、今回は逆です。

* **`PRIOR manager_id`**: 「1行前のデータにおける**上司のID**」を指します。
* **`employee_id = ...`**: 「次の行では、その上司IDを**社員IDとして**検索してね」という意味になります。

これにより、「自分 $\rightarrow$ 自分の上司 $\rightarrow$ 上司の上司……」という形で、家系図を遡るようにデータが芋づる式に抽出されていきます。

### 2. 文法・テクニックのポイント

#### ① `CONNECT_BY_ROOT` の面白い逆転現象

```sql
CONNECT_BY_ROOT(employee_id) AS start_emp_id
```

`CONNECT_BY_ROOT` 演算子は、その探索における「一番の根本（起点）」となった行のデータを全行にコピーする機能です。
トップダウン探索であれば「社長の名前」が取れる関数ですが、今回は**ボトムアップ（下から上）に探索しているため、起点である「末端の従業員（自分自身）」の情報がルートとして扱われます。** 結果として、期待する結果の通り「誰から始まった承認ラインのデータか」を綺麗に横並びに保持することができます。

#### ② `LEVEL` 擬似列の意味

通常、`LEVEL` は組織の深さ（社長が1、部長が2……）を表しますが、ボトムアップ探索では「自分自身から何階層上に位置するか」という、自分発信のステップ数（自分が1、直属の上司が2、その上が3……）に変化します。

#### ③ 最上位のボス（社長）をサブクエリで仕留める理由

今回の解答例では、最上位のボス（`FINAL_LEADER_NAME`）をスカラ・サブクエリで別途取得しています。

```sql
(SELECT ... FROM hr.employees WHERE manager_id IS NULL)
```

> **なぜ関数で一発で取れないの？**
> 本来、トップダウン探索であれば `CONNECT_BY_ROOT` を使えば一撃で社長が取れます。しかし今回は探索が逆方向（ボトムアップ）なため、関数だけでは「探索の終着点（一番上）」をスマートに全行へ配ることができません。そこで「上司がいない人（`manager_id IS NULL`）＝社長」という絶対的なルールをサブクエリでドッキングさせる手法をとっています。非常に現実的で賢いワークフローです。

### 3. 実務での活用シーン：ワークフローとBOM（部品表）

この逆引きのテクニックは、一般的なデータ抽出だけでなく、システムのバックエンドロジックで極めて重要な役割を果たします。

* **ワークフロー・電子決裁システム**:
「金額が100万円を超えたから、申請者（起点）から社長（ルート）に到達するまでの全承認者のリストを動的に生成して経路テーブルに一括挿入する」という処理。
* **製造業の部品構成表（BOM: Bill of Materials）逆引き**:
「ある特定のネジ（末端コンポーネント）」に欠陥が見つかった際、そのネジが**最終的にどの完成品（車や家電などのルート）に組み込まれているか**を芋づる式に特定し、リコール対象を絞り込む。
* **Webサイトのパンくずリスト**:
「現在の詳細記事ページ（起点）」から、所属する「中カテゴリ」「大カテゴリ」「トップページ（ルート）」へと遡り、パンくずリスト（`ホーム > グルメ > ラーメン > 店舗詳細`）のリンクを動的に生成する。


----
<br><br>

# 問題14-6：階層を維持した兄弟間のソート（ORDER SIBLINGS BY）

### 難易度：★★★☆☆ (Lv.3)

## 問題
経営層から「全社の組織図をツリー形式で出力してほしい」との要望がありました。ただし、単に階層化するだけでなく、「同じ上司を持つ部下（同一部門・同一チームのメンバー）の中では、**給与（`SALARY`）が高い順**に並べて表示してほしい」という追加の条件が提示されました。

階層の親子関係（誰が誰の部下かという枝分かれ）を一切崩すことなく、同じ親を持つ「兄弟間」だけで動的にソートを制御するクエリを構築してください。

HRスキーマ`employees` テーブルを参照し、社長（`MANAGER_ID` が `NULL` の人物）をルートとして、トップダウンの組織ツリーを取得してください。

出力にあたっては以下の条件を満たしてください。

1. **EMPLOYEE_NAME**：階層レベルに応じて、氏名の前に **`-`を2つずつ** インデントとして付与し、先頭に「レベル番号. 」を付けて表示する（例：`1. Steven King`、`--2. Neena Kochhar`）。氏名は `FIRST_NAME` と `LAST_NAME` を半角スペースで結合すること。
2. **SALARY**：各従業員の給与を表示する。
3. **ソート順**：全体のツリー構造を維持したまま、同じ上司を持つ従業員同士（兄弟ノード）の間でのみ、給与の降順（高い順）で並び替えること。給与が同額の場合は、`EMPLOYEE_ID` の昇順とする。


## 期待する結果
| MANAGER_ID | EMPLOYEE_ID | EMPLOYEE_NAME              | SALARY | 
| ---------- | ----------- | -------------------------- | ------ | 
|            | 100         | 1. Steven King             | 24000  | 
| 100        | 101         | --2. Neena Yang            | 17000  | 
| 101        | 108         | ----3. Nancy Gruenberg     | 12008  | 
| 108        | 109         | ------4. Daniel Faviet     | 9000   | 
| 108        | 110         | ------4. John Chen         | 8200   | 
| 108        | 112         | ------4. Jose Manuel Urman | 7800   | 
| 108        | 111         | ------4. Ismael Sciarra    | 7700   | 
| 108        | 113         | ------4. Luis Popp         | 6900   | 
| 101        | 205         | ----3. Shelley Higgins     | 12008  | 
| 205        | 206         | ------4. William Gietz     | 8300   | 
| 101        | 204         | ----3. Hermann Brown       | 10000  | 
| 101        | 203         | ----3. Susan Jacobs        | 6500   | 
| 101        | 200         | ----3. Jennifer Whalen     | 4400   | 
| 100        | 102         | --2. Lex Garcia            | 17000  | 
| 102        | 103         | ----3. Alexander James     | 9000   | 
| 103        | 104         | ------4. Bruce Miller      | 6000   | 
| 103        | 105         | ------4. David Williams    | 4800   | 
| 103        | 106         | ------4. Valli Jackson     | 4800   | 
| 103        | 107         | ------4. Diana Nguyen      | 4200   | 
| 100        | 145         | --2. John Singh            | 14000  | 
| 145        | 150         | ----3. Sean Tucker         | 10000  | 
| 145        | 151         | ----3. David Bernstein     | 9500   | 
| 145        | 152         | ----3. Peter Hall          | 9000   | 
| 145        | 153         | ----3. Christopher Olsen   | 8000   | 
| 145        | 154         | ----3. Nanette Cambrault   | 7500   | 
| 145        | 155         | ----3. Oliver Tuvault      | 7000   | 
| 100        | 146         | --2. Karen Partners        | 13500  | 
| 146        | 156         | ----3. Janette King        | 10000  | 
| 146        | 157         | ----3. Patrick Sully       | 9500   | 
| 146        | 158         | ----3. Allan McEwen        | 9000   | 
| 146        | 159         | ----3. Lindsey Smith       | 8000   | 
| 146        | 160         | ----3. Louise Doran        | 7500   | 
| 146        | 161         | ----3. Sarath Sewall       | 7000   | 
| 100        | 201         | --2. Michael Martinez      | 13000  | 
| 201        | 202         | ----3. Pat Davis           | 6000   | 
| 100        | 147         | --2. Alberto Errazuriz     | 12000  | 
| 147        | 162         | ----3. Clara Vishney       | 10500  | 
| 147        | 163         | ----3. Danielle Greene     | 9500   | 
| 147        | 164         | ----3. Mattea Marvins      | 7200   | 
| 147        | 165         | ----3. David Lee           | 6800   | 
| 147        | 166         | ----3. Sundar Ande         | 6400   | 
| 147        | 167         | ----3. Amit Banda          | 6200   | 
| 100        | 114         | --2. Den Li                | 11000  | 
| 114        | 115         | ----3. Alexander Khoo      | 3100   | 
| 114        | 116         | ----3. Shelli Baida        | 2900   | 
| 114        | 117         | ----3. Sigal Tobias        | 2800   | 
| 114        | 118         | ----3. Guy Himuro          | 2600   | 
| 114        | 119         | ----3. Karen Colmenares    | 2500   | 
| 100        | 148         | --2. Gerald Cambrault      | 11000  | 
| 148        | 168         | ----3. Lisa Ozer           | 11500  | 
| 148        | 169         | ----3. Harrison Bloom      | 10000  | 
| 148        | 170         | ----3. Tayler Fox          | 9600   | 
| 148        | 171         | ----3. William Smith       | 7400   | 
| 148        | 172         | ----3. Elizabeth Bates     | 7300   | 
| 148        | 173         | ----3. Sundita Kumar       | 6100   | 
| 100        | 149         | --2. Eleni Zlotkey         | 10500  | 
| 149        | 174         | ----3. Ellen Abel          | 11000  | 
| 149        | 175         | ----3. Alyssa Hutton       | 8800   | 
| 149        | 176         | ----3. Jonathon Taylor     | 8600   | 
| 149        | 177         | ----3. Jack Livingston     | 8400   | 
| 149        | 178         | ----3. Kimberely Grant     | 7000   | 
| 149        | 179         | ----3. Charles Johnson     | 6200   | 
| 100        | 121         | --2. Adam Fripp            | 8200   | 
| 121        | 184         | ----3. Nandita Sarchand    | 4200   | 
| 121        | 185         | ----3. Alexis Bull         | 4100   | 
| 121        | 186         | ----3. Julia Dellinger     | 3400   | 
| 121        | 129         | ----3. Laura Bissot        | 3300   | 
| 121        | 187         | ----3. Anthony Cabrio      | 3000   | 
| 121        | 130         | ----3. Mozhe Atkinson      | 2800   | 
| 121        | 131         | ----3. James Marlow        | 2500   | 
| 121        | 132         | ----3. TJ Olson            | 2100   | 
| 100        | 120         | --2. Matthew Weiss         | 8000   | 
| 120        | 125         | ----3. Julia Nayer         | 3200   | 
| 120        | 180         | ----3. Winston Taylor      | 3200   | 
| 120        | 181         | ----3. Jean Fleaur         | 3100   | 
| 120        | 183         | ----3. Girard Geoni        | 2800   | 
| 120        | 126         | ----3. Irene Mikkilineni   | 2700   | 
| 120        | 182         | ----3. Martha Sullivan     | 2500   | 
| 120        | 127         | ----3. James Landry        | 2400   | 
| 120        | 128         | ----3. Steven Markle       | 2200   | 
| 100        | 122         | --2. Payam Kaufling        | 7900   | 
| 122        | 188         | ----3. Kelly Chung         | 3800   | 
| 122        | 189         | ----3. Jennifer Dilly      | 3600   | 
| 122        | 133         | ----3. Jason Mallin        | 3300   | 
| 122        | 134         | ----3. Michael Rogers      | 2900   | 
| 122        | 190         | ----3. Timothy Venzl       | 2900   | 
| 122        | 191         | ----3. Randall Perkins     | 2500   | 
| 122        | 135         | ----3. Ki Gee              | 2400   | 
| 122        | 136         | ----3. Hazel Philtanker    | 2200   | 
| 100        | 123         | --2. Shanta Vollman        | 6500   | 
| 123        | 192         | ----3. Sarah Bell          | 4000   | 
| 123        | 193         | ----3. Britney Everett     | 3900   | 
| 123        | 137         | ----3. Renske Ladwig       | 3600   | 
| 123        | 138         | ----3. Stephen Stiles      | 3200   | 
| 123        | 194         | ----3. Samuel McLeod       | 3200   | 
| 123        | 195         | ----3. Vance Jones         | 2800   | 
| 123        | 139         | ----3. John Seo            | 2700   | 
| 123        | 140         | ----3. Joshua Patel        | 2500   | 
| 100        | 124         | --2. Kevin Mourgos         | 5800   | 
| 124        | 141         | ----3. Trenna Rajs         | 3500   | 
| 124        | 142         | ----3. Curtis Davies       | 3100   | 
| 124        | 196         | ----3. Alana Walsh         | 3100   | 
| 124        | 197         | ----3. Kevin Feeney        | 3000   | 
| 124        | 143         | ----3. Randall Matos       | 2600   | 
| 124        | 198         | ----3. Donald OConnell     | 2600   | 
| 124        | 199         | ----3. Douglas Grant       | 2600   | 
| 124        | 144         | ----3. Peter Vargas        | 2500   | 
## 解答例
```sql
SELECT
    manager_id,
    employee_id,
    LPAD('-', 2 * (LEVEL - 1), '-') || LEVEL || '. ' || first_name || ' ' || last_name AS employee_name,
    salary
FROM
    hr.employees
START WITH
    manager_id IS NULL
CONNECT BY
    PRIOR employee_id = manager_id
-- 階層構造を壊さずに兄弟間だけでソートする魔法のキーワード
ORDER SIBLINGS BY
    salary DESC,
    employee_id ASC
```

## 解説
階層問い合わせ（`CONNECT BY`）の強力な応用技、`ORDER SIBLINGS BY`（兄弟間のソート）です。
ツリー構造をSQLで扱う際、多くのエンジニアが一度は通る致命的なトラップがあります。それは、「ツリーを出力したあとに、普通の `ORDER BY` でソートをかけると、せっかくの階層構造（親子関係）が完全にバラバラに破壊されてしまう」という問題です。
今回の解答例は、その崩壊を完璧に防ぎつつ、同じ親を持つ「兄弟（シブリング）」の間だけで綺麗に給与順に並び替える、実務のレポート作成で必須のテクニックです。ポイントを絞って解説します。

### 1. スクリプトの解説：ツリーを壊さない「内なるソート」
このクエリの最大の主役は、もちろん **`ORDER SIBLINGS BY`** 句です。

#### 通常の ORDER BY との違い

* **`ORDER BY salary DESC` を使った場合（失敗）**:
ツリー構造（誰が誰の部下か）をすべて無視して、全社員が単純に給与の高い順に並び替わります。社長の Steven King の下に、突然別ラインの部長や課員が混ざり合い、組織図としては全く意味をなさなくなってしまいます。
* **`ORDER SIBLINGS BY salary DESC` を使った場合（成功）**:
「 Steven King の直属の部下（レベル2のメンバー）」という**同じ親を持つ兄弟の集まりの中だけで**、給与の高い順（Neena $\rightarrow$ Lex $\rightarrow$ John ...）にソートされます。そして、それぞれの部下の配下（レベル3のメンバー）に移動した際にも、やはりその兄弟間だけで給与順にソートされます。

ツリーの「枝分かれの形」をしっかりとキープしたまま、各階層の並び順だけをコントロールできる唯一無二のコマンドです。

### 2. 文法・テクニックのポイント：美しいインデントの作り方
今回の `EMPLOYEE_NAME` 列では、視覚的に組織の深さがひと目で伝わるような文字列結合を行っています。

```sql
LPAD('-', 2 * (LEVEL - 1), '-') || LEVEL || '. ' || first_name || ' ' || last_name
```

この部分を分解して見てみましょう。

#### ① `LPAD` による動的インデント

`LPAD(文字列, 総文字数, 埋める文字)` は、指定した文字数になるまで左側を特定の文字で埋める関数です。

* **レベル1（社長）の場合**: `2 * (1 - 1) = 0` 文字になるため、ハイフンは **0個** になります。
* **レベル2（副社長など）の場合**: `2 * (2 - 1) = 2` 文字になるため、左側に **`--`** がきれいに付与されます。
* **レベル3（マネージャーなど）の場合**: `2 * (3 - 1) = 4` 文字になり、**`----`** となってさらに深くインデントされます。

#### ② 階層番号のドッキング

`|| LEVEL || '. '` を後ろに繋げることで、期待する結果の通り `--2. Neena Yang` や `----3. Nancy Gruenberg` といった、**現在の階層深さが数字でも視覚（ハイフンの量）でも一発でわかる**、極めて洗練されたテキストが完成します。

### 3. 実務での活用シーン：構造を保った並び替え

この `ORDER SIBLINGS BY` は、以下のような「ツリー型の画面表示」のバックエンドで大活躍します。

* **ファイルシステムのディレクトリツリー**:
エクスプローラーのように、フォルダの階層構造は絶対に崩さないまま、同じフォルダ内にあるファイルや子フォルダだけを「名前順」や「最終更新日順」「ファイルサイズ順」で並び替える処理。
* **Webサイトのコメント欄（スレッド表示）**:
ある投稿に対する「親コメント」と「返信（子コメント）」のツリー構造を維持したまま、同じ親に対する返信同士（兄弟）だけを「タイムスタンプの古い順（時系列）」や「いいね！の多い順」にソートするロジック。
* **ECサイトの商品カテゴリマスタ**:
`大カテゴリ > 中カテゴリ > 小カテゴリ` の親子関係は保ったまま、同じ中カテゴリに属する小カテゴリ同士を「表示順の優先度（マスタのソート定義）」に従って並べる処理。

----
<br><br>

# 【完全版】問題14-7：組織データの整合性チェック（リーフ判定と循環参照の検出）

### 難易度：★★★★☆ (Lv.4)

## 問題
データ移行やシステムの統合、あるいは手動での人事マスター更新の際、「Aの上司がBで、Bの上司がCで、Cの上司がA」といった不正な「循環参照（無限ループ）」がデータに紛れ込んでしまうことがあります。
この状態のデータに対して通常の階層問い合わせを実行すると、データベースは無限ループに陥り、最悪の場合システム停止（クラッシュ）を招きます。

コンプライアンスおよびデータガバナンスの観点から、システムを安全に保護しながら不正なループデータをピンポイントで検出し、同時に「これ以上部下がいない末端の従業員（リーフ）」を特定するための、高度なデータ整合性チェッククエリを構築してください。

HRスキーマの `EMPLOYEES` テーブルの一部のデータをベースとし、インラインビュー（WITH句）を用いて**擬似的に循環参照（100番の上司が102番、102番の上司が100番）を発生させたデータソース**を使用します。

```sql
-- 本問題の検証用データソース（クエリの先頭に配置します）
WITH test_employees AS (
    SELECT 
        employee_id, 
        -- 100番（本来の社長）のマネージャーを敢えて102番に書き換えてループを偽装
        CASE WHEN employee_id = 100 THEN 102 ELSE manager_id END AS manager_id, 
        first_name || ' ' || last_name AS name 
    FROM hr.employees 
    WHERE employee_id IN (100, 101, 102, 103, 104)
)
```

上記の `test_employees` をデータソースとして参照し、`employee_id = 100` を開始地点（`START WITH`）とするトップダウンの階層問い合わせを作成してください。

出力にあたっては以下の条件を満たしてください。

1. **INDENTED_NAME**：階層レベル（`LEVEL`）に応じて、氏名の前に**「-」を2つずつ**インデントとして付与して表示する。
2. **IS_LEAF**：その従業員に部下がいない（末端ノードである）場合は `1`、部下がいる場合は `0` を出力する。
3. **IS_LOOP**：その従業員のデータが原因で**循環参照（ループ）が発生している場合は `1**`、それ以外は `0` を出力する。
4. データ内に循環参照が含まれていても、**エラー（ORA-01436）を出さずに安全にクエリを完結させる**制御を行うこと。

## 期待する結果
| EMPLOYEE_ID | INDENTED_NAME       | LEVEL | IS_LEAF | IS_LOOP | 
| ----------- | ------------------- | ----- | ------- | ------- | 
| 100         | Steven King         | 1     | 0       | 0       | 
| 101         | --Neena Yang        | 2     | 1       | 0       | 
| 102         | --Lex Garcia        | 2     | 0       | 1       | 
| 103         | ----Alexander James | 3     | 0       | 0       | 
| 104         | ------Bruce Miller  | 4     | 1       | 0       | 

## 解答例
```sql
WITH test_employees AS (
    SELECT 
        employee_id, 
        CASE WHEN employee_id = 100 THEN 102 ELSE manager_id END AS manager_id, 
        first_name || ' ' || last_name AS name 
    FROM hr.employees 
    WHERE employee_id IN (100, 101, 102, 103, 104)
)
SELECT
    employee_id,
    LPAD('-', 2 * (LEVEL - 1), '-') || name AS indented_name,
    LEVEL,
    -- 末端ノード（葉）を判定する擬似列
    CONNECT_BY_ISLEAF  AS is_leaf,
    -- 循環参照の発生源を特定する擬似列
    CONNECT_BY_ISCYCLE AS is_loop
FROM
    test_employees
START WITH
    employee_id = 100
-- NOCYCLEキーワードにより無限ループエラーを回避
CONNECT BY NOCYCLE
    PRIOR employee_id = manager_id
```

## 解説
今回は階層問い合わせ（`CONNECT BY`）の最高峰とも言える「循環参照（無限ループ）の検知とデータクリーニング」です。
実務で手動の人事マスター変更や、他システムからのデータ移行を繰り返していると、「Aの上司がB、Bの上司がC、Cの上司がA」といったゴーストデータ（循環参照）がどうしても数件紛れ込んでしまうことがあります。
これを知らずに通常のクエリでツリー化しようとすると、データベースが無限ループに陥り、エラー（`ORA-01436`）でシステムが停止してしまいます。今回の解答例は、データベースを安全に保護しながら、そのバグの発生源をピンポイントで炙り出すプロ必須の防衛テクニックです。

### 1. スクリプトの解説：無限ループの恐怖と「安全弁」

このクエリの最も重要なポイントは、`CONNECT BY` 句の直後に仕込まれた **`NOCYCLE`（ノーサイクル）** というキーワードです。

#### NOCYCLE がない場合の悲劇

今回の検証データでは、社長である 100番（Steven）のマネージャーが 102番（Lex）に設定されています。一方で、102番のマネージャーは 100番です。

通常の `CONNECT BY` だと、`100 → 102 → 100 → 102 → ...` と永遠にぐるぐると巡回し続け、最終的にデータベースが「これ以上は危険だ！」と判断して `ORA-01436` エラーを吐いてクエリを強制終了させてしまいます。

#### NOCYCLE という安全弁

キーワード `CONNECT BY NOCYCLE` と記述しておくと、Oracleは内部で「一度通ったルート（履歴）」を記憶しながら探索します。
そして、「あ、この先に進むと過去に通った行（100番）に戻ってしまうな」と察知した瞬間に、**エラーを出さずに、その枝の探索を安全に打ち切ってくれる**のです。


### 2. 文法・テクニックのポイント：データを見分ける2つの疑似列

安全にクエリが完結したら、次は「どこにバグがあるか」を人間が見つけなければなりません。そこで大活躍するのが、Oracleが提供する2つの強力な疑似列です。

#### ① `CONNECT_BY_ISCYCLE`（ループの戦犯を特定）

「この行の次に進むと、過去のデータに戻ってループが完成してしまう」という、まさに**ループの引き金（発生源）になっている行にだけ `1**` を立ててくれます。

期待する結果の **`EMPLOYEE_ID = 102`** を見てください。

```text
100（Steven） → 102（Lex） ──(次に行くと100に戻る！)──> [ループ発生]
```

102番の行で `IS_LOOP` が `1` になっていますね。これを見つけるだけで、管理者は「102番のマネージャー設定（または100番のマネージャー設定）が間違っている！」と一瞬で原因を特定できます。

#### ② `CONNECT_BY_ISLEAF`（組織の末端を特定）

ツリー構造において、それより下に部下が一人もいない「末端の行（葉：リーフ）」である場合に **`1`** を、下にまだ部下がぶら下がっている場合は **`0`** を返します。

* **101番（Neena）**: 下に部下がいないので `1`（リーフ）。
* **104番（Bruce）**: 彼がこのラインの最末端なので `1`（リーフ）。

> **102番（Lex）のリーフ判定に注目**
> 本来のデータでは 102番の下には 103番や104番が続いているため、`IS_LEAF` は `0` になります。もし `NOCYCLE` がなければ、この102番から下の安全なメンバー（103, 104）のデータごとエラーで吹き飛んで見えなくなってしまいます。`NOCYCLE` を使うことで、バグを抱えつつも、その配下のメンバーまで正しくタイムラインを展開できているのがこの機能の素晴らしいところです。

### 3. 実務での活用シーン：データガバナンスとクリーニング

この循環参照の検知ロジックは、以下のような「親子のネットワークデータ」を扱うシステムの裏側で、データの健全性を保つための「監査バッチ」として頻繁に組み込まれます。

* **人事・組織マスターの夜間チェック**:
「毎日深夜にこのクエリを走らせ、`IS_LOOP = 1` の行が1件でも見つかったら、人事担当者に『データに不正なループがあります』とアラートメールを飛ばす」という自動運用。
* **製造業の部品構成表（BOM）の整合性チェック**:
「製品 A の組み立てには部品 B が必要で、部品 B の製造には部品 C が必要で、部品 C の材料に製品 A が指定されている」といった、**設計段階のあり得ないマスタ登録バグ**を検知する。
* **SNSのデッドロック・配送経路の循環検知**:
配送ネットワークや、データが特定のハブをぐるぐる回って目的地に届かないようなルート不備の自動検出。