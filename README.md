# howto-install-arch-linux-ja Arch Linuxのインストールガイド(2024/Oct)

Arch Linuxのインストール手順をまとめました。情報源はこちら：  
https://wiki.archlinux.org/title/Installation_guide

オフィシャルWikiは多彩な状況での選択肢を網羅している反面、選ぶに迷う局面が多々あります。現時点でわたくしが実行している手順をまとめました。

## インストールメディアの入手

- こちらからISOファイルを取得してください。  
https://www.archlinux.jp/download/  
- microSDなどを使って起動可能なUSBドライブを作成します。Win環境ならRufusが楽です(※ 作成時は『DDモード』で書き込む必要があります)。  
https://rufus.ie/en/  
- または複数のISOイメージを起動時に選択/起動できるVentoyが便利です。64Gなり128GなりのmicroSD1枚にWindows / Linux / その他レスキューイメージ等を１つのUSBデバイスに集約できます。さらにはISOイメージを改変することなく特定ファイルを置換して起動するなど突っ込むと相当便利なのでお勧め。    
https://www.ventoy.net/  

## USBデバイス等からUEFIモードで起動
通常のPCの起動プロセスは大昔からあるBIOSとUEFIと2種類あります。可能ならばUEFIモードで起動 & UEFIでインストールすると何かと便利です。  
BIOS起動かEFI起動かはArchLinuxの起動画面(=grubのメニュー画面)に表示されるので判別できます。

起動したらまずキーボード配列を設定します。(US配列の人はもちろん不要です)  
```zsh
loadkeys jp106
```  
以下のフォルダが存在していればUEFIモードです。もしなければ ＆ UEFIモードに変更したければBIOS設定画面で変更する必要があります。どちらがよいかはターゲットハードウェア次第ですが、本ガイドではUEFIモードのみ解説しています。
```zsh
ls /sys/firmware/efi/efivars
```  

## インストール作業の開始

- (Optional)ssh経由で作業をする場合  
最近のArchLinuxISO環境にはsshdが稼働しています。ターゲットマシンがDHCP経由でIPを取得していて、慣れた環境からSSH経由でインストールしたい場合以下の手順で可能となります。  

```zsh
ip a
```  
IPアドレスが表示されますので記録します。  
ISO環境のsshdはrootによるログインが可能ですがパスワードが設定されてないのでrootのパスワードを設定します。
```zsh
passwd root
```  
これでssh経由で作業が可能となります(作業内容は以降差はありません)。  

- 時刻を設定する  
たまーにGnuGPからみで面倒なエラーが出る場合があるので、ISO環境時点でも以下のコマンドを実行して時刻を同期させておくことをお勧めいたします。
```zsh
timedatectl set-ntp true
ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
hwclock --systohc
timedatectl status 
```  

## ディスクのパーティション作成 & ファイルシステム選択
USBブートのLinuxからターゲットPCのディスクは /dev/sda や /dev/nvme0n1 や /dev/mmcblk0 のようなブロックデバイス名称が割り当てられます。デバイスの名称を確認するには、lsblk を実行します。  
```zsh
lsblk
```  
以降の例では通常の内蔵HDD/SSDに割り当てられる/dev/sdaをターゲットのディスクとして解説します。  
まずはディスク詳細とパーティション情報を表示させて/dev/sdaで間違いないか確認するのが安全です。  
```zsh
fdisk -l /dev/sda
```  
その後fdiskでパーティションを作成します。
```zsh
fdisk /dev/sda
```  
EFIではEFI専用のパーティションとLinux本体を格納する2つのパーティションが必要です。スワップはパーティションではなくスワップファイルで同じこと実現できるので専用にパーティション作成はもはや不要です。

```zsh
fdisk /dev/sda

Welcome to fdisk (util-linux 2.40.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help):
```
あらたにEFI対応パーティションテーブル作成はgコマンドです。

```zsh
Command (m for help): g
Created a new GPT disklabel (GUID: 304C35A2-BF5F-472A-953D-5C2F31FD80A6).
```
その後/bootになる予定のEFI専用パーティションと残り全てをLinux用に使います(追加でパーティション作成するならば以下のコマンドを参考に追加・調整してください)。
まず/boot予定のパーティションです。200Mとかで十分のはずですが500Mくらい割り当てると何かのときに安心です(というような状況になったことはありませんが)。
```zsh
Command (m for help): n
Partition number (1-128, default 1): 1
First sector (2048-1000215182, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-1000215182, default 1000214527): +500M

Created a new partition 1 of type 'Linux filesystem' and of size 500 MiB.
Partition #1 contains a vfat signature.

Do you want to remove the signature? [Y]es/[N]o: y

The signature will be removed by a write command.
```
過去のパーティションのデータがあると↑のように『ホントにいいのか？』と尋ねてきます。
```zsh
Command (m for help): n
Partition number (2-128, default 2):
First sector (1026048-1000215182, default 1026048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (1026048-1000215182, default 1000214527):

Created a new partition 2 of type 'Linux filesystem' and of size 476.5 GiB.
```
作成した状況を確認するにはpコマンドです。
```zsh
Command (m for help): p
Disk /dev/sda: 476.94 GiB, 512110190592 bytes, 1000215216 sectors
Disk model: INTEL SSDPEKNU512GZ
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 304C35A2-BF5F-472A-953D-5C2F31FD80A6

Device           Start        End   Sectors   Size Type
/dev/sda1    2048    1026047   1024000   500M Linux filesystem
/dev/sda2 1026048 1000214527 999188480 476.5G Linux filesystem

Filesystem/RAID signature on partition 1 will be wiped.

Command (m for help):
```
どちらもファイルタイプがLinux filesystemになっています。/boot予定の/dev/sda1はEFI systemである必要があるので、tコマンドで変更します。
```zsh

Command (m for help): t
Partition number (1,2, default 2): 1
Partition type or alias (type L to list all): uefi

Changed type of partition 'Linux filesystem' to 'EFI System'.
```
以下のようになればwコマンドでホントにドライブに書き込みを実行します。
```zsh
Command (m for help): p
Disk /dev/sda: 476.94 GiB, 512110190592 bytes, 1000215216 sectors
Disk model: INTEL SSDPEKNU512GZ
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 304C35A2-BF5F-472A-953D-5C2F31FD80A6

Device           Start        End   Sectors   Size Type
/dev/sda1    2048    1026047   1024000   500M EFI System
/dev/sda2 1026048 1000214527 999188480 476.5G Linux filesystem

Filesystem/RAID signature on partition 1 will be wiped.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

root@archiso ~ #
```
続いて各パーティションをフォーマットします。
```zsh
# root@archiso ~ # mkfs.fat -F 32 /dev/sda1
mkfs.fat 4.2 (2021-01-31)
# root@archiso ~ # mkfs.ext4 /dev/sda2
```
続いてこれらをマウントします。
```zsh
# mount /dev/sda2 /mnt
# mount --mkdir /dev/sda1 /mnt/boot
```
## ミラーリストの更新とカーネルを含む各種パッケージのインストール

続いてミラーリストファイルを編集します。本文書を含めて文章化されたミラーリスト例は現実に即していない可能性があります。ぱっちりミラーを計測して速い上位20をリストに記載すると高速に更新できるようになります。ISOイメージ起動時にミラーリストはこの作業が実行されていて、最速のもの10個が並んでいますが、意図的に日本とUSAのものに絞って20個並べるとこうなります。
```zsh
# reflector -c jp -c us -n 20 >! /etc/pacman.d/mirrorlist
################################################################################
################# Arch Linux mirrorlist generated by Reflector #################
################################################################################

# With:       reflector -c jp -c us -n 20
# When:       2024-10-17 01:38:38 UTC
# From:       https://archlinux.org/mirrors/status/json/
# Retrieved:  2024-10-17 01:36:06 UTC
# Last Check: 2024-10-17 01:17:19 UTC

Server = http://mirrors.rit.edu/archlinux/$repo/os/$arch
Server = https://mirrors.rit.edu/archlinux/$repo/os/$arch
Server = rsync://mirrors.rit.edu/archlinux/$repo/os/$arch
Server = http://www.gtlib.gatech.edu/pub/archlinux/$repo/os/$arch
Server = http://mirror.umd.edu/archlinux/$repo/os/$arch
Server = https://mirror.umd.edu/archlinux/$repo/os/$arch
Server = rsync://mirror.umd.edu/archlinux/$repo/os/$arch
Server = http://ftp.osuosl.org/pub/archlinux/$repo/os/$arch
Server = https://ftp.osuosl.org/pub/archlinux/$repo/os/$arch
Server = rsync://ftp.osuosl.org/archlinux/$repo/os/$arch
Server = http://mirrors.lug.mtu.edu/archlinux/$repo/os/$arch
Server = https://mirrors.lug.mtu.edu/archlinux/$repo/os/$arch
Server = rsync://mirrors.lug.mtu.edu/archlinux/$repo/os/$arch
Server = http://mirrors.xmission.com/archlinux/$repo/os/$arch
Server = http://mirrors.kernel.org/archlinux/$repo/os/$arch
Server = https://mirrors.kernel.org/archlinux/$repo/os/$arch
Server = rsync://ftp.jaist.ac.jp/pub/Linux/ArchLinux/$repo/os/$arch
Server = http://mirrors.cat.pdx.edu/archlinux/$repo/os/$arch
Server = rsync://mirrors.cat.pdx.edu/archlinux/$repo/os/$arch
Server = http://mirrors.rutgers.edu/archlinux/$repo/os/$arch
```
JPのみにしたい場合は『-c us』を削除してください。近隣アジアの国を追加変更その他は-cオプションを調整してください。

続いてターゲットドライブにカーネルその他をダウンロード＆インストールします。
まずpacman.confを編集して高速化を図ります。
```zsh
# vim /etc/pacman.conf
```
以下の2つがコメント行になっているので有効化&数を増やします。

```zsh
Color
ParallelDownloads = 20
```

次にISOファイルは1ヶ月毎の更新でkeyringを先に更新しておくと大量のパッケージインストールも途中で『更新するか？』と尋ねてこなくなります。

```zsh
# pacman -S archlinux-keyring
```

そしてターゲットドライブにカーネルその他をインストールします。『linux』を抜くとkernel無しでその他のものだけが入り起動不可となりますが特に警告も出ないので忘れずに指定してください(複数のlinuxイメージを選べるため&linuxが不要な状況があるにはあります)。
以下はわたくしの例ですが、linux-firmware以降は適当に増減してください。
```zsh
# pacstrap /mnt linux base base-devel linux-firmware man openssh vim neovim nano zsh zsh-lovers grml-zsh-config git wget mtools efitools dosfstools tmux go moreutils net-tools mc usbutils 
```
続いてターゲットドライブにマウント情報を書き込みます。
```zsh
# genfstab -U /mnt >> /mnt/etc/fstab
```
このgenfstabコマンドはUUID指定でfstabリストを生成してくれる便利なスクリプトですが本体には入っていませんのでコピーしておくと将来ドライブを増減したときに便利です(-Uオプションをわすれずに)。
```zsh
# cp /usr/bin/genfstab /mnt/root/
```
## ターゲットドライブにchrootする

続いてターゲットドライブにchrootして各種調整を行います。デフォルトではbashですが便利なzshにします。
```zsh
# arch-chroot /mnt
[root@archiso /]# zsh
root@archiso / #
```
ここでの作業は色々ありますが最低限passwdの設定とgrubを導入しないと起動しません。
```zsh
root@archiso / # passwd
New password:
Retype new password:
passwd: password updated successfully
```
ターゲットマシンが起動時に時間があうように調整します。ついでにハードウェアクロックはUTCに、表示時刻はJSTにします。
```zsh
# ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
# hwclock --systohc
```
続いてロケールの設定をします。日本語も使いたい方は追加・変更してください。
```zsh
# mv /etc/locale.gen /etc/locale.gen.original
# echo "en_US.UTF-8 UTF-8" > /etc/locale.gen  && cat /etc/locale.gen
# locale-gen
Generating locales...
  en_US.UTF-8... done
Generation complete.
```
ホストネーム設定は本来はhostnamectlコマンドを使いますがchroot環境では動作しないので直接ファイルに書き込みます(または起動後に編集してください)。
```zsh
touch /etc/hostname
echo "9020mt" >! /etc/hostname && cat /etc/hostname
```
## ネットワーク設定
このあたりが公式wikiでは多様な状況下を列挙していてよくわかりません。わたくしの経験と理解の範囲で以下にまとめました。

- DHCPからIPを取得する場合
  dhcpcdを導入します。
```zsh
# pacman -S dhcpcd
```
  ネットワークインターフェイスが1つだけならそのまま起動設定してもいいですが、インターフェイス名を確認して指定すると盤石です。
```zsh
# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 98:90:96:be:9e:f1 brd ff:ff:ff:ff:ff:ff
    altname enp0s25```
```
この場合eno1(またはenp025)なので、以下のコマンドとなります。
```zsh
# systemctl enable dhcpcd@eno1.service
```
こまかな調整は/etc/dhcpcd.confを編集してください。

- 固定IPを設定する場合
  
  dhcpcdでも固定IP設定できますが、IPエイリアスできない(または困難)その他所詮はDHCPクライアントなので、ArchLinux製のnetctlを使います。
  ついでにopenresolvを導入してsystemd-reoslvconfに依存しないようにします。

```zsh
pacman -S netctl openresolv
```
/etc/netctlに移動します。examplesフォルダに各種の設定プロファイルがありますので、その中からethernet-staticを使います。
無線/ppp/vlanその他は該当のプロファイルを参照してください。
```zsh
# cd /etc/netctl && ls /etc/netctl/examples
```
- ネットワークインターフェイス名称を変更する場合

　netctlではインターフェイス名が必要です。が、eno1やenp0s25などの名称が気に入らない場合、先にインターフェイス名称の変更を行います。不要な方は次のステップへ進んでください。
　ここではudevルールを変更します(macアドレスが固定&取得できる必要があります。そうでない場合Arch Wikiのドキュメントを参照してください)。
 
まずmacアドレスを表示させます。 
```zsh
# cat /sys/class/net/*/address
98:90:96:be:9e:f1
00:00:00:00:00:00
```
この例ではネットワークインターフェイスは1つだけなので(00の連続ははloです)判別できますが、複数ある場合は「ip a」コマンドを使ってください。
取得したmacアドレスをudev設定ファイルに記載します。
```zsh
# touch /etc/udev/rules.d/20-network.rules
# vim /etc/udev/rules.d/20-network.rules
```
以下のうちaddressの場所にmacアドレスをコピペして(英字は小文字である必要があるのでコピペが一番です)、NAMEに指定したい名称をつけます。
```zsh
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="98:90:96:be:9e:f1", NAME="net0"
```
 > [!WARNING]
> eth0を使いたいところですが、『固定の名前を付けるときに "ethX" や "wlanX" というような名前を使ってはいけません、起動時にカーネルと udev で競合状態が発生する可能性があります。』とあるように正常起動しない場合があります。
> 昔はただの脅し(?)でしたが最近は本当に起動しない場合も増えてきましたので諦めるのが楽です。どうしてもeth0にしたい方はArch Wikiを参照してください。

```zsh
# cat /etc/udev/rules.d/10-network.rules
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="98:90:96:be:9e:f1", NAME="net0"
```
(起動時に反映されます。手動でトリガーするなら以下のコマンドですがchroot環境では動作しません)
```zsh
# udevadm trigger --verbose --subsystem-match=net --action=add 
```
続いてようやくnetctlファイルです。examplesフォルダからethernet-staticをコピーして編集します。

```zsh
# cd /etc/netctl
# cp examples/ethernet-static profile && cat profile
```
見ればわかるようなファイルですので環境に適切に編集してください。なおDNSを複数したい場合、Address同様に以下のようにします。
```zsh
DNS=('202.232.2.2 202.232.2.3')
```
編集が完了したらこのprofileを有効化します。
```zsh
# netctl start profile && netctl enable profile
```
以下執筆予定
★openresolvの設定
```zsh
```
またssdを有効化します。(必要に応じてsshd_configファイルを編集してください。)
```zsh
vim /etc/ssh/sshd_config
systemctl enable sshd
```
root以外の一般ユーザーを作成します(rootではyayその他動作しないコマンドが多々あります)。
```zsh
# useradd -m -G wheel -s /bin/bash yourname
# passwd yourname
```
sudoを使う場合以下のファイルを編集します。
```zsh
# vim /etc/sudoers
```
### ブートローダの導入と設定

最後にブートローダとしてgrubを導入します。先にCPUにあわせてマイクロコードをインストールしてください。以下のいずれか1つだけです。

```zsh
# pacman -S intel-ucode
```
または
```zsh
pacman -S amd-ucode
```
grubと関連パッケージをインストールします。

```zsh
# pacman -S grub efibootmgr
```
grubを/bootパーティションに導入します。
```zsh
# grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```
設定値を変更する場合は以下のファイルを編集します。その後grub-mkconfigコマンドを実行して反映させます。
```zsh
# vim /etc/default/grub
# grub-mkconfig -o /boot/grub/grub.cfg
```
これで作業は完了です。chroot環境を脱して再起動するとArch Linuxが起動するはずです。

```zsh
# exit
# reboot
```
## 起動後の各種設定
★執筆予定
```zsh
```
```zsh
```





















