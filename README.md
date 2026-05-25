# MediaMTX Docker Compose

公式 Docker イメージ [`bluenviron/mediamtx:1`](https://hub.docker.com/r/bluenviron/mediamtx) を使い、MediaMTX を Docker Compose で簡単に起動するための構成です。独自の Dockerfile は使用しません。

## 特徴

- RTSP / RTMP / HLS / WebRTC を有効化
- `network_mode: host` を使用し、MediaMTX がホスト上で直接ポートを待ち受け
- タイムゾーンを `Asia/Tokyo` に設定
- 起動、更新、停止は `systemctl` ではなく `docker compose` で管理

## 必要環境

- Linux ホスト
- Docker Engine
- Docker Compose v2 (`docker compose` コマンド)

`network_mode: host` を使用するため、Linux サーバーでの利用を想定しています。Compose ファイルでの `ports:` マッピングは不要です。ファイアウォールやクラウドのセキュリティグループでは、利用するポートを必要に応じて許可してください。

## ファイル構成

| ファイル | 用途 |
| --- | --- |
| `docker-compose.yml` | MediaMTX コンテナの定義 |
| `mediamtx.yml` | RTSP / RTMP / HLS / WebRTC の基本設定 |
| `install.sh` | イメージを取得してサービスを起動 |
| `update.sh` | 最新の同一メジャー版イメージを取得して再作成 |
| `uninstall.sh` | コンテナを停止・削除 |

## インストール

```bash
git clone <this-repository-url>
cd mediamtx-docker-compose-github-bluenviron-mediamtx
chmod +x install.sh update.sh uninstall.sh
./install.sh
```

起動状態とログは `docker compose` で確認できます。

```bash
docker compose ps
docker compose logs -f mediamtx
```

## 更新

```bash
./update.sh
```

`bluenviron/mediamtx:1` の最新イメージを取得し、必要に応じてコンテナを再作成します。

## アンインストール

```bash
./uninstall.sh
```

コンテナは削除されますが、このリポジトリの設定ファイルと取得済み Docker イメージは保持されます。

## 主要ポート

この構成では `network_mode: host` により、以下のポートがホスト上で直接使用されます。

| プロトコル | ポート | トランスポート | 用途 |
| --- | ---: | --- | --- |
| RTSP | `8554` | TCP / UDP | 配信の送信・視聴 (`rtsp://HOST:8554/STREAM`) |
| RTMP | `1935` | TCP | OBS 等からの送信 (`rtmp://HOST:1935/STREAM`) |
| HLS | `8888` | TCP (HTTP) | ブラウザ等での HLS 視聴 (`http://HOST:8888/STREAM/index.m3u8`) |
| WebRTC | `8889` | TCP (HTTP) | WebRTC のシグナリング・閲覧ページ (`http://HOST:8889/STREAM`) |
| WebRTC | `8189` | UDP | WebRTC メディア通信 |

WebRTC をインターネット越しに使用する場合は、`mediamtx.yml` の `webrtcAdditionalHosts` にクライアントから到達可能なホスト名またはグローバル IP アドレスを追加してください。

```yaml
webrtcAdditionalHosts: [stream.example.com]
```

## 利用例

FFmpeg で RTSP ストリームを送信します。

```bash
ffmpeg -re -stream_loop -1 -i input.mp4 -c copy -f rtsp rtsp://localhost:8554/mystream
```

送信したストリームは、次の URL で利用できます。

| 方式 | URL |
| --- | --- |
| RTSP | `rtsp://HOST:8554/mystream` |
| RTMP | `rtmp://HOST:1935/mystream` |
| HLS | `http://HOST:8888/mystream/index.m3u8` |
| WebRTC | `http://HOST:8889/mystream` |

## 設定変更

`mediamtx.yml` を編集後、必要に応じてコンテナを再起動してください。

```bash
docker compose restart mediamtx
```

公式ドキュメント:

- [Installation](https://mediamtx.org/docs/kickoff/installation)
- [Configuration](https://mediamtx.org/docs/features/configuration)
