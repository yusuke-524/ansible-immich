# Ansible Immich Setup
# Ansible Immich Setup

このリポジトリは、Immich を動作させる Ubuntu VM 内部を Ansible で自動構成するためのものです。

概要
- NFS マウント（TrueNAS）設定
- Docker と docker-compose の導入と Immich の起動
- systemd によるサービス化と自動起動設定

主なドキュメント
- `QUICKSTART.md` — 最短手順（数コマンド）
- `SETUP_GUIDE.md` — 詳細手順・トラブルシューティング（一次情報源）
- `REFERENCE.md` — よく使うコマンドのリファレンス

構成（抜粋）

```
ansible-immich/
├── immich-setup.yml       # メイン Playbook
├── ansible.cfg            # Ansible 設定
├── inventories/hosts.ini  # ホスト定義
├── group_vars/            # 環境変数（編集箇所）
└── roles/immich-setup/    # ロール（処理本体）
```

まずは `QUICKSTART.md` を実行し、問題があれば `SETUP_GUIDE.md` を参照してください。

----

