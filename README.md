# Ansible Immich Setup

Immich フォトマネージメントを Ubuntu VM 上に自動構築する Ansible Playbook です。

## 機能

- NFS マウント設定（TrueNAS 連携）
- Docker / Docker Compose のインストール
- Immich アプリケーションのデプロイ
- systemd によるサービス化と自動起動
- セキュリティ設定（SSH 強化、自動アップデート）

## 必要条件

- Ansible 2.9 以上
- ターゲット: Ubuntu 24.04 LTS（SSH 接続可能、sudo 権限）
- NFS サーバー（TrueNAS 等）が稼働していること

## クイックスタート

### 1. 設定ファイルを編集

```bash
# インベントリ（ターゲット VM の IP を設定）
vim inventories/hosts.ini

# 環境変数（NFS パス、URL 等を設定）
vim group_vars/immich_servers.yml
```

`inventories/hosts.ini` の例:
```ini
[immich_servers]
immich_vm ansible_host=10.0.20.100 ansible_user=immich
```

`group_vars/immich_servers.yml` で必須の設定:
```yaml
nfs_server: 10.0.20.10
nfs_export_path: /mnt/pool/immich_data
immich_server_url: http://10.0.20.100
```

### 2. Vault でパスワードを管理（推奨）

```bash
ansible-vault create group_vars/immich_servers/vault.yml
# 内容例:
# vault_immich_db_password: "your-strong-password"
```

### 3. Playbook 実行

```bash
# 構文チェック
ansible-playbook immich-setup.yml --syntax-check

# 接続テスト
ansible all -i inventories/hosts.ini -m ping -K

# 本実行
ansible-playbook -i inventories/hosts.ini immich-setup.yml -K --ask-vault-pass
```

### 4. 動作確認

```bash
ssh immich@10.0.20.100 'cd ~/immich-app && docker compose ps'
```

ブラウザで `http://<immich_server_url>:2283` にアクセス。

## ディレクトリ構成

```
ansible-immich/
├── immich-setup.yml           # メイン Playbook
├── ansible.cfg                # Ansible 設定
├── inventories/
│   └── hosts.ini              # ホスト定義
├── group_vars/
│   ├── immich_servers.yml     # 環境変数（編集必須）
│   └── immich_servers/
│       └── vault.yml          # 機密情報（Vault）
└── roles/
    └── immich-setup/          # ロール本体
        ├── tasks/main.yml
        ├── defaults/main.yml
        ├── templates/
        └── README.md
```

## 詳細ドキュメント

- 詳細な手順、変数一覧、トラブルシューティングは **[SETUP_GUIDE.md](SETUP_GUIDE.md)** を参照
- ロール固有の情報は **[roles/immich-setup/README.md](roles/immich-setup/README.md)** を参照

## ライセンス

MIT
