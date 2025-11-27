# Immich Ansible セットアップ - 環境構築ガイド

このファイルは本リポジトリにおける「詳細な一次情報源」です。
`QUICKSTART.md` と `REFERENCE.md` は要点・コマンド参照に絞っているため、背景や手順の理由、トラブルシューティングはこの `SETUP_GUIDE.md` を参照してください。

## プロジェクト概要

このプロジェクトは、Immich フォトマネージメント VM 内部を Ansible で自動セットアップするものです。

**特徴：**
- ✅ 完全変数化（環境ごとに簡単にカスタマイズ可能）
- ✅ Vault による機密情報管理
- ✅ 冪等性（何度実行しても安全）
- ✅ 詳細なドキュメント
- ✅ エラーハンドリング充実

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
| `immich_servers.yml` | Immich サーバーのグループ変数（パスワード、ポート等） |
| `immich_servers/vault.yml` | Vault で暗号化された機密情報（DB パスワード等） |

### `roles/immich-setup/`

| ファイル | 説明 |
|---------|------|
| `tasks/main.yml` | タスク定義（実行される処理） |
| `defaults/main.yml` | デフォルト変数 |
| `templates/immich-env.j2` | 環境ファイルテンプレート |
| `templates/immich-systemd.service.j2` | systemd サービステンプレート |
| `README.md` | ロール説明書 |
