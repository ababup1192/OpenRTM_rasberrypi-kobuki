# raspberry pi 講習会ドキュメント

## 講習会ページ
http://openrtm.org/openrtm/ja/tutorial/robomec2013
http://openrtm.org/openrtm/ja/content/raspberrypi-openrtm-tutorial

## raspberry pi OS導入

https://www.raspberrypi.org/downloads/
RASPBIANのOSイメージをダウンロード
イメージをsdカードに焼きます。
micro sdカードをrasberry pi 本体にぶっさします。

初期のユーザ名とパスワードは、

	id: pi
	pass: raspberry

に設定されている。

## raspberry pi 初期設定

拡張Unitをピンに挿して、電源アダプタにぶっ挿してディスプレイとキーボードとマウス（無くてもいい)、LANケーブルを挿して。起動。

Expand Filesystem を選択。(SDカードのファイルサイズをフルで使う。)

- Change User Password (デフォルトユーザ pi のパスワードを変えたかったらどうぞ)
- Advanced Options
     - SSH Enableにする。
	 - Update しておく。

- Finishを選択

とりあえず一通り設定したら reboot するといいんじゃなかろうか。

	$ sudo reboot

## 無線LAN設定

ホストマシンからraspberry pi へsshで アクセス
ログイン時にディスプレイにローカルのipアドレスが出るはず(同じネットワーク内でアクセスしないとダメ。) たぶん eth0: offered 192.168.1.hoge みたいな感じ。

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
	// 問題無ければ以下のコマンドで無線LANを再起動
	$ ifdown wlan0 && ifup wlan0
	// このコマンドでIPアドレス確認。
	$ ifconfig wlan0

※ raspberry piへ sshするときは、同じローカルネットワーク下に無ければ接続できない。複数回IPが被った時は、RSAキーを削除して再生成しないと行けないことに注意！
ここまで設定出来てしまえば、LANケーブルをraspberry piから抜いてOK

## ホスト名の設定
無線LAN設定で確認したIPアドレスを使ってsshアクセスする。
毎回 ssh pi@192.168.~~ とアクセスはツライので、*avahi*というツールを使ってLAN内のホスト名を割り振る。

デフォルトでは、[raspberrypi]というホスト名。同じLAN内でホスト名の衝突があると困るので、rasp1 rasp2 などの連番を振るか、必ず被らないホスト名に設定する。以下のファイルの一行目を書き換えてホスト名を設定する。

	$ sudo vi /etc/hostname

続いて、以下のファイルを指定通りに編集する。

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

	$ sudo apt-get  install git // たぶん標準で入っている。
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

- ネーミングサービスを起動。
- ConsoleInComp コンポーネントを起動(C++でもPythonでも可)
- System Editor起動
- パレットを開いて、ConsoleInCompを配置
- rasberry pi のIPアドレス(Bonjourを入れてる場合は、[ホスト名].localでいける・・・？)を入れる
- するとConsoleOutCompが項目に出てくるはずなので、繋いでActivateする。
- ConsoleInのほうのプロンプトに数字を打ち込む。
- rasberry pi で結果を確認する。


## kobukiのRTCをコンパイル

	$ svn co http://svn.openrtm.org/components/trunk/mobile_robots/kobuki

