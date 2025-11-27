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
│   └── hosts.ini                    # ホストインベントリ
├── 📁 group_vars/
│   ├── immich_servers.yml           # グループ変数
│   └── immich_servers/
│       └── vault.yml                # Vault（機密情報）
└── 📁 roles/
    └── immich-setup/                # immich-setup ロール
        ├── tasks/
        ├── defaults/
        ├── templates/
        └── README.md
```

# Immich — 実行リファレンス（抜粋）

よく使うコマンドだけをまとめた短いリファレンスです。詳細は `SETUP_GUIDE.md` を参照してください。

インベントリ例

```ini
[immich_servers]
immich_vm ansible_host=10.0.20.13 ansible_user=immich
```

主要コマンド

```bash
# 構文チェック
ansible-playbook immich-setup.yml --syntax-check

# 接続確認
ansible all -i inventories/hosts.ini -m ping

# ドライラン（実行シミュレーション）
ansible-playbook immich-setup.yml --check

# 本実行（Vault 利用時は --ask-vault-pass）
ansible-playbook -i inventories/hosts.ini immich-setup.yml --ask-vault-pass

# Vault の作成/編集
ansible-vault create group_vars/immich_servers/vault.yml
ansible-vault edit group_vars/immich_servers/vault.yml

# 実行後の確認（VM 側）
ssh immich@10.0.20.13 'sudo systemctl status immich'
ssh immich@10.0.20.13 'cd ~/immich-app && docker compose ps'
ssh immich@10.0.20.13 'mount | grep immich-data'
```

トラブルシュート用ショートカット

```bash
# 詳細ログ
ansible-playbook immich-setup.yml -vvv

# 特定タスクから開始
ansible-playbook immich-setup.yml --start-at-task="NFS マウントポイントを作成"

# タスクをスキップ
ansible-playbook immich-setup.yml --skip-tags "ssh"
```

チェック（実行前）
- SSH キーが登録されている
- `inventories/hosts.ini` の IP を確認
- `group_vars/immich_servers.yml` を環境に合わせて編集
- Vault に DB パスワードを設定（推奨）

----
詳細や背景、手順の理由は `SETUP_GUIDE.md` を参照してください。
