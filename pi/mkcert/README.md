# pi: mkcert

## 導入手順

### `libnss3-tools` のインストール

`certutil` を含む `libnss3-tools` をインストールする。

```bash
sudo apt install libnss3-tools
```

Linux環境では、NSSベースのブラウザ（Firefoxなど）の証明書ストアを操作する `certutil` が必要

### `mkcert` のダウンロード

ARM用の最新バイナリをダウンロードする。

```bash
wget -q https://dl.filippo.io/mkcert/latest?for=linux/arm -O mkcert
chmod +x mkcert
sudo mv mkcert /usr/local/bin/
```

### ローカルCAの登録

ローカルCA（認証局）を作成し、OSやブラウザに信頼済みとして登録する。

```bash
mkcert -install
```

### ローカル証明書の生成

指定したホスト名やIPアドレスのローカル証明書を生成する。

```bash
mkcert localhost 127.0.0.1 192.168.x.x
```

生成結果の例

```text
./localhost+2.pem
./localhost+2-key.pem
```

## Nginxの設定

Nginxの設定ファイルを作成する。

```bash
sudo vi /etc/nginx/sites-available/mkcert-local.conf
```

### 設定例

```nginx
server {
  listen 443 ssl http2;
  server_name localhost 192.168.x.x;                   # サーバのホスト名やIP

  ssl_certificate     /home/xxxx/localhost+2.pem;      # xxxx: ユーザ名
  ssl_certificate_key /home/xxxx/localhost+2-key.pem;

  root /var/www/html;                                  # Webのルートディレクトリ
  index index.html;

  allow 192.168.x.x;                                   # 許可するクライアントIP
  deny all;

  location / {
    try_files $uri $uri/ =404;
  }

  location /api/ {
    proxy_pass http://localhost:8080/api/;             # バックエンドAPIのURL
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```

## 関連URL

[mkcert GitHub](https://github.com/FiloSottile/mkcert)

[mkcert公式サイト](https://mkcert.org)
