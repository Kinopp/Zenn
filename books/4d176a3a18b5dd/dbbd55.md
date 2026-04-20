---
title: "Abstract Factoryパターン"
free: false
---

## 概要
**Abstract Factory（アブストラクト・ファクトリー）パターン**は、**「関連する部品の『セット』を、具体的なクラスを気にせずに一気に作り替える」** ためのパターンです。別名「工場の工場」とも呼ばれます。

わかりやすい例えは、**WindowsとMacのUI（ボタンやテキストボックス）の切り替え**です。

* **ボタン（部品A）:** Windows風、Mac風
* **テキストボックス（部品B）:** Windows風、Mac風

もし、あなたのアプリが「今はWindowsモード」で動いているなら、ボタンもテキストボックスも**Windows用で統一**されていなければなりません。ここで「ボタンだけMac風」になってしまうと、デザインが崩れてしまいますよね。

Abstract Factoryは、**「Windows工場」を呼べばWindows用の部品セットだけが、「Mac工場」を呼べばMac用の部品セットだけが確実に手に入る**ようにする仕組みです。

## クラス構造
![](https://storage.googleapis.com/zenn-user-upload/3d94d1a20e9b-20260407.png =580x)


## イメージ
![](https://storage.googleapis.com/zenn-user-upload/c921c232c5be-20260407.png =600x)

### 【解説】
### 1. 抽象的な工場と製品（中央の青い枠）
ここが「ルール」です。
* **抽象的な工場：** 「うちの系列は、必ず『本体』と『芯』の2つを作ります」と宣言します。
* **抽象的な本体 / 芯：** 具体的な形は決まっていませんが、「書くための道具」としての共通の型です。

### 2. 具体的な工場（左右の緑の枠）
ここが「ブランド（製品ライン）」です。
* **ボールペン工場（左）：** 「ペン用の本体」と「インクの芯」をセットで作ります。
* **シャープペンシル工場（右）：** 「シャーペン用の本体」と「黒鉛の芯」をセットで作ります。

### 3. Client（利用側）のメリット
Clientは、中央の「抽象的な工場」の窓口だけを使います。
「ボールペン工場」をセットすれば、自動的にペン用のパーツが揃います。**「うっかりシャーペンの本体にボールペンの芯を入れてしまった！」という事故が構造的に起こらなくなる**のです。

### まとめ
この図のキモは、**「横のつながり（継承）」と「縦のつながり（生成）」がマトリックス状に整理されている** 点にあります。
「ボールペン」と「シャーペン」という別々の世界（ConcreteFactory）を完全に分離しつつ、Clientには「文房具を作る」という共通の窓口（AbstractFactory）だけを見せる。この **「一貫性のあるセット販売」** こそがAbstract Factoryの真髄です。


## メリット
### 1. 部品同士の「整合性」を絶対に見逃さない
例えば「Windows風のボタン」と「Mac風のチェックボックス」が混ざってしまうと、画面がチグハグになります。このパターンを使えば、「Windows工場」からは必ず「Windows用の部品セット」だけが出てくるようになるため、**デザインや仕様の統一感**が保証されます。

### 2. 具体的なクラス名を隠せる（疎結合）
使う側（クライアント）は「何のボタンを作るか」を知る必要がなく、ただ「工場にボタンを注文する」だけで済みます。これにより、将来的に部品のクラス名が変わっても、使う側のコードを修正する必要がありません。

### 3. 「セットの切り替え」が魔法のように一瞬
「今はダークモード工場を使う」「次はライトモード工場を使う」と工場を入れ替えるだけで、アプリ全体の見た目や挙動を一気に変えることができます。



## デメリット
### 1. 新しい「種類の部品」を追加するのが地獄
これが最大の弱点です。例えば「ボタン」と「テキスト」を作っていた工場に、新しく「スライダー」を追加したいとなった場合、**「抽象工場」だけでなく、今まで作った「すべての具体的な工場」を書き換えなければなりません。**
> **例：** Windows工場、Mac工場、Linux工場……すべてに「スライダーの作り方」を追記して回る必要があります。

### 2. 構造が非常に複雑（クラスの山）
インターフェース、抽象クラス、具体的なクラスが何層にも重なるため、初めてコードを見る人が構造を理解するのに時間がかかります。小さなプロジェクトで使うと「大げさすぎる」設計になりがちです。

## 適用パターン
| 状況 | Abstract Factoryを使うべき？ |
| :--- | :--- |
| **関連する部品をセット（家族）として扱いたい** | **最適！** |
| **プラットフォームごとに処理を丸ごと差し替えたい** | **最適！** |
| **新しい部品の種類が頻繁に増える予定がある** | **不向き（Factory Methodなどの方がマシ）** |

> **例え話で理解：インテリアのコーディネート**
> 
> 「北欧スタイル」の家具セットと「和モダン」の家具セットがあるとします。
> 北欧スタイルの店（Factory）に行けば、椅子もテーブルも照明も、すべて「北欧風」で統一されたものが出てきます。
> 客（クライアント）は、個別の家具の製造工程を知らなくても、**「北欧風のセットを一式ください」**と言うだけで、統一感のある部屋が完成します。


## サンプルソース
#### `abstractfactory`パッケージ

```Java:AbstractFactory.java
package AbstractFactory.abstractfactory;

public abstract class AbstractFactory {

public static AbstractFactory getInstance(String className) throws Exception{   
       AbstractFactory abstractFactory = null;
       try {
            abstractFactory = (AbstractFactory)Class.forName(className).newInstance();
        } catch (ClassNotFoundException ex) {
            System.err.println("クラスの指定が正しくありません");
            throw ex;
        }
       return abstractFactory;
    }
   
    public void makeProduct(){
        System.out.println("---- 製品生成開始 ----");
        createAbstractProduct1().makeProduct1();
        createAbstractProduct2().makeProduct1();
        System.out.println("---- 製品生成終了 ----");
    }

    public abstract AbstractProduct1 createAbstractProduct1();
    public abstract AbstractProduct2 createAbstractProduct2();
}
```
```Java:AbstractProduct1.java
package AbstractFactory.abstractfactory;

public abstract class AbstractProduct1 {  
    public void makeProduct1(){
        System.out.println("---- Product1生成過程開始 ----");
        execute1();
        execute2();
        System.out.println("---- Product1生成過程終了 ----");
    }
    public abstract void execute1();
    public abstract void execute2();
}
```
```Java:AbstractProduct2.java
package AbstractFactory.abstractfactory;

public abstract class AbstractProduct2 {
    public void makeProduct1(){
        System.out.println("---- Product2生成過程開始 ----");
        perform1();
        perform2();
        System.out.println("---- Product2生成過程終了 ----");
    }
    
    public abstract void perform1();
    public abstract void perform2();
}
```

#### `concretefactory`パッケージ
```Java:ConcreteFactory.java
package AbstractFactory.concretefactory;

import AbstractFactory.abstractfactory.*;

public class ConcreteFactory extends AbstractFactory{
    @Override
    public AbstractProduct1 createAbstractProduct1() {
        System.out.println("Product1生成");
        return new ConcreteProduct1();
    }

    @Override
    public AbstractProduct2 createAbstractProduct2() {
        System.out.println("Product2生成");
        return new ConcreteProduct2();
    }
}
```
```Java:ConcreteProduct1.java
package AbstractFactory.concretefactory;

import AbstractFactory.abstractfactory.*;

public class ConcreteProduct1 extends AbstractProduct1{
    @Override
    public void execute1() {
        System.out.println("ConcreteProduct1.execute1実行");
    }

    @Override
    public void execute2() {
        System.out.println("ConcreteProduct1.execute2実行");
    }
}
```
```Java:ConcreteProduct2.java
package AbstractFactory.concretefactory;

import AbstractFactory.abstractfactory.AbstractProduct2;

public class ConcreteProduct2 extends AbstractProduct2{
    @Override
    public void perform1() {
        System.out.println("ConcreteProduct2.perform1実行");
    }

    @Override
    public void perform2() {
        System.out.println("ConcreteProduct2.perform2実行");
    }   
}
```
#### `Main`

```Java:Client
package AbstractFactory;

import AbstractFactory.abstractfactory.AbstractFactory;

public class Client {
    public static void main(String... args) throws Exception{
        final String fullyQualifiedClassName = "AbstractFactory.concretefactory.ConcreteFactory";
        AbstractFactory factory = AbstractFactory.getInstance(fullyQualifiedClassName); 
        factory.makeProduct();
    }
}
```


### 実行結果
```
---- 製品生成開始 ----
Product1生成
---- Product1生成過程開始 ----
ConcreteProduct1.execute1実行
ConcreteProduct1.execute2実行
---- Product1生成過程終了 ----
Product2生成
---- Product2生成過程開始 ----
ConcreteProduct2.perform1実行
ConcreteProduct2.perform2実行
---- Product2生成過程終了 ----
---- 製品生成終了 ----
```


