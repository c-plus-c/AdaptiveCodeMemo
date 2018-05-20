# インタフェースとデザインパターン

# インタフェース
* 1つのクラスで本当に意味を持つインタフェースがいくつであるかを見極めることが課題となる。
* 明示的な実装
    * どのインタフェースのどのメソッドに対して実装するかを明示して実装する
    * 実装するインタフェース群の中でメソッド名が被った場合に有効

# アダプティブデザインパターン
## Null Objectパターン
* nullを返すのではなくNullを示す具象オブジェクトを返すパターン
    * IUserインタフェースを実装したNullUserクラスを定義するなど
* NullPointerExceptionを防いだりnullチェックを省くために用いる
* isNullプロパティアンチパターン
    * 抽象オブジェクトにIsNullプロパティを定義し、具象オブジェクトでそれがnullか否かをBooleanで返すような仕様
    * カプセル化を目的としているオブジェクトからロジックが零れ落ちてしまう
        * 本来の実装とNull Object実装とを区別するためにクライアントコードにif分が入り込むようになる

## Adapterパターン
* オブジェクトインスタンスが実装していないインタフェースにクライアントが依存していたとしても、そのオブジェクトインスタンスを提供できるようする。
* Class Adapter パターン
* Object Adapter パターン
    * 合成を用いることで、インタフェースメソッドを外側のカプセル化されているオブジェクトへ委譲する
    * 例: クライアントが期待する機能を扱うインタフェース(IExpectedInterface)を定義し、AdapterはTargetとなるオブジェクトを受け取り、IExpectedInterfaceを実装する。クライアントはIExpectedInterface（Adapterインスタンス）からTargetの機能にアクセスする

# Strategyパターン
* 再コンパイルを行わずにクラスのふるまいを変更できる
* あるインタフェースがあってそのインタフェースを実装したクラスが複数パターンあり、クライアントはその状態に応じて使用する実装を切り替える。

# さらなる汎用性を求めて
## ダックタイピング
アヒルのように歩き、アヒルのように泳ぎ、アヒルのように鳴くものはアヒルである
* そのクラスCはインタフェースAの条件を満たしているが、インタフェースAを実装していないときでもインタフェースAとみなして使うようなこと
* C#で実現するならdynamicあるいはImpromptu Interfaceを使う
* 但し、Enumeratorの実装のみダックタイピングが適用される
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace TheInterface
{
    // こうすることでforeachに適用できる（IEnumeratorの実装とか明示していない）
    public class DuckEnumerator
    {
        int i = 0;

        public bool MoveNext()
        {
            return i++ < 10;
        }

        public int Current
        {
            get
            {
                return i;
            }
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            var duck = new Duck();

            foreach (var duckling in duck)
            {
                Console.WriteLine("Quack {0}", duckling);
            }

            Console.ReadKey();
        }
    }
}
```

## ミックスイン
* 実装の継承を用いず、他の複数のクラスの実装を含んでいるクラスのこと
    * C#では拡張メソッドを用いることで実現可能
        * 拡張メソッドの難点
            * テストが難しくなる
            * 静的クラスとして定義されるので、オブジェクトに関連するインスタンスごとの状態を保持することは不可能
            * 拡張メソッドのターゲットがすべて同じインタフェースになる
            * 拡張の対象となるインスタンスがすべてこのインタフェースを実装しなければいけない
    * Re-motion Re-mixを使用しても実装可能

## 流れるようなインタフェース
* メソッドチェーン的な使い方ができるインタフェース

