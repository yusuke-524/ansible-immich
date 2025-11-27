# Ansible Immich Setup

Immich フォトマネージメントアプリケーション の Ansible 自動化スクリプト

## 概要

このプロジェクトは、Ubuntu VM上で Immich を自動デプロイします。以下を自動化します：

- システム初期設定（タイムゾーン、APT キャッシュ設定）
- NFS マウント設定（TrueNAS からのストレージマウント）
- Docker・Docker Compose インストール
- Immich アプリケーションのダウンロードと設定
- systemd サービスの設定
- SSH セキュリティ設定
- 自動アップデート設定

## 前提条件

### ホスト側（実行環境）
- Ansible 2.9 以上
- SSH クライアント
- Python 3.6 以上

### ターゲット VM 側
- Ubuntu 24.04 LTS
- SSH でアクセス可能
- sudo 権限を持つユーザー（デフォルト: `immich`）
- インターネット接続

## ディレクトリ構成

```
ansible-immich/
├── immich-setup.yml           # メイン Playbook
├── ansible.cfg                 # Ansible 設定
├── inventories/
│   └── hosts.ini              # ホストインベントリ
├── group_vars/
│   └── immich_servers.yml     # グループ変数
├── roles/
│   └── immich-setup/
│       ├── defaults/
│       │   └── main.yml       # ロール用デフォルト変数
│       ├── tasks/
│       │   └── main.yml       # タスク定義
│       └── templates/
│           ├── immich-env.j2              # .env テンプレート
│           └── immich-systemd.service.j2  # systemd サービステンプレート
└── docs_private/              # プロジェクトドキュメント
```

## 使用方法

### 1. 環境変数のカスタマイズ

`group_vars/immich_servers.yml` を環境に合わせて編集：

```yaml
# IP アドレス設定
immich_ip: 10.0.20.13
nfs_server: 10.0.20.10

# URL設定
immich_server_url: https://immich.int.sirosuzu.net

# その他環境に合わせた設定
```

### 2. Vault を使用したパスワード管理（推奨）

PostgreSQL パスワードなど機密情報は Ansible Vault で管理：

```bash
# Vault ファイルを作成（初回のみ）
ansible-vault create group_vars/immich_servers/vault.yml

# 以下の内容を入力
vault_immich_db_password: "YourStrongPasswordHere"
```

### 3. Playbook の実行

```bash
# SSH鍵を使用した実行（推奨）
ansible-playbook -i inventories/hosts.ini immich-setup.yml

# パスワード認証を使用する場合
ansible-playbook -i inventories/hosts.ini immich-setup.yml -k

# Vault パスワードを入力する場合
ansible-playbook -i inventories/hosts.ini immich-setup.yml --ask-vault-pass
```

### 4. 実行結果の確認

Playbook 実行後、以下で確認：

```bash
# SSH で VM にログイン
ssh immich@10.0.20.13

# Immich サービスの状態を確認
sudo systemctl status immich

# Docker Compose の状態を確認
cd ~/immich-app
docker compose ps

# NFS マウント確認
mount | grep immich-data
```

## 各タスクの詳細

### 1. システム準備
- タイムゾーンを Asia/Tokyo に設定
- APT キャッシュサーバーを設定
- システムパッケージを更新
- 必要なツール（nfs-common、docker など）をインストール

### 2. NFS マウント設定
- マウントポイント `/mnt/immich-data` を作成
- `/etc/fstab` に NFS マウント情報を追加
- NFS を実際にマウント
- マウント確認を実施

### 3. Docker インストール
- Docker 公式リポジトリを追加
- Docker CE と docker-compose-plugin をインストール
- Docker サービスを有効化・起動
- immich ユーザーを docker グループに追加

### 4. Immich アプリケーション設定
- 作業ディレクトリ `~/immich-app` を作成
- 公式リポジトリから最新の docker-compose.yml をダウンロード
- Jinja2 テンプレートから `.env` ファイルを生成
- 環境変数を設定（DB パスワード、タイムゾーン、URL など）

### 5. systemd サービス設定
- Immich サービスユニットファイルを生成
- systemd でサービス起動・停止を管理
- 自動起動を有効化

### 6. SSH セキュリティ
- パスワード認証を無効化（公開鍵認証のみ）

### 7. 自動アップデート
- unattended-upgrades を設定

## トラブルシューティング

### NFS マウント失敗
```bash
# NFS サービスが動作しているか確認（VM内）
sudo systemctl status nfs-client

# NAS 側の NFS 共有を確認
showmount -e 10.0.20.10

# マウント手動実行
sudo mount 10.0.20.10:/mnt/pool4tb1/media/immich_photo /mnt/immich-data
```

### Docker の起動失敗
```bash
# Docker サービスの状態確認
sudo systemctl status docker

# Docker ログ確認
sudo journalctl -u docker -n 50

# immich ユーザーで docker 実行確認
docker ps
```

### Immich サービスが起動しない
```bash
# systemd のログを確認
sudo journalctl -u immich -n 50

# docker-compose.yml の妥当性確認
cd ~/immich-app
docker compose config

# 手動で起動試行
cd ~/immich-app
docker compose up -d
```

## カスタマイズ

### 環境変数の追加

`group_vars/immich_servers.yml` に変数を追加し、テンプレートで参照：

```yaml
# group_vars/immich_servers.yml
custom_setting: value
```

```jinja2
{# templates/immich-env.j2 #}
CUSTOM_SETTING={{ custom_setting }}
```

### タスクの追加

`roles/immich-setup/tasks/main.yml` に新しいタスクセクションを追加

## セキュリティ考慮事項

1. **パスワード管理**: 機密情報は Ansible Vault で管理
2. **SSH 認証**: 公開鍵認証のみ有効化
3. **NFS 権限**: データセットの UID/GID を www-data(33) に設定
4. **ファイアウォール**: 本番環境ではファイアウォール設定を確認

## ドキュメント参照

詳細な構築ドキュメント：
- `docs_private/VM/サービス系VM/Immich.md` - Immich 構築手順
- `docs_private/Proxmox共通.md` - Proxmox VE 全般情報

## ライセンス

プロジェクトドキュメントは内部用です

## サポート

問題が発生した場合は、以下を確認してください：
1. ドキュメント内のトラブルシューティング
2. VM のログ出力（journalctl）
3. Docker の状態（docker ps、docker logs）
