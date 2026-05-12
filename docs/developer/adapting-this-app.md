# BirdNET Live を別の Flutter アプリに作り替えるためのガイド

このメモは、Flutter が初めてで、BirdNET Live を少し直すのではなく、
別のアプリの土台として使いたい人向けの概要です。

## 1. まず押さえたい Flutter の基本

### Flutter を一言でいうと

Flutter は、Dart でウィジェットを組み合わせながら画面を作る UI
フレームワークです。1 つのコードベースから Android、iOS、デスクトップ
向けのアプリを作れます。

### 最初に見るとよいファイル

- `pubspec.yaml`:
  パッケージ情報、依存関係、アセット、アプリのバージョン
- `lib/main.dart`:
  起動時の初期化処理
- `lib/app.dart`:
  `MaterialApp`、テーマ、ローカライズ、最初に開く画面
- `lib/`:
  アプリ本体の UI とロジックのほぼすべて

### Flutter アプリの基本的な流れ

- `main()` でアプリを起動する
- `runApp()` でルートウィジェットを載せる
- ウィジェットが UI を表す
- 状態が変わると UI も変わる
- 再ビルドは普通の動作

このリポジトリでは、`main()` でプラットフォーム向けの初期化をしたあと、
Riverpod の `ProviderScope` を立ち上げ、最後に `App` を読み込みます。

### ウィジェットの基本

- **StatelessWidget**: 入力が決まれば表示も決まる
- **StatefulWidget**: ローカルな可変状態を持てる
- **ConsumerWidget**: Riverpod の provider を読める

このアプリでは Riverpod を中心に状態管理しているので、
`ConsumerWidget` が多く使われています。

### 状態管理の基本

BirdNET Live は **Riverpod** を使っています。

よくある流れは次のとおりです。

1. 値やサービスの provider を定義する
2. ウィジェット側で `ref.watch(...)` で読む
3. notifier やサービス経由で更新する

つまり、アプリの挙動を変えたいときは、画面側のコードだけでなく、
その背後にある provider も一緒に見ることが多いです。

### アセットと設定

Flutter では `pubspec.yaml` に静的アセットを宣言します。
このプロジェクトでは画像、モデルファイル、種データなどをここで扱っています。

主な例:

- `assets/images/`
- `assets/models/`
- `assets/species_data/`

### ローカライズの基本

画面に出す文字列は、ウィジェットへ直接書かないのが基本です。
このリポジトリでは次の ARB ファイル群で管理しています。

- `lib/l10n/`

表示テキストを変えたいときは、ここを最初に確認すると把握しやすいです。

### 開発時によく使うコマンド

```bash
flutter pub get
flutter analyze
flutter test
flutter run
```

- **Hot reload**: 状態をなるべく維持したまま変更を反映する
- **Hot restart**: Dart 側の状態を立ち上げ直す
- **flutter analyze**: まず最初に回したい静的チェック

## 2. このリポジトリは何をするものか

BirdNET Live は、端末上で ONNX 推論を動かし、鳥の音声をリアルタイムに
識別する Flutter アプリです。マイク入力を取り込み、ローカルでモデルを実行し、
検出結果をライブスペクトログラムや調査向けの機能と一緒に表示します。

小さなサンプルアプリではなく、すでに次のような要素を持っています。

- リアルタイム音声入力
- オンデバイス推論
- GPS を使ったワークフロー
- セッション保存とエクスポート
- ローカライズ
- 複数の動作モード

別アプリの土台として見ると強力ですが、そのぶん学ぶ範囲は広めです。

## 3. リポジトリ全体の見取り図

### 重要なルートファイル

- `pubspec.yaml`:
  依存関係、アセット、バージョン
- `analysis_options.yaml`:
  Analyzer と lint の設定
- `mkdocs.yml`:
  ドキュメントサイトの設定
- `README.md`:
  プロジェクト概要と基本コマンド

### アプリ本体

- `lib/core/`:
  定数、テーマ、基盤ユーティリティ
- `lib/shared/`:
  共通モデル、サービス、provider、再利用ウィジェット
- `lib/features/`:
  機能ごとのモジュール

### 主な feature モジュール

- `lib/features/live/`:
  ライブ識別モード
- `lib/features/point_count/`:
  時間制限付きの定点観測
- `lib/features/survey/`:
  GPS を使う長時間の調査モード
- `lib/features/file_analysis/`:
  音声ファイルのオフライン解析
- `lib/features/explore/`:
  位置情報ベースの種の探索
- `lib/features/inference/`:
  ONNX モデルの読み込みと推論
- `lib/features/audio/`:
  音声入力とバッファ処理
- `lib/features/history/`:
  セッション保存、レビュー、エクスポート
- `lib/features/settings/`:
  設定画面と設定関連ロジック
- `lib/features/home/`:
  ホーム画面と入口になる画面群
- `lib/features/onboarding/`:
  初回オンボーディングと利用規約フロー
- `lib/features/about/`:
  クレジット、リンク、法的情報

### プラットフォーム別ディレクトリ

- `android/`
- `ios/`

アプリ名、パッケージ ID、権限、署名、プラグイン連携などを変えるときは、
このあたりも触ることになります。

### テストとドキュメント

- `test/`: 単体テスト
- `integration_test/`: 結合テスト
- `docs/`: ユーザー向け・開発者向けドキュメント

## 4. アプリの起動順

起動まわりは短いので、最初に理解しておくと楽です。

1. `lib/main.dart`
2. `lib/app.dart`
3. オンボーディングまたはホーム画面

起動時には次のような初期化が行われます。

- フォアグラウンドタスク通信用の初期化
- Survey 用通知機能の初期化
- システム UI の設定
- `SharedPreferences`
- Riverpod provider 群

その後 `App` が `MaterialApp` を組み立て、テーマとローカライズを設定し、
オンボーディング画面かホーム画面へ進みます。

## 5. このアプリの設計上の大きな特徴

このリポジトリは、画面ごと・サービスごとに分けるよりも、
**feature 単位**で整理されています。

これは別アプリに作り替えるときに便利です。機能を大きな単位で見られるので、
次のように考えやすくなります。

- ある feature を丸ごと外す
- ある feature の名前や役割を変える
- feature の内部実装だけ差し替える
- 共通基盤を残しつつ、業務ロジックだけ入れ替える

別アプリ化を考えると、この feature ベースの構成はかなり扱いやすいです。

## 6. BirdNET Live 固有の部分

もし新しいアプリが鳥の音声識別ではないなら、次の領域は特に BirdNET 色が強いです。

- `assets/models/`
- `assets/species_data/`
- `lib/features/inference/`
- `lib/features/audio/`
- `lib/features/explore/`
- `lib/features/live/`
- `lib/features/point_count/`
- `lib/features/survey/`

ただし、新しいアプリでもセンサー入力、オフライン推論、現地調査、
セッション記録のような考え方を使うなら、構造自体はかなり流用できます。

## 7. 比較的流用しやすい部分

アプリの題材が変わっても、次は残しやすいです。

- Flutter + Riverpod の起動・依存解決部分
- テーマとローカライズの枠組み
- `SharedPreferences` を使った設定保存
- feature ベースのディレクトリ構成
- 履歴・セッション管理の考え方
- エクスポートの仕組み
- MkDocs によるドキュメント構成

## 8. 別アプリ化するときの最初の一歩

別プロダクトとして作り替えるなら、最初は次の順で考えるのが安全です。

1. ブランド名とアプリの識別子を変える
2. 残す機能と消す機能を決める
3. ホーム画面を新しい目的に合わせて作り替える
4. BirdNET 固有のモデルやデータを差し替える
5. 設定項目や文言を新しいドメインに合わせる

アプリの識別情報を変えるときに重要な場所:

- `pubspec.yaml`
- `android/app/build.gradle`
- `android/app/src/main/AndroidManifest.xml`
- `ios/Runner/Info.plist`

## 9. このコードベースの学び方

Flutter が初めてなら、最初から全部を追わないほうが理解しやすいです。

おすすめの順番は次のとおりです。

1. `README.md` を読む
2. `lib/main.dart` を読む
3. `lib/app.dart` を読む
4. `lib/features/home/` を見る
5. いちばん興味のある feature を 1 つ追う
6. そのあとで provider、service、platform 固有コードへ進む

この順番なら、把握する範囲を少しずつ広げられます。

## 10. まとめ

BirdNET Live は間違いなく Flutter アプリで、再利用しやすい形にもなっています。

- Flutter がクロスプラットフォーム UI を担当する
- Riverpod が状態管理と依存の配線を担う
- `lib/features/` に業務機能がまとまっている
- `lib/shared/` と `lib/core/` に共通基盤が集まっている
- BirdNET 固有のモデル・音声・調査ロジックは比較的まとまっている

そのため、別アプリに変えていくときは、アプリの土台と共通基盤を活かしつつ、
BirdNET 固有の feature を段階的に自分のアプリ向けへ置き換えていくやり方が
取りやすいです。
