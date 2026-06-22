**【ステップ 3. GPU ドライバをインストールしよう】**

2026/06/21 登 初版作成

- [1. GPU ドライバのインストールの必要性](#1-gpu-ドライバのインストールの必要性)
  - [1.1. GPU の搭載](#11-gpu-の搭載)
  - [1.2. GPU のドライバの必要性](#12-gpu-のドライバの必要性)
- [2. GPU ドライバのインストール方法](#2-gpu-ドライバのインストール方法)
  - [2.1. 前提](#21-前提)
  - [2.2. 必要ユーティリティのインストール](#22-必要ユーティリティのインストール)
  - [2.3. 呪文の入力](#23-呪文の入力)
  - [2.4. GPU ドライバのダウンロード](#24-gpu-ドライバのダウンロード)
  - [2.5. GPU ドライバのインストール](#25-gpu-ドライバのインストール)
  - [2.6. GPU ドライバのインストールが完了したことの確認](#26-gpu-ドライバのインストールが完了したことの確認)
  - [2.7. 念のため再起動](#27-念のため再起動)
- [3. GPU 用のメモリサイズの調整](#3-gpu-用のメモリサイズの調整)
  - [3.1. 事前準備](#31-事前準備)
  - [3.2. GPU 用メモリサイズの拡大](#32-gpu-用メモリサイズの拡大)
  - [3.3. GPU 用メモリサイズの拡大結果の確認](#33-gpu-用メモリサイズの拡大結果の確認)


# 1. GPU ドライバのインストールの必要性
## 1.1. GPU の搭載
本講義の受講者全員に対して、1 人あたり 1 台配布するサーバーマシンは、

```
GMKtec EVO-X2 AI ミニPC AMD Ryzen AI Max+ 395、LPDDR5X 8000MHz 128GB +2TB SSD (Max.8TB) Windows 11 PRO ゲーミングPC、AMD Radeon 8060S RJ45/Wi-Fi 7/BT 5.4、4画面 HDMI 2.1/DP 1.4/USB4 235Bパラメータ対応 静音ビジネス ゲーミング向け
```
という AI マシンである。このマシンには、かなり強力な GPU (Graphics Processing Unit) という半導体チップが搭載されている (より厳密には、CPU と GPU が一体となったチップが搭載されている)。GPU により、LLM を実用的速度で動作させることが可能になる。

## 1.2. GPU のドライバの必要性
GPU のような特別なハードウェアを、Linux 等の OS 上で動作させる各種アプリケーションソフトウェア (AI ソフトウェア等) から利用するためには、「ドライバ」というシステム系のソフトウェアが必要になる。ドライバは、特別なハードウェアを、実際にそのハードウェアを利用したいと考えるソフトウェアのために、抽象化するとともに、同種の他のハードウェアと同一の方法で取り扱うことができるように、ハードウェア間の差異を吸収するための、中間的なソフトウェアである。

仮にドライバという概念がなければ、アプリケーションソフトウェアは、いろいろなハードウェアを直接駆動させるための癖をすべて認識把握している必要がある。しかしながら、アプリケーションの開発時においては、ユーザによってそのアプリケーションがどのようなハードウェアとともに動作させられるのか、事前に予測することはできず、不都合が生じる。

そこで、ハードウェアを抽象化し、統一された操作方法でアプリケーションからハードウェアを駆動する「ドライバ」という仕組みが生み出された。新しいハードウェアを作った人は、その新しいハードウェア用ドライバを書けば、既存のアプリケーションソフトウェアを改変することなく、そのハードウェアをスムーズに呼び出すことができる。

# 2. GPU ドライバのインストール方法
今回必要となる GPU ドライバは、AMD 社独自のソフトウェアであり、標準の Ubuntu Linux には入っていない。GPU ドライバをインストールするには、結構複雑な手順が必要である。本来は、各自が自ら AMD 社のドキュメントなどを熟読して、それぞれ試行錯誤しながらインストールすべきものである。

しかし、そのような試行錯誤プロセスは、あまりにも難しすぎると、頓挫してしまい、そこから先に進めなくなる。そこで、本講義では、GPU ドライバのインストールについて、だいたいはこれをコピー＆ペーストすれば動くというような具合の手順を提示することにした。

## 2.1. 前提
【ステップ 2】までの手順がすでに完了しており、Linux に root 権限で SSH でログインできることを前提とする。以下、`sudo bash` (手抜き) 状態の状態か、あるいは root ユーザで Linux にログインした状態 (さらに手抜き) を想定して、具体的手順を解説する。

## 2.2. 必要ユーティリティのインストール
ドライバのインストールにあたって必要となるユーティリティを、以下のコマンドでインストールする。

```

apt-get -y update && apt-get -y install curl jq git build-essential python3-pip curl p7zip-full build-essential unzip python3-setuptools python3-wheel pipx

```

上記を実行すると、画面に、インターネット上の Ubuntu 社のサーバーから色々な雑多なソフトウェアパッケージファイルが自動的にダウンロードされる様子が表示される。これは眺めていて楽しいものである。無事すべてのパッケージのダウンロードが終わったら、インストールも自動的になされる。その結果、画面に以下のようなメッセージが表示されれば、成功である。

```
(... 中略 ...)

Scanning processes...
Scanning processor microcode...
Scanning linux images...

Running kernel seems to be up-to-date.

The processor microcode seems to be up-to-date.

No services need to be restarted.

No containers need to be restarted.

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.
```

## 2.3. 呪文の入力
GPU を利用するに先立ち、呪文のように、次の 2 つのコマンドを実行する。

第一のコマンドは、次のとおりである。

```
usermod -aG video $USER
```

第二のコマンドは、次のとおりである。

```
newgrp video
```

## 2.4. GPU ドライバのダウンロード
AMD の GPU ドライバは、「rocm」という名前である。本講義では、「7.2.0.1」というバージョンをインストールすることにする。

まず、ファイルを次のコマンドでダウンロードする。このテキストの表示上、横方向にはみ出てしまうかもしれないが、単にコピー＆ペーストすればよい。

```

curl --fail --raw --location --create-dirs --output-dir /data1/tmp/ --remote-name https://download1.start.coins.tsukuba.ac.jp/01_public/001_download1/amdgpu/7.2.1/rocm-installer_1.2.6.70201-48-81~24.04.run

```

ダウンロードが終わったら、ダウンロード済みファイルを確認する。

```
ls -la /data1/tmp/
```

次のようになっていればよい。

```
-rw-r--r-- 1 root root 9388029929 ＜日時＞ rocm-installer_1.2.6.70201-48-81~24.04.run
```

また、一応、ファイルが破損していないかどうか確認する。

```
cd /data1/tmp/

sha1sum *
```

次のようになっていればよい。

```
1023dc9ab40c752e25b2b95ed1fb3c8ef5d88e09  rocm-installer_1.2.6.70201-48-81~24.04.run
```

この sha1sum というコマンドは、ダウンロードしたファイルが破損していないかどうか確認するために、よく利用する。上記例における「1023dc9ab40c752e25b2b95ed1fb3c8ef5d88e09」という数字の羅列は、「ハッシュ値」と呼ぶ。1 ビットでもファイルが破損していれば、ハッシュ値が異なるので、すぐに気付くことができる。

対象ファイルのハッシュ値が異なる場合は、必ず異なるファイルである。対象ファイルのハッシュ値が同じであれば、おそらく同じファイルである可能性が極めて高い。しかし、厳密にはハッシュ値の衝突は確率的に発生し得るので、異なるファイル同士でも同じハッシュ値を持つ可能性がある。

余談であるが、ハッシュ値を求めるために利用できる方式として、上記のコマンドの SHA-1 という方式のほか、MD5 や SHA-2 方式などがある。暗号学者によると、SHA-1 は、特殊な条件下で、意図的に衝突を引き起こせるらしい。近年では SHA-2 を利用することが推奨されている。しかし、本講義のように、厳格なセキュリティ担保を不要とし、単にファイルが壊れていないかどうかだけを検証する目的であれば、SHA-1 でも、MD5 でも、実用上差し支え無い。

[ハッシュ値あるいはハッシュ関数](https://ja.wikipedia.org/wiki/%E3%83%8F%E3%83%83%E3%82%B7%E3%83%A5%E9%96%A2%E6%95%B0) という概念は、コンピュータに接する人生において、いろいろな文脈で、頻出であると思われるので、簡単に覚えておくとよい。

## 2.5. GPU ドライバのインストール
いよいよ GPU ドライバをインストールする。以下を実行する。

```

bash /data1/tmp/rocm-installer_1.2.6.70201-48-81~24.04.run deps=install target="/" rocm postrocm

```

このコマンドを実行すると、進捗状況がだらだらと表示される。最後に、次のように表示されれば完了である。

```
Setting up extra ROCm post install...
/etc content for component: rocm-opencl
Processing for rocm-opencl.
/etc/OpenCL/vendors/
Setting up extra ROCm post install...Complete.
Exiting Installer.
Installer log stored in: /root/rocm-installer/logs/install_1782031906.log
/root
Cleaning up rocm components...
Cleaning up rocm components...Complete
Cleaning up amdgpu components...
Cleaning up amdgpu components...Complete
==== Removing install-init.sh ====
==== Removing rocm-installer.sh ====
==== Removing deps-installer.sh ====
==== Removing cleanup-install.sh ====
```

## 2.6. GPU ドライバのインストールが完了したことの確認
次のコマンドを実行する。

```
amd-smi
```

次のように表示されれば、ドライバのインストールは成功している。
```
root@サーバー名:~# amd-smi
+------------------------------------------------------------------------------+
| AMD-SMI 26.2.2+e1a6bc5663    amdgpu version: 6.17.0-35 ROCm version: 7.2.1    |
| VBIOS version: 023.011.000.039.000001                                        |
| Platform: Linux Baremetal                                                    |
|-------------------------------------+----------------------------------------|
| BDF                        GPU-Name | Mem-Uti   Temp   UEC       Power-Usage |
| GPU  HIP-ID  OAM-ID  Partition-Mode | GFX-Uti    Fan               Mem-Usage |
|=====================================+========================================|
| 0000:c5:00.0  Radeon 8060S Graphics | N/A        N/A   0                 N/A |
|   0       0     N/A             N/A | N/A        N/A             155/2048 MB |
+-------------------------------------+----------------------------------------+
+------------------------------------------------------------------------------+
| Processes:                                                                   |
|  GPU        PID  Process Name          GTT_MEM  VRAM_MEM  MEM_USAGE     CU % |
|==============================================================================|
|  No running processes found                                                  |
+------------------------------------------------------------------------------+
```

また、次のように入力してみる。
```
rocm-smi
```

これに対して、次のように表示されれば良い。

```
======================================== ROCm System Management Interface ========================================
================================================== Concise Info ==================================================
Device  Node  IDs              Temp    Power     Partitions          SCLK  MCLK  Fan  Perf  PwrCap  VRAM%  GPU%  
              (DID,     GUID)  (Edge)  (Socket)  (Mem, Compute, ID)                                              
==================================================================================================================
0       1     0x1586,   8797   36.0°C  4.028W    N/A, N/A, 0         N/A   N/A   0%   auto  N/A     7%     0%    
==================================================================================================================
============================================== End of ROCm SMI Log ===============================================
```

## 2.7. 念のため再起動
どうやら、このドライバは、再起動しなくても動作するようだが、GPU ドライバのような大層なものをインストールした後は、念のため再起動することが望ましい。

```
reboot
```

# 3. GPU 用のメモリサイズの調整
前述のとおり、今回利用する GPU は CPU チップと一体化されており、CPU と GPU との間で、合計 128GB のメモリを共有する (取り合う) 関係になっている。

CPU と GPU のメモリ分配は、Linux のカーネルによって制御される。デフォルトで約 61GB (半分程度) が GPU メモリとして割り当てられているが、これでは不足する。100GB 程度を GPU メモリ、20GB 程度を CPU メモリとして利用するように調整してみよう。

## 3.1. 事前準備

まず、次のコマンドを実行する。

```
pipx ensurepath
```

そして、次のコマンドを実行する。

```
pipx install amd-debug-tools
```

## 3.2. GPU 用メモリサイズの拡大

いずれのコマンドも無事成功したら、

```
~/.local/bin/amd-ttm
```

と入力すると、現在の GPU 利用可能メモリサイズが表示されるはずである。おそらくは、おおむね以下のように表示されると思われる。

```
💻 Current TTM pages limit: 16182747 pages (61.73 GB)
💻 Total system memory: 123.46 GB
```

そこで、この設定を変更し、100GB を GPU に割り当てるようにしよう。

```
~/.local/bin/amd-ttm --set 100
```

と実行する。すると、次のように表示される。

```
🐧 Successfully set TTM pages limit to 26214400 pages (100.00 GB)
🐧 Configuration written to /etc/modprobe.d/ttm.conf
🐧 Checking if the initramfs image needs to be regenerated
🐧 Found initramfs images: ['/boot/initrd.img-6.8.0-71-generic', '/boot/initrd.img-6.17.0-35-generic']
✅ TTM module is included in initramfs: /boot/initrd.img-6.8.0-71-generic
🐧 The initramfs image needs to be regenerated
🐧 Updating initramfs using update-initramfs
update-initramfs: Generating /boot/initrd.img-6.17.0-35-generic
✅ Initramfs updated successfully
○ NOTE: You need to reboot for changes to take effect.
Would you like to reboot the system now? (y/n): 
```

いろいろ面白いアイコンやメッセージなどが出るが、要するに、カーネルの設定をすこしいじって、GPU メモリサイズを変更することに成功した、ということを意味している。

ご丁寧に、`Would you like to reboot the system now? (y/n): ` と聞いているので、`y` と入力して再起動する。

(間違えて `n` を入力してしまった場合は、`reboot` コマンドで再起動すればよい。)

## 3.3. GPU 用メモリサイズの拡大結果の確認
再起動後、

```
~/.local/bin/amd-ttm
```

を再度実行すると、

```
💻 Current TTM pages limit: 26214400 pages (100.00 GB)
💻 Total system memory: 123.46 GB
```

のように表示され、GPU メモリが 100GB に拡大されたことが分かる。




[**トップページに戻る**](../README.md)

