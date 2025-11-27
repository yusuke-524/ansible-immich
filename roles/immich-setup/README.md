# immich-setup ロール

Immich を動作させるための VM 内設定とアプリ起動を自動化するロールです。

## ロール概要

このロールは以下を実行します：

1. **システム初期化**
   - タイムゾーン設定
   - APT パッケージマネージャー設定
   - システムパッケージ更新

2. **ストレージ設定**
   - NFS マウント設定
   - TrueNAS との連携

3. **Docker インストール・設定**
   - Docker CE と docker-compose
   - docker グループ権限設定

4. **Immich アプリケーション**
   - docker-compose.yml のダウンロード
   - 環境設定ファイルの生成
   - systemd サービスの設定

5. **セキュリティ設定**
   - SSH パスワード認証無効化
   - 自動セキュリティアップデート

## 変数

### デフォルト変数（`defaults/main.yml`）

```yaml
# ユーザー設定
immich_user: immich              # アプリケーション実行ユーザー
immich_group: www-data           # アプリケーション実行グループ
immich_uid: 33                   # www-data UID

# パス設定
immich_home_dir: /home/immich
immich_app_dir: /home/immich/immich-app
immich_data_mount_point: /mnt/immich-data

# NFS 設定
nfs_server: 10.0.20.10
nfs_export_path: /mnt/pool4tb1/media/immich_photo
nfs_mount_opts: defaults

# 時刻設定
timezone: Asia/Tokyo

# APT キャッシュ
apt_cache_server: apt.int.sirosuzu.net
apt_cache_port: 3142

# Immich 環境変数
immich_upload_location: /mnt/immich-data
immich_server_url: https://immich.int.sirosuzu.net
immich_db_password: (Vault で管理推奨)
```

### グループ変数（`group_vars/immich_servers.yml`）

環境固有の設定をここで上書きします。

## 依存性

### Ansible 最小バージョン
- Ansible 2.9 以上

### 必要なコレクション
- `community.general` （timezone モジュール使用）

### ターゲットホスト
- Ubuntu 24.04 LTS
- sudo 権限
- インターネット接続

## 使用方法

### 基本的な実行

```yaml
---
- name: Immich セットアップ
  hosts: immich_servers
  become: yes

  roles:
    - immich-setup
```

### 変数のオーバーライド

```yaml
---
- name: Immich セットアップ（カスタマイズ）
  hosts: immich_servers
  become: yes

  vars:
    immich_db_password: "{{ vault_immich_db_password }}"
    immich_server_url: "https://photos.example.com"

  roles:
    - immich-setup
```

## タスク詳細

### 1. システム準備
- タイムゾーンを設定
- APT キャッシュサーバー設定
- パッケージ更新・アップグレード

### 2. NFS マウント
- マウントポイント作成
- /etc/fstab に追加
- マウント実行・確認

### 3. Docker セットアップ
- GPG キー追加
- リポジトリ追加
- docker-ce インストール
- サービス有効化
- ユーザーをグループに追加

### 4. Immich アプリケーション
- ディレクトリ作成
- docker-compose.yml ダウンロード
- .env ファイル生成（テンプレート使用）

### 5. systemd サービス
- サービスユニットファイル生成
- デーモンリロード
- サービス有効化・起動

### 6. セキュリティ
- SSH パスワード認証無効化

### 7. 自動アップデート
- unattended-upgrades 設定

## テンプレートファイル

### `immich-env.j2`
`.env` ファイルテンプレート。以下の変数で置換されます：
- `{{ immich_upload_location }}`
- `{{ timezone }}`
- `{{ immich_db_password }}`
- `{{ immich_server_url }}`

### `immich-systemd.service.j2`
systemd サービスファイルテンプレート。以下の変数で置換されます：
- `{{ immich_user }}`
- `{{ immich_group }}`
- `{{ immich_app_dir }}`

## トラブルシューティング

### タスクが失敗した場合

```bash
# より詳細なログで実行
ansible-playbook immich-setup.yml -vvv

# 特定のタスクのみ実行
ansible-playbook immich-setup.yml --start-at-task="タスク名"

# 特定のタスクをスキップ
ansible-playbook immich-setup.yml --skip-tags="タグ"
```

### よくある問題

**問題：NFS マウント失敗**
- NAS が起動しているか確認
- NFS 共有権限を確認
- 手動でマウント試行

**問題：Docker コンテナ起動失敗**
- docker.service が実行中か確認
- メモリ・ストレージを確認
- docker logs で確認

**問題：SSH 接続失敗**
- SSH キー設定を確認
- ホストファイアウォールを確認

## ベストプラクティス

1. **本番環境では Vault を使用**
   ```bash
   ansible-vault create group_vars/immich_servers/vault.yml
   ```

2. **変数をグループ/ホスト単位で管理**
   - `group_vars/` - グループ共通設定
   - `host_vars/` - ホスト固有設定

3. **定期的にテスト実行**
   ```bash
   ansible-playbook immich-setup.yml --check
   ```

4. **変更履歴を記録**
   - ansible.cfg で log_path を設定
   - git で管理

## カスタマイズ例

### メモリ設定のカスタマイズ

`docker-compose.yml` をオーバーライドする場合、新しいタスクを追加：

```yaml
- name: docker-compose.yml のメモリ設定を更新
  replace:
    path: "{{ immich_app_dir }}/docker-compose.yml"
    regexp: 'mem_limit: 4g'
    replace: 'mem_limit: 8g'
  register: compose_memory
```

### ログローテーション設定

```yaml
- name: Docker ログローテーション設定
  copy:
    dest: /etc/docker/daemon.json
    content: |
      {
        "log-driver": "json-file",
        "log-opts": {
          "max-size": "10m",
          "max-file": "3"
        }
      }
  register: docker_config

- name: Docker サービス再起動
  systemd:
    name: docker
    state: restarted
  when: docker_config.changed
```

## ライセンス・参考資料

- Immich: https://immich.app/
- Docker: https://www.docker.com/
- Ansible: https://www.ansible.com/
