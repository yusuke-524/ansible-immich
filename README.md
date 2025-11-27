# Ansible Immich Setup

Immich フォトマネージメントアプリケーション の Ansible 自動化スクリプト
# Ansible Immich Setup

このリポジトリは、Immich を動かす Ubuntu VM 内部を Ansible で自動構成するためのものです。

簡潔な目的:
- NFS マウント（TrueNAS）を設定
- Docker と Docker Compose を用意して Immich を起動
- systemd でサービス化し自動起動を有効化

主なドキュメント:
- `QUICKSTART.md` — 最短で動かすための実行手順（数コマンド）
- `SETUP_GUIDE.md` — 詳細な手順とトラブルシューティング（一次情報源）
- `REFERENCE.md` — よく使うコマンドのみを集めたリファレンス

ディレクトリ構成（概要）:

```
ansible-immich/
├── immich-setup.yml       # メイン Playbook
├── ansible.cfg            # Ansible 設定
├── inventories/hosts.ini  # ホスト定義
├── group_vars/            # 環境変数（編集箇所）
└── roles/immich-setup/    # 実際の処理を行うロール
```

まずは `QUICKSTART.md` を参照し、問題が発生したら `SETUP_GUIDE.md` を参照してください。

※ 詳細な変数や長い説明はすべて `SETUP_GUIDE.md` に集約しています。

----
├── immich-setup.yml           # メイン Playbook
