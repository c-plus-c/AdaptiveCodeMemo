# 依存性逆転の原則

# 最初が肝心
* プロキシ化が可能: 別の実装をクライアントに提供できるクラス（クラスの場合は全てのメソッドがvirtualで宣言されている場合、インタフェースは常に可能）
## オブジェクトグラフの作成
* 依存性のグラフを作成する
### Poor Man's Dependency Injection
* 外部の依存関係がなくても機能する
* 冗長だが柔軟

```cs
public partial class App: Application{
    private void OnApplicationStartup(object sender, StartupEventArgs e){
        CreateMappings();

        // ここでDIしてる
        var settings = new ApplicationSettings();
        var taskService = new TaskServiceAdo(settings); // ISettingsを要求
        var objectMapper = MapperAutoMapper();
        controller = new TaskListController(taskService, objectMapper); // ITaskService, IObjectMapperを要求
        MainWindow = new TaskListView(controller);
        MainWindow.Show();

        controller.OnLoad();
    }

    private void CreateMappings(){
        AutoMapper.Mapper.CreateMap<TaskDto, TaskViewModel>();
    }

    private TaskListController controller;
}
```
### メソッド注入
* コンストラクターだけでなくメソッドやプロパティを使用してDIする
* 呼び出されるメソッドでのみ依存関係が必要になる場合に役立つ
    * 依存関係を利用する機会が少ない場合はメソッドに渡してしまう方が合理的
* そのメソッドを呼び出すクライアントの依存先のインスタンスを確保しなければいけないのが難点
```cs
public class TaskListController: INotifyProperyChanged{
    public TaskListController(ITaskService taskService, IObjectMapper mapper, ISettings settings){
        this.taskService = taskService;
        this.mapper = mapper;
        this.settings = settings;
    }

    public void OnLoad(){
        var taskDtos = taskService.GetAllTasks(settings);
        AllTasks = new ObservableCollection<TaskViewModel>(mapper.Map<IEnumerable<TaskViewModel>>(taskDtos));
    }
}
```
### プロパティ注入
* 省略
### Inversion of Controlパターン
* オブジェクトグラフの生成を実行時まで先送りできる
* IoCコンテナ
    * アプリケーションのインタフェースをそれらの実装と結び付け、依存関係を全て解決することにより、クラスのインスタンスを取得できる
```cs
public partial class App: Application{
    private void OnApplicationStartup(object sender, StartupEventArgs e){

        // ここでDIしてる
        CreateMappings();
        container = new UnityContainer();
        container.RegisterType<ISettings, ApplicationSettings>();

        // MapperAutoMapperのコンストラクタにはすでにcontainerにRegisterされたISettingsの実装（ApplicationSettings）が注入される
        container.RegisterType<IObjectMapper, MapperAutoMapper>(); 

        // 以下略
        container.RegisterType<ITaskService, TaskServiceAdo>();
        container.RegisterType<TaskListController>();
        container.RegisterType<TaskListView>();

        // 依存関係が解決できなかったらエラー
        MainWindow = contaienr.Resolve<TaskListView>();
        MainWindow.Show();

        ((TaskListController).MainWindow.DataContext).OnLoad();
    }

    private void CreateMappings(){
        AutoMapper.Mapper.CreateMap<TaskDto, TaskViewModel>();
    }

    private IUnityContainer controller;
}
```
### Register, Resolve, Release パターン
* IoCコンテナは全て、メソッドが3つのシンプルなインタフェースとして単純化できる（Unityも例外ではない）
```cs
public interface IContainer: IDisposable{

    // TInteface型の受け口にTImplementation型を注入するようにする
    void Register<TInterface, TImplementation>() where TImplementation: TInterface;

    // 全ての依存を解決する
    TImplementation Resolve<TInterface>();

    // コンテナにRegisterされたインスタンスを開放する
    // ライフタイムなどの都合により明示的な開放が必要になったときに使う
    void Release();
}
```

コンテナによる依存関係の解決処理をIocConfigurationクラスに委譲する使用例

```cs
public partial class App: Application{
    private void OnApplicationStartup(object sender, StartupEventArgs e){

        // ここでDIしてる
        CreateMappings();
        ioc = new IocConfiguration();
        ioc.Register();

        // 依存関係が解決できなかったらエラー
        MainWindow = ioc.Resolve<TaskListView>();
        MainWindow.Show();

        ((TaskListController).MainWindow.DataContext).OnLoad();
    }

    private void OnApplicationExit(object sender, ExitEventArgs e){
        ioc.Release();
    }

    private void CreateMappings(){
        AutoMapper.Mapper.CreateMap<TaskDto, TaskViewModel>();
    }

    private IocConfiguration ioc;
}

public class IocConfiguration{
    public IocConfiguration(){
        container = new UnityContainer();
    }

    public void Register(){
        container.RegisterType<ISettings, ApplicationSettings>();
        container.RegisterType<IObjectMapper, MapperAutoMapper>(); 
        container.RegisterType<ITaskService, TaskServiceAdo>();
        container.RegisterType<TaskListController>();
        container.RegisterType<TaskListView>();
    }

    public Window Resolve(){
        return container.Resolve<TaskListView>();
    }

    public void Release(){
        container.Dispose();
    }

    private IUnityContainer controller;
}
```

### 命令型の登録と宣言型の登録
* これまでやってきたのはコンテナオブジェクトのメソッドに対する手続き型の呼び出しに基づいた命令型の登録
* 簡素で読みやすくタイポによる不安がないが、実装がコンパイル時に決定される
    * 依存する実装を切り替える際には再コンパイルが必要
* 宣言型の場合は、登録をXMLに書き込み、その決断を設定時へ先送りする。
    * 再コンパイル不要だが、タイポの危険性がある
    * また、冗長性が増える
    * IoCコンテナの機能を満足に引き出せないことからあまりお勧めできない（らしい）
### オブジェクトのライフタイム
* オブジェクトの生存期間とかを意識すること
* クラスが依存関係にあるオブジェクトがコンストラクタを通じて渡される場合h、そのオブジェクトを明示的に削除すべきではない
* 以下の例ではIDbConnectionに依存するTaskServiceAdoが破棄されるタイミング（TaskServiceAdo.Dispose()が呼ばれる）にて呼ぶ

```cs
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Data.SqlClient;
using System.Data;

using ServiceInterfaces;

namespace ServiceImplementations
{
    public class TaskServiceAdo : ITaskService, IDisposable
    {
        public TaskServiceAdo(IDbConnection connection)
        {
            this.connection = connection;
        }

        public IEnumerable<TaskDto> GetAllTasks()
        {
            var allTasks = new List<TaskDto>();

            connection.Open();

            try
            {
                using (var transaction = connection.BeginTransaction())
                {
                    var command = connection.CreateCommand();
                    command.Transaction = transaction;
                    command.CommandType = CommandType.StoredProcedure;
                    command.CommandText = "[dbo].[get_all_tasks]";

                    using (var reader = command.ExecuteReader(CommandBehavior.CloseConnection))
                    {
                        while (reader.Read())
                        {
                            allTasks.Add(
                                new TaskDto
                                {
                                    ID = reader.GetInt32(IDIndex),
                                    Description = reader.GetString(DescriptionIndex),
                                    Priority = reader.GetString(PriorityIndex),
                                    DueDate = reader.GetDateTime(DueDateIndex),
                                    Completed = reader.GetBoolean(CompletedIndex)
                                }
                            );
                        }
                    }
                }
            }
            finally 
            {
                connection.Close();
            }

            return allTasks;
        }

        public void Dispose()
        {
            connection.Dispose();
        }

        private readonly IDbConnection connection;

        private const int IDIndex = 0;
        private const int DescriptionIndex = 1;
        private const int PriorityIndex = 2;
        private const int DueDateIndex = 3;
        private const int CompletedIndex = 4;
    }
}
```

### 接続ファクトリ
* Factoryパターン
    * Create~~~メソッドを作る

### Responsible Owner パターン
* IDisposableインタフェースを全ての実装で強制的に適用する代わりに、本当な必要でのみ使用することもできる。
    * 但し、ファクトリ（インタフェース）によって生成されたオブジェクトがIDisposableを実装していない場合、usingブロックを使ってスコープを外れたオブジェクトを削除することはできなくなる。
    * このような場合、Responsible Ownerパターンを使用する
        * usingブロックをtry/finallyブロックに置き換え、ファクトリによって生成されたオブジェクトがIDisposableインタフェースを実装しているかどうかを実行時にチェックする
        * 但し、デコレータを使用しているときにデコレータのツリーに入っているコンポーネントでIDisposableが実装されていないものがある場合、その子供のコンポーネントはIDisposableが実装されているか否かにかかわらず解放されない
            * そういうときはFactory Isolationパターンを使う

### Factory Isolation パターン
* よくわからん
* Withメソッドを使う
    * ファクトリによって作成されたオブジェクトをパラメータとして受け取るラムダ式を受け取る
    * ファクトリによって作成されたオブジェクトのライフタイムがラムダ式のスコープに明示的にリンクされるという利点がある
        * ラムダ式のスコープ外からはファクトリが生成したオブジェクトを原則参照できないので、オブジェクトのライフタイムを制御することはできないということをクライアントに示すことができる
            * 広いスコープの変数に代入すれば操作できてしまうが、原則禁止

#高度な注入
## Service Locatorアンチパターン
```cs
public interface IServiceLocator: IServiceProvider{
    // 関数名はResolveにすべき    
    object GetInstance(Type serviceType);
    object GetInsntance(Type serviceType, string key);

    // そもそも邪道
    IEnumerable<object> GetAllInstances(Type serviceType);

    TService GetInstance<TService>();

    TService GetInstance<TService>(string key);

    IEnumerable<TService> GetAllInstances<TService>();
}

public static class ServiceLocator{
    private static ServiceLocatorProvider currentProvider;

    public static IServiceLocator Current{
        get{
            return currentProvider();
        }
    }

    public static void SetLocatorProvider(ServiceLocatorProvider newProvider){
        currentProvider = newProvider
    }
}
```
* コンテナを使用するにはServiceLocatorクラスを呼び出さなきゃいけない
* しかもタイミングを問わずコンテナから何でも取り出せてしまう
    * DIにおける「ハリウッドの原則」
        * 「こちらから連絡するから、連絡してこないように」
* 場合によっては、使わなきゃいけないときがある
    * コンストラクタ注入が使えないときとか
* コンテナ注入も同じようなことなのでよくない
    * コンテナのアセンブリも参照しないといけないし...
* ただ、DIやらないよりかはまし

## Illegitimate Injection パターン
* 依存するインタフェース一覧を引数にとるコンストラクタ以外に、既定のコンストラクタを定義してその内部で依存する型のインスタンスを直接生成するようなパターン
* ユニットテストをサポートするために使用されることがある
    * 最初からモック用のコンテナ作っとけよという話

## 合成ルート
* アプリケーションでは、DIについて何かを知っている部分を一つだけにすべき
* 合成ルート: Poor Man's Dependency Injectionを使用するときにクラスが生成される場所（依存ツリーを生成するところ）
    * アプリケーションのエントリポイントのできるだけチアックに置けばいい

### 解決ルート
* 解決の対象となるオブジェクトグラフのルートを形成するオブジェクト型
* 本番用だったりモック用だったりを切り替えられるようにする

## 設定より規約
* 設定に基づく登録では、インタフェースを実装に一つ一つマッピングする必要がある
    * 時間がかかる
    * 単純作業の繰り返し
* 代わりに、規約を使ってコードの量を減らすことができる
* 規約はコンテナに対する命令
    * インタフェースを実装に自動的にマッピングする方法を示す
    * 登録の代わりにコンテナに入力として渡される
* 短所
    * 規約をセットアップするのは難しい
        * インタフェースの実装が一つだけであるとしたら、それ自体がコードの匂いになる
            * インタフェースごとに複数の実装を作成し、規約に基づいて登録することを心掛けるべき
* 設定を使うか規約を使うかのトレードオフは有用性と複雑さの2つの基準で判断される（強い型付けのときのみ）
    * プロジェクトが単純で、マッピングの量が限られている場合は、Poor Man's Dependency Injection
    * プロジェクトがより複雑で、マッピングの登録となるインタフェースとクラスの数が多い場合は、登録の大部分を規約で処理し、それ以外のより具体的なマッピングを明示的に登録する

```cs
private void OnApplicationStartup(object sender, StartupEventArgs e){
    CreateMappings();

    container = new UnityContainer();

    // 以下のRegisterTypeの引数によって、注入に使う実装を指定する

    // ベースパスであるbinフォルダのアセンブリに含まれているクラスをすべて登録する
    // それらのクラスを、クラスの名前とマッチするインタフェースにマッピングする。（Service実装クラスがIServiceインタフェースに対して登録）
    // 各まっぴんっぐを登録する際には、マッピングの命名に規定値を使用する。既定値はnullであり、マッピングに名前を付けないことを意味する。
    container.RegisterTypes(AllClasses.FromAssembliesInBasePath(),
                            WithMappings.FromMatchingInterface,
                            WithName.Default);
    
    MainWindow = container.Resolver<TaskListView>();
    MainWindow.Show();

    ((TaskListController)MainWindow.DataContext).OnLoad();
}
```
# まとめ
* DIがなければ、クラスから依存関係を抜き出し、それらの実装を汎用的なインタフェースで覆い隠すことに焦点を絞るのは不可能