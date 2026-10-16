# NEON JUMP - 光の跳躍

ネオンカラーのランナーゲーム。ランキング上位10人のゴーストと同時プレイで、自分の最高記録に挑戦！

![NEON JUMP](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)

## 🎮 ゲーム内容

光の壁を跳んで越え、ネオンの高さを狙え。  
ランキング上位10人のジャンプ軌跡をゴーストとしてリアルタイム表示、彼らを超えよう！

### 特徴
- **🏆 マルチランキング**: 全プレイヤー共通のランキング（上位100保持）
- **👥 ゴースト対戦**: ランキング上位10人のジャンプ軌跡をリアルタイム表示
- **⚡ 変量ジャンプ**: 押下時間でジャンプ高さを変化
- **🔁 ダブルジャンプ**: クールタイム付きの2段ジャンプ
- **✨ ネオンエフェクト**: パーティクル、画面シェイク、グローエフェクト

## 🕹️ 操作方法

| 操作 | 説明 |
|------|------|
| `スペース` / `↑` / `W` / タップ | ジャンプ（長押しで高く） |
| 空中でもう一度押す | ダブルジャンプ（クールタイム 0.9秒） |
| `Escape` | ランキングモーダルを閉じる |

### ダブルジャンプの仕組み
- 着地すると使用可能状態にリセット
- 使用後 0.9秒はクールタイム（再使用不可）
- HUDの左下に `DJ READY`（緑）/ `DJ 0.9s`（ピンク）で表示

## 🚀 動作環境

### 必要条件
- **PHP 8.0+**（FPMまたはCLI）
- **Webサーバー**: nginx / Apache など
- **ブラウザ**: 最新バージョンの Chrome / Firefox / Safari / Edge

### インストール

```bash
# 1. リポジトリをクローン
git clone https://github.com/n-nagmn/neonjump.git

# 2. Webサーバーのドキュメントルートに配置
cp -r neonjump/* /var/www/html/neonjump/

# 3. 権限設定（PHP-FPMが実行ユーザーの場合）
chown -R www-data:www-data /var/www/html/neonjump
chmod 775 /var/www/html/neonjump
chmod 644 /var/www/html/neonjump/*.php /var/www/html/neonjump/*.html
```

### nginx設定例

```nginx
server {
    listen 80;
    server_name example.com;
    root /var/www/html;

    location /neonjump/ {
        try_files $uri $uri/ =404;
        
        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        }
    }
}
```

## 📁 ファイル構成

```
neonjump/
├── index.html          # ゲーム本体（HTML5 Canvas + Vanilla JS）
├── api.php             # サーバーサイドAPI
├── README.md           # このファイル
├── LICENSE             # MIT License
├── records.json        # [自動生成] スコア記録
├── ghosts.json         # [自動生成] ゴースト軌跡データ
└── neonjump.lock       # [自動生成] flock用ロックファイル
```

## 🔌 API

### エンドポイント

| メソッド | エンドポイント | 説明 |
|---------|---------------|------|
| GET | `/api.php?action=rank&limit=10&with_ghosts=1` | ランキング取得（上位N人、ゴースト含む） |
| POST | `/api.php` | スコア記録（`action: record`） |
| POST | `/api.php` | ゴースト記録（`action: ghost`） |
| GET | `/api.php?action=ghost&name=プレイヤー名` | 指定プレイヤーのゴースト取得 |

### スコア記録

```bash
curl -X POST 'http://localhost/neonjump/api.php' \
  -H 'Content-Type: application/json' \
  -d '{
    "action": "record",
    "name": "プレイヤー名",
    "score": 1234
  }'
```

**レスポンス**:
```json
{"status": "success"}
```

### ゴースト記録

```bash
curl -X POST 'http://localhost/neonjump/api.php' \
  -H 'Content-Type: application/json' \
  -d '{
    "action": "ghost",
    "name": "プレイヤー名",
    "score": 1234,
    "frames": [[0, 300, 0], [10, 280, -100], [20, 300, 50]]
  }'
```

**framesの形式**: `[距離(px), y座標(地面基準), 速度(py/s)]` の配列

**レスポンス**:
```json
{"status": "success"}
```

### ランキング取得

```bash
curl 'http://localhost/neonjump/api.php?action=rank&limit=10&with_ghosts=1'
```

**レスポンス**:
```json
{
  "status": "success",
  "ranking": [
    {
      "name": "プレイヤー名",
      "score": 1234,
      "ts": 1792127876,
      "ghost": [[0, 300, 0], [10, 280, -100], [20, 300, 50]]
    }
  ]
}
```

## ⚙️ 設定

### ゲーム定数（index.html内）

```javascript
const GRAVITY = 2600;           // 重力加速度
const JUMP_V = -980;            // 基本ジャンプ速度
const JUMP_V_MAX = -1380;       // 長押し最大のジャンプ速度
const HOLD_RATE = 900;          // 長押し加速率
const DJ_V = -820;              // ダブルジャンプ速度
const DJ_HOLD_MAX = -1150;      // ダブルジャンプ長押し最大
const DJ_COOLDOWN = 0.9;        // ダブルジャンプクールタイム（秒）
const SCROLL_BASE = 330;        // 基本スクロール速度
const MAX_SCROLL = 860;         // 最大スクロール速度
const COYOTE = 0.09;            // コヨーテタイム（秒）
const BUFFER = 0.12;            // 入力バッファ（秒）
```

### サーバー設定（api.php内）

```php
$RECORDS_LIMIT = 100;           // 最大保存ランキング数
$GHOSTS_SIZE_LIMIT = 1500000;   // ゴーストデータサイズ上限（約1.5MB）
$NAME_MAX_LENGTH = 20;          // プレイヤー名最大文字数
```

## 🛡️ セキュリティ

- **flock ロック**: 同時書き込み時の競合防止
- **アトミック書き込み**: tmpファイル経由でファイル破損を防止
- **入力検証**: スコア範囲（0-999999）、フレーム数上限（60000）、名前長制限
- **CORS**: クロスオリジンアクセスを許可（`Access-Control-Allow-Origin: *`）
- **表示制御**: `display_errors` を無効化（本番環境）

## 📊 データ管理

### records.json
- 上位100件のスコアを保持
- 同名プレイヤーは上位スコアのみ更新
- スコア降順ソート

### ghosts.json
- ゴースト軌跡データ（最大約1.5MB）
- 同名プレイヤーは常に最新プレイを上書き
- サイズ超え時は末尾（スコア下位）から削除
- スコア降順ソート

### データのリセット

```bash
# サーバー上で実行
cd /var/www/html/neonjump
sudo rm -f records.json ghosts.json neonjump.lock
```

## 🔧 トラブルシューティング

### APIが動作しない
1. PHP-FPMが起動しているか確認: `systemctl status php8.3-fpm`
2. 権限を確認: `ls -la /var/www/html/neonjump/`
   - ディレクトリ: `www-data` が所有、`775`
   - ファイル: `644`
3. nginxログを確認: `tail -f /var/log/nginx/error.log`

### ゴーストが表示されない
1. APIが正常にゴーストを返しているか確認:
   ```bash
   curl 'http://localhost/neonjump/api.php?action=rank&limit=10&with_ghosts=1'
   ```
2. `ghosts.json` にデータがあるか確認
3. プレイヤーが5m以上のスコアを出しているか確認（それ未満はゴースト送信されない）

### スコアが記録されない
1. POSTリクエストが成功しているか確認
2. `records.json` が書き込めるか権限を確認
3. スコアが有効範囲（0-999999）内か確認

## 📄 License

[MIT License](LICENSE)

## 👤 作者

**nagmn**

- GitHub: [@n-nagmn](https://github.com/n-nagmn)

## 🙏 謝辞

- HTML5 Canvas API
- PHP
- nginx / PHP-FPM

## 📝 更新履歴

### v1.0.0 (2026-10-16)
- 初版リリース
- マルチランキング（上位100保持）
- ゴースト対戦（上位10人リアルタイム表示）
- 変量ジャンプ（長押しで高さ変化）
- ダブルジャンプ（クールタイム付き）
- ネオンビジュアルエフェクト
- ローカル + サーバーサイド高スコア保存
