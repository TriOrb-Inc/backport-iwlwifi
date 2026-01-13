# backport-iwlwifi
intel wifi driver backport
intel WiFi,Bluetooth firmware binaries  

## PPAで公開するまでの手順
1. パッケージのビルドに必要な依存を導入します。
   ```sh
   sudo apt-get update
   sudo apt-get install -y devscripts debhelper lintian build-essential
   ```
2. `debian/rules` が存在するパッケージング用ディレクトリで作業します（`debuild` は `debian/rules` が必須です）。
   - `debian/rules` はディレクトリではなく **実行可能なファイル** です。
   - 最低限、以下のファイルが揃っている必要があります。
     - `debian/rules`
     - `debian/control`
     - `debian/changelog`
     - `debian/source/format`（または `debian/compat` などビルドシステムに必要なファイル）
   - このリポジトリには現状 `debian/changelog` しかないため、PPA 用には `debian/rules`/`debian/control` などを追加で用意してください。
3. ソースパッケージのメタデータを確認します（`debian/changelog` の対象ディストリ名など）。
4. ソースをビルドして `.changes` と `.dsc` を生成します。
   ```sh
   debuild -S -sa
   ```
5. 生成物が正しいか確認します。
   ```sh
   lintian ../*.changes
   ```
6. Launchpad の PPA にアップロードします。
   ```sh
   dput ppa:triorb/ppa <your-package>.changes
   ```
7. Launchpad のビルド完了後、以下の PPA ページで公開状態を確認します。
   https://launchpad.net/~triorb/+archive/ubuntu/ppa
8. 配布先で PPA を追加してインストールします。
   ```sh
   sudo add-apt-repository ppa:triorb/ppa
   sudo apt-get update
   sudo apt-get install -y <package-name>
   ```

## develop版をPPAにアップロードする推奨手順
1. develop 向けのバージョンを `debian/changelog` に追記します（例: `1.2.3~dev1`）。
2. ソースをビルドして `.changes` と `.dsc` を生成します。
   ```sh
   debuild -S -sa
   ```
3. Launchpad の PPA にアップロードします。
   ```sh
   dput ppa:triorb/ppa <your-package>.changes
   ```

## ソースコードからビルドしてインストールする手順
1. ソースツリーでビルドします。
2. カーネルモジュールをインストールし、依存関係を更新します。
   ```sh
   sudo make install
   sudo depmod --all
   ```
3. ファームウェアを配置します。
   ```sh
   cd ../fw-binaries
   sudo cp -rf ./* /usr/lib/firmware/
   ```
