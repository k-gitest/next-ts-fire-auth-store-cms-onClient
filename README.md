## 目的
next.jsを使用してclient側でfirebase各種サービスを使用する技術選定における検証である。

## app概要
create-next-appで構築されたNext.jsとfirabaseのfirestoreとauthenticationを利用したCMSプロジェクトです。

* 複数会員での利用を想定している
* 会員向けと一般向けで表示画面を分ける前提
* クライアント側のフロントはSSG、APIはSSRを想定している
* 認証のみapiを使用し、それ以外は全てclient側で処理する

## 開発環境

* next 13.4.2
* typescript 5.0.4
* firebase 9.22.0
* firebase-admin 11.9.0
* tailwind 3.3.2
* swr 2.1.5
* axios 1.4.0

## ディレクトリ構成

<pre>
myapp...プロジェクトディレクトリ
  ├── components ...呼び出し用コンポーネントファイル
  │     ├── FormParts ...フォームコンポーネント
  │     ├── Private ...ログインユーザー向けコンポーネント
  │     ├── Public ...非ログインユーザー向けコンポーネント
  │     ├── layout ...メインレイアウト
  │     └── provider ...ユーザー認証チェック
  ├── lib ...firebaseなど外部設定ファイル
  ├── pages ...初期生成されるメインファイル
  │     ├── [uid] ...一般向け画面
  │     │     └── [pid] ... 投稿表示画面
  │     ├── api ...サーバー側処理
  │     │     └── admin ... adminSDK使用ファイル
  │     ├── login ...ログイン画面
  │     ├── signup ...登録画面
  │     └── user ...会員向け画面
  ├── public ...画像ファイル
  ├── styles ...css設定ファイル
  └── types ...型定義ファイル
</pre>

* typesには共用できる型定義を入れる
* 単一コンポーネントのみ適用の型定義の場合はファイル内に記述する
* firebase認証によるCRUD処理はlib/authsubmitに記述する
* deleteはサーバー側からでないと削除できないためapi内に記述する
* ユーザー登録データはlib/userStoreに記述する
* 投稿データ処理はlib/postStoreに記述する
* components/provider/AuthProviderのuseContextで状態管理をする

## 認証方法

Firebase AuthenticationはJWTを使用して認証情報をクライアントに渡すため、認証の検証はapi側でfirebase-adminを使用しtokenを渡しJWTを検証する。

認証できた場合、認証情報であるuidやtokenはindexedDBに保存される。cookieやローカルストレージではない。
ログアウトやブラウザを閉じると削除される。

## 注意点

Firebase Authenticationは認証状態を管理するサービス、firestoreはデータを格納するサービスであり、別物であることに注意する必要がある。

リレーションは出来ないがuidは共通である為、uidを活用してデータを呼び出す必要がある。

firestoreはテーブル/レコード形式ではなくコレクション/ドキュメント形式であり、サブドキュメントと組み合わせてデータを格納する必要がある。

スキーマはないがデータ形式を幾つか指定する事は可能である。
SQLではなくmap等を使用してデータを指定や検索などして取得する必要がある。

タイムスタンプ形式で格納すると呼び出した時に型エラーになってしまう。
その為adminでタイムスタンプをtoMillis()などで加工し格納する必要がある。

## 結論

Firebase Authenticationは簡単に実装できるclient側の認証サービスであり、管理もしやすい。

firestoreは注意点に挙げたとおり癖が強いと感じるが、手軽に使えるDBであることが分かる。
しかしながら前述の通りコレクション／ドキュメント形式は大規模なDBには向いていないと感じる。
