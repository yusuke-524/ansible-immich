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

```ini
[immich_servers]
immich_vm ansible_host=10.0.20.13 ansible_user=immich
```

ホスト名やIPを実環境に合わせて変更してください。

### 2.2 グループ変数の編集

`group_vars/immich_servers.yml` を編集：

```yaml
# 例：別のホストで実行する場合
immich_ip: 10.0.30.13
nfs_server: 10.0.30.10
```

### 2.3 Vault で機密情報を管理（推奨）

Vault ファイルを新規作成：

```bash
ansible-vault create group_vars/immich_servers/vault.yml
```

以下の内容を入力：

```yaml
---
vault_immich_db_password: "your_strong_postgres_password_here"
```

Playbook で参照：

```yaml
immich_db_password: "{{ vault_immich_db_password | default('ChangeMe') }}"
```

## 3. SSH 接続確認

Playbook 実行前に、SSH で接続可能か確認：

```bash
# SSH 接続テスト
ssh immich@10.0.20.13 "whoami"

# SSH 鍵が必要な場合
ssh -i ~/.ssh/id_rsa immich@10.0.20.13 "whoami"

# SSH 公開鍵がまだ登録されていない場合（1回目）
ssh-copy-id -i ~/.ssh/id_rsa.pub immich@10.0.20.13
```

## 4. Playbook の実行

### 4.1 事前チェック（ドライラン）

```bash
# Playbook の構文チェック
ansible-playbook immich-setup.yml --syntax-check

# ホストの接続確認
ansible all -i inventories/hosts.ini -m ping

# ドライラン（実際には何も実行しない）
ansible-playbook immich-setup.yml --check
```

### 4.2 本実行

```bash
# 基本的な実行（SSH 鍵を使用）
ansible-playbook immich-setup.yml

# 詳細なログ出力
ansible-playbook immich-setup.yml -v

# より詳細なログ出力
ansible-playbook immich-setup.yml -vvv

# Vault パスワードを指定
ansible-playbook immich-setup.yml --vault-password-file=.vault_pass
```

### 4.3 実行時の表示例

```
PLAY [Immich VM セットアップ] *********************************************************

TASK [Gathering Facts] *******************************************
ok: [immich_vm]

TASK [immich-setup : タイムゾーン設定] *****************************
ok: [immich_vm]

TASK [immich-setup : APT キャッシュサーバー設定] *********************
changed: [immich_vm]

...

PLAY RECAP *******************************************************
immich_vm : ok=30 changed=12 unreachable=0 failed=0

===================================
Immich セットアップが完了しました
===================================
```

## 5. 実行後の確認

### 5.1 VM 内での確認

```bash
# SSH で VM にログイン
ssh immich@10.0.20.13

# Immich サービスの状態
sudo systemctl status immich

# Docker コンテナの確認
cd ~/immich-app
docker compose ps

# ログの確認
docker compose logs -f

# NFS マウントの確認
mount | grep immich-data
```

### 5.2 Web UI へのアクセス

```
URL: https://immich.int.sirosuzu.net:2283

（初回アクセス時に自己署名証明書の警告が出た場合は進行）
```

### 5.3 初期設定

1. **管理者アカウント作成**
   - メール: admin@example.com
   - パスワード: 強力なパスワード

2. **ユーザーアカウント作成**
   - 通常使用するアカウントを作成

3. **ストレージ確認**
   - 管理 > 設定 > ストレージテンプレート
   - ストレージテンプレートエンジンを有効化

## 6. トラブルシューティング

### 接続エラー

```bash
# SSH を verbose モードで実行
ssh -v immich@10.0.20.13

# Ansible の verbose オプション
ansible-playbook immich-setup.yml -vvv
```

### パーミッションエラー

```bash
# ユーザーが docker グループに所属しているか確認
groups immich

# docker グループに追加（手動）
sudo usermod -aG docker immich
newgrp docker
```

### NFS マウントエラー

```bash
# NAS との接続確認
ping 10.0.20.10

# NFS 共有が見えるか確認
showmount -e 10.0.20.10

# マウント情報確認
cat /etc/fstab

# マウント手動実行
sudo mount -a
```

## 7. ロールバック・リセット

Playbook の実行を失敗した場合のリセット：

```bash
# SSH で VM にログイン
ssh immich@10.0.20.13

# Immich を停止
sudo systemctl stop immich

# Docker コンテナをクリア
cd ~/immich-app
docker compose down -v

# 必要に応じて、再度 Playbook を実行
```

## 8. 定期実行・スケジューリング

Playbook を定期実行する場合：

```bash
# crontab で毎日午前2時に実行
0 2 * * * cd /path/to/ansible-immich && ansible-playbook immich-setup.yml >> /var/log/immich-ansible.log 2>&1
```

## 参考リンク

- [Ansible 公式ドキュメント](https://docs.ansible.com/)
- [Immich 公式ドキュメント](https://docs.immich.app/)
- [Docker 公式ドキュメント](https://docs.docker.com/)
