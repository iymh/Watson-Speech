# Watson Libraries for Embed 使用方法

## 製品情報
https://www.ibm.com/jp-ja/partnerworld/program/embeddableai-jp
* "トライアル" ボタンからデモサイトを表示します
![HP](hp.jpg)

## IBMidの登録
* IBMidでのログインが必要です。
<div>
  <img src="ibmid_login.jpg" width="50%">
</div>

* アカウントが無い場合は作成します
<div>
  <img src="ibmid_create1.jpg" width="50%"><img src="ibmid_create2.jpg" width="50%">
</div>

* Eメールに検証コードが送信されます
<div>
  <img src="ibmid_create3.jpg" width="50%"><img src="ibmid_create4.jpg" width="50%">
</div>

* 多要素認証を登録します(Eメールを選択)
<div>
  <img src="ibmid_create5.jpg" width="50%"><img src="ibmid_create6.jpg" width="50%">
</div>

* Eメールに認証コードが送信されます
<div>
  <img src="ibmid_create7.jpg" width="50%"><img src="ibmid_create8.jpg" width="50%">
</div>

* IBMid登録が成功したとEメールが届いているはずです
<div>
  <img src="ibmid_create9.jpg" width="50%">
</div>

## デモサイト https://dsce.ibm.com
* "Package and deploy AI" を選択します
  * "Start" を押します
<div>
  <img src="dsce_1.jpg" width="80%">
</div>

* "Watson Speech" を選択します
* "Text to Speech" を選択します
<div>
  <img src="dsce_2.jpg" width="80%">
</div>

* "Deploy a model container" を選択します
  * "Next" を押します
<div>
  <img src="dsce_3.jpg" width="80%">
</div>

* 使い方の概要が表示されます
<div>
  <img src="dsce_4.jpg" width="80%">
</div>

* "See build,run,and test steps" を押します
<div>
  <img src="dsce_5.jpg" width="80%">
</div>

## トライアル登録
* "使用方法の手順のページが表示されます
<div>
  <img src="git_1.jpg" width="80%">
</div>

* "entitlement key" のリンクを押します
<div>
  <img src="myibm_1.jpg" width="80%">
</div>

* "Add new key"を押します
  * キーが発行されます。コピーしておきます。
<div>
  <img src="myibm_2.jpg" width="80%">
</div>

* Gitのページの "Get a Watson Text to Speech trial license" のリンクを押します
<div>
  <img src="trial_1.jpg" width="80%">
<div>
</div>
<div>
  <img src="trial_3.jpg" width="80%">
</div>

* "Continue" を押します
<div>
  <img src="myibm_3.jpg" width="80%">
</div>

* Libraryが追加されています
<div>
  <img src="myibm_4.jpg" width="80%">
</div>


## サンプルソース https://github.com/ibm-build-lab/Watson-Speech
* GitコマンドかZipのダウンロードでソースを取得します。
<div>
  <img src="git_2.jpg" width="80%">
</div>

## ”single-container-tts”を構築
* Gitから取得したソースツリーから
* "cd single-container-tts" 
  * 発行したEntitlement Keyを以下パスワード欄に貼る

* Dockerレジストリにログイン
  ```
  docker login cp.icr.io -u cp -p ********** 
  ```
  <div>
    <img src="docker_1.jpg" width="80%">
  </div>

* Dockerビルド
  ```
  docker build . -t tts-standalone
  ```
  <div>
    <img src="docker_2.jpg" width="80%">
  </div>

  * 初回は数十分かかります(8GBほどモジュールをダウンロード)

* Docker実行
  ```
  docker run --rm -it --env ACCEPT_LICENSE=true --publish 1080:1080 tts-standalone
  ```
  <div>
    <img src="docker_3.jpg" width="80%">
  </div>

  * コンテナでWebサーバが起動されています
  * "http://localhost:1080/"で接続確認をします
  <div>
    <img src="docker_4.jpg" width="80%">
  </div>


## 言語モデルの入れ替え
<div>
  <img src="git_3.jpg" width="80%">
</div>

* "Dockerfile"
  ```
  # Add additional models here
  FROM cp.icr.io/cp/ai/watson-tts-en-us-michaelv3voice:1.0.0 AS en-us-voice
  FROM cp.icr.io/cp/ai/watson-tts-ja-jp-emiv3voice:1.0.0 AS ja-jp-voice
  ```
  ```
  # For each additional models, copy the line below with the model image
  COPY --chown=watson:0 --from=en-us-voice model/* /models/pool2/
  COPY --chown=watson:0 --from=ja-jp-voice model/* /models/pool2/
  ```


* /single-container-tts/config/ 下のファイルを修正する
  * "env_config.json"
    ```
    "clusterGroups": {
      "default": {
        "service_type": "text-to-speech",
        "component": "runtime",
        "group": "default",
        "models": [
          "en-US_MichaelV3Voice",
          "ja-JP_EmiV3Voice"
        ]
      }
    },
    "defaultTTSVoice": "ja-JP_EmiV3Voice",
    ```

  * "sessionPools.yaml"
    ```
    PreWarmingPolicy:
    # UPDATE BELOW WITH MODELS USED
      - name: en-US_MichaelV3Voice
      - name: ja-JP_EmiV3Voice
    ```
* 再度ビルドして実行
  * ターミナルからコンテナを終了してから
  ```
  docker build . -t tts-standalone
  ```
  ```
  docker run --rm -it --env ACCEPT_LICENSE=true --publish 1080:1080 tts-standalone
  ```