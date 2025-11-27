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
```bash
# Immich — Reference (commands)

このファイルはよく使うコマンドだけを短くまとめたリファレンスです。詳細は `SETUP_GUIDE.md` を参照してください。

インベントリ例:

```ini
[immich_servers]
immich_vm ansible_host=10.0.20.13 ansible_user=immich
```

基本コマンド:

```bash
# 構文チェック
ansible-playbook immich-setup.yml --syntax-check

# 接続確認
ansible all -i inventories/hosts.ini -m ping

# ドライラン
ansible-playbook immich-setup.yml --check

# 本実行（Vault利用時は --ask-vault-pass など）
ansible-playbook -i inventories/hosts.ini immich-setup.yml --ask-vault-pass

# Vault 作成
ansible-vault create group_vars/immich_servers/vault.yml

# サービス確認（実行後）
ssh immich@10.0.20.13 'sudo systemctl status immich'

# Docker Compose 状態
ssh immich@10.0.20.13 'cd ~/immich-app && docker compose ps'

# NFS マウント確認
ssh immich@10.0.20.13 'mount | grep immich-data'
```

トラブルシュートのためのショートカット:

```bash
# 詳細ログ
ansible-playbook immich-setup.yml -vvv

# 特定タスク開始
ansible-playbook immich-setup.yml --start-at-task="NFS マウントポイントを作成"

# タスクをスキップ
ansible-playbook immich-setup.yml --skip-tags "ssh"
```

チェックリスト（要確認）:
- SSH キーが登録されている
- `inventories/hosts.ini` の IP が正しい
- `group_vars/immich_servers.yml` を環境に合わせて編集
- Vault に DB パスワードを設定（推奨）

----
詳細や背景、手順の理由は `SETUP_GUIDE.md` に集約しています。
mount | grep immich-data
