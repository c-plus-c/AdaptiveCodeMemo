# ユニットテストとリファクタリング

* ユニットテスト（単体テスト）
    * 他のコードをテストするためのコードを記述する手法
    * 各ユニットテストは、以下の3つのセクションで構成される(AAA)
        * Arrange: テストの事前条件のセットアップ
        * Act: テストの対象となるアクションの実行
        * Assert: 振る舞いが期待通りであることの検証
    * テストの対象となるシステム（クラスとか）をSUT(System Under Test)という
* リファクタリング
    * 挙動が変わらない範囲でのコード改善
    * ユニットテストはリファクタリングの際のセーフティネットとして生きる

# ユニットテスト

* テスト駆動開発
    * テストをかいて、そのテストを満たすように実装するというのを繰り返してシステムを開発すること
    * 1段階: 失敗するテストを作成する
        * テストは動作の保証が取れるまで容赦なく書く
        * 最初からすべてのパターンを網羅するのではなく、単純なものから少しずつ書くことで直感を働かせてアサーションをコーディングする
    * 2段階: この失敗するテストを成功させ、グリーンのアイコンが表示されるようにする
    * 3段階: テストが崩れないという制約の下で、リファクタリングを行う

## より複雑なテスト
* TDDとAAAを使って記述する
* モック実装を使う
    * インフラストラクチャとか時間によって状態が変わったり、こちらから自由に状態を変化させられないものとかに使う
    * Moqパッケージを使うと、モック対象のインタフェースを指定するだけでその振る舞いを定義できる
    * オーバースペックのテストはアサーションの対象を変更することで回避できる
## その他のテスト
* ハッピーパス: エラーや問題を発生させないコードを通る実行パス
    * Null参照やアカウント検出、例外スローなどのケースは考慮されていない
    * ExpectedExceptionAttributeによってテスト中に投げられる必要のある例外を指定できる
    * 事前条件も考慮すること
* 各レイヤーは下位のレイヤーの例外をラッピングするための例外型を定義する
## 不具合修正のテスト
1. 失敗するユニットテストを作成する
2. 二つのことを明らかにする
    * 不具合を確実に再現する手順
    * （現在適用されていない）期待される振る舞い

# リファクタリング
* マジックナンバーを定数に置き換える
* 条件式を多態性に置き換える
* コンストラクタをファクトリメソッドに置き換える
* コンストラクタをファクトリクラスに置き換える
* 継承を委譲に置き換える