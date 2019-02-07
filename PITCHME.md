### 極めて高い安全性を保証する`Rust`での組込み開発

---

### [Rust](https://www.rust-lang.org/)

Mozillaが支援する**システムプログラミング言語**

言語の目的として、以下の3つを挙げている

- 速度
- 並行性
- 安全性

GCがなく、バイナリの動作速度は[C言語に匹敵](https://github.com/ixy-languages/ixy-languages)
強力な型システムとリソース管理システムにより、**メモリセーフ**かつ**スレッドセーフ**

---

### 最近のRust

- Firefoxの次世代レンダリングエンジンServo
- AWSのVMM firecracker
- Rustの安全性を形式手法で証明したRustBelt
- OS
- Idein actcast

### 組込み`Rust`

Rustの注力分野は、CLI / WebAssembly / Network / **Embedded**

ベアメタル (no_std)なバイナリも可能

[アーキテクチャサポート](https://forge.rust-lang.org/platform-support.html)

バックエンドはLLVMで、x86 / ARM (aarch64, v6, v7) / RISC-Vなどをサポート

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
| 記述の自由度 | 極めて高い | 強い型付けシステム、所有権、借用、ライフタイムというルールを守る義務 |
| 安全性の高め方 | 職人芸と外部ツール、動的プロファイル | コンパイラでのルールチェック |
| 安全でないコード | 動作する | コンパイルエラーになる |
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

### 強力な型システム

強力な型システムにより、**型安全**である。

型安全である、とは未定義動作が発生しない、ということ
(`unsafe`という抜け道はある)

型は常に厳密にチェックされる。
参照外し以外、暗黙の型変換は発生しない

---

### 所有権

全ての値には、その値を変更できる唯一の所有者が存在する

```c
    int32_t value = 42;
    int32_t *value_alias = &value;

    value += 1;
    *alias += 1;
```

Cでは`value`も`*value_alias`も同じ値を変更できる。

Rustでは**できない**

---


### 所有権

```rust
fn main() {
    let mut value = 42;
    let mut value_alias = &mut value;

    value += 1;  // compile error!
    *value_alias += 1;
}
```

動的確保したメモリでも、マルチスレッドでも同じ。
同時に複数箇所から書き換えられることはない。

### 借用

所有権は貸すことができる。これを借用と呼ぶ。

借用は下記の**どちらか一方だけ**の状態を持つ

- 唯一の可変参照を持つ
- 複数の不変参照を持つ

つまり、誰かが書き換えられる変数は、誰からも**読むことすらできない**。

書き換わらない変数は、複数の場所から読み込まれる。

### ライフタイム

全ての変数には生存期間がある。
動的確保したメモリも例外ではない。

```rust
    {
        let heap = Box::new(1);
    }  // release allocated memory here
```

大抵の場合、コンパイラが面倒を見てくれる
複雑なライフタイムを持つ場合、明示的に書く必要がある