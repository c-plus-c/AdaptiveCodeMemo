# リスコフの置き換え原則

# 定義
* DerivedがBaseの派生型であるとすれば、Base型のオブジェクトをDerived型のオブジェクトと置き換えたとしても、従来通りプログラムは動作し続けるはずである
* 基底型の変数に対してどの継承された型が入っているかはクライアントは知るべきではない
* コンテクスト
    * クライアントが派生型を操作する方法。クライアントが派生型を操作しないとしたら、LSPに従うことも、違反することもあり得ない。
## コントラクトのルール
* 事前条件を派生型で強化することはできない
* 事後条件は派生型で緩和することはできない
* 基底が他の普遍条件（常に満たさなければならない条件）は派生型でも維持されなければいけない
## 変性のルール
* 派生型のメソッドの引数には反変性がなければいけない
* 派生型の戻り値の型には今日変性がなければいけない
* 既存の例外改装に含まれていない新しい例外を派生型からスローしてはいけない
* .NET FrameworkのCLR言語における編成の概念は、ジェネリック型と委譲に限定される
# コントラクト
* 開発者は、インタフェースに対してプログラムするべき
    * コントラクトに対してプログラムするという表現もある
## 事前条件
* メソッドを失敗させずに確実に実行するために必要なすべての条件
    * メソッドの先頭にガード句を配置する（例外のスローなど）
    * 事前条件のターゲットとしていいのは、ユーザが操作できるもののみ
        * メソッドの引数・パブリックフィールドのみ
## 事後条件
* メソッドの終了時にオブジェクトが有効な状態のままであるかをチェックするための条件
    * メソッド終了直前にメソッド内部で事後条件のチェックを行う
        * メソッド終了条件が不正の場合は例外をスローするなどする
    * メソッドの戻り値などが主なターゲットとなる
* 戻り値が常にゼロ以外の製の値になることをメソッドのインタフェースが伝えていない
    * インタフェースとクライアントのコントラクトの特徴
## データ不変条件
* オブジェクトのライフタイムにわたって変化しない述語
* データ不変条件はオブジェクトが作成された時点で満たされ、オブジェクトがスコープを外れるまでその状態が維持される必要がある
    * publicプロパティのsetterに不変条件をチェックする記述を挿入する
## カプセル化
* 確実にコントラクトを持たせたいならプリミティブ型をラップした独自クラスを用意するのもよい
    * 重さを示す変数をWeightと定義し、double型をラップしてそのdouble型が負の値にならないようにする
## LSPのコントラクトのルール
* 事前条件は強化できない
    * クライアントがサブクラスのどのメソッドでも最も厳格な事前条件コントラクトが定義されると想定していた場合、事前条件を強化すると、クライアントのコードが動作しなくなる恐れがある
* 事後条件は緩和できない
    * 新しいサブクラスが作成視された時に既存のクライアントが動作しなくなる恐れがあるから
    * LSPに準拠していれば、新に作成されたサブクラスを既存のすべてのクライアントで使用できるはずであり、思わぬ方法で失敗することはないはず
* 不変条件は維持しなけばいけない
    * 基底クラスに含まれるすべてのデータ不変条件は新しいサブクラスでも維持しなければいけない
        * 基底クラスのプロパティAは正の値しか取れないのなら、派生クラスでもプロパティAは正の値しか取れないようにしなければいけない
## コードコントラクト
* if文と例外を使ったコントラクトのガード句よりもくどくないやつ
* System.Diagnostics.Contracts.Contractクラスを使う
    + コードコントラクトを使用するとContractクラスがコードベースに浸透してしまうので、プロジェクトで最初からそれを使用するか、最後まで使用しないでおくことが肝心

### 事前条件コントラクト
* Contract.Requiresメソッドは事前条件のチェックに用い、Boolean型の述語を受け取ってこの内部の述語がtrueを返したら事前条件が満たされているとする

### 事後条件コントラクト
* Contract.Ensuresメソッドは事後条件のチェックの用い、Boolean型の述語を受け取ってこの内部の述語がtrueを返したら事後条件が満たされているとする
    * Contract.Resultメソッドを使うとメソッドから最終的に返される結果を得られる

### 不変条件コントラクト
* Contract.Invariantメソッドはデータ不変条件のチェックに用いる
    * データ不変条件のチェックを行うメソッドを定義し、そのメソッドにContractInvariantMethodAttributeを付与する
        * この属性をつけることで、クラスのデータ不変条件に違反していないことを確認するために、メソッドの開始時と終了時にこのメソッドがコールされる
        * その内部でContract.Invariantメソッドによるデータ不変条件のチェックを書くこと

```cs
public class ShippingStrategy
{
    protected decimal flatRate;
    public ShippingStrategy(decimal flatRate){
        this.flatRate = flatRate
    }

    // ShippingStrategyメソッドの開始時と終了時にこのメソッドがコールされる
    [ContractInvariantMethod]
    private void ClassInvariant(){
        Contract.Invariant(this.flatRate > 0m, "Flat rate must be positive and non-zero");
    }
}
```

### インタフェースコントラクト
* Contract静的クラスを使ったコントラクトの確認の問題点を解決する
* Contractの対象となるinterface（IShippingStrategy）があるとして、そのinterfaceを実装してコントラクトの処理をまとめたクラス(ShippingStrateyContract)を作る
    * ShippingStrateyContractクラス内部でContract.RequiresとかContract.Ensuresを呼び出す
* Contractに関する処理は全てこのShippingStrategyContractに書けばよい

```cs

[ContractClass(typeof(ShippingStrategyContract))]
interface IShippingStrategy{
    decimal CalculateShippingCost(float packageWeightInKilograms, 
                                Size<float> packageDimensionsInInches,
                                RegionInfo destication);
}

// IShippingStrategyを実装したすべてのクラスに対してこのコントラクトが適用される
[ContractClassFor(typeof(IShippingStrategy))]
public abstract class ShippingStrategyContract: IShippingStrategy
{
    protected decimal flatRate;
    public decimal CalculateShippingCost(float packageWeightInKilograms, 
                                Size<float> packageDimensionsInInches,
                                RegionInfo destication){
        Contract.Requires<ArgumentOutOfRangeException>(packageWeightInKilograms > 0f,
        "Package weight must be positive and non-zero");
        Contract.Requires<ArgumentOutOfRangeException>(
            packageDimensionsInInches.X > 0f && packageDimensionsInInches.Y > 0f,
            "Package dimensions must be positive and non-zero");

        Contract.Ensures(Contract.Result<decimal>() > 0m);
    }

    // ShippingStrategyメソッドの開始時と終了時にこのメソッドがコールされる
    [ContractInvariantMethod]
    private void ClassInvariant(){
        Contract.Invariant(this.flatRate > 0m, "Flat rate must be positive and non-zero");
    }
}
```
# 共変性と反変性
* 共変: out
    * 継承先のクラスではそのメソッドの戻り値の型はベースクラスの引数型と同一あるいはベースクラスの引数型の継承型のみ
* 反変: in
    * 継承先のクラスでのそのメソッドの引数の型はベースクラスと同一あるいはベースクラスの引数型の基底型のみ
* 不変性: 何も指定しない
    * 共変でも不変でもない
## LSPの型システムのルール
* 派生型のメソッドの引数には反変性がなければいけない
* 派生型の戻り値の方には共変性がなければいけない
* 新しい例外は許可されない
### 新しい例外は許可されない
* 派生クラスで基底クラスにない例外がスローされてもクライアントはそれがスローされることを想定していないのでcatchできずに落ちる
    * 基底クラス例外を統一して扱うこと（基底例外クラスを作り、新しく発生しうる例外はそこから派生する）