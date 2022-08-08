# howto-install-arch-linux-ja Arch Linuxのインストールガイド(2022/Feb)

Arch Linuxのインストール手順をまとめました。情報源はこちら：  
https://wiki.archlinux.org/title/Installation_guide

オフィシャルWikiは多彩な状況での選択肢を網羅している反面、選ぶに迷う局面が多々あります。現時点でわたくしが実行している手順をまとめました。

## 1. インストールメディアの入手

- こちらからISOファイルを取得してください。  
https://www.archlinux.jp/download/  
- microSDなどを使って起動可能なUSBドライブを作成します。Win環境ならRufusが楽です(※ 作成時は『DDモード』で書き込む必要があります)。  
https://rufus.ie/en/  
- または複数のISOイメージを起動時に選択/起動できるVentoyが便利です。64Gなり128GなりのmicroSD1枚にWindows / Linux / その他レスキューイメージ等を１つのUSBデバイスに集約できます。  
https://www.ventoy.net/  

## 2. UEFIモードで起動
通常のPCの起動プロセスは大昔からあるBIOSとUEFIと2種類あります。可能ならばUEFIモードで起動 & UEFIでインストールすると何かと便利です。  
BIOS起動かEFI起動家はArchLinuxの起動画面(=grubのメニュー画面)が変わりますので判別できます(詳細な判別方法は後述します)。  
起動したらまずはキーボード配列を設定します。  
```zsh
# loadkeys jp106
```  
以下のフォルダが存在していればUEFIモードです。もしなければBIOS設定画面で変更する必要があります。
```zsh
# ls /sys/firmware/efi/efivars
```  

## 3. インストール作業の開始
★
## 4. ディスクのパーティション作成 & ファイルシステム選択
★



