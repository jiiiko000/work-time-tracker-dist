# work-time-tracker

開始時刻を積むだけで工数が貯まる、1人用のセルフホスト型タイムトラッカーです。課題を選んで「開始」を押し、作業が変わったら切り替える。それだけで1日の実績が記録され、あとから履歴の編集・丸め・集計・CSV 出力ができます。

**Docker があれば動きます。** Node のインストールも、このアプリのソースコードも要りません。データはすべて手元の `./data` に貯まり、外部に送信されません。

## 必要なもの

- Docker（`docker compose` が使えること）

## 起動する

空のフォルダを作り、その中で次の2行を実行します。

```sh
curl -O https://raw.githubusercontent.com/jiiiko000/work-time-tracker-dist/main/docker-compose.yml
docker compose up -d
```

ブラウザで http://localhost:8080 を開きます。止めるときは `docker compose down` です。

compose を使わない場合はこちら（同じ内容です）。

```sh
mkdir -p data
docker run -d --name work-time-tracker \
  -p 127.0.0.1:8080:3000 \
  -e TZ=Asia/Tokyo -e DB_PATH=/data/app.db \
  -v "$PWD/data:/data" \
  --restart unless-stopped \
  ghcr.io/jiiiko000/work-time-tracker:latest
```

- イメージは公開されているので `docker login` は要りません。
- Linux では、コンテナが root で動くため `./data` が root 所有になることがあります。先に `mkdir -p data` してから起動すると詰まりにくくなります。詰まったら `sudo chown -R "$USER" data` で戻せます。

## 設定できること

環境変数で動きを変えられます。**左の「アプリ単体の既定値」は Docker を使わずに動かす場合の値で、Docker で動かすときは一部が違います。** 実際に効いている値（実効値）は次のとおりです。

| 環境変数 | アプリ単体の既定値 | コンテナでの実効値 | 内容 |
| --- | --- | --- | --- |
| `HOST` | `127.0.0.1` | `0.0.0.0` | 待ち受けアドレス。認証がないため、既定では localhost からのみ。コンテナでは `0.0.0.0` で待ち受け、ホスト側は compose の `ports` で `127.0.0.1` に絞っています |
| `PORT` | `3000` | `3000` | コンテナ内で待ち受けるポート |
| `DB_PATH` | `./data/app.db` | `/data/app.db` | データベースの保存先（コンテナでは `/data` の中だけを使います） |
| `BACKUP_KEEP` | `14` | `14` | 自動バックアップの残す世代数 |
| `TZ` | `Asia/Tokyo` | `Asia/Tokyo` | 日付の境界に使うタイムゾーン。画面は `Asia/Tokyo` 固定で解釈するため、変えると画面とずれる場合があります |

設定を変えるときは `docker-compose.yml` の `environment` に足します。ただし次の3つに注意します。

- **`DB_PATH` は必ず `/data` の中にします。** 表の `./data/app.db` はアプリ単体の値で、コンテナでそのまま使うと `/data` の外（`/app/data/app.db`）を指し、`./data` に残らずコンテナを作り直すと消えます。標準の compose は `./data:/data` をマウントしているので、既定の `/data/app.db` から変えないでください。
- **`PORT` を変えるときは、`ports` の右側も同じ値にします。** たとえば `PORT=8080` にするなら `ports` を `"127.0.0.1:8080:8080"` にします。左側はホストから開く番号、右側はコンテナ内の番号で、片方だけ変えるとつながりません。あわせて `healthcheck` の `test` に書いてある `3000` も同じ値に変えてください。そのままだと `unhealthy` と表示されます（アプリ自体は動いています）。
- **`HOST` は標準の compose では変えません。** `0.0.0.0` でないとホストからつながりません。外からの見え方は `ports` の左側で調整します（例: `"127.0.0.1:9090:3000"` にすると http://localhost:9090 で開きます）。

詳しくは [MANUAL.md](MANUAL.md) を見てください。

## データとバックアップ

- データは起動したフォルダの `./data` に貯まります（バックアップも同じ場所）。
- 起動時に1回、そのあと1時間ごとに、当日分がまだ無ければ `./data/backups/` に1日1つバックアップを作ります（`app-YYYY-MM-DD.db`）。失敗してもアプリは止まりません。
- 更新は `docker compose pull` → `docker compose up -d` です。データは `./data` にあるのでそのまま残ります。
- 復元の手順も含めて、詳しくは [MANUAL.md](MANUAL.md) を見てください。

## マニュアル

画面ごとの使い方と、バックアップ・復元・更新の手順は [MANUAL.md](MANUAL.md) にまとめてあります。

## ライセンス

Apache License 2.0（`LICENSE` を参照）。

<!-- 同期の確認用。あとで消します -->
