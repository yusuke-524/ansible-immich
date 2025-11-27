# Immich Ansible セットアップ - クイックスタートガイド

## 1. 初期準備（実行前）

### 1.1 Ansible のインストール

```bash
# Ubuntu/Debian
sudo apt install ansible

# macOS
brew install ansible

# Python で実行環境を構築（推奨）
python3 -m venv venv
source venv/bin/activate
pip install ansible==2.10.0 or later
```

### 1.2 プロジェクトのセットアップ

```bash
# リポジトリをクローン
cd ~/Documents/repos/ansible-immich

# 必要なディレクトリ構造を確認
tree -L 2

# 依存するコレクションをインストール（オプション）
ansible-galaxy collection install community.general
```

### 1.3 Vault パスワードファイルの作成

```bash
# .vault_pass を作成（chmod 600）
echo "your_vault_password_here" > .vault_pass
chmod 600 .vault_pass
```

## 2. 設定のカスタマイズ

### 2.1 インベントリの確認

`inventories/hosts.ini` を確認：


# Immich: Quickstart (minimal)

最短でセットアップするための最小手順のみを載せます。詳細は `SETUP_GUIDE.md` を参照してください。

1) インベントリを編集（`inventories/hosts.ini`）

```ini
[immich_servers]
immich_vm ansible_host=10.0.20.13 ansible_user=immich
```

2) Vault（任意だが推奨）

```bash
ansible-vault create group_vars/immich_servers/vault.yml
# 例: vault_immich_db_password: "<strong-password>"
```

3) 構文チェック / 接続確認

```bash
ansible-playbook immich-setup.yml --syntax-check
ansible all -i inventories/hosts.ini -m ping
```

4) 実行

```bash
ansible-playbook -i inventories/hosts.ini immich-setup.yml --ask-vault-pass
```

5) 実行後の簡易確認

```bash
ssh immich@10.0.20.13
sudo systemctl status immich
cd ~/immich-app && docker compose ps
mount | grep immich-data
```

----
このファイルは最小手順に特化しています。手順の背景や詳細・トラブルシューティングは `SETUP_GUIDE.md` にまとめています。
# SSH 接続テスト
