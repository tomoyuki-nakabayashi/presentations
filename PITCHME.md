### 極めて高い安全性を保証する`Rust`での組込み開発

---

### 背景

組込み開発でもプログラミング言語が選べる時代になりつつある

- LLVMの発展が発端
    - GCCに匹敵するコンパイラを作るのが難しかった

目的に応じた言語選択ができるように準備することが重要

---

### 候補言語

いずれもLLVMをバックエンドに持つ言語

- Go
- Rust
- Zig

---

### `Go`

- google
- スクリプト言語の生産性とコンパイル言語の速度
- シンプルな言語仕様
- GC (ガベージコレクション)有り
- システムコールを自前実装
- MCUで使えるTinyGo

---

### `Rust`

- Mozilla
- 速度、並行性、安全性
- 言語仕様は簡単ではない
- GCなしでメモリ安全性を保証
- オブジェクト指向、関数型を取り入れた手続き型言語
- 自前実装のlibc

---

### `Zig`

- connectFree
- 堅牢性、最適性、明瞭性
- Goより攻めていて、Rustよりシンプル
- メモリアロケータを明示的に管理
- C言語のヘッダを直接include可能
- システムコールを自前実装

---

### ユースケース別の選択例

- Go
    - GCがあっても良いユーザーアプリ
- Rust
    - 安全性第一
- Zig
    - ベアメタル、Cとの連携が多い
- CおよびC++
    - 既存ソフトの改修、ライブラリ

---

### 今回のテーマは`Rust`

安全性が高く、高い評価を得ており、勢いがある

IoTのビジネスモデルが普及

- MCUがネットワークに接続
- デバイスも認証情報を取り扱う
- 限られたリソース

ユースケースにぴったり

---

### [`Rust`](https://www.rust-lang.org/)

Mozillaが支援する**システムプログラミング言語**

言語の目的として、以下の3つを挙げている

- 速度
- 並行性
- 安全性

GCがなく、バイナリの動作速度は[C言語に匹敵](https://github.com/ixy-languages/ixy-languages)

強力な型システムとリソース管理システムにより、**メモリセーフ**かつ**スレッドセーフ**

---

### `Rust`

パラダイム
- 手続き型
- trait (型クラス)によるオブジェクト指向
- 関数型の機能を多く提供

C言語からのギャップは大きいが、得られるメリットも大きい

---

### 最近の`Rust`

プロダクトや様々な分野で使用されるように

- Firefoxの次世代レンダリングエンジンServo
- AWSのVMM firecracker
- Rustの安全性を形式手法で証明したRustBelt
- OS
- Idein actcast

---

### 組込み`Rust`

2018年 **組込み** に注力された

ベアメタル (no_std) なバイナリも可能

[アーキテクチャサポート](https://forge.rust-lang.org/platform-support.html)

x86 / ARM (aarch64, v6, v7) / RISC-Vなどをサポート

主要なボードにはライブラリが提供されている

[The Embedded Rust Book](https://tomoyuki-nakabayashi.github.io/book/)は和訳済

---

### 安全性の観点で見たCと`Rust`の違い

プログラマに与えるものと、負わせるもの。

C言語は、**自由**を与えて、**責任**を課す。

Rustは、**義務**を課して、**保証**を与える。

C言語でのコーディングは自然石を使った壁の構築、対して、Rustでのコーディングは加工された石を使った壁の構築

---

### 安全性の定量的な比較

[Rustで普通にプログラミングするだけでMISRA-Cのルールを90%満足できる](https://blog.hatena.ne.jp/tomo-wait-for-it-yuki/tomo-wait-for-it-yuki.hatenablog.com/edit?entry=98012380858802545)

Rustでコンパイルが通った時点で、MISRA-Cの**80%**のコーディングルールを満たす。

有料のMISRA-Cチェックツールで検出できないバグが、Rustでコンパイルエラーとなる例

[The Challenge of Using C in Safety-Critical Applications](https://static1.squarespace.com/static/5a60ec649f8dce866f011db6/t/5ad016871ae6cf72ec208cb8/1523586697234/The+Challenge+of+Using+C+in+Safety-Critical+Applications.pdf)

`Rustコンパイラ > C + コーディング基準 + 静的解析ツール`

---

### 自由度と安全性の定性的な比較

| 項目 | C言語 | Rust |
| --- | --- | --- |
| 記述の自由度 | 極めて高い | 厳格なルールがある |
| 安全性の高め方 | 職人芸と外部ツール、動的プロファイル | コンパイラでのルールチェック |
| 安全でないコード | 動作する | コンパイルエラーになる |

---

### 自由度と安全性の定性的な比較

| 項目 | C言語 | Rust |
| --- | --- | --- |
| 安全でないコードの特定方法 | 把握が困難 | `unsafe`ブロック内にしか書けない |
| 安全性を保証する責任は | プログラマにある | 一部はプログラマ、ルールの範囲内はコンパイラが保証する |

---

### どういう言語か？

> Rustを使い始めてわかったのは、C/C++では長い間かけてゆっくり学んでいたような「良い書き方」を学ばないことには、
> Rustではプログラムをコンパイルすることすらできない、ということだ。
> 
> Mitchell Nordine

---

### なぜ安全なのか？

次のルール (義務) を守らなければコンパイルできないため

- 強力な型システム
- 所有権
- 借用
- ライフタイム

---

### 強力な型システム

強力な型システムにより、**型安全**である。

型安全である、とは未定義動作が発生しない、ということ
(`unsafe`という抜け道はある)

型は常に厳密にチェックされる。
参照外し以外、暗黙の型変換は発生しない

---

### 強力な型システム

型プログラミングにより、静的な検証が可能

![GPIO型状態プログラミング](https://pbs.twimg.com/media/Dych45YUUAAYctc.jpg)

---

### 所有権

全ての値には、その値を変更できる唯一の所有者が存在する

```c
    int32_t value = 42;
    int32_t *alias = &value;

    value += 1;
    *alias += 1;
```

Cでは`value`も`*value_alias`も同じ値を変更でき、バグの温床に

Rustでは**できない**

---

### 所有権

```rust
fn main() {
    let mut value = 42;
    let mut alias = &mut value;

    value += 1; // compile error!
    *alias += 1;
}
```

動的確保したメモリでも、マルチスレッドでも同じ。
同時に複数箇所から書き換えるコードはコンパイルエラー

---

### 借用

所有権は貸すことができる。これを借用と呼ぶ。

借用は下記の**どちらか一方だけ**の状態を持つ

- 唯一の可変参照を持つ
- 複数の不変参照を持つ

つまり、誰かが書き換えられる変数は、誰からも**読むことすらできない**。

書き換わらない変数は、複数の場所から読み込むことができる。

---

### ライフタイム

全ての変数には生存期間がある。
動的確保したメモリも例外ではない。

```rust
    {
        let heap = Box::new(1);
    }  // release allocated memory here
```

複雑なライフタイムを持つ場合、明示的に書く必要がある
基本は、所有権と借用の情報からコンパイラが自動的にやってくれる

---

### 少し実践的な例

Zephyrの参照カウンタを使ったメモリ管理機構のアプリ例

```c
    struct net_pkt *reply_pkt;
    reply_pkt = net_app_get_net_pkt(ctx, net_pkt_family(pkt), BUF_TIMEOUT);

    ret = net_app_send_pkt(ctx, reply_pkt, NULL, 0, K_NO_WAIT,
                   UINT_TO_POINTER(net_pkt_get_len(reply_pkt)));
    if (ret < 0) {
        NET_ERR("Cannot send data to peer (%d)", ret);
        net_pkt_unref(reply_pkt);
    }
```

---

### 非同期処理のためエラー時のみcallerがメモリ解放

APIドキュメント

> If the function return < 0, then it is caller responsibility to unref the pkt.

```c
    if (ret < 0) {
        net_pkt_unref(reply_pkt);
    }
```

上の処理がないとメモリリーク、条件を間違うと2重解放

**人間が**、API仕様を把握するしかない

---

### Rustの例（API提供側）

```rust
// `pkt`の所有権を要求する
fn consume_or(pkt: Box<u32>) -> Result<(), Box<u32>> {
    if *pkt == 0 {
        return Ok(())
    } // `pkt`のライフタイムが終了し、リソースが解放される

    // エラーの場合は、所有権を返す
    Err(pkt)
}
```

---

### Rustの例

https://wandbox.org/permlink/0Swez4QYqVNxOEMX

```rust
    let reply_pkt = Box::new(1);

    if let Err(_reply_pkt) = consume_or(reply_pkt) {
        // エラーハンドリングする
    } // _reply_pktはここでリソースを解放する

    // 解放したリソースへのアクセスなのでコンパイルエラー
    // println!("{}", reply_pkt);
```

---

### more information

- [Rustの日本語ドキュメント](https://doc.rust-jp.rs/)
- [The Embedded Rust Book 和訳](https://tomoyuki-nakabayashi.github.io/book/)

---

### 組込み`Rust`の課題

C言語のギャップが大きい
- シンタックス
- セマンティクス

関数型、式指向、ムーブ、トレイト、ジェネリクス、代数的データ型、パターンマッチ、コレクション、マクロ

---

### 今後の組込みプログラマに求められるもの

組込みでデファクトになる言語は予測できないが、LLVM言語が普及する可能性が高い

- LLVMの知識
- 近代的プログラミング言語の機能

この2つを兼ね備えるプログラマを擁する組織が有利

---

### LLVMの知識

今後、よりLLVM言語が使われる比率が増える

- 最適化
- ツールチェイン
- ターゲットアーキテクチャ
- コミュニティでの影響力

---

### 近代的プログラミング言語の機能

- 強い型付け、型推論
- 継承に依らないポリモーフィズム
- 関数型言語の機能
- 所有権
- エコシステム
    - フォーマッタ
    - パッケージシステム

---

### `Rust`に話を戻すと

RustはLLVM言語で、近代的プログラミング言語の機能を兼ね備えている

C言語とRustのギャップは、

組込みプログラマが次世代で活躍するために習得が必要な素養

---

### まとめ

- 組込みでも言語が選べる時代へ
    - LLVMの知識と近代プログラミング言語の機能が重要に
- 現状、Rustが有力候補
- Rustは安全性の高いプログラミングが可能
    - 型システム、所有権、借用、ライフタイム