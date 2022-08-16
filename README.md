# howto-install-arch-linux-ja Arch Linuxのインストールガイド(2022/Aug)

Arch Linuxのインストール手順をまとめました。情報源はこちら：  
https://wiki.archlinux.org/title/Installation_guide

オフィシャルWikiは多彩な状況での選択肢を網羅している反面、選ぶに迷う局面が多々あります。現時点でわたくしが実行している手順をまとめました。

## 1. インストールメディアの入手

- こちらからISOファイルを取得してください。  
https://www.archlinux.jp/download/  
- microSDなどを使って起動可能なUSBドライブを作成します。Win環境ならRufusが楽です(※ 作成時は『DDモード』で書き込む必要があります)。  
https://rufus.ie/en/  
- または複数のISOイメージを起動時に選択/起動できるVentoyが便利です。64Gなり128GなりのmicroSD1枚にWindows / Linux / その他レスキューイメージ等を１つのUSBデバイスに集約できます。さらにはISOイメージを改変することなく特定ファイルを置換して起動するなど突っ込むと相当便利なのでお勧め。    
https://www.ventoy.net/  

## 2. USBデバイス等からUEFIモードで起動
通常のPCの起動プロセスは大昔からあるBIOSとUEFIと2種類あります。可能ならばUEFIモードで起動 & UEFIでインストールすると何かと便利です。  
BIOS起動かEFI起動かはArchLinuxの起動画面(=grubのメニュー画面)に表示されるので判別できます。
起動したらまずキーボード配列を設定します。  
```zsh
# loadkeys jp106
```  
以下のフォルダが存在していればUEFIモードです。もしなければ ＆ UEFIモードに変更したければBIOS設定画面で変更する必要があります。どちらがよいかはターゲットハードウェア次第ですが、本ガイドではUEFIモードのみ解説しています。
```zsh
# ls /sys/firmware/efi/efivars
```  

## 3. インストール作業の開始

- (Optional)ssh経由で作業をする場合  
最近のArchLinuxISO環境にはsshdが稼働しています。ターゲットマシンがDHCP経由でIPを取得していて、慣れた環境からSSH経由でインストールしたい場合以下の手順で可能となります。  

```zsh
# ip a
```  
IPアドレスが表示されますので記録します。  
ISO環境のsshdはrootによるログインが可能ですがパスワードが設定されてないのでrootのパスワードを設定します。
```zsh
# passwd root
```  
これでssh経由で作業が可能となります(作業内容は以降差はありません)。  

- 時刻を設定する  
たまーにGnuGPからみで面倒なエラーが出る場合があるので、ISO環境時点でも以下のコマンドを実行して時刻を同期させておくことをお勧めいたします。
```zsh
# timedatectl set-ntp true
# ln -sf /usr/share/zoneinfo/Asia/Tokyo /etc/localtime
# hwclock --systohc
```  

## 4. ディスクのパーティション作成 & ファイルシステム選択
★



