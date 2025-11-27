# Immich Ansible セットアップ - 環境構築ガイド

## プロジェクト概要

このプロジェクトは、Immich フォトマネージメント VM 内部を Ansible で自動セットアップするものです。

**特徴：**
- ✅ 完全変数化（環境ごとに簡単にカスタマイズ可能）
- ✅ Vault による機密情報管理
- ✅ 冪等性（何度実行しても安全）
- ✅ 詳細なドキュメント
- ✅ エラーハンドリング充実

## 作成されたファイル一覧

### ルートディレクトリ

| ファイル | 説明 |
|---------|------|
| `immich-setup.yml` | メイン Playbook |
| `ansible.cfg` | Ansible 全体設定 |
| `README.md` | プロジェクト全体の説明書 |
| `QUICKSTART.md` | クイックスタートガイド |
| `.gitignore` | Git 除外ファイル設定 |

### `inventories/`

| ファイル | 説明 |
|---------|------|
| `hosts.ini` | ホストインベントリ（ターゲットVMの定義） |

### `group_vars/`

| ファイル | 説明 |
|---------|------|
| `immich_servers.yml` | グループ共通変数（環境固有の設定） |
| `immich_servers/vault.yml` | Vault ファイル（機密情報、未作成時は手動作成） |

### `roles/immich-setup/`

#### `defaults/main.yml`
ロールのデフォルト変数。以下を定義：
- ユーザー情報（immich, www-data）
- ディレクトリパス
- NFS マウント設定
- Docker、Immich 環境変数

#### `tasks/main.yml`
実行タスク定義（8つのセクション）：
1. システム準備
2. NFS マウント設定
3. Docker インストール
4. Immich アプリケーション準備
5. systemd サービス設定
6. SSH セキュリティ設定
7. 自動アップデート設定
8. ヘルスチェック

#### `templates/`

| ファイル | 説明 |
|---------|------|
| `immich-env.j2` | Immich `.env` ファイルテンプレート |
| `immich-systemd.service.j2` | systemd サービスファイルテンプレート |

#### `README.md`
ロール固有の詳細ドキュメント

## 変数の階層構造

変数の優先度（高→低）：

```
プレイブック内の vars
  ↓
ホスト変数 (host_vars/<hostname>/)
  ↓
グループ変数 (group_vars/immich_servers/)
  ↓
ロールのデフォルト (roles/immich-setup/defaults/)
```

## セットアップ手順

### ステップ 1: 前提条件確認

```bash
# Ansible インストール確認
ansible --version

# Python 3.6+ インストール確認
python3 --version

# SSH キーのセットアップ
ssh-keygen -t ed25519 -f ~/.ssh/id_immich -C "immich-automation"
ssh-copy-id -i ~/.ssh/id_immich.pub immich@10.0.20.13
```

### ステップ 2: インベントリのカスタマイズ

`inventories/hosts.ini` を環境に合わせて編集：

```ini
[immich_servers]
immich_vm ansible_host=<your_vm_ip> ansible_user=immich
```

### ステップ 3: グループ変数の設定

`group_vars/immich_servers.yml` を編集：

```yaml
# 環境に合わせて値を変更
immich_ip: 10.0.20.13
nfs_server: 10.0.20.10
immich_server_url: https://immich.int.sirosuzu.net
```

### ステップ 4: 機密情報の管理

Vault ファイルを作成：

```bash
ansible-vault create group_vars/immich_servers/vault.yml
```

内容：
```yaml
vault_immich_db_password: "YourSecurePostgresPassword123!"
```

### ステップ 5: 実行

```bash
# 構文チェック
ansible-playbook immich-setup.yml --syntax-check

# ドライラン（何も実行しない）
ansible-playbook immich-setup.yml --check

# 本実行
ansible-playbook immich-setup.yml --ask-vault-pass
```

## 実行結果の確認

### ホスト側

```bash
# ホストに SSH ログイン
ssh immich@10.0.20.13

# Immich サービス状態確認
sudo systemctl status immich

# Docker コンテナ確認
cd ~/immich-app
docker compose ps

# NFS マウント確認
mount | grep immich

# ログ確認
sudo journalctl -u immich -n 20
```

### Web UI

ブラウザで以下にアクセス：
```
https://immich.int.sirosuzu.net:2283
```

初回ログイン：
1. 管理者アカウントを設定
2. 普段使用するユーザーアカウントを作成
3. ストレージテンプレートを有効化

## カスタマイズ例

### メモリ設定の変更

`group_vars/immich_servers.yml` に以下を追加：

```yaml
immich_memory: 8G  # デフォルトは 4G
```

その後、`roles/immich-setup/templates/immich-systemd.service.j2` を編集するか、
または新しいタスクを追加します。

### NFS マウント先の変更

```yaml
immich_data_mount_point: /data/immich  # デフォルト: /mnt/immich-data
immich_upload_location: /data/immich
```

### DB パスワードの複雑さ要件

Vault で強力なパスワードを設定：

```bash
ansible-vault edit group_vars/immich_servers/vault.yml
```

生成例（bash）：
```bash
openssl rand -base64 32 | tr -d '/' | cut -c1-25
```

## トラブルシューティング

### エラー: "NFS マウント失敗"

```bash
# SSH で VM にログイン後
ssh immich@10.0.20.13

# NFS が見えるか確認
showmount -e 10.0.20.10

# 手動マウント試行
sudo mount -v 10.0.20.10:/mnt/pool4tb1/media/immich_photo /mnt/immich-data
```

### エラー: "Docker コンテナ起動失敗"

```bash
# Docker ログ確認
cd ~/immich-app
docker compose logs

# 環境設定ファイル確認
cat .env
```

### エラー: "SSH 接続失敗"

```bash
# SSH の詳細なログを出力
ssh -vvv immich@10.0.20.13

# 公開鍵がサーバーにコピーされているか確認
ssh immich@10.0.20.13 "cat ~/.ssh/authorized_keys | grep $(ssh-keygen -y -f ~/.ssh/id_rsa | md5sum)"
```

## ロールバック手順

実行に失敗した場合：

```bash
# SSH で VM にログイン
ssh immich@10.0.20.13

# Immich サービス停止
sudo systemctl stop immich

# Docker コンテナをクリア
cd ~/immich-app
docker compose down -v

# 必要に応じてディレクトリを削除
rm -rf ~/immich-app

# Playbook を再実行
```

## セキュリティベストプラクティス

1. **Vault でパスワード管理**
   ```bash
   ansible-vault create group_vars/immich_servers/vault.yml
   ```

2. **SSH 鍵認証を使用**
   ```bash
   ssh-key-gen -t ed25519
   ssh-copy-id -i ~/.ssh/id_ed25519.pub immich@10.0.20.13
   ```

3. **inventory ファイルの権限**
   ```bash
   chmod 600 inventories/hosts.ini
   chmod 600 ansible.cfg
   ```

4. **ログファイルは機密情報を含む可能性**
   ```bash
   # ログを保存する場合は保護
   chmod 600 ansible.log
   ```

## 定期実行の設定

Playbook を定期的に実行（例：毎週日曜 2 時）

```bash
# Crontab に以下を追加
0 2 * * 0 cd /path/to/ansible-immich && ansible-playbook immich-setup.yml --extra-vars "ansible_become_pass=$(cat /etc/immich-ansible-pass)" >> /var/log/immich-ansible.log 2>&1
```

## パフォーマンス最適化

`ansible.cfg` で以下を有効化済み：
- `pipelining = True` - SSH パイプラインを有効化
- `ControlMaster = auto` - SSH 接続の再利用

複数ホストの並行実行：

```bash
ansible-playbook immich-setup.yml --forks 5
```

## ドキュメント参照

- 📖 **README.md** - プロジェクト全体の説明
- ⚡ **QUICKSTART.md** - クイックスタート
- 🔧 **roles/immich-setup/README.md** - ロール詳細
- 📋 **docs_private/VM/サービス系VM/Immich.md** - Immich 構築手順（オリジナル）
- 📋 **docs_private/Proxmox共通.md** - Proxmox VE 情報

## 次のステップ

1. **他の VM への展開**
   - OPNsense, TrueNAS, Caddy など用のロールを作成
   - 同じ構造で管理可能

2. **CI/CD パイプライン統合**
   - GitHub Actions で自動実行
   - Ansible Tower で集中管理

3. **モニタリング・ログ集約**
   - Prometheus でメトリクス収集
   - ELK Stack でログ管理

## サポート・フィードバック

構築時に問題が発生した場合：

1. ドキュメント内の「トラブルシューティング」を確認
2. Ansible の `-vvv` フラグで詳細ログを確認
3. `ansible.log` のログファイルを確認

---

**作成日**: 2025年11月27日
**バージョン**: 1.0
**対応環境**: Ubuntu 24.04 LTS, Ansible 2.9+
