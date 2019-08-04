## イントロ
---
### UiPath 開発に便利な「Attended Framework」テンプレート

- 「設定ファイルを読み込むための処理を、別ワークフローからコピペして、、」
- 「エラー処理(例外処理)」
- 「ログ出力形式」など

---

### UiPath開発において、**共通化・ルール化したほうがよいモノがいくつかあったり**します。

---
### UiPath 社が [UiPath Go!](https://go.uipath.com/)で公開している「Attended Framework」テンプレートを用いることで、これらの**「メンドクサイ、、だけど大事」な部分をテンプレートにおまかせする**ことができます。


---
![000.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/73777/3e2ca286-89a8-9b7b-16be-1ec35da28737.png)

---
![001.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/73777/66a0ba62-a596-2fc7-03a3-0d37c4e510d8.png)

---
### この記事の対象の方
- プロジェクトを作るたびに、設定ファイルExcelを読み込んでDataTableへ入れるSequenceのコピペをしている方
- 定番の例外処理はどうやるんだっけ？という方
- ElasticsearchやKibanaでのログの収集・解析に興味がある方

---
## 準備・環境

- UiPath Studioは、2018.4.5を用いています。https://go.uipath.com/component/attended-framework を見ると Studio 2018.3.2で開発って書いてありますので、バージョンはそれほどシビアじゃなさそうです。

---
## やってみる
---

### UiPath Studio で開いてつかってみる

- テンプレートはメインの業務処理を ``Process.xaml`` に書いていく仕組み
- (デモ実行)

---
## いいところやTIPS集

---
### 設定ファイル読み込みの標準化
- **``Data/Config.xlsx``に、設定値をKey/Value形式で管理する機能**を提供してくれます。

```
in_Config("logF_BusinessProcessName")  → AttendedFramework  
```

---

- ``in_Config``変数は ``Process.xaml`` 全域(?)で利用可能
![T03_0.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/73777/150e0b2d-b012-50a4-be7b-fd5b44d23495.png)

---
```
in_Config("logF_BusinessProcessName") = AttendedFramework
in_Config("Enable_Screenshot") = False  ← あとで出てくる、ApplicationException発生時にスクリーンショットをとるかどうかフラグ
...
in_Config("logF_RobotType") = Attended
in_Config("Framework_Version") = 1.1.2.0
```
- Excelファイルに追記することで、もちろん任意の値を追加することが可能です。

---
### 定番の例外処理
- テンプレートは``Process.xaml`` をinvokeしている箇所をまるっと ``Try/Catch/Finally`` で囲っている
![T03.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/73777/db2b3030-067a-5ed7-1a46-314205e4d474.png)

---
- 続いてFinallyで実際の例外処理(含む正常時の処理)が行われます。
![T05.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/73777/3e0a7816-4f82-c38b-7c75-a8d30bcbba7b.png)

---
### Finallyの条件分岐

- 例外が発生しなかったとき
    - 正常終了のログ出力("Process successful.")
- BusinessRuleExceptionがスローされたとき、
    - 特に何もせず、発生した例外を再スロー
- それ以外の例外がスローされたとき 
    - スクリーンショットをとって、発生した例外を再スロー

---
### デフォルトのテンプレはたいしたことをしていないので、**各社のRPA導入プロジェクトでルールを定め、定めた処理をココに追記しそれを自社独自のテンプレとして配布する**ようにしましょう。

---
### **テンプレ側に記述する、ルールとして定める例外処理の例**

- 「``Process.xaml`` を作成する開発者は、業務例外(ビジネス例外)発生時は``BusinessRuleException``をスローすること」としておき、
- BusinessRuleExceptionがスローされたら、Message Boxでワークフローを起動した当人にエラーを通知
- それ以外の例外がスローされたら、(たとえば周辺システムの)想定外の障害の可能性があるので、システム管理者にもメールで通知

などが考えられます。

---
### ログ機能について。

- テンプレートではFinallyの処理のところで「ログフィールドを追加」アクティビティでログのフィールドを追加しています。
![E01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/73777/d22a67f1-9381-8c78-ceb0-e7104871b7f2.png)

---

- ワークフロー終了時に自動出力される実行ログ( C:\Users\[ユーザ名]\AppData\Local\UiPath\Logs]\2019-08-03_Execution.log など)にも、

```json:(2019-08-03_Execution.logのある1行。整形してます)
{
  "message": "AttendedFramework の実行が終了しました",
  "level": "Information",
  "logType": "Default",
  "timeStamp": "2019-08-03T22:53:08.5117584+09:00",
  "fingerprint": "f18c7d9b-7871-4022-a413-1f849f86e351",
  "windowsIdentity": "WINDOWS\\m-kino",
  "machineName": "WINDOWS",
  "processName": "AttendedFramework",
  "processVersion": "1.0.0.0",
  "jobId": "f7b31386-dd88-4b15-ba3f-e67d4ec1f78e",
  "robotName": "WINDOWS_m-kino",
  "machineId": 6,
  "totalExecutionTimeInSeconds": 1,
  "totalExecutionTime": "00:00:01",
  "fileName": "Main",
  "logF_RobotType": "Attended",  ←コレ
  "logF_BusinessProcessName": "AttendedFramework",   ←コレ
  "logF_TransactionStatus": "ApplicationException",   ←コレ
  "logF_TransactionID": "WINDOWS\\m-kino_WINDOWS_2019/08/03_22:53:07.417"   ←コレ
}
```
と追加フィールドが出力されるようになりました。

- とくに``logF_TransactionStatus`` は先の条件分岐のどこを通ったかを示す結果ステータスなので、このフィールドを確認することでワークフローの正常/異常を確認することができます。

---

### ログ機能について補足

- さらにOrchestratorに接続されているロボットの場合、これらのログは自動的にOrchestratorにアップされるようになっています[^3]。そしてOrchestratorを導入することで、これら**アップされた大量のログを使って、ワークフローの稼働件数や稼働時間を集計する**ことができたりします。

- それをするには大量のログから上記の「UiPathが自動出力する実行完了ログ」だけ取り出してその件数を集計すればよいので、このログを区別する情報がほしいですが、よく見るとこの「実行完了ログ」にだけ ``totalExecutionTime`` フィールドが存在するので、結論をいうと**totalExecutionTimeフィールドが存在するログだけを取り出せばOK**です。

- Attended Frameworkはさらにこのログに「ワークフローのステータス」も出力するようになっているので、正常終了ワークフロー件数やエラーワークフロー件数などを簡単に集計することができるようになっています。

- 下記のキャプチャは**Elasticsearchに蓄積された大量ログのうち、実行完了ログだけをKibanaで抽出し、Kibanaのダッシュボードでロボット達の稼働状況の可視化を行ったサンプル**です。
![D01.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/73777/8993812b-6b21-2d58-0e5e-8e5fdc2654b9.png)

- Kibanaは上記のように「処理件数」「エラー件数」「ロボごとの、時間帯ごとの稼働件数」「指定した時間帯のエラーメッセージ」などなど、ユーザが必要とする情報をキレイに可視化することができて、とてもイイですね。

- またこのAttended Frameworkが出力するログフォーマットは、さきほどの RE Framework(Robotic Enterprise Framework) と同じフォーマットになっているので、RE Frameworkの出力ログ向けに作られたKibanaのダッシュボードは、そのままAttended Frameworkのワークフローでも利用可能になっているそうです。いい感じです。

---
Attended Frameworkのよいところのご説明でした。

---
### テンプレートとして保存しておくと便利

さてダウンロード・解凍したテンプレートファイルをそのまま使ってきましたが、UiPath Studioで開いた後「テンプレートとして保存」をクリックすると
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/73777/ab218267-c7f0-c177-d860-357835a0647b.png)


テンプレート作成画面が表示されます。適当に名前を入力後「作成」をクリックしてテンプレートを作成すると、、、
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/73777/6e76a223-5a9f-dcd9-3d5e-93693423fea4.png)

プロジェクトの新規作成画面に、自分が作成したテンプレートが表示されるようになりました！
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/73777/000491ec-96fa-b142-1d51-91a61bab27e9.png)


他のテンプレートと同様、今後はココからプロジェクトを作成すれば、まさにテンプレートとして使用することが可能です。
![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/73777/b7714a70-5e8b-e5ca-94be-815a94a2b654.png)

---
### テンプレートを複数人で共有する

例外処理のところで「テンプレートには社内独自の例外処理を追記していきましょう」と書きましたが、このようにStudioで改修したプロジェクトファイル群をそのままzipなどでアーカイブして、テンプレートして使いたい方(社内のプロジェクトなら、UiPath Studioをお使いの各開発者)に配布しましょう。配布されたzipファイルは、各人が自分のUiPath Studioで一度開いて、上記の「テンプレートとして保存」をおこなう事で、独自のテンプレートを複数人で共有することが可能となります。

---
## まとめ

- 設定ファイル読み込みを自動でやってくれるのはとても助かる
- 定番の例外処理も、BusinessRuleException などUiPath社の指針が示されているので、例外設計がしやすくなる。
- RE Frameworkとも思想が類似しているので、今後RE Frameworkを理解するための事前知識としてちょうどよい
- ログ出力もElasticsearchやKibanaなどによる集計も見据えた設計になっていて、とても参考になる
- UiPath Go!にサインアップしないとダウンロード出来ないのは非常にメンドクサイ


などなど。**目先の作業をさらっと自動化するなど、小規模の場合でも生産性は落ちない**ので、使わない手はないなーーとおもいました。


おつかれさまでした。

---
## 追記

テンプレートっていちど使い始めると「テンプレート自体に更新がはいったとき」に、どうやって各開発者の端末に反映させるの？って話がでてきます。
たとえば、テンプレ側に記載された例外処理を変更した場合とかですが、、通常であればテンプレを再配布することになりますよね。再配布ってメンドウだし、テンプレを利用している側も、``Process.xaml`` 以外を新しいテンプレに差し替えて、、、などなどメンドウですね。 

当方では Attended Framework が提供する機能を「カスタムアクティビティ」にしておき、テンプレートがそのカスタムアクティビティを呼び出すようにしたテンプレートを作ったりしています。テンプレート開発者がテンプレを改修したいときは、

- テンプレ開発者はカスタムアクティビティを改修・リリースする
- ``Process.xaml`` 開発者側は、テンプレが依存しているアクティビティのバージョンを変更する

ことで、その改修を反映させることができるようになりました。そのカスタムアクティビティはOrchestratorのNugetフィードを使って配布できるので、テンプレ利用者側の負担はほぼなくなり、とても使い勝手がよくなります。


また、Attended Frameworkの設定ファイルは、配置場所が「``Data/Config.xlxs``」に固定化されているので「開発時は他の設定ファイルを見たいんだよなぁ」なんて事がやりたくなります。ってことで「xxにあるファイルを優先して参照、なかったら``Data/Config.xlxs``をつかう」なんて機能も入れてみました。


Attended Framework の拡張版は下記で配布しているので、興味があったら見てみてください。

- https://github.com/masatomix/My_Attended_Framework

---
## 関連リンク

- [Attended Framework(UiPath Go!)](https://go.uipath.com/ja/component/attended-framework)
- [Enhanced REFrameWork(UiPath Go!)](https://go.uipath.com/ja/component/enhanced-reframework-57011)
- [自分が書いてるUiPathのQiita記事一覧](https://qiita.com/search?q=user%3Amasatomix+tag%3Auipath&sort=like)
- [Attended Framework にすこし改修を加えたテンプレート](https://github.com/masatomix/My_Attended_Framework)
- [そのテンプレートで使用するカスタムライブラリを含んだアクティビティパッケージ](https://github.com/masatomix/kino.UiPath.MyAttendedFramework)





[^1]: 記事を書こうと再ランしてみたら、ダウンロードリンクがいつまでも 「プロフィール入力」ってなってたり、Safariでダウンロードすると空振りしたり、、なんだか不安定な感じでした。。何かったら中のヒトに問い合わせてみましょう！
[^2]: ようになっています、というかそのデータが再度キューに投入されるので、結果リトライされるってことですね
[^3]: さらにElasticsearch/Kibanaを使用している場合はそちらにも連携されます
