# backport-iwlwifi
intel wifi driver backport
intel WiFi,Bluetooth firmware binaries  

## PPAで公開するまでの手順
1. パッケージのビルドに必要な依存を導入します。
2. ソースをビルドして `.changes` と `.dsc` を生成します（例: `debuild -S -sa`）。
3. 生成物が正しいか確認します（例: `lintian`）。
4. Launchpad の PPA にアップロードします。
   ```sh
   dput ppa:triorb/ppa <your-package>.changes
   ```
5. Launchpad のビルド完了後、以下の PPA ページで公開状態を確認します。
   https://launchpad.net/~triorb/+archive/ubuntu/ppa
