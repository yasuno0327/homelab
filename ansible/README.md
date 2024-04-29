# ansible適用前のセットアップ

## raspberry-piのOSセットアップ
CentOS Stream 9を以下のURLからダウンロード
https://people.centos.org/pgreco/CentOS-Userland-9-stream-aarch64-RaspberryPI-Minimal-4/
ダウンロードしたxyzファイルを解凍せずにMacのRaspberry pi Imagerを使って書き込みを行う
※ Imager側で指定できるユーザー名などはCentOS Stream 9では利用出来ないので、何も設定せずに書き込む

m2SSD経由でブートさせるような場合は書き込み後のストレージをMacで直接開いてboot/config.txtに以下の内容を書き込む
```config.txt
dtparam=pciex1
dtparam=nvme
```

## 初回セットアップ
一旦ディスプレイにraspberrypiを繋いで,sshの設定などをしてMac上でsshして作業出来るようにする

### パスワードの変更
初回はcentosのデフォルトユーザーでログインする
デフォルトはname: root, password: centos
その後passwdコマンドでrootのパスワードを変更しておく

```shell
passwd root
```

### パーティション拡張
デフォルトではストレージの全量分のパーティションが割り当てられていないので拡張しておく
以下のコマンドを実行するだけでOK

```shell
rootfs-expand
```

### hostnameの変更、ローカルネットワークからの名前解決
private ipはdhcpで動的に割り当てられるので、avahiを利用してローカルネットワークから名前解決出来るようにしておく

```shell
sudo dnf install -y avahi
sudo systemctl start avahi-daemon
sudo systemctl enable avahi-daemon

sudo hostnamectl set-hostname <任意のホスト名>
```

### rootでsshを許可する
sshで作業出来た方が楽なのでやっておく
/etc/ssh/sshd_configを弄る

```/etc/ssh/sshd_config
PermitRootLogin yes
```

sshデーモンを再起動して反映させる
```shell
systemctl restart sshd
```

### SELinuxとfirewallを一時無効化
初回のセットアップには邪魔なので無効化しちゃう
後述の方法で後ほど適切な設定を行う

```shell
vim /etc/selinux/config

SELINUX=enforcing # ここを削除
SELINUX=disabled
```

```shell
systemctl stop firewalld
systemctl disable firewalld
```

### オプション Wifi,Bluetooth無効化
消費電力が気になる場合は/boot/config.txtから無効化

```config.txt
dtoverlay=disable-bt
dtoverlay=disable-wifi
```

### 再起動
selinuxの設定などを適用するため再起動しておく

```shell
sudo reboot
```

## Mac上でsshする
ssh出来るか試しておく

```shell
ssh root@<設定したホスト名>
```
