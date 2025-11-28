# immich-setup ロール

Immich を Ubuntu VM 上にデプロイするための Ansible ロールです。

## 実行内容

1. **システム準備**
   - タイムゾーン設定
   - APT キャッシュサーバー設定
   - 必要パッケージのインストール

2. **NFS マウント**
   - マウントポイント作成
   - `/etc/fstab` 設定
   - マウント実行

3. **Docker セットアップ**
   - Docker CE / Docker Compose インストール
   - サービス有効化
   - ユーザーを docker グループに追加

4. **Immich デプロイ**
   - docker-compose.yml ダウンロード
   - `.env` ファイル生成
   - systemd サービス設定

5. **セキュリティ**
   - SSH パスワード認証無効化
   - 自動セキュリティアップデート設定

## 変数

### 必須変数（`group_vars/immich_servers.yml` で定義）

| 変数名 | 説明 |
|--------|------|
| `nfs_server` | NFS サーバー IP |
| `nfs_export_path` | NFS エクスポートパス |
| `immich_server_url` | Immich アクセス URL |

### デフォルト変数（`defaults/main.yml`）

```yaml
# ユーザー設定
immich_user: immich
immich_group: www-data
immich_uid: 33

# ディレクトリ設定
immich_home_dir: /home/immich
immich_app_dir: /home/immich/immich-app
immich_data_mount_point: /mnt/immich-data
immich_db_data_location: /var/lib/immich-db  # ローカルストレージ

# NFS 設定（group_vars で上書き必須）
nfs_server: ""
nfs_export_path: ""
nfs_mount_opts: defaults

# タイムゾーン
timezone: Asia/Tokyo

# APT キャッシュ
apt_cache_server: apt.int.sirosuzu.net
apt_cache_port: 3142

# Immich 環境変数
immich_upload_location: /mnt/immich-data
immich_server_url: https://immich.int.sirosuzu.net
immich_db_password: "{{ vault_immich_db_password | default('ChangeMeToStrongPassword') }}"
```

## テンプレート

### `templates/immich-env.j2`

Immich の `.env` ファイルを生成します。主要な設定：

- `UPLOAD_LOCATION` — ユーザーデータ保存先（NFS）
- `DB_DATA_LOCATION` — PostgreSQL データ保存先（ローカル）
- `DB_HOSTNAME=database` — Docker Compose サービス名
- `REDIS_HOSTNAME=redis` — Docker Compose サービス名

### `templates/immich-systemd.service.j2`

systemd サービスユニットファイルを生成します。

## 依存関係

- Ansible 2.9+
- `community.general` コレクション（timezone モジュール）

## 使用例

```yaml
- name: Immich セットアップ
  hosts: immich_servers
  become: yes
  vars_files:
    - "group_vars/immich_servers.yml"
  roles:
    - immich-setup
```

## 注意事項

- **DB データはローカルストレージに配置**: NFS 上では PostgreSQL が権限エラーで起動失敗するため、`immich_db_data_location` はローカルパスを使用
- **NFS 設定は group_vars で必須**: `nfs_server` と `nfs_export_path` は空のデフォルトのため、必ず `group_vars/immich_servers.yml` で設定してください
