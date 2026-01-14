# backport-iwlwifi
intel wifi driver backport
intel WiFi,Bluetooth firmware binaries  

## PPAで公開するまでの手順
1. パッケージのビルドに必要な依存を導入します。
   ```sh
   sudo apt-get update
   sudo apt-get install -y devscripts debhelper lintian build-essential
   ```
2. GPG秘密鍵を作成します（未作成・未登録の場合）。
   ```sh
   export DEBFULLNAME="Tobeta Masakazu"
   gpg --full-generate-key
   gpg --list-secret-keys --keyid-format=long
   # ubuntuのキーサーバーに転送します
   gpg --send-keys --keyserver keyserver.ubuntu.com <KEY_ID> # (ex.2254AC1F8F75D5B8D1A728C1B28F612689549D7D)
   # 作成したGPGキーのフィンガープリントを表示します
   gpg --fingerprint 
   # Launchpadの「OpenPGP keys」からフィンガープリントを登録します
   ```
3. パッケージソースディレクトリに移動します。
   ```sh
   cd ./src
   ```
4. (備考) `debian`以下のファイルを確認します
   - `debian/rules` はディレクトリではなく **実行可能なファイル** です。
   - 最低限、以下のファイルが揃っている必要があります。
     - `debian/rules`
     - `debian/control`
     - `debian/changelog`
     - `debian/source/format`（または `debian/compat` などビルドシステムに必要なファイル）
5. ソースパッケージのメタデータを確認します（`debian/changelog` の対象ディストリ名など）。
6. ソースをビルドして `.changes` と `.dsc` を生成します。
   ```sh
   # (任意) 古い生成物を削除します
   # rm -f ../*.build ../*.changes ../*.dsc ../*.tar.xz ../*.buildinfo ../*.ppa.upload
   debuild -S -k<KEY_ID> # (ex. debuild -S -k2254AC1F8F75D5B8D1A728C1B28F612689549D7D)
   ```
7. 生成物が正しいか確認します。
   ```sh
   lintian ../*.changes
   ```
8. Launchpad の PPA にアップロードします。
   ```sh
   dput ppa:triorb/ppa <your-package>.changes #(ex. dput ppa:triorb/ppa ../backport-iwlwifi_74.60~dev1.3_source.changes)
   ```
   - `dput` が署名検証で失敗する場合は、生成した `.changes` が正しく署名されているかを確認してください。
   - PPAページに反映されるまで数分から数十分かかる場合があります。
9. Launchpad のビルド完了後、以下の PPA ページで公開状態を確認します。
   https://launchpad.net/~triorb/+archive/ubuntu/ppa
   - 次のコマンドでも確認できます```wget -qO- http://ppa.launchpad.net/triorb/ppa/ubuntu/dists/$(lsb_release -sc)/main/binary-amd64/Packages | grep 'Package: '```
10. 配布先で PPA を追加してインストールします。
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

## GPG 秘密鍵の作成と KEY_ID の指定方法
1. 秘密鍵を作成します。
   ```sh
   gpg --full-generate-key
   ```
2. 署名に使う秘密鍵の `KEY_ID` を確認します。
   ```sh
   gpg --list-secret-keys --keyid-format=long
   ```
   - `sec   rsa4096/XXXXXXXXXXXXXXXX` の **`/` の右側**（16 文字の ID）が `KEY_ID` です。
3. Launchpad 登録用に公開鍵をエクスポートします。
   ```sh
   gpg --armor --export <KEY_ID>
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
   - apt でインストールする場合は `debian/rules` の `override_dh_auto_install` で
     `fw-binaries/*` をパッケージに取り込み、`/lib/firmware` に展開されます。
