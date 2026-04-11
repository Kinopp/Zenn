---
title: "階層構造の取り扱い"
free: false
---

# 問題14-1（従業員と上司関係のツリー構造1） *Lv3*
### HRスキーマ`EMPLOYEES`テーブルを参照し、従業員と従業員の上司関係をツリー構造で取得してください。

* **「上司 <-- 従業員」で表現する**
* **上司に対して、さらに上司が存在する場合は、**
  **「その上の上司 <-- 上司 <-- 従業員」の形で表現する。**
  **（例）「100 <-- 101 <-- 108」**

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


<br><br>
----

# 【完全版】問題14-2（従業員と上司関係のツリー構造2） *Lv4*
### HRスキーマ`EMPLOYEES`テーブルを参照し、従業員と従業員の上司関係をツリー構造で取得してください。

* **最上位の従業員を「1」、その下位の従業員を「2」という形で氏名に番号を付与して表示する。**
* **階層を下げるごとに、氏名の前にスペースを2つ付与する**
  **（例）「1. Steven King」、「&nbsp;&nbsp;2. Neena Kochhar」、「&nbsp;&nbsp;&nbsp;&nbsp;3. Nancy Greenberg」**
* **深さ優先探索順でソートする**


### 期待する結果
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


### 解答例
- 再帰的with句を使用
```sql
with recursive_pr (
  employee_id,
  manager_id,
  first_name,
  last_name,
  lvl
) as (
  select
    e1.employee_id,
    e1.manager_id,
    e1.first_name,
    e1.last_name,
    1 as lvl
  from employees e1
  where
    e1.manager_id is null

  union all

  select
    e2.employee_id,
    e2.manager_id,
    e2.first_name,
    e2.last_name,
    rpr.lvl + 1
  from recursive_pr rpr
    inner join employees e2 
      on e2.manager_id = rpr.employee_id
) 
    search breadth first by manager_id set rpr_order
select
  manager_id,
  employee_id,
  lpad(' ', 2 *(lvl -1)) || lvl || '. ' || first_name || ' ' || last_name as employee_name
from recursive_pr
order by rpr_order
```

- 階層問合せ演算子を使用
```sql
select
  manager_id,
  employee_id,
  lpad(' ', 2 *(level -1)) || level || '. ' || first_name || ' ' || last_name as employee_name
from employees 
    start with manager_id is null 
    connect by prior employee_id = manager_id
```

### 解説


### 参考リンク
https://www.shift-the-oracle.com/sql/with.html#with-recursive


----
<br><br>


# 問題X-X（XXXXX）
- 1. 組織ツリーの完全走査と部門全体の給与ロールアップ（再帰CTE）
マネージャーと部下の関係（manager_id）をたどり、**「自分自身だけでなく、配下のすべての階層の従業員を含めた組織全体の人件費」**を計算するクエリです。階層の深さがどれだけあっても動的に計算します。

### 期待する結果
件数多い

### 解答例
```sql
WITH OrgTree (
        employee_id,
        manager_id,
        emp_name,
        salary,
        org_level,
        hierarchy_path
)AS (
    -- 1. ベースクエリ：トップマネジメント（社長など、マネージャーがいない従業員）
    SELECT
        employee_id,
        manager_id,
        first_name || ' ' || last_name AS emp_name,
        salary,
        1 AS org_level,
        CAST(TO_CHAR(employee_id) AS VARCHAR2(1000)) AS hierarchy_path
    FROM hr.employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- 2. 再帰クエリ：直属の部下をツリーに結合していく
    SELECT
        e.employee_id,
        e.manager_id,
        e.first_name || ' ' || e.last_name,
        e.salary,
        t.org_level + 1,
        t.hierarchy_path || '->' || TO_CHAR(e.employee_id)
    FROM hr.employees e
    JOIN OrgTree t ON e.manager_id = t.employee_id
),
OrgRollup AS (
    -- 3. 階層パス（hierarchy_path）を利用して、自身の配下（間接的な部下も含む）を特定し集計
    SELECT
        t1.employee_id AS manager_node,
        t1.emp_name,
        t1.org_level,
        t1.salary AS own_salary,
        SUM(t2.salary) AS total_org_salary,
        COUNT(t2.employee_id) - 1 AS total_subordinates
    FROM OrgTree t1
    JOIN OrgTree t2 ON t2.hierarchy_path LIKE t1.hierarchy_path || '%'
    GROUP BY t1.employee_id, t1.emp_name, t1.org_level, t1.salary
)
-- 4. 最終結果の整形（組織全体の給与コストが高い順）
SELECT
    manager_node,
    LPAD(' ', (org_level - 1) * 4) || emp_name AS indented_emp_name,
    own_salary,
    total_org_salary,
    total_subordinates,
    ROUND(own_salary / total_org_salary * 100, 2) AS pct_of_total_org_salary
FROM OrgRollup
ORDER BY total_org_salary DESC;
```

### 解説

----

