# raspberry pi 講習会ドキュメント

## 講習会ページ
http://openrtm.org/openrtm/ja/tutorial/robomec2013
http://openrtm.org/openrtm/ja/content/raspberrypi-openrtm-tutorial

## raspberry pi OS導入

https://www.raspberrypi.org/downloads/
RASPBIANのOSイメージをダウンロード
イメージをSDカードに焼きます(必ず最初にフォーマットしてね)。
(SDカード焼き方
Win: http://openrtm.org/openrtm/ja/content/raspberrypi_sdcard#toc4,
Mac: http://qiita.com/ttyokoyama/items/7afe6404fd8d3e910d09
)
micro SDカードをrasberry pi 本体にぶっさします。(USBが一杯あるほうの反対側の底面)

初期のユーザ名とパスワードは、

	id: pi
	pass: raspberry

に設定されている。

## raspberry pi 初期設定

HDMIとディスプレイとキーボードとマウス、無線LANアダプタ（マウスは無くてもいい)、LANケーブルを挿す。MicroUSBとホストPCを繋いで起動！

しばらくするとGUIぽいCUIメニューが出てくる。以下の項目をカーソルキーとエンターキーでポチポチ選択していく。

- Expand Filesystem (SDカードをフルに使う)
- Change User Password (デフォルトユーザ pi のパスワードを変えたかったらどうぞ)
- Advanced Options
     - SSH [Enable] (SSH接続の有効)
	 - Update (Raspberry piのアップデート)

- Finishを選択

とりあえず一通り設定したらシェルが起動するので、再起動するといいんじゃなかろうか。

	$ sudo reboot

## 無線LAN設定

ホストPCからraspberry piへsshで アクセス(Putty等)する。
ログイン時にディスプレイにローカルのIPが出るはず(同じネットワーク内でアクセスしないとダメ！) たぶん **eth0: offered 192.168.1.hoge** みたいな感じ。ログインに成功したらシェルで以下のコマンドを打ち込んでいく。

無線LANはDHCPでIPを動的に振るようにする。viもちろん使えると思うけど基本操作(http://www.envinfo.uee.kyoto-u.ac.jp/user/susaki/command/vi.html)
(宗教上の理由でvi使えない人は、 $ apt-get install emacs 以下 vi → emacsに脳内補完)

	$ sudo vi /etc/network/interfaces

以下の1箇所(一箇所はRASPBIAN最新版では必要なし。)を修正。

	iface wlan0 inet manual
           ↓
	iface wlan0 inet dhcp

次に、ルータのESSIDとパスフレーズを以下の方法に従って設定。

	$ sudo bash
	$ cd /etc/wpa_supplicant
	// iwlist wlan0 scan | grep ESSID で目的のルータのESSIDを探す。
	// [pass]は、ルータに記載されている。(バッファロールータだったらKEYと書かれている箇所。)
	$ wpa_passphrase [ESSID] [pass] >> wpa_supplicant.conf
	$ cat wpa_supplicant.conf // で追記されているか確認。不安な場合はバックアップを取る。
	// 問題無ければ以下のコマンドで無線LANを再起動。成功確率五分ぐらいなので、接続出来ない人は有線で先を進めるか、SDカードにOS焼くところからやり直してください・・・。
	// 対策として無線LAN子機を抜き差し、ルータの再起動、Raspberry piの再起動を試してね。
	$ ifdown wlan0
	$ ifup wlan0
	// このコマンドでIPアドレス確認。
	$ ifconfig wlan0

※ raspberry piへ sshするときは、同じローカルネットワーク下に無ければ接続できない。複数回IPが被った時は、RSAキーを削除して再生成しないと行けないことに注意！
ここまで設定出来てしまえば、LANケーブルをraspberry piから抜いてOK

## ホスト名の設定
無線LAN設定で確認したIPアドレスを使ってsshアクセスする。
毎回 ssh pi@192.168.~~ とアクセスはツライので、*avahi*というツールを使ってLAN内のホスト名を割り振る。

デフォルトでは、[raspberrypi]というホスト名。同じLAN内でホスト名の衝突があると困るので、rasp1 rasp2 などの連番を振るか、必ず被らないホスト名に設定する。以下のファイルの一行目を書き換えてホスト名を設定する。

	$ sudo vi /etc/hostname

	raspberrypi
		↓
	rasp*

続いて、以下の通りに編集する。

	$ sudo vi /etc/hosts

	127.0.1.1       raspberrypi
		  ↓
	127.0.1.1       [新しいホスト名]

以下のコマンドで*avahi-daemon*をインストール(RASPBIAN最新版では標準でインストールされている模様)
	sudo apt-get update && sudo apt-get install avahi-daemon

一度再起動をする。

	sudo reboot

すると、

	ssh pi@[新しいホスト名].local

で、接続できるはず。WindowsだとBonjourというツールを入れる必要アリ
(参考:http://openrtm.org/openrtm/ja/content/raspberrypi_initial_setting)

## Open-RTMの準備

	$ sudo apt-get install git // たぶん標準で入っているのでいらない
	$ cd $HOME && git clone https://gist.github.com/62d0ec4da753d6997dc1.git
	// 15分くらい掛かるかも
	$ cd 62d0ec4da753d6997dc1 && chmod 700 provision.sh && ./provision.sh

## Open-RTMのテスト

### rasberrypi側の準備
ネーミングサービスを起動して、サンプルからConsoleOutコンポーネントを起動。

	$ rtm-naming
	$ /usr/share/openrtm-1.1/example/ConsoleOutComp

### ホストPC側の準備

以下に従って起動していく。(スタートメニュー)
分からない場合は2015年の講習会資料を参考するように。
(http://openrtm.org/openrtm/ja/tutorial/robomech2015)

- ネーミングサービスを起動。
- ConsoleInComp コンポーネントを起動(C++でもPythonでも可)
- System Editor起動
- パレットを開いて、ConsoleInCompを配置
- rasberry pi のIPアドレス(Bonjourを入れてる場合は、[ホスト名].localでいける・・・？)を入れる
- するとConsoleOutCompが項目に出てくるはずなので、繋いでActivateする。
- ConsoleInのほうのプロンプトに数字を打ち込む。
- rasberry pi で結果を確認する。


## KobukiのRTCの作成

	// svn経由でソースコードをチェックアウト。そしてmakeへ・・・
	$ cd $HOME && svn co http://svn.openrtm.org/components/trunk/mobile_robots/kobuki
	$ cd Kobuki
	$ mkdir build
	$ cd build
  	$ cmake -DCMAKE_INSTALL_PREFIX=/usr ..
  	$ make
  	$ cd src
  	$ sudo make install

## Raspberry pi と Kobukiをドッキング！

写真の状態にする。もし電源をKobukiからRaspberry piへ給電出来ない場合は、スマホ用のモバイルバッテリーなどから給電すると良いよ。KobukiとRaspberry piの接続はプリンター用のUSBケーブルでブスッと挿します。

![](http://openrtm.org/openrtm/sites/default/files/273/kobuki_and_raspi.png)

側面のスイッチからKobukiの電源をON! Raspberry pi側から以下のコマンドと表示になれば接続確認！

	$ ls /dev/ttyUSB*
    /dev/ttyUSB0

## PCからKobukiをLEDチカチカ！
KobukiRTCをRaspberry pi側から起動。

	$ rtm-naming // 既に起動してる場合は打たなくてOK
	$ sudo /usr/lib/openrtm-1.1/rtc/KobukiAISTComp

### ホストPCから操作

コンソールからのテスト同様に、システムエディタを開いてRaspberry piと接続。コンポーネントを設置してActivateにする。コンポーネントのコンフィグ(編集ボタン)からLEDのラジオボタンをいじってみよう！

## Kobuki猪突猛進！

ホストPCからKobukiを自律操作させます(自律とは一体)。ズルい人は↓から解答をコピー
http://openrtm.org/openrtm/sites/default/files/273/KobukiAutoMove.zip
cmakeしてコンポーネントを起動しましょう。cmakeが分からない子は、Flipコンポーネントのcmakeを真似してみよう！
(http://openrtm.org/openrtm/ja/node/5022)
Kobukiがところかまわず衝突したら実験成功です。お疲れ様です。ジョイスティックのプログラムは起動できませんでしたorz