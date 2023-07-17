# Symbol Shoestring β 起動方法まとめ

symbol-bootstrap に代わる新たな node 起動用ツールとして symbol-shoesting が β 公開されました。現時点ではテストネットにのみ対応し、各種不具合も存在しますが、もし検証可能な方は以下の手順を参考にインストールを行い、フィードバックをお寄せください。

## 検証時点の環境情報

```
Ubuntu 20.04.6 LTS 
Python 3.8.10
gettext (GNU gettext-runtime) 0.19.8.1
docker
docker compose
```

## 目次

(a) wizardを起動させるまで
(b) wizardのsetupを完了させるための修正
(c) wizardのsetup実行
(d) nodeを起動するまで 
(a) wizardを起動させるまで

## (a) wizardを起動させるまで

### 以前の残骸があったら念のために消す

```shell
cd 
rm -rf .local
```

### 各種ミドルウェアを以下の順番で追加する

```shell
sudo apt install -y libssl-dev gettext
pip install PyOpenSSL --upgrade
pip install symbol-shoestring
```

### githubからlangファイルを取得（バグへの暫定対応）

```
cd .local/lib/python3.8/site-packages/shoestring/
mkdir -p lang/en/LC_MESSAGES
cd lang/en/LC_MESSAGES
wget raw.githubusercontent.com/symbol/product/dev/tools/shoestring/lang/en/LC_MESSAGES/messages.po
cp -p messages.po messages.po.org
```

### messages.po へコード追記（バグへの暫定対応）

以下を `messages.po` の 6 行目に追加してください。次の手順でエラーが生じます。

**修正前**
```messages.po
# Translations for Shoestring.
# Copyright (C) 2023 Symbol Contributors
# This file is distributed under the same license as the Shoestring project.
msgid ""
msgstr ""

#: shoestring/commands/announce_transaction.py:31
msgid "announce-transaction-announce-successful"
msgstr "transaction was successfully sent to the network"
```

**修正後**
```messages.po
# Translations for Shoestring.
# Copyright (C) 2023 Symbol Contributors
# This file is distributed under the same license as the Shoestring project.
msgid ""
msgstr ""
"Content-Type: text/plain; charset=UTF-8\n"

#: shoestring/commands/announce_transaction.py:31
msgid "announce-transaction-announce-successful"
msgstr "transaction was successfully sent to the network"
```

### poからmo生成

```
msgfmt messages.po -o messages.mo
```

### wizard起動

```shell
cd ~/.local/lib/python3.8/site-packages/shoestring/
python3 -m shoestring.wizard
```

## (b) wizardのsetupを完了させるための修正

### 端末のサイズの設定(環境により不要）
画面が小さいと以下のようなエラーがでる場合は画面幅を広く取ってください。

```
"Window too small..."
```

### 以下のファイルでエラーがでるので修正

```
cd ~/.local/lib/python3.8/site-packages/shoestring/
cp -p internal/PackageResolver.py internal/PackageResolver.py.org
vi internal/PackageResolver.py
```

**修正内容**
```PackageResolver.py
for file in subdir.glob('*'):
-    shutil.move(file, destination_directory)
+    shutil.move(str(file), destination_directory)
     subdir.rmdir()
```

### startup/templatesディレクトリをshoestringにコピー
https://github.com/symbol/product/tree/dev/tools/shoestring/startup
https://github.com/symbol/product/tree/dev/tools/shoestring/templates 

```shell
cd ~/.local/lib/python3.8/site-packages/shoestring/
mkdir startup
cd startup
wget https://raw.githubusercontent.com/symbol/product/dev/tools/shoestring/startup/delayrestapi.sh
wget https://raw.githubusercontent.com/symbol/product/dev/tools/shoestring/startup/mongors.sh
wget https://raw.githubusercontent.com/symbol/product/dev/tools/shoestring/startup/startBroker.sh
wget https://raw.githubusercontent.com/symbol/product/dev/tools/shoestring/startup/startServer.sh
wget https://raw.githubusercontent.com/symbol/product/dev/tools/shoestring/startup/wait.sh

cd ~/.local/lib/python3.8/site-packages/shoestring/
mkdir templates
cd templates
wget https://raw.githubusercontent.com/symbol/product/dev/tools/shoestring/templates/docker-compose-dual.yaml
wget https://raw.githubusercontent.com/symbol/product/dev/tools/shoestring/templates/docker-compose-peer.yaml
wget https://raw.githubusercontent.com/symbol/product/dev/tools/shoestring/templates/nginx.conf.erb
```
 
## (c) wizardのsetup実行

### symbolディレクトリ作成

```
cd ~
mkdir symbol
```

### wizard起動

```
cd ~/.local/lib/python3.8/site-packages/shoestring/
python3 -m shoestring.wizard
```

### welcome画面で<setup>を選択しリターン
タブで選択項目を移動可能

### Obligatory settingで以下を実行

1. Configuration destination directoryが(1)で作成したディレクトリになっていること
  * 上記が赤字になっていたらうまくディレクトリが作成されていないので、最初からやりなおす
2. CA PEM file path(main account)
  * 最初赤字になっているはず
3. タブで"ca.key.pem"と初期値が表示されている箇所に移動。
4. フルパスに修正
  * 例: /home/pasomi/ca.key.pem
5. 3番目の入力フィールドの"* CA PEM file"にタブで移動
6. 右矢印キー(>)で2番目の"Generate randam private keyに移動(タブでは移動できないので注意)して、リターンキーを実行。
7. 次にタブを押して、"Specify output location of private key PEM file above"フィールドに移動。
8. <Generate!>にカーソルがくるので、リターンを押すと"Key generated and saved to file"と表示される。
9. タブを押して<Next>に移動して、リターンを実行する
10. Choose network typeでtestnetを選択し、<Next>
11. testnetは下矢印キーでtestnetの項目に移動して、スペースを実行
12. Choose node typeでDualを選択して、<Next>
13. Havest settingsは何も選択せずに、<Next>
14. Voter settingsも何も選択せずに、<Next>
15. Node settingsを指定して<Next>
  * 今回とりあえずIP or domain nameのみ設定
16. CA name + node cert nameはデフォルトのままで<Next>
17. writing configuraionが大丈夫そうだったら<Finish>

## (d) nodeを起動するまで

### dockerをインストールして起動

```
curl https://get.docker.com | sh
sudo usermod -aG docker pasomi
sudo reboot
sudo systemctl start docker
sudo systemctl enable docker
```

### docker-composeをインストール

```
sudo curl -L "https://github.com/docker/compose/releases/download/v2.2.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### nodeを起動

```
cd ~/symbol
docker-compose up -d
```

以上
