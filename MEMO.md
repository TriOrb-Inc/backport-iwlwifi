export DEBFULLNAME="Tobeta Masakazu"
gpg --full-generate-key
gpg --list-secret-keys --keyid-format=long
gpg --armor --export 2254AC1F8F75D5B8D1A728C1B28F612689549D7D
debuild -S -k2254AC1F8F75D5B8D1A728C1B28F612689549D7D

dput ppa:triorb/ppa ../backport-iwlwifi_74-60~dev1.1_source.changes

sudo rm -r debian
dh_make --createorig --copyright mit -s -p backport-iwlwifi_74-60~dev1.1
git checkout debian/changelog
sudo rm debian/*.ex
sudo rm debian/*.EX