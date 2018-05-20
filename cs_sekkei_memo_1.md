# スクラムの紹介
### 非対称な階層化
サーバに送られたリクエストの流れは常に同じであるとは限らない：階層化が非対称
コマンド/クエリ債務分離（CQRS）: 最近注目の非対称の階層化パターン
#### CQS
* コマンド
    * アクションに対する命令的な呼び出しで、コードに何かを実行させる
    * 値を返してはいけない
    * メソッドが値を返してかつCQS純く尾ならそのメソッドはオブジェクトの状態を変更すると考えることができる
        * ゆえに呼び出しの順序に注意を払う必要がある

```cs
// CQS準拠のコマンドメソッド
public void SaveUser(string name)
{
    session.Save(new User(name));
}

// CQS非準拠のコマンドメソッド
public User SaveUser(string name){
    var user = new User(name);
    session.Save(user);
    return user;
}
```

* クエリ
    * データに対するリクエストであり、コードに何かを取得させる
    * システムの状態を変更しない

```cs
// CQS準拠のクエリメソッド
public IEnumerable<User> FindUserByID(Guid userID)
{
    return session.Get<User>(userID);
}

// CQS非準拠のクエリメソッド
public IEnumerable<User> FindUserByID(Guid userID)
{
    var user = session.Get<User>(userID);
    user.LastAccessed = DateTime.Now;
    return user;
}
```

