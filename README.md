# backport-iwlwifi
intel wifi driver backport
intel WiFi,Bluetooth firmware binaries  

## PPAで公開するまでの手順
1. パッケージのビルドに必要な依存を導入します。
   ```sh
   sudo apt-get update
   sudo apt-get install -y devscripts debhelper lintian build-essential
   ```
2. ソースパッケージのメタデータを確認します（`debian/changelog` の対象ディストリ名など）。
3. ソースをビルドして `.changes` と `.dsc` を生成します。
   ```sh
   debuild -S -sa
   ```
4. 生成物が正しいか確認します。
   ```sh
   lintian ../*.changes
   ```
5. Launchpad の PPA にアップロードします。
   ```sh
   dput ppa:triorb/ppa <your-package>.changes
   ```
6. Launchpad のビルド完了後、以下の PPA ページで公開状態を確認します。
   https://launchpad.net/~triorb/+archive/ubuntu/ppa
