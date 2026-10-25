
# curl
- URLを指定してHTTP・HTTPSなどの通信を行うコマンド
- WebサーバーやAPIにリクエストを送り、レスポンスを取得できる
- HTTP通信ではデフォルトでGETメソッドを使用する
- Webサーバーの稼働確認、APIの動作確認、リバースプロキシの接続確認などに利用する

## options

### -i (--include)
- HTTPレスポンスのヘッダーと本文の両方を表示する　
- 使う場面：FastAPIのAPIが正常に応答しているか、HTTPステータスコードと返されたJSONデータを同時に確認したいとき
- ＊サーバーがHTTPリクエストを受け取り、レスポンス本文までしっかり正しく返しているかを確認したい時


```bash
curl -i http://127.0.0.1:8000/openapi.json
```

### -I (--head)
- HTTPのHEADメソッドでリクエストを送り、レスポンスヘッダーのみを取得する 
- 使う場面：NginxのHTTPステータスコードやレスポンスヘッダーを確認したいとき
- ＊サーバーがHTTPリクエストを受け取り、正常に応答しているかだけを確認したい時

```bash
curl -I -H "Host: example.com" http://127.0.0.1/
```

### -H (--header)
- HTTPリクエストに任意のヘッダーを追加・指定する
- 使う場面：Nginxにアクセス先のドメインを伝えたり、FastAPIに認証用パスワードを送信したいとき

```bash
curl -i \
  -H "Host: example.com" \
  -H "enterd-password: $QUIZ_PASSWORD" \
  "http://127.0.0.1/api/quiz?attempt_count=2"
```

- `Host`：Nginxが対応するserverブロックを選ぶ際に使用する
- `enterd-password`：今回のFastAPIで定義している認証用ヘッダー

### -X (--request)
- HTTPリクエストで使用するメソッドを指定する
- 使う場面：GET、POST、PUT、DELETEなどを明示的に指定したいとき

```bash
curl -X GET http://127.0.0.1:8000/openapi.json
```

- GETはデフォルトなので通常は省略可能

### -d (--data)
- HTTPリクエストのボディに送信するデータを指定する
- デフォルトではPOSTメソッドが使用される
- 使う場面：APIにJSON形式の回答データなどを送信したいとき

例（JSONを受け付けるAPIの場合）：

```bash
curl -X POST \
  -H "Content-Type: application/json" \
  -d '{"name":"test"}' \
  http://127.0.0.1:8000/example
```

- JSONを送信するときは、`Content-Type: application/json` を指定する
- 上記は構文説明用の例で、今回のクイズAPIとは異なる

### -v (--verbose)
- 通信の詳細情報を表示する
- 接続先、リクエストヘッダー、レスポンスヘッダーなどを確認できる
- 使う場面：NginxやFastAPIへの通信でエラーが発生した場合に原因を調査したいとき

```bash
curl -v http://127.0.0.1:8000/openapi.json
```

- 認証情報などの秘密情報も表示されることがあるため注意する


