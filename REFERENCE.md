# Immich Ansible セットアップ - 実行リファレンス

## 📋 ファイル一覧と構成

### 作成されたファイル総数: 12ファイル

```
ansible-immich/
├── 📄 immich-setup.yml              # メイン Playbook（実行コマンドで指定）
├── 📄 ansible.cfg                   # Ansible 全体設定
├── 📄 README.md                     # プロジェクト全体説明
├── 📄 QUICKSTART.md                 # クイックスタート
├── 📄 SETUP_GUIDE.md                # 詳細セットアップガイド
├── 📁 inventories/
│   └── 📄 hosts.ini                 # ホスト定義（必須：IP設定）
├── 📁 group_vars/
│   ├── 📄 immich_servers.yml        # グループ変数（必須：環境設定）
│   └── 📁 immich_servers/           # Vault ファイル用ディレクトリ
├── 📁 roles/immich-setup/
│   ├── 📄 README.md                 # ロール説明
│   ├── 📄 defaults/main.yml         # デフォルト変数
│   ├── 📄 tasks/main.yml            # タスク定義（231行、8セクション）
│   └── 📁 templates/
│       ├── 📄 immich-env.j2         # .env ファイルテンプレート
│       └── 📄 immich-systemd.service.j2  # systemd サービステンプレート
```

## 🚀 最小限の実行手順（3ステップ）

### 1️⃣ SSH キーのセットアップ（初回のみ）

```bash
# VM に SSH キーをコピー
ssh-copy-id -i ~/.ssh/id_rsa.pub immich@10.0.20.13

# 接続確認
ssh immich@10.0.20.13 "echo OK"
```

### 2️⃣ Vault ファイル作成（初回のみ）

```bash
# Vault ファイルを作成
ansible-vault create group_vars/immich_servers/vault.yml
```

以下の内容を入力：
```yaml
---
vault_immich_db_password: "StrongPassword123!"
```

### 3️⃣ Playbook 実行

```bash
# 構文チェック
ansible-playbook immich-setup.yml --syntax-check

# 本実行（Vault パスワード入力が求められる）
ansible-playbook immich-setup.yml --ask-vault-pass
```

## 📝 実行後の確認コマンド

```bash
# VM にログイン
ssh immich@10.0.20.13

# Immich サービス状態
sudo systemctl status immich

# Docker コンテナ一覧
cd ~/immich-app && docker compose ps

# NFS マウント確認
mount | grep immich-data

# ログ確認
sudo journalctl -u immich -n 30
```

## 🔧 主要な変更ポイント

### `inventories/hosts.ini`（必須編集）
```ini
[immich_servers]
immich_vm ansible_host=10.0.20.13 ansible_user=immich  # ← IP アドレスを確認
```

### `group_vars/immich_servers.yml`（環境に応じて編集）
```yaml
# NAS の IP アドレス
nfs_server: 10.0.20.10                    # 環境に合わせて変更

# Immich へのアクセス URL
immich_server_url: https://immich.int.sirosuzu.net

# APT キャッシュサーバー
apt_cache_server: apt.int.sirosuzu.net
```

## 📊 実行されるタスク（8セクション）

| # | セクション | タスク数 | 説明 |
|----|-----------|--------|------|
| 1️⃣ | システム準備 | 5 | タイムゾーン、APT キャッシュ、パッケージ更新 |
| 2️⃣ | NFS マウント | 3 | マウントポイント作成、fstab 設定、マウント |
| 3️⃣ | Docker インストール | 5 | GPG キー、リポジトリ追加、インストール、起動 |
| 4️⃣ | Immich 準備 | 4 | ディレクトリ作成、docker-compose.yml DL、.env 生成 |
| 5️⃣ | systemd 設定 | 4 | サービスファイル生成、デーモンリロード、有効化 |
| 6️⃣ | SSH セキュリティ | 2 | パスワード認証無効化 |
| 7️⃣ | 自動アップデート | 2 | unattended-upgrades 設定 |
| 8️⃣ | ヘルスチェック | 2 | Docker Compose 状態確認、ログ出力 |

**合計: 約 27 タスク**

## 🛠️ デバッグ・トラブルシューティング

### verbose オプションで詳細ログ表示

```bash
# レベル 1（-v）
ansible-playbook immich-setup.yml -v

# レベル 2（-vv）
ansible-playbook immich-setup.yml -vv

# レベル 3（-vvv）最も詳細
ansible-playbook immich-setup.yml -vvv
```

### ドライラン（実行シミュレーション）

```bash
# 何も実行せず、実行予定内容を表示
ansible-playbook immich-setup.yml --check

# verbose 付き
ansible-playbook immich-setup.yml --check -v
```

### 特定タスクのスキップ

```bash
# ssh セキュリティ設定をスキップ
ansible-playbook immich-setup.yml --skip-tags "ssh"

# 複数スキップ
ansible-playbook immich-setup.yml --skip-tags "ssh,nfs"
```

### 特定タスクから実行

```bash
# NFS マウント設定から開始
ansible-playbook immich-setup.yml --start-at-task="NFS マウントポイントを作成"
```

## 🔐 セキュリティ設定

### Vault パスワード管理（推奨）

```bash
# パスワードファイルを作成（600 権限）
echo "my_vault_password" > ~/.vault_pass
chmod 600 ~/.vault_pass

# 以後、パスワード入力不要
ansible-playbook immich-setup.yml --vault-id ~/.vault_pass
```

### Vault ファイルの編集

```bash
# Vault ファイルの確認・編集
ansible-vault edit group_vars/immich_servers/vault.yml
```

## 📈 パフォーマンス

最適化済みの設定（`ansible.cfg`）:
- ✅ Pipelining 有効（SSH 効率化）
- ✅ SSH ControlMaster 有効（接続再利用）
- ✅ タイムアウト: 30秒

**推定実行時間**:
- 初回: 3-5 分
- 再実行: 1-2 分（冪等性）

## 🔄 再実行・更新

Playbook は冪等性があるため、何度実行しても安全です。

```bash
# 環境変数を更新した場合
# → .env ファイルが更新される
# → Immich サービスが自動的に再起動

ansible-playbook immich-setup.yml --ask-vault-pass
```

## 📞 トラブルシューティング参照

以下を参照してください：

1. **README.md** - プロジェクト全体のトラブルシューティング
2. **QUICKSTART.md** - セットアップ各段階での問題
3. **roles/immich-setup/README.md** - ロール固有の問題
4. **SETUP_GUIDE.md** - 詳細なトラブルシューティング

## 🎯 チェックリスト

セットアップ実施時の確認事項：

```
□ Ansible がインストール済み
□ SSH キーが VM に登録されている
□ inventories/hosts.ini で IP アドレスを確認
□ group_vars/immich_servers.yml で環境を確認
□ Vault ファイルを作成済み
□ 構文チェック実行（--syntax-check）
□ ドライラン実行（--check）
□ 本実行実施
□ VM 内で確認（systemctl status immich）
□ Web UI にアクセス可能（https://immich.int.sirosuzu.net:2283）
□ 管理者アカウント作成
□ ストレージテンプレート有効化
```

## 📖 関連ドキュメント

| ドキュメント | 内容 |
|-----------|------|
| README.md | プロジェクト全体 |
| QUICKSTART.md | クイックスタート |
| SETUP_GUIDE.md | 詳細セットアップ |
| roles/immich-setup/README.md | ロール詳細 |
| docs_private/VM/サービス系VM/Immich.md | オリジナル構築手順 |
| docs_private/Proxmox共通.md | Proxmox VE 全般 |

---

**Tips**: Ansible 実行時の出力は自動的に `ansible.log` に記録されます。
