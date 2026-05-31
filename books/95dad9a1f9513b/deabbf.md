---
title: "第12章 分析関数②（全0問）"
free: false
---

# 【完全版】問題12-15：基準値との比較（FIRST_VALUE / LAST_VALUE）

### 難易度：★★★☆☆ (Lv.3)

## 問題
人事部より、部門内の給与格差に関する分析レポートの依頼がありました。
各部門において、「その部門で最も早く採用された従業員（最初に入社した人）」の給与を基準値（ベースライン）とし、他の従業員との給与差を可視化したいと考えています。

これにより、勤続年数と給与の相関関係や、初期メンバーと後発メンバーの待遇差を調査します。

HRスキーマの`EMPLOYEES` テーブルより、部門ID（`DEPARTMENT_ID`）が `60`（IT）および `90`（Executive）の従業員を対象として、以下の情報を取得してください。

**【取得項目】**

1. **`DEPARTMENT_ID`**: 部門ID
2. **`FIRST_NAME`**: 名前
3. **`HIRE_DATE`**: 採用日
4. **`SALARY`**: 給与
5. **`BASE_SALARY`**: その部門で **最も早く採用された人** の給与（`FIRST_VALUE` を使用）
6. **`DIFF_BASE`**: 基準給与との差分（`SALARY` - `BASE_SALARY`）

**【抽出・算出ルール】**

* 部門（`PARTITION BY department_id`）ごとに計算してください。
* 「最も早く採用された」の判定は、採用日（`HIRE_DATE`）の昇順とします。
* 同日に採用された人が複数いる場合は、`EMPLOYEE_ID` が小さい方を優先してください。
* 結果は、部門IDの昇順、次に採用日の昇順で表示してください。

## 期待する結果
| DEPARTMENT_ID | FIRST_NAME | HIRE_DATE            | SALARY | BASE_SALARY | DIFF_BASE | 
| ------------- | ---------- | -------------------- | ------ | ----------- | --------- | 
| 60            | David      | 2015-06-25T00:00:00Z | 4800   | 4800        | 0         | 
| 60            | Alexander  | 2016-01-03T00:00:00Z | 9000   | 4800        | 4200      | 
| 60            | Valli      | 2016-02-05T00:00:00Z | 4800   | 4800        | 0         | 
| 60            | Diana      | 2017-02-07T00:00:00Z | 4200   | 4800        | -600      | 
| 60            | Bruce      | 2017-05-21T00:00:00Z | 6000   | 4800        | 1200      | 
| 90            | Lex        | 2011-01-13T00:00:00Z | 17000  | 17000       | 0         | 
| 90            | Steven     | 2013-06-17T00:00:00Z | 24000  | 17000       | 7000      | 
| 90            | Neena      | 2015-09-21T00:00:00Z | 17000  | 17000       | 0         | 


## 解答例、解説
[![](https://static.zenn.studio/user-upload/d958a6990064-20260508.png)](https://zenn.dev/kinopp/books/0b24d659785f31)

----
<br><br>

# 【完全版】問題12-16：分析関数版 LISTAGG（明細へのリスト付与）

### 難易度：★★★★☆ (Lv.4)

## 問題
注文管理システム（OEスキーマ）において、カスタマーサポート向けの注文明細照会画面を作成しています。

担当者は、ある注文（`ORDER_ID`）に含まれる各商品（`PRODUCT_ID`）の詳細を確認しながら、同時に「その注文の中に他にどんな商品が一緒に買われているか」をひと目で把握したいと考えています。

行を1行に集約してしまうのではなく、全明細行を表示したまま、その横に「注文内全商品リスト」を添えて出力してください。

OEスキーマの **`ORDER_ITEMS`** および **`PRODUCT_INFORMATION`** テーブルを使用し、`ORDER_ID` が `2355` および `2374` のデータを対象として、以下の情報を取得してください。

**【取得項目】**

1. **`ORDER_ID`**: 注文ID
2. **`PRODUCT_ID`**: 商品ID
3. **`PRODUCT_NAME`**: 商品名
4. **`ALL_ITEMS_IN_ORDER`**: **その注文に含まれる全商品のリスト**

**【抽出・算出ルール】**
* `ALL_ITEMS_IN_ORDER` は、注文（`PARTITION BY order_id`）ごとに商品名（`PRODUCT_NAME`）をカンマ＋スペース（`', '`）で区切って連結してください。
* リスト内の商品名の並び順は、商品名のアルファベット昇順（`WITHIN GROUP (ORDER BY product_name)`）としてください。
* 結果は、注文IDの昇順、次に商品名の昇順で表示してください。

## 期待する結果
| ORDER_ID | PRODUCT_ID | PRODUCT_NAME            | ALL_ITEMS_IN_ORDER                                                                                                                                    | 
| -------- | ---------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | 
| 2355     | 2289       | KB 101/ES               | KB 101/ES, LCD Monitor 9/PM, PS 220V /L, Paper - Std Printer, Plastic Stock - R, Plastic Stock - Y, Screws <B.32.P>, Screws <Z.28.P>, Video Card /E32 | 
| 2355     | 2359       | LCD Monitor 9/PM        | KB 101/ES, LCD Monitor 9/PM, PS 220V /L, Paper - Std Printer, Plastic Stock - R, Plastic Stock - Y, Screws <B.32.P>, Screws <Z.28.P>, Video Card /E32 | 
| 2355     | 2311       | PS 220V /L              | KB 101/ES, LCD Monitor 9/PM, PS 220V /L, Paper - Std Printer, Plastic Stock - R, Plastic Stock - Y, Screws <B.32.P>, Screws <Z.28.P>, Video Card /E32 | 
| 2355     | 2339       | Paper - Std Printer     | KB 101/ES, LCD Monitor 9/PM, PS 220V /L, Paper - Std Printer, Plastic Stock - R, Plastic Stock - Y, Screws <B.32.P>, Screws <Z.28.P>, Video Card /E32 | 
| 2355     | 2330       | Plastic Stock - R       | KB 101/ES, LCD Monitor 9/PM, PS 220V /L, Paper - Std Printer, Plastic Stock - R, Plastic Stock - Y, Screws <B.32.P>, Screws <Z.28.P>, Video Card /E32 | 
| 2355     | 2326       | Plastic Stock - Y       | KB 101/ES, LCD Monitor 9/PM, PS 220V /L, Paper - Std Printer, Plastic Stock - R, Plastic Stock - Y, Screws <B.32.P>, Screws <Z.28.P>, Video Card /E32 | 
| 2355     | 2323       | Screws <B.32.P>         | KB 101/ES, LCD Monitor 9/PM, PS 220V /L, Paper - Std Printer, Plastic Stock - R, Plastic Stock - Y, Screws <B.32.P>, Screws <Z.28.P>, Video Card /E32 | 
| 2355     | 2322       | Screws <Z.28.P>         | KB 101/ES, LCD Monitor 9/PM, PS 220V /L, Paper - Std Printer, Plastic Stock - R, Plastic Stock - Y, Screws <B.32.P>, Screws <Z.28.P>, Video Card /E32 | 
| 2355     | 2308       | Video Card /E32         | KB 101/ES, LCD Monitor 9/PM, PS 220V /L, Paper - Std Printer, Plastic Stock - R, Plastic Stock - Y, Screws <B.32.P>, Screws <Z.28.P>, Video Card /E32 | 
| 2374     | 2423       | C for SPNIX4.0 - 1 Seat | C for SPNIX4.0 - 1 Seat, OSI 1-4/IL, SPNIX4.0 - SAL, SPNIX4.0 - UL/D                                                                                  | 
| 2374     | 2449       | OSI 1-4/IL              | C for SPNIX4.0 - 1 Seat, OSI 1-4/IL, SPNIX4.0 - SAL, SPNIX4.0 - UL/D                                                                                  | 
| 2374     | 2422       | SPNIX4.0 - SAL          | C for SPNIX4.0 - 1 Seat, OSI 1-4/IL, SPNIX4.0 - SAL, SPNIX4.0 - UL/D                                                                                  | 
| 2374     | 2467       | SPNIX4.0 - UL/D         | C for SPNIX4.0 - 1 Seat, OSI 1-4/IL, SPNIX4.0 - SAL, SPNIX4.0 - UL/D                                                                                  | 

## 解答例、解説
[![](https://static.zenn.studio/user-upload/d958a6990064-20260508.png)](https://zenn.dev/kinopp/books/0b24d659785f31)

----
<br><br>

# 【完全版】問題12-17：統計的分布（CUME_DIST / PERCENT_RANK）

### 難易度：★★★☆☆ (Lv.3)

## 問題
マーケティング部門では、製品の価格戦略を検討するために、各製品カテゴリ内における製品価格の「相対的なポジション」を分析しています。

単なる順位（1位、2位…）ではなく、「その製品よりも高い製品がどれくらい存在するか」という割合を数値化することで、高級ラインの製品（上位10%など）を動的に特定したいと考えています。

SHスキーマの`PRODUCTS`テーブルより、製品カテゴリ（`PROD_CATEGORY_ID`）が `205`（Electronics）の製品を対象として、以下の情報を取得してください。

**【取得項目】**

1. **`PROD_ID`**: 製品ID
2. **`PROD_NAME`**: 製品名
3. **`PROD_LIST_PRICE`**: 定価
4. **`CUME_DIST_VAL`**: **累積分布**
5. **`PERCENT_RANK_VAL`**: **パーセンタイル順位**

**【抽出・算出ルール】**

* 価格（`PROD_LIST_PRICE`）の**降順**（高い順）で並べた際の分布を計算してください。
* 計算結果は小数点第4位を四捨五入して表示してください。
* 結果は、定価の降順で表示してください。

## 期待する結果
| PROD_ID | PROD_NAME                       | PROD_LIST_PRICE | CUME_DIST_VAL | PERCENT_RANK_VAL | 
| ------- | ------------------------------- | --------------- | ------------- | ---------------- | 
| 28      | English Willow Cricket Bat      | 199.99          | 0.0385        | 0                | 
| 19      | Cricket Bat Bag                 | 55.99           | 0.0769        | 0.04             | 
| 123     | Helmet                          | 49.99           | 0.1154        | 0.08             | 
| 40      | Team shirt                      | 44.99           | 0.3462        | 0.12             | 
| 44      | Team shirt                      | 44.99           | 0.3462        | 0.12             | 
| 45      | Team shirt                      | 44.99           | 0.3462        | 0.12             | 
| 41      | Team shirt                      | 44.99           | 0.3462        | 0.12             | 
| 42      | Team shirt                      | 44.99           | 0.3462        | 0.12             | 
| 43      | Team shirt                      | 44.99           | 0.3462        | 0.12             | 
| 126     | Spiked Shoes                    | 28.99           | 0.3846        | 0.36             | 
| 113     | Cricket Ball                    | 22.99           | 0.4231        | 0.4              | 
| 23      | Plastic Cricket Bat             | 21.99           | 0.4615        | 0.44             | 
| 124     | Wicket Keeper Gloves            | 18.99           | 0.5769        | 0.48             | 
| 122     | Wide Brim Hat                   | 18.99           | 0.5769        | 0.48             | 
| 114     | Cricket Ball - Training Ball    | 18.99           | 0.5769        | 0.48             | 
| 125     | Bucket Hat                      | 15.99           | 0.6154        | 0.6              | 
| 116     | Catchers Helmet                 | 11.99           | 0.6923        | 0.64             | 
| 48      | Indoor Cricket Ball             | 11.99           | 0.6923        | 0.64             | 
| 121     | Cricket - Athletic Guard Cup    | 10.99           | 0.7308        | 0.72             | 
| 30      | Linseed Oil                     | 9.99            | 0.7692        | 0.76             | 
| 117     | Plastic - Beach Cricket Wickets | 8.99            | 0.8846        | 0.8              | 
| 31      | Fiber Tape                      | 8.99            | 0.8846        | 0.8              | 
| 115     | Plastic - Beach Cricket Ball    | 8.99            | 0.8846        | 0.8              | 
| 118     | Cricket Bails                   | 7.99            | 0.9231        | 0.92             | 
| 120     | Cricket Peak Cap                | 6.99            | 1             | 0.96             | 
| 119     | Cricket Bails - Junior          | 6.99            | 1             | 0.96             | 

## 解答例、解説
[![](https://static.zenn.studio/user-upload/d958a6990064-20260508.png)](https://zenn.dev/kinopp/books/0b24d659785f31)

----
<br><br>


# 【完全版】問題12-18：売上の前月比と成長率の算出（LAG関数の応用）

### 難易度：★★★☆☆ (Lv.3)

## 問題
SH（売上履歴）スキーマを使用し、特定の商品について月ごとの売上推移を分析します。単なる売上額だけでなく、「前月からの増減額」**および**「売上成長率（前月比%）」を算出してください。
これにより、売上の勢いが加速しているのか、鈍化しているのかを判断します。

SHスキーマの`SALES`テーブルより、商品ID（`PROD_ID`）が「**13**」の商品を対象に、2021年の月別売上レポートを作成してください。

**【算出項目】**

1. **MONTH**: 売上月（`YYYY-MM`形式）。
2. **MONTHLY_SALES**: その月の売上合計（`AMOUNT_SOLD`の合計）。
3. **PREV_MONTH_SALES**: **前月**の `MONTHLY_SALES` 。
4. **SALES_DELTA**: 前月からの増減額（`MONTHLY_SALES` - `PREV_MONTH_SALES`）。
5. **GROWTH_RATE**: 前月比成長率（小数第3位を四捨五入し、XX.XX% 形式で表示）。

## 期待する結果
| MONTH   | MONTHLY_SALES | PREV_MONTH_SALES | SALES_DELTA | GROWTH_RATE | 
| ------- | ------------- | ---------------- | ----------- | ----------- | 
| 2021-01 | 85697.96      |                  |             | %           | 
| 2021-02 | 215811.29     | 85697.96         | 130113.33   | 151.83%     | 
| 2021-03 | 160276.7      | 215811.29        | -55534.59   | -25.73%     | 
| 2021-04 | 177791.56     | 160276.7         | 17514.86    | 10.93%      | 
| 2021-05 | 169206.96     | 177791.56        | -8584.6     | -4.83%      | 
| 2021-06 | 152892.03     | 169206.96        | -16314.93   | -9.64%      | 
| 2021-07 | 174558.16     | 152892.03        | 21666.13    | 14.17%      | 
| 2021-08 | 221150.28     | 174558.16        | 46592.12    | 26.69%      | 
| 2021-09 | 163743.33     | 221150.28        | -57406.95   | -25.96%     | 
| 2021-10 | 227343.36     | 163743.33        | 63600.03    | 38.84%      | 
| 2021-11 | 150036.7      | 227343.36        | -77306.66   | -34%        | 
| 2021-12 | 230453.02     | 150036.7         | 80416.32    | 53.6%       |


## 解答例、解説
[![](https://static.zenn.studio/user-upload/d958a6990064-20260508.png)](https://zenn.dev/kinopp/books/0b24d659785f31)

----
<br><br>


# 【完全版】問題12-19：パレート分析（ABC分析）による重要商品の特定

### 難易度：★★★★☆ (Lv.4)

## 問題
「売上の8割は全商品のうちの2割が作っている」というパレートの法則（80:20の法則）に基づき、SHスキーマの商品別売上を分析します。
各商品が全体売上に占める**累計構成比**を計算し、商品をA・B・Cランクに分類してください。

2019年の売上データに基づき、以下の条件で商品リストを作成してください。

**【集計ルール】**

1. **PROD_REVENUE**: 商品ごとの2019年の売上合計を算出。
2. **CUMULATIVE_PCT**: 売上額の**降順**で並べた際の、売上の**累計構成比**（その商品までの累計売上 ÷ 全体売上）を算出 。


3. **ABC_RANK**: 累計構成比に基づき以下のランクを付与。
* 70%まで：**'A'**
* 70%超〜90%まで：**'B'**
* 90%超：**'C'**

## 期待する結果
| PROD_NAME                                  | PROD_REVENUE | CUMULATIVE_PCT | ABC_RANK | 
| ------------------------------------------ | ------------ | -------------- | -------- | 
| Lithium Electric Golf Caddy                | 5477218.04   | 22.74          | A        | 
| Pitching Machine and Batting Cage Combo    | 2733887.43   | 34.09          | A        | 
| Soccer Goal - Official                     | 2239127.88   | 43.39          | A        | 
| Speed Trainer Bats and Training Program    | 1535187.44   | 49.77          | A        | 
| Right-Handed Graphite Shaft Iron Set       | 1368317.88   | 55.45          | A        | 
| Match Used Autograph Racquet               | 990525.95    | 59.56          | A        | 
| Soccer Goal - Portable                     | 936197.53    | 63.45          | A        | 
| Team shirt                                 | 851093.17    | 66.98          | A        | 
| Pro Maple Youth Bat                        | 691798.97    | 69.85          | A        | 
| English Willow Cricket Bat                 | 644480.02    | 72.53          | B        | 
| Genuine Series MIX Wood Bat                | 611329.86    | 75.07          | B        | 
| Limited Edition Racquet                    | 578374.62    | 77.47          | B        | 
| Pro Style Batting Tee                      | 567533.83    | 79.83          | B        | 
| 5 Point Batting Tee                        | 522713.71    | 82             | B        | 
| Pro Maple Bat                              | 496483.76    | 84.06          | B        | 
| 12.75" Premium Series Glove                | 293152.28    | 85.27          | B        | 
| Catchers Mitt                              | 269009.61    | 86.39          | B        | 
| Cricket Bails                              | 226875.98    | 87.33          | B        | 
| Slugger Youth Series Maple Bat             | 221494.36    | 88.25          | B        | 
| Cricket Bat Bag                            | 182670.35    | 89.01          | B        | 
| Spiked Shoes                               | 172907.76    | 89.73          | B        | 
| Endurance Coolcore 1/2 Zip Pullover        | 163929.27    | 90.41          | C        | 
| Helmet                                     | 163865.43    | 91.09          | C        | 
| 13" Field Master Series Glove              | 149108.82    | 91.71          | C        | 
| 12" Premium Series Glove                   | 131170.12    | 92.25          | C        | 
| Bucket of 24 Leather Baseballs             | 124081.8     | 92.77          | C        | 
| Bucket Hat                                 | 117214.73    | 93.26          | C        | 
| Baseball Is Life Cap                       | 110987.48    | 93.72          | C        | 
| Bucket of 24 Synthetic Baseballs           | 107968.24    | 94.17          | C        | 
| 11.5" Youth Triple Stripe Series Glove     | 106525.01    | 94.61          | C        | 
| Tennis Strings Natural Gut                 | 97259.84     | 95.01          | C        | 
| Cricket Ball                               | 91540.88     | 95.39          | C        | 
| Plastic Cricket Bat                        | 85211.28     | 95.75          | C        | 
| Soccer Ball - Size 5                       | 83353.36     | 96.09          | C        | 
| Goal Keeper Gloves                         | 76010        | 96.41          | C        | 
| 2 Competition Grade NFHS Baseballs         | 73544.8      | 96.71          | C        | 
| Tennis Balls 12 Pack                       | 72058.92     | 97.01          | C        | 
| Cricket Ball - Training Ball               | 71070.25     | 97.31          | C        | 
| Soccer Ball - Size 4                       | 70645.28     | 97.6           | C        | 
| 6 Gallon Empty Ball Bucket                 | 70288.37     | 97.89          | C        | 
| Indoor Cricket Ball                        | 69656.88     | 98.18          | C        | 
| Fiber Tape                                 | 64464.83     | 98.45          | C        | 
| 11" Youth Field Master Glove               | 61209.1      | 98.7           | C        | 
| Linseed Oil                                | 59391.8      | 98.95          | C        | 
| Catchers Helmet                            | 56622.64     | 99.18          | C        | 
| Plastic - Beach Cricket Wickets            | 40338.17     | 99.35          | C        | 
| Cricket Bails - Junior                     | 39241.12     | 99.52          | C        | 
| Regular Duty Tennis Balls                  | 35771.66     | 99.66          | C        | 
| MLB Official Game Baseball w/ Display Case | 31853.11     | 99.8           | C        | 
| Cricket Peak Cap                           | 27333.37     | 99.91          | C        | 
| Plastic - Beach Cricket Ball               | 10505.43     | 99.95          | C        | 
| Soccer Jersey                              | 8989.83      | 99.99          | C        | 
| Cushioned Grip                             | 1685.89      | 100            | C        | 
| Wicket Keeper Gloves                       | 624.82       | 100            | C        | 
| Sonic Core Graphite Racquet                | 11.99        | 100            | C        |

## 解答例、解説
[![](https://static.zenn.studio/user-upload/d958a6990064-20260508.png)](https://zenn.dev/kinopp/books/0b24d659785f31)

----
<br><br>

# 【完全版】問題12-20：中央値と外れ値の検出（PERCENTILE_CONT）

### 難易度：★★★★☆ (Lv.4)

## 問題
人事部では、平均給与だけでなく「中央値（メジアン）」を用いて部門間の給与バランスを調査しています。平均値は極端な高額所得者（外れ値）に引きずられやすいため、中央値との乖離を見ることで実態に近い給与水準を把握します。

HRスキーマの `EMPLOYEES` テーブルを使用し、各部門（`DEPARTMENT_ID`）における以下の統計量を算出してください。

**【算出項目】**

1. **AVG_SAL**: 部門の平均給与。
2. **MEDIAN_SAL**: 部門の**給与中央値**（`PERCENTILE_CONT(0.5)` を使用）。
3. **DIFF**: 平均と中央値の差分（`AVG_SAL` - `MEDIAN_SAL`）。
4. 対象は `DEPARTMENT_ID` が 50, 80, 100 の部門とします。

## 期待する結果
| DEPARTMENT_ID | AVG_SAL | MEDIAN_SAL | DIFF   | 
| ------------- | ------- | ---------- | ------ | 
| 50            | 3475.56 | 3100       | 375.56 | 
| 80            | 8955.88 | 8900       | 55.88  | 
| 100           | 8601.33 | 8000       | 601.33 | 


## 解答例、解説
[![](https://static.zenn.studio/user-upload/d958a6990064-20260508.png)](https://zenn.dev/kinopp/books/0b24d659785f31)

