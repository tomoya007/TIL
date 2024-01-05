React, TypeScriptでのプロジェクトを管理しているとき、開発、本番といった環境ごとに異なる設定をする必要があります。
これらの環境ごとに異なる環境変数を設定する方法を書いていきます。

前提として、```create-react-app```には、dotenvというパッケージが既に入っています。


## 環境変数の設定
プロジェクトルートに環境ごとの```.env```ファイルを作成します。

・```.env```：開発環境用の環境変数を設定します。

.```env.production```：本番環境用の環境変数を設定します。

例）

```.env
REACT_APP_API_BASE_URL=http://localhost
```


## 環境変数の読み込み方
jsファイルには、```REACT_APP_```のように設定することで読み込むことができます。

例）```LoginComponent.js```など

```js
const apiURL = process.env.REACT_APP_API_BASE_URL
```

## Gitで.envファイルを無視する方法
これらの ```.env``` ファイルを Git のバージョン管理から除外します。
これにより、セキュリティ上の問題を防ぐことができます。
```.gitignore``` ファイルに以下の行を追加します：

```.gitignore
.env
```

## 参考
https://qiita.com/itinerant_programmer/items/a2875b9bd63d1f6c5912

https://qiita.com/naogify/items/37de1baf1aebb648273b

