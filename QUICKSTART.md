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


# Immich — クイックスタート

ここでは最短で環境を立ち上げるための最小手順だけを示します。手順の背景や詳細、トラブルシューティングは `SETUP_GUIDE.md` を参照してください。

1) インベントリを編集

`inventories/hosts.ini` を開き、ターゲット VM の IP とユーザーを設定します。

```ini
[immich_servers]
immich_vm ansible_host=10.0.20.13 ansible_user=immich
```

2) Vault（機密情報の管理、推奨）

```bash
ansible-vault create group_vars/immich_servers/vault.yml
# 例: vault_immich_db_password: "<strong-password>"
```

3) 構文チェックと接続確認

```bash
ansible-playbook immich-setup.yml --syntax-check
ansible all -i inventories/hosts.ini -m ping
```

4) Playbook 実行

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
短くまとまった手順のみを載せています。詳細な手順やトラブルシューティングは `SETUP_GUIDE.md` を参照してください。
