# Immich Ansible セットアップガイド

このドキュメントは詳細な手順、変数リファレンス、トラブルシューティングを提供します。
クイックスタートは [README.md](README.md) を参照してください。

## 目次

1. [前提条件](#前提条件)
2. [設定ファイル詳細](#設定ファイル詳細)
3. [コマンドリファレンス](#コマンドリファレンス)
4. [トラブルシューティング](#トラブルシューティング)

---

## 前提条件

### コントロールノード（Ansible 実行側）

```bash
# Ubuntu/Debian
sudo apt install ansible

# macOS
brew install ansible

# Python venv（推奨）
python3 -m venv venv && source venv/bin/activate
pip install ansible

# コレクションのインストール
ansible-galaxy collection install community.general
```

### ターゲットノード（Immich VM）

- Ubuntu 24.04 LTS
- SSH 公開鍵認証でログイン可能
- sudo 権限（NOPASSWD 推奨、または `-K` オプションで対応）
- NFS サーバーへのネットワーク接続

---

## 設定ファイル詳細

### `inventories/hosts.ini`

```ini
[immich_servers]
immich_vm ansible_host=10.0.20.100 ansible_user=immich

[immich_servers:vars]
ansible_python_interpreter=/usr/bin/python3
```

### `group_vars/immich_servers.yml`

環境固有の設定を定義します（必須）。

```yaml
# ユーザー情報
immich_user: immich
immich_group: www-data

# ホスト名
immich_vm_hostname: pve-immich

# ネットワーク設定
immich_ip: 10.0.20.100

# NFS マウント設定（必須）
nfs_server: 10.0.20.10
nfs_export_path: /mnt/pool/immich_data

# Immich 設定
immich_server_url: http://10.0.20.100

# APT キャッシュサーバー（オプション）
apt_cache_server: apt.example.com
apt_cache_port: 3142

# タイムゾーン
timezone: Asia/Tokyo
```

### `group_vars/immich_servers/vault.yml`

機密情報を Vault で暗号化して管理します。

```bash
# 作成
ansible-vault create group_vars/immich_servers/vault.yml

# 編集
ansible-vault edit group_vars/immich_servers/vault.yml

# 内容例
vault_immich_db_password: "your-strong-password-here"
```

---

## コマンドリファレンス

### 基本コマンド

```bash
# 構文チェック
ansible-playbook immich-setup.yml --syntax-check

# 接続テスト
ansible all -i inventories/hosts.ini -m ping -K

# ドライラン（実際には変更しない）
ansible-playbook -i inventories/hosts.ini immich-setup.yml --check

# 本実行
ansible-playbook -i inventories/hosts.ini immich-setup.yml -K --ask-vault-pass

# 詳細ログ付き実行
ansible-playbook -i inventories/hosts.ini immich-setup.yml -K --ask-vault-pass -vvv
```

### 部分実行

```bash
# 特定タスクから開始
ansible-playbook -i inventories/hosts.ini immich-setup.yml --start-at-task="Docker CE をインストール"

# タグでフィルタ（タグが定義されている場合）
ansible-playbook -i inventories/hosts.ini immich-setup.yml --tags "docker"
ansible-playbook -i inventories/hosts.ini immich-setup.yml --skip-tags "ssh"
```

### 変数確認

```bash
# 特定変数の値を確認
ansible immich_vm -i inventories/hosts.ini -m debug -a "var=nfs_export_path"

# 複数変数を確認
ansible immich_vm -i inventories/hosts.ini -m debug -a 'msg="NFS={{ nfs_server }}:{{ nfs_export_path }}"'
```

### VM 側での確認

```bash
# SSH 接続
ssh immich@10.0.20.100

# Immich サービス状態
sudo systemctl status immich

# Docker コンテナ状態
cd ~/immich-app && docker compose ps

# ログ確認
docker compose logs --tail=100 immich-server
docker compose logs --tail=100 database

# NFS マウント確認
mount | grep immich-data
```

---

## トラブルシューティング

### SSH 接続エラー

```
UNREACHABLE! => {"msg": "Permission denied (publickey,password)."}
```

**原因**: SSH 公開鍵が登録されていない
**対処**:
```bash
ssh-copy-id immich@10.0.20.100
```

### sudo パスワードエラー

```
Missing sudo password
```

**原因**: become（sudo）にパスワードが必要
**対処**:
```bash
# 実行時にパスワードを入力
ansible-playbook ... -K

# または Vault に保存
# vault.yml に追加: ansible_become_password: "password"
```

### NFS マウントエラー

```
mount.nfs: access denied by server while mounting ...
```

**原因**: NFS サーバー側でエクスポートされていない、または権限がない
**対処**:
1. NFS サーバーで `showmount -e` を実行してエクスポート一覧を確認
2. `group_vars/immich_servers.yml` の `nfs_export_path` を正しいパスに修正
3. NFS サーバー側で適切な権限を設定

### group_vars が読み込まれない

```
nfs_server is defined and nfs_server != "" ... evaluated_to: false
```

**原因**: `group_vars` が正しく読み込まれていない
**対処**: `immich-setup.yml` に `vars_files` を明示的に指定（現在は設定済み）

### Postgres 起動失敗

```
chown: changing ownership of '/var/lib/postgresql/data': Operation not permitted
```

**原因**: NFS 上で PostgreSQL データを配置しようとしている（root_squash による権限エラー）
**対処**: DB データはローカルストレージに配置（`immich_db_data_location: /var/lib/immich-db`）

### Docker サービス名解決エラー

```
Error: getaddrinfo ENOTFOUND immich-db
```

**原因**: `.env` の `DB_HOSTNAME` が docker-compose のサービス名と一致していない
**対処**: `.env` を確認し、`DB_HOSTNAME=database`、`REDIS_HOSTNAME=redis` に修正

---

## 変数一覧

| 変数名 | デフォルト値 | 説明 |
|--------|--------------|------|
| `immich_user` | `immich` | アプリケーション実行ユーザー |
| `immich_group` | `www-data` | アプリケーション実行グループ |
| `immich_home_dir` | `/home/immich` | ホームディレクトリ |
| `immich_app_dir` | `/home/immich/immich-app` | アプリディレクトリ |
| `immich_data_mount_point` | `/mnt/immich-data` | NFS マウントポイント |
| `immich_db_data_location` | `/var/lib/immich-db` | DB データディレクトリ（ローカル） |
| `nfs_server` | （必須） | NFS サーバー IP |
| `nfs_export_path` | （必須） | NFS エクスポートパス |
| `nfs_mount_opts` | `defaults` | NFS マウントオプション |
| `timezone` | `Asia/Tokyo` | タイムゾーン |
| `immich_server_url` | （必須） | Immich アクセス URL |
| `immich_db_password` | Vault 参照 | PostgreSQL パスワード |

---

## 関連ドキュメント

- [README.md](README.md) — クイックスタート
- [roles/immich-setup/README.md](roles/immich-setup/README.md) — ロール詳細
