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

## debian/配下のファイルの用意方法
`debuild` でソースパッケージを作るには、最低限の `debian/` ファイルを自分で用意する必要があります。ここでは**最小構成の作り方**を例示します。

1. ひな型を生成します（推奨）。
   ```sh
   sudo apt-get install -y dh-make
   dh_make --createorig -s -p <package-name>_<version>
   ```
   - 実行すると `debian/` が生成されるので、不要なサンプルファイルを削除します。

2. 主要ファイルを編集します。
   - `debian/control`: パッケージ名、依存、説明を記載します。
   - `debian/changelog`: バージョンと配布先ディストリ、変更点を記載します。
   - `debian/rules`: `dh` を使う場合の最小例は以下です。
     ```makefile
     #!/usr/bin/make -f
     %:
     \tdh $@
     ```
     その後 `chmod +x debian/rules` を実行します。
   - `debian/source/format`: ソース形式を指定します（例: `3.0 (quilt)`）。

3. 既存パッケージを参考にする方法（代替）。
   - 既存のカーネルモジュール系パッケージの `debian/` をコピーして必要箇所を置き換えると、手早く整備できます。

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
