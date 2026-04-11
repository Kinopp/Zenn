---
title: "Flyweightパターン"
free: false
---

## 概要
**Flyweight（フライウェイト）パターン**は、**「大量の細かいオブジェクトを、みんなで共有してメモリを節約する」** パターンです。「フライウェイト」はボクシングの軽量級を意味し、その名の通りプログラムを「軽く」するための工夫です。
同じ内容のオブジェクトを何千、何万個も作るとメモリが足りなくなります。そこで、**「共通の部分（内部状態）」** を使い回し、**「個別の事情（外部状態）」** だけを後から外から与えることで、インスタンスの総数を劇的に減らします。

## クラス構造
![](https://storage.googleapis.com/zenn-user-upload/f5ffe3d2c913-20260404.png =270x)


## イメージ
【効果】
同一のオブジェクトの場合、新たにインスタンス化しないのでリソースの無駄を省くことができる。
![](https://storage.googleapis.com/zenn-user-upload/1d48ed7eef6a-20260404.png =580x)



## メリット
### 1. メモリ使用量を圧倒的に削減できる
全く同じデータを持つオブジェクトを重複して作らなくて済むため、メモリ消費を最小限に抑えられます。特に、数万個のキャラクターを出すゲームや、膨大な文字を扱うテキストエディタなどで真価を発揮します。

### 2. パフォーマンスの向上（GCの抑制）
オブジェクトの生成数が減るため、メモリを確保する時間が短縮され、Javaなどの言語では「ガベージコレクション（GC）」の発生頻度を下げることにも繋がります。



## デメリット
### 1. 「内部」と「外部」の状態分離が面倒
オブジェクトの情報を「全インスタンス共通のデータ（内部状態）」と「その時々で変わるデータ（外部状態）」に分ける設計が必要です。この切り分けが難しく、コードが複雑になりがちです。

### 2. 実行時の計算コストが増える場合がある
共有オブジェクトを使う際、外部状態（座標や色など）をその都度計算したり引数で渡したりする必要があるため、メモリは節約できても、**CPUの処理時間はわずかに増える**ことがあります。

### 3. 共有しているため、勝手に中身を変えられない
一つのオブジェクトをみんなで共有しているため、ある場所で「中身を書き換えよう」とすると、**それを使っている全箇所に影響が出てしまいます。** 基本的に共有部分は「不変（Immutable）」に保つ必要があります。

## 適用パターン

| 状況 | Flyweightを使うべき？ |
| :--- | :--- |
| **同じオブジェクトが大量に必要** | **最適！** |
| **オブジェクトの生成にメモリを食いすぎている** | **最適！** |
| **オブジェクトごとにデータがバラバラ** | **不向き（共有できない）** |
| **メモリに余裕がある** | **不要（普通の `new` でOK）** |

> **例え話で理解：学校の「教科書」**
> * **普通の方法：** 全生徒に全科目の教科書を配る（大量の紙＝メモリを消費）。
> * **Flyweight：** 図書室に1セットだけ教科書を置き、生徒は必要な時だけそこへ読みに行く。
>   * **内部状態（共有）：** 教科書に書いてある内容。
>   * **外部状態（個別）：** 誰が（どの生徒が）、いつ（何時間目に）読んでいるか。


## サンプルソース


```Java:Flyweight.java
public class Flyweight {
}
```
```Java:FlyweightFactory.java
public class FlyweightFactory {
    private Map pool = new HashMap();
    
    private static FlyweightFactory singleton = new FlyweightFactory();
    
    private FlyweightFactory(){
    }
    
    public static FlyweightFactory getInstance(){
        return singleton;
    }
    
    public synchronized Flyweight getFlyweight(String key){
        Flyweight flyweight = (Flyweight)pool.get(key);
        if (flyweight == null){
            flyweight = new Flyweight();
            pool.put(key, flyweight);
            System.out.println("インスタンスを生成しました：" + key);
        }
        return flyweight;
    }    
}
```
```Java:Client.java
public class Client {
    public static void main(String... args){
        FlyweightFactory flyweightFactory = FlyweightFactory.getInstance();
        Flyweight flyweight1 = flyweightFactory.getFlyweight("a");
        Flyweight flyweight2 = flyweightFactory.getFlyweight("b");
        Flyweight flyweight3 = flyweightFactory.getFlyweight("a");
        Flyweight flyweight4 = flyweightFactory.getFlyweight("c");
    }
}
```


### 実行結果
```
インスタンスを生成しました：a
インスタンスを生成しました：b
インスタンスを生成しました：c
```


