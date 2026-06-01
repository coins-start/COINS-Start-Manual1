**【ステップ 2.  AI サーバーマシンをセットアップして筑波大学のグローバル IPv4 アドレスと大学サブドメインを付与し、簡単な Web サーバを立ち上げよう】**

2026/06/01 登 初版作成

# 1. この度提供する AI サーバーマシン

この度、本講義の受講者全員に対して、1 人あたり 1 台配布するサーバーマシンは、

```
GMKtec EVO-X2 AI ミニPC AMD Ryzen AI Max+ 395、LPDDR5X 8000MHz 128GB +2TB SSD (Max.8TB) Windows 11 PRO ゲーミングPC、AMD Radeon 8060S RJ45/Wi-Fi 7/BT 5.4、4画面 HDMI 2.1/DP 1.4/USB4 235Bパラメータ対応 静音ビジネス ゲーミング向け
```
という AI マシンである。このマシンは、GPU 用メモリとして約 100GB を割り当てることができ、2025 年 ～ 2026 年ごろに次々に出てきた高性能のローカル LLM モデルデータを、軽々と読み込んで、高速に推論することができる。

![alt text]({4ED4E381-60E1-4838-94F0-BB767CCEF40E}.png)


# 2. AI サーバーマシンの BIOS 初期設定

この AI マシンにキーボードとモニタ (HDMI ケーブル) をつないで、電源を入れ、起動する。

注意点として、モニタは、HDMI 端子を利用することをお勧めする。DisplayPort 端子も付いているが、うまくモニタに表示されないことがあったためである。

ここでは、まだ、LAN ケーブルは、抜いておく。

キーボードは、近くの水色のボックスに、「構築設定作業用キーボード」と書かれたキーボードが合計 7 枚くらい入っている。

![alt text](image-1.png)

上記写真のとおり、RealForce というブランドのキーボードがある。これはとても快適なキーボードであり、コンピュータを用いる作業のストレスを大幅に軽減してくれる可能性があるので、見つけたら利用してみるとよい。

## 2.1. BIOS 画面に入る

マシンを起動したら、すかさず、Esc キーを連打する。うまくいくと、青っぽい BIOS 画面に入る。BIOS (Basic Input/Output System) とは、コンピュータの OS が起動するよりも前の段階で起動する、より基礎的なソフトウェアのことである。たとえば、人間で言うと、眠っている間でも呼吸や心臓などが動作しているが、BIOS は、少し不正確だが、一応そのような基礎的機能のようなものであると考えればよい。

## 2.2. 動作速度の最速化

BIOS 画面では、まず、下図のとおり、「Main」における「Power Mode Select」を開き、「Performance Mode」に設定する。これにより、動作速度の最速化設定がなされる (おそらく)。

![alt text](image-7.png)

## 2.3. 自動電源 ON の設定

次に、「Advanced」へ行き、次に「Auto Power On」を「Power On」に設定する。

これは、マシン稼働中に、万一停電や法定点検 (筑波大学では毎年秋にある) が発生して給電が途絶え、のちに復電した場合に、自動的にマシンの電源が ON になるような設定である。この設定をしておかなければ、停電の都度、マシンが止まってしまい、教室まで行って電源を ON にする必要が生じてしまう。

![alt text](image-2.png)

## 2.4. GPU に割り当てるメモリサイズの最小化 (逆説的)

次に、BIOS 画面で「Advanced」の「GFX Configuration」を開く。

![alt text](image-5.png)

![alt text](image-4.png)

ここで、「iGPU Configuration」を「UMA_SPECIFIED」に設定する。

次に、下図の通り、「UMA Frame buffer Size」を、「2G」または「512MB」などの最小値 (ハードウェアの世代によって、値が異なるようである) に設定変更する。これは、後に AI 目的で利用する際に重要である。逆説的であるが、この 「UMA Frame buffer Size」 (GPU に専用で割り当てるメモリ分量) を BIOS 上は最小にしておき、後でソフトウェア (カーネル) において大きな容量を GPU に割り当てることが、良好な AI 動作の秘訣である。

![alt text](image-6.png)

## 2.5. Secure Boot の無効化

次に、BIOS 画面で「Security」の「Secure Boot」を開く。

![alt text](image-8.png)

「Secure Boot」を「Disabled」に設定する (最初から「Disabled」になっている場合が多い)。Secure Boot を Disabled にしなければ、Linux のインストールができない可能性があるためである。

![alt text](image-9.png)

## 2.6. BIOS 設定の変更点の保存

最後に、BIOS 設定の変更点を本体に保存する。下図のように、「Save & Exit」から「Save & Reset」を開き、「Yes」を選択して、設定を保存して再起動する。

![alt text](image-10.png)

再起動すると、Windows 11 あるいは前の人がインストールしていた OS が起動してしまうと思われるが、それはひとまず無視してよい。

# 3. OS は Ubuntu 24.04.3 x64 版がお勧め
上記の AI マシンの能力をできるだけ発揮するためには、OS のバージョンが肝要である。著者がいろいろ試したところ、Ubuntu 24.04.3 x64 版を動作させることが最も無難であり、最適であるようである。そこで、Ubuntu 24.04.3 x64 版のインストールをお勧めする。


Ubuntu 24.04.3 x64 版には、サーバー版と、デスクトップ版がある。安定性・予測可能性・省メモり性などの観点から、サーバー版をインストールすることを強くお勧めする。


# 4. OS をインストールしてみよう
## 4.1. ubuntu-24.04.3-live-server-amd64.iso という ISO ファイルの取得

Ubuntu 24.04.3 x64 版のサーバー版をインストールすることに決めた場合、**「ubuntu-24.04.3-live-server-amd64.iso」** という ISO ファイルを用いてクリーンインストールすることをお勧めする。ISO ファイルとは、拡張子が「.iso」のファイルのことであり (「ISO」の由来は、おそらく、「ISO 9660」という規格の先頭部分から来ていると思われる)、CD-ROM や DVD-ROM の記録データをそのままファイル化したものである。

「ubuntu-24.04.3-live-server-amd64.iso」は、インターネットのいろいろな場所 (Ubuntu の公式サイトやミラーサーバ等) に置いてある。また、本講義のために立ち上げた以下の URL からでもダウンロード可能である。

**https://download1.start.coins.tsukuba.ac.jp/01_public/001_download1/ubuntu-24.04/260125_ubuntu_2404_noble_cdimage/x64/ubuntu-24.04.3-live-server-amd64.iso**

手元のノートパソコン (私物。無い場合はご相談) を用いて、ubuntu-24.04.3-live-server-amd64.iso をダウンロードしよう。

## 4.2. USB メモリへの書き込み

手元のノートパソコンに、ubuntu-24.04.3-live-server-amd64.iso をダウンロードしたら、これを USB メモリに書き込む必要がある。

USB メモリは、下記写真のように、3C206 の室の隅のコーナーのキャスター付き物置の一番上の **「USB メモリ (安物)」** と書かれた引出しに入っている。これを用いるとよい。

なお、ここの USB メモリに、誰かがいたずらでコンピュータ・ウイルス等を書き込んでいる可能性もあるので、使用にあたっては、十分注意すること。

![alt text](image.png)

USB メモリに ubuntu-24.04.3-live-server-amd64.iso を書き込む際には、注意する必要がある。USB メモリ上の FAT32 等でフォーマットされたファイルシステム上に ubuntu-24.04.3-live-server-amd64.iso を保存しても、効果がない。そうではなく、USB メモリの生データ (ファイルシステムという抽象的枠組みの内側ではなく、ファイルシステム等のデータがいわば物理的に記録される実在的データ) として USB メモリに書き込まれる必要がある。

そのためには、以下のようなフリーのツールを利用する。

Windows パソコン用:

- **Win32 Disk Imager Renewal**  (改良品・登がビルド/配布)  
  https://github.com/dnobori/DN-Win32DiskImagerRenewal/
- **Win32 Disk Imager** (オリジナル品)  
  https://sourceforge.net/projects/win32diskimager/

macOS パソコン用:

- macOS で USB 書き込みをやったことがないので分からないが、定評がある安全なツールを自ら調べて用いるとよい。(マルウェアに注意する必要がある。)

Solaris パソコン用:
- dd コマンドを用いること。極めて注意して用いる必要がある。

コンピュータや、クラウド基盤、AI システム等を取り扱う仕事に就く場合、USB メモリへの ISO イメージの書き込みという作業は、今後、人生において何百回、何千回も行なう可能性がある。これを機会に修得しておくと有利である。


## 4.3. USB メモリからサーバコンピュータを起動

サーバの電源が入っている場合は、一度、電源コードを抜くなどして (あるいは、再起動操作でもよい)、ubuntu-24.04.3-live-server-amd64.iso を書き込んだ USB メモリを本体の USB ポートに差し込み、再度電源を投入する。

**LAN ケーブルは、抜いておく。**

電源投入後、F7 キーを連打する。すると、添付のように、起動メニューが表示される。

![alt text](image-11.png)

ここでの表示の方法・内容は環境によって異なるが、色々吟味し、試行錯誤して、先ほど差し込んだ USB メモリから起動するように操作する。上記写真においては、この USB メモリは「UEFI: General UDisk 5.00」と記載されているが、この文字列のみを見てそれが USB メモリであることを見抜くことは至難の業である。

ここで、先ほど差し込んだ USB メモリから起動するように操作すると、一度コンピュータが再起動し、USB メモリからの起動が開始される。

## 4.4. Ubuntu のインストーラを実行して OS を入れる

USB メモリから起動開始すると、画面上に「GNU GRUB」と表示され、起動するシステムを選択できるようになる。ここで、「Try or install Ubuntu Server」を選択する。

![alt text](image-12.png)

すると、下記画面のように、小さい文字で大量の情報が流れながら、OS をセットアップするための OS が、USB メモリから起動する。

ここで、もし、USB メモリからの起動に失敗する場合は、少し前の USB メモリへの ubuntu-24.04.3-live-server-amd64.iso の書き込みが正常に行なわれていない可能性が高い。再度 ubuntu-24.04.3-live-server-amd64.iso を USB メモリに書き込んで試行してみる。

![alt text](image-13.png)

USB メモリからの起動が一段落すると、下図の写真のように、言語を選択する画面が表示される。ここでは、日本語を表示したい誘惑に惑わされずに、「English」を選択する。

Linux ベースの OS で言語設定を選択できる場合は、「English」を選択したほうが、日本語あるいはその他の言語を選択した場合と比較して、良い結果となる場合が多い。

![alt text](image-14.png)

次に、少しややこしいが、今度は接続されているキーボードの種類を選択する画面が表示される。ここで、デフォルト (初期値) は「English (US)」キーボードになっているが、日本語キーボードを接続している場合は、「Japanese」を選択したほうが良いかも知れない。

![alt text](image-15.png)

Ubuntu のインストール種類を選択する画面が表示されたら、「Ubuntu Server」を選択する。「Ubuntu Server (minimized)」ではないので、注意する。

![alt text](image-16.png)

その後、「Network configuration」という画面に進む。ここで、マシン本体からネットワーク (LAN ケーブル) を抜いているので、ネットワークへの検出は失敗するはずであるが、それは正常である。「Continue without network」を選択する。

![alt text](image-17.png)

「Proxy configuration」は、空欄のまま、「Done」を押し、次に進む。

![alt text](image-18.png)

「Ubuntu archive mirror configuration」は、設定値を変更することなく、「Done」を押し、次に進む。

![alt text](image-19.png)

「Checking for installer update...」が表示されるが、無視して、「Continue without updating」を選択して先に進む。

![alt text](image-20.png)

「Guided storage configuration」画面では、「Use an entire disk」を ON にし (初期状態で ON になっているはずである)、「Set up this disk as an LVM group」を ON にする。「Encrypt the LVM group with LUKS」は OFF のままとする。「Done」に進む。

![alt text](image-21.png)

「Storage configuration」画面で、コンピュータ本体の内蔵ストレージ (SSD) をどのように使うのかの方針が表示される。ここでは、あまり内容を変更せずに、「Done」で先に進む。

![alt text](image-22.png)

本当に実行してよいか? という最終確認が表示されるので、「Continue」を選択して先に進む。

![alt text](image-23.png)

「Profile configuration」画面が表示される。

- 「Your name」と「Pick a username」は、あなた (受講者) の好きなユーザ名を指定する。ユーザ名は、半角英数字 (1 文字目は英字) で小文字で数文字程度が望ましい。「Your name」と「Pick a username」には同じ文字列を入れるのがよい。
- 「Your servers name」には、このコンピュータに付ける愛称のようなものを入力する。半角英数字 (1 文字目は英字) で小文字で数文字程度が望ましい。「Pick a username」とは異なるものにする必要がある。
- 「Choose a password」および「Confirm your password」には、相当程度複雑なパスワードを指定する。簡単なパスワードを設定してしまうと、総当たり攻撃で、インターネット側から破られて不正アクセスされるリスクがあるため、かなり強固なパスワードを設定する必要がある。

ここで設定したユーザ名とパスワードは、メモしておくこと。

![alt text](image-24.png)

「Upgrade to Ubuntu Pro」画面が表示されるが、「Skip Ubuntu Pro setup for now」を選択し、「Continue」で次に進む。

![alt text](image-25.png)

「SSH configuration」画面では、必ず「Install OpenSSH server」を ON にした状態で、「Done」で次に進む。ここで、「SSH key」はインポートする必要はない (本来はここで SSH 公開鍵をインポートすると、よりセキュアになるが、本講義の受講者の観点からみると挫折の原因となるので、今回は SSH 公開鍵を用いた認証は行なわず、パスワードを用いた認証にとどめる)。

![alt text](image-26.png)

上記の「SSH configuration」画面が済むと、突然インストールが開始され、数秒 ～ 数十秒で完了する (このサーバーマシンはとても高速なので、数秒で完了するかも知れない)。インストールが完了すると、「Installation complete!」と画面上部に表示され (わかりにくい)、「Reboot Now」が選択できるようになる。「Reboot Now」を選択する。

![alt text](image-27.png)

「Please remove the installation medium, then press ENTER:」と表示される。ここで、USB メモリを抜き、Enter キーを押す。そうすると、コンピュータが再起動される。

![alt text](image-28.png)

コンピュータが再起動し、無事、先ほど Ubuntu がインストールされた内蔵 SSD から起動することに成功すると、画面上に、「login:」というプロンプトが表示される。

なお、LAN ケーブルは抜かれたままの状態であるため、起動時に LAN を検出しようとして、約 120 秒間、待機状態で止まることがあるが、これは正常である。LAN 検出中の画面が表示されたならば、120 秒間待つと、タイムアウトして、OS の起動が完了する。

![alt text](image-29.png)

OS の初期起動時は、画面上に、「login:」というプロンプトが表示された後に、さらに 15 秒くらい待つと、下記画面のように「cloud-init」という初期化プログラムが起動し、内部的にユーザの作成処理等が実施される。これが完了すると、「Cloud init (...略) finished」と表示される。

![alt text](image-30.png)

この状態で、Enter キーを何度か適当に押してから、「login:」というプロンプトで、先ほど指定したユーザ名とパスワードを入力する。すると、Linux のシェル画面 (bash というシェル) 上で OS に対して自由にコマンドを入力できる状態に入ることができる。


# 5. OS の初期設定をしよう

## 5.1. Ubuntu のソフトウェア自動アップデート機能を無効にする
Ubuntu 24.04 には、各種のソフトウェア自動アップデート機能が搭載されている。これは、インターネット上で新たに公開されたアップデートプログラムを自動的にダウンロードしてインストールする機能である。

一般的に、ソフトウェア自動アップデート機能はセキュリティ上有益であることが多い。しかし、今回のハードウェアと Linux を AI マシンとして利用する場合、カーネルやライブラリ、ドライバ等と呼ばれるソフトウェアが勝手にアップデートされてしまうと、AI エンジンの安定稼働が突然損なわれるリスクがあり、自動アップデート機能による不具合の不利益が大きいかも知れない。また、自動アップデート機能を OFF にしても、アップデート自体は手動で行なうことが可能である。そこで、ソフトウェア自動アップデート機能は OFF にすることをお勧めする。

### 5.1.1. まず root 権限に昇格する
Ubuntu にログインした最初の状態は、一般ユーザー権限である。重要な作業を行なう際には、「root 権限」という、すべての事柄が可能な権限に権限昇格する必要がある。これには `sudo` というコマンドを用いる。`sudo` の `su` とは、「Super User」(特権ユーザ) の略である。「do」は、実行する、という意味だと思われる。

本来、`sudo` コマンドは、1 個の処理ごとに呼び出すことが望ましい。たとえば、`abc` というコマンドを root 権限で実行したい場合は、`sudo abc` と入力する、という具合である。

だが、それは面倒なので、次のように入力すると、bash (シェル) 自体が root 権限で再度起動し、その上のすべての操作は root 権限で行なわれることとなる。

```
sudo bash
```

上述のコマンドについては、批判の対象になり得る。`sudo bash` は、sudo が折角実現しようとしている、1 コマンド単位における最小限特権実現によるメリットを損なってしまい、操作ミスによる危険な状態を招来させるというのが、批判の主な内容である。

それは、そのとおりである。確かに、複数の管理者ユーザが管理する重要なシステム等においては、事故の可能性を最小限にするため、sudo コマンドにおける権限昇格は、1 コマンドごとに実行することが望ましい。しかし、それはとても煩雑であるし、実は sudo コマンドを付けることによる副作用が発生し得るのである (少し複雑だが、「パイプ」と呼ばれる処理の挙動が変質してしまうことがある)。



このような点で、1 コマンドあたり 1 回の sudo 呼出しという理想を貫かせることは、初心者の視点からみると、楽しいはずのコンピュータが苦痛なものに感じられ、むしろ不利益のほうが大きい場合もある。また、本講義は、いわば遊びのための自作 AI やサーバシステム入門であり、重要な機密情報を取り扱ったり、あるいは、消失すると困るデータを取り扱ったりする局面はほとんどないか、限定的である。


加えて、特権制御は Linux のカーネルと呼ばれるソフトウェアが行なっているが、近年、AI を用いたコード分析により、Linux カーネルには多数の脆弱性が潜んでいることが明らかとなっており、Linux カーネル上で一般の低権限ユーザが sudo コマンドの実行なくしてカーネル上の root 特権を奪取できてしまう脆弱性は多数存在し得る。1 つの脆弱性をとり除いても、他の脆弱性が複数残っている状態なのである。そして、これらは攻撃者によって AI を用いて以前よりも簡単に発見され、突かれてしまう。サイバー攻撃に対する防御の観点では、ほとんど、一般ユーザ権限がひとたび奪取されれば root 特権も奪取されてしまう状態だと言っても誤りではない。これでは、厳格な sudo の利用を堅持するメリットが比較相対的に少ないことになってしまう。


そこで、本講義では、最小特権原則の原理主義を常に貫くのではなく、できるだけ、初心者が快適に作業を行なうことができるように、**快適のために、常に root (管理者特権) で作業することをそれなりに許容する** という方針で説明を行なう。

```
sudo bash
```

を実行すると、パスワードの再入力を求められる可能性があるので、入力する。

自らが現在一般ユーザ権限であるか、それとも root 権限であるかは、コマンドのプロンプトの末尾右端の 1 文字で判別できる。

```
$
```

の場合は、一般ユーザ権限である。他方で、

```
#
```

の場合は、root 権限である。または、

```
whoami
```

というコマンドの結果が `root` であるか否かによって判別することも可能である。

### 5.1.2. 次に、各種自動アップデート機能を無効にする

root 権限が取得できたら、次のコマンドをそれぞれを実行して、各種自動アップデート機能を無効にする。

```
apt-get -y remove unattended-upgrades
```

```
apt-get -y remove cloud-init
```

```
systemctl disable apt-daily.timer
systemctl mask apt-daily.timer

systemctl disable apt-daily.service
systemctl mask apt-daily.service
```

```
systemctl disable apt-daily-upgrade.timer
systemctl mask apt-daily-upgrade.timer

systemctl disable apt-daily-upgrade.service
systemctl mask apt-daily-upgrade.service
```

```
apt-mark hold snapd
```

# 6. 内蔵 SSD (2TB) の容量を限界まで使う (使用領域の容量拡大)

本サーバでは、内蔵 SSD として 2TB の SSD が付属している。しかしながら、初期状態では、Ubuntu のインストーラにより、この SSD のうち 100GB 程度しか使用されない状態になっている。AI モデルをダウンロードしたり、色々入れ換えたりするためには、使用容量を 2TB の限界まで拡大させることが有益である。

## 6.1. 現状の確認
まず、
```
df /
```
というコマンドを用いて、現在 OS がどれだけの容量を認識しているかを確認する。おそらく、合計容量 100G 程度しかなく、空きが 80GB 程度しかない結果になるであろう。

## 6.2. 容量の拡大
そこで、

```
lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
```

というコマンドを入力する。その後、

```
df /
```
を再度実行し、合計容量が 2TB に拡張されたら、容量拡大は成功である。


# 7. グローバル IPv4 アドレスを設定する
[**【ステップ 1. グローバル IP アドレスとサブドメインの割り当てを受けよう】**](../Step1_IP_Address_and_Domain_Name_Assignment/Step1_IP_Address_and_Domain_Name_Assignment.md) で割り当てを受けた IPv4 アドレス (`130.158.230.x` という形式の IPv4 アドレス) を、本サーバで利用できるようにする。

## 7.1. LAN ケーブルの接続
まず、本サーバの LAN ポートに LAN ケーブル (教室の隅の本講義のためのコーナーに置いてあるはず) の一端を刺す。LAN ケーブルは、写真のような場所に入っている。

より長い LAN ケーブルが必要な場合は、鹿野先生が、拾ってきた LAN ケーブルをたくさん持っているらしいので、鹿野先生に相談をするとよい。

![alt text](image-31.png)

LAN ケーブルのもう一端を、いよいよ、本講義のために用意されている上流ネットワークスイッチに接続する。室内にある「start-asw1」という水色のスイッチのポート 1 ～ 23 の空いているポートのうち 1 つを利用すること。詳しくはスイッチに貼られている注意書き (下記写真) を熟読すること。

![alt text](image-32.png)

LAN ケーブルを刺しただけでは、まだ、通信はできない。さらに、以下の設定をするまでは、LAN ポートはリンクダウン状態になっている可能性があるが、それは正常である。


## 7.2. LAN ポートの設定 (Linux OS 上)

先ほど、Linux 上で root 権限を取得した状態 (`sudo bash` を実行) になったが、この root 権限で、次を実行する。

まず、
```
ls -la /etc/netplan/
```
というコマンドを実行し、`/etc/netplan/` というディレクトリにすでに設定ファイル (拡張子が `.yaml` のファイル) が存在していないかどうか確認する。もし存在している場合は、`rm` コマンドを用いて、既存の当該ファイルを削除する。

次に、
```
ip address
```
というコマンドを実行し、物理的な LAN ポートの Linux 上でのデバイス名 (識別子) を確認する。おそらくは、`ip address` の実行結果は、

```
# ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eno1: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether 84:47:09:77:7a:26 brd ff:ff:ff:ff:ff:ff
    altname enp193s0
3: wlp195s0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether dc:56:7b:fb:c0:7d brd ff:ff:ff:ff:ff:ff
```

のように表示される。ここで、「2:」のところにある「eno1」が、LAN ポートの識別子であると考えられる。その理由であるが、「1:」の「lo」は、ループバックデバイスという特殊なデバイスを意味していて可能性から除外できるし、「3:」の「wl」で始まるデバイスは、WiFi アダプタを意味しているので除外できるためである。

そこで、LAN ポートの識別子は `eno1` であると仮定することができる。

次に、
```
nano /etc/netplan/01-static-ip.yaml
```
というコマンドを実行する。この `nano` とは、メモ帳のようなテキストエディタである。`/etc/netplan/01-static-ip.yaml` とは、当該ファイル名を開け (存在しなければ、作成せよ) という意味である。現在は `/etc/netplan/01-static-ip.yaml` は存在しないので、作成されることになる。nano が起動したら、本文部分で、

```
network:
 ethernets:
  eno1:
   dhcp4: false
   dhcp6: false
   addresses:
    - 130.158.230.★/26
   routes:
    - to: default
      via: 130.158.230.62
   nameservers:
    addresses:
     - 8.8.8.8
     - 1.1.1.1
```

のように入力し (サーバマシンはまだネットワークにつながっていない訳だから、これは、サーバマシンに物理的に接続したキーボードを用いて、頑張って入力する必要がある。コピー＆ペーストするというわけにはいかない。一種の苦行である)、保存する。

注意点として、スペース文字 (空白文字) によるインデントが極めて重要であるという点である。

上記のうち、`★` の部分は、[**【ステップ 1. グローバル IP アドレスとサブドメインの割り当てを受けよう】**](../Step1_IP_Address_and_Domain_Name_Assignment/Step1_IP_Address_and_Domain_Name_Assignment.md) で割り当てを受けた IPv4 アドレス (`130.158.230.★` という形式の IPv4 アドレス) の ★ 部分を入力する。

nano で、上記 `/etc/netplan/01-static-ip.yaml` を保存したら、nano を終了する。

念のため

```
cat /etc/netplan/01-static-ip.yaml
```

で意図した内容になっているか確認する。


その後、まず、
```
chmod 600 /etc/netplan/01-static-ip.yaml
```

を実行する。次に、文法チェックとして、

```
netplan generate
```

を実行する。ここでエラーが出たら、`/etc/netplan/01-static-ip.yaml` の内容に誤りがあるということになるので、よく吟味して修正する。

最後に、
```
netplan apply
```

を実行する。

ここで、うまくいけば、LAN 通信が可能となり、
```
ip address
```
コマンドの結果は、
```
root@dnaiserver1:~# ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 84:47:09:77:7a:26 brd ff:ff:ff:ff:ff:ff
    altname enp193s0
    inet 130.158.230.1/26 brd 130.158.230.63 scope global eno1
       valid_lft forever preferred_lft forever
    inet6 2001:2f8:3a:1731:8647:9ff:fe77:7a26/64 scope global dynamic mngtmpaddr noprefixroute 
       valid_lft 2591999sec preferred_lft 604799sec
    inet6 fe80::8647:9ff:fe77:7a26/64 scope link 
       valid_lft forever preferred_lft forever
3: wlp195s0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether dc:56:7b:fb:c0:7d brd ff:ff:ff:ff:ff:ff
```

のように変化する。

## 7.3. LAN ポートを用いたグローバル IPv4 アドレスの疎通確認

ここまでが正しく行えている場合は、

```
ping 8.8.8.8
```

と入力すると、Google の Public DNS サーバー (よく ping 検査で利用される) に ping が飛ぶはずである (ping を終了する場合は、Ctrl + C を押すこと)。

また、
```
ping -4 www.google.com
```
や
```
ping -4 m.root-servers.net
```
を実行しても、ping が飛ぶはずである。

これらは、インターネット上の著名なサーバに ping テストを行ない、インターネットの疎通を確認することとなるコマンドである。インターネット上の対象サーバに若干の負荷をかけることになるが、これらの対象サーバはもともと極めて膨大な件数のリクエストを捌いており、通常の ping コマンドで ping を 1 秒おきに合計数回送付する程度であれば、問題はほとんどないと考えられる。


# 8. SSH でクライアント PC からサーバに接続してみる

ここまでの設定は、サーバに物理的に接続されたキーボードやモニタを用いていた。しかし、この方法だと、少し設定変更するだけでも、物理的に 3C206 室に来ないといけないので、大変である。

そこで、大学内の別の室や、あるいは、自宅やモバイル回線などのインターネット接続環境から、自分のサーバに対して管理のための接続をすることができれば便利である。そのためには、SSH (Secure Socket Shell, 上記の bash のようなシェル環境にリモート PC から接続して遠隔管理するための比較的安全な仕組み) をいう仕組みが利用できる。

## 8.1. 手元のパソコンの準備

受講者の手元のパソコン (ノートパソコン等) を、任意の方法でインターネットに接続する。

- coins-wireless
- 学情センター WiFi
- 大学内の他の室の学内 LAN
- 自宅のインターネット接続 (光ファイバ等)
- モバイルインターネット接続

などのいずれの回線でもよい。

## 8.2. ping の確認

次に、当該コンピュータから、

```
ping ★.start.coins.tsukuba.ac.jp
```
(★ は、[【ステップ 1. グローバル IP アドレスとサブドメインの割り当てを受けよう】](../Step1_IP_Address_and_Domain_Name_Assignment/Step1_IP_Address_and_Domain_Name_Assignment.md) で割り当てを受けた、「.start.coins.tsukuba.ac.jp」の、固有のサブドメインラベル)

というコマンドで、まず ping が届くかどうか確認する。

もし、上記で名前解決に失敗している場合は、
```
ping 130.158.230.◎
```
(◎ は、割り当て IPv4 アドレスの固有の可変部分)

のように、IPv4 アドレス直打ちで、ping を飛ばしてみるとよい。これで成功するのに、「★.start.coins.tsukuba.ac.jp」のドメイン名を指定した場合は失敗する場合は、[【ステップ 1. グローバル IP アドレスとサブドメインの割り当てを受けよう】](../Step1_IP_Address_and_Domain_Name_Assignment/Step1_IP_Address_and_Domain_Name_Assignment.md) のサブドメインの設定が誤っている可能性が高いので、確認してみるとよい。

## 8.3. SSH 接続をしてみる

ping が通れば、今度は、SSH 接続を行なってみる。SSH 接続を行なうためには、手元の作業用ノートパソコンに SSH クライアントソフトウェアが必要である。

ssh の接続先は、上記の ping と同じ
```
★.start.coins.tsukuba.ac.jp
```

である。ユーザ名とパスワードは、ローカル画面でログインする際と同様である。

### 8.3.1. Windows パソコンの SSH クライアント

- Poderosa  
  https://ja.wikipedia.org/wiki/Poderosa/
- Tera Term  
  https://teratermproject.github.io/
- Windows 標準の OpenSSH クライアント (ssh コマンド)

などが存在する。いずれも一長一短があるので、試してみて気に入ったものを利用するとよい。


### 8.3.2. macOS や Linux 等の SSH クライアント

ssh コマンドが標準で付いているので、これを利用するとよい。

## 8.4. SSH で可能な操作

ssh で接続が成功したら、接続先の Linux サーバにキーボードと画面を接続して直接操作している場合とほとんど同様に、接続先の Linux サーバをリモート管理することができる。

`sudo` コマンドを用いて、管理者権限に昇格できる点も同一である。


## 8.5. (希望する場合のみ) root で SSH 接続を可能にする

Linux (UNIX) のセキュリティ上の慣習として、SSH 等でログインする際に、直接 root 権限でログインすることは禁止し、まず一般ユーザでログインし、その後に `sudo` コマンド等で root 権限に昇格して作業をする、という手法が一般的である。

しかし、1 人で管理する実験的なサーバ等で、ほとんど root 権限で作業するような場合で、十分複雑なパスワードあるいは公開鍵認証をかけるなどセキュリティ対策をそれなりに取っているならば、root ユーザで直接 SSH 接続することを許容することも、場合によっては選択肢となり得る。

この場合は、サーバ上で、次のような設定を行なうとよい (SSH 経由でも設定できる)。

まず、テキストエディタで、`/etc/ssh/sshd_config` を開く。

```
sudo nano /etc/ssh/sshd_config
```

次に、

```
#PermitRootLogin prohibit-password
```
となっている行を
```
PermitRootLogin yes
```
に置換する。

そして、`/etc/ssh/sshd_config` を上書き保存して nano を終了し、

```
/etc/init.d/ssh restart
```

を実行する。

また、
```
sudo passwd root
```
コマンドで、root ユーザーのパスワードを設定する。(初期状態では、root のパスワードは設定されていない。)

この状態で、クライアント端末から、当該サーバに root ユーザとして直接 SSH 接続することが可能となる。

## 8.6. (発展) SSH の公開鍵認証の設定
近年、キーロガーや、パスワードの使い回しなどが原因で、パスワードリストがダークウェブ (サイバー犯罪者のための情報交換サイト) に流出することがある。万一利用しているパスワードが流出すると、当該パスワードを用いてログイン可能な SSH サーバーは、攻撃者によって不正侵入されるリスクが生じる。

これを防ぐために、SSH では、公開鍵認証の設定が可能である。具体的には、SSH 用の公開鍵・秘密鍵のペアを作成しておき、公開鍵を SSH サーバ側に設置する。SSH クライアントは秘密鍵を保持する。この方法により、安全性を一定程度向上させることができる。

ただし、SSH クライアントパソコンに入っている秘密鍵 (これは、原則として、クライアントパソコンのディスク上に記録されており、クライアントパソコンを侵害した攻撃者に盗まれるリスクがある) が結局奪取された場合は、パスワードが漏洩したのと同じ結果となる。不正侵入のリスクがゼロになるわけではない。緩和策として、秘密鍵を利用する都度、パスフレーズを入力する方法が存在する。しかし、これは結構煩雑であり、また、キーロガーが仕掛けられた場合はパスフレーズも奪取されるリスクがある。公開鍵認証を過信するのではなく、本質的にはパスワード認証と同様のリスクがあることを認識して利用するのがよい。

SSH の公開鍵認証の設定方法は、インターネットで検索すると、解説記事がいくつも見つかるので、それらをいくつか見比べて実施するとよい。


# 9. Ubuntu の OS をアップデートしよう

SSH でログインができたので、Ubuntu の OS の各種コンポーネントおよびカーネルをアップデートする操作をしてみよう。

以下の作業は、手元の作業用ノートパソコン等から、SSH で、対象サーバーに接続した状態で操作することを想定している。また、(手抜きであるが) `sudo bash` を実行し常時 root 権限でシェルを操作している状態を想定している (そのため、各行に `sudo` を入れていない)。

まず、
```
uname -a
```

というコマンドで、現在のカーネルのバージョンを確認する。おそらくは、

```
Linux <サーバ名> 6.8.0-71-generic #71-Ubuntu SMP PREEMPT_DYNAMIC Tue Jul 22 16:52:38 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```

という結果になるはずである。これは、`ubuntu-24.04.3-live-server-amd64.iso` の ISO イメージに入っているカーネルである。

次に、
```
apt-get -y update
```

というコマンドを実行してみる。`apt-get -y update` は、インターネット上にある、Ubuntu の最新パッケージリストを取得するコマンドである。update とあるが、インストール済みソフトウェアを更新するコマンドではない。


なお、`apt-get` というコマンドは、Ubuntu 系 (Debian 系) の OS で、パッケージ化されているソフトウェアをダウンロード・インストールしたり、更新したりするためのコマンドである。`apt-get` コマンドを利用する場合は、いつでも、まず `apt-get -y update` を実行してから、目的のコマンドを実行するのがよい。

`apt-get -y update` が成功したら、次に、以下を実行する。

```
apt-get -y upgrade
```

このコマンドは、すでにインストールされている各種コンポーネントを更新するコマンドである。数分間時間がかかる。

上記の `upgrade` コマンドでは、カーネルと呼ばれる、OS の核心部分が更新されないことが多い。また、AI の実験を後に行なうためには、`hwe` と呼ばれる、新しめのカーネルを手動で入れたほうがよい。そこで、最後に、

```
apt-get -y install linux-generic-hwe-24.04 linux-headers-generic-hwe-24.04 linux-image-generic-hwe-24.04
```

を実行する。これにより、最新のカーネル (hwe 版という、AI 実験に向いたバージョン) がインストールされる。

その後、

```
reboot
```

というコマンドでコンピュータを再起動する。

再起動すると、SSH が一度切れてしまう。再起動が完了した後に SSH を再接続し、

```
uname -a
```

というコマンドで、アップデート後のカーネルのバージョンを確認する。おそらくは、

```
Linux <サーバ名> 6.17.0-35-generic #35~24.04.1-Ubuntu SMP PREEMPT_DYNAMIC Tue May 26 19:30:42 UTC 2 x86_64 x86_64 x86_64 GNU/Linux
```

というように、少なくとも `6.17.0-35-generic` というバージョンか、それよりも新しいバージョンに更新されているはずである。

SSH 経由でカーネルを更新することに成功できたならば、SSH を初めて利用する受講者の視点でも、一応の Linux に関するリモートの基本操作の方法が習得できたことになる。

基本的に、Linux のサーバを手動で構築・運用する場合は、ほとんどこれと同様の操作方法を日々行なうことになる。


# 10. Web サーバを立ててみよう
折角、
```
★.start.coins.tsukuba.ac.jp
```
(★ は自分のハンドルネーム)

というサブドメインを取得したのだから、サーバやグローバル IPv4 アドレスの動作テストを兼ねて、
```
https://★.start.coins.tsukuba.ac.jp/
```
という URL でアクセス可能な Web サイトを立ち上げてみよう。

以下の作業は、手元の作業用ノートパソコン等から、SSH で、対象サーバーに接続した状態で操作することを想定している。また、(手抜きであるが) `sudo bash` を実行し常時 root 権限でシェルを操作している状態を想定している (そのため、各行に `sudo` を入れていない)。

## 10.1. nginx (Web サーバ) のインストール
近時は、ngnix という Web サーバが人気である。安定性、高速性、リバースプロキシという機能の柔軟性、セキュリティ (脆弱性が比較的少ない) 等の原因で人気があるようである。

以下の方法で、ngnix をインストールする。

```
apt-get -y update && apt-get -y install nginx libnginx-mod-http-fancyindex apache2-utils
```

ここで、`&&` は、その左右のコマンドを、左 → →の順で、両方実行する (左側が失敗した場合は、右側は実行しない) という意味である。

## 10.2. nginx (Web サーバ) でホストしたいコンテンツ (仮) の作成

nginx のインストールで、特にエラーが出なければ、次に、nginx で Web サーバとしてホスティングしたい Web コンテンツを作成する。といっても、とても手抜きなので、

```
mkdir -p /data1/web_server/default/

echo "Welcome to My Web Site! by ●●" > /data1/web_server/default/index.html
```

のように実行するとよい。(●● の部分には、ローマ字などで、自分の氏名を入れることが望ましい。[「筑波大学における情報システム利用のガイドライン」の「3.4 情報発信における責任の明示」](https://www.t-act.tsukuba.ac.jp/flow/pdf/net-use.pdf)参照。)

その結果を

```
cat /data1/web_server/default/index.html
```

として確認し、
```
Welcome to My Web Site! by ●●
```
という文字列が入っていることを確認する (これは、実は HTML ですらないテキスト文字列であるが、一応の実験としてはこれで十分である)。


## 10.3. nginx の HTTPS サーバー機能で使用する SSL 証明書の取得

nginx は、https (SSL。暗号化通信の仕組み) に対応した Web サーバとして稼働することができる。SSL 証明書は、`https://` で始まる URL の Web サーバーと、クライアント端末 (Web ブラウザ) との通信の暗号化のセキュリティを高めるための仕組みである。

今回の実習では、`◎.start.coins.tsukuba.ac.jp` というドメイン名を用いて、各受講者の方々は、それぞれ SSL で Web サーバを立ち上げることを想定している。ここで、本来であれば、`◎.start.coins.tsukuba.ac.jp` というそれぞれのドメインごとに SSL 証明書を有償で取得したり、あるいは、Let's Encrypt 等の無償 SSL 証明書取得サービスを利用したりする必要がある。しかし、これは少し大変である。Let's Encrypt の場合、1 つのドメインの配下から多数の異なるリクエストが飛ぶと、rate limit という閾値に引っかかってエラーになる可能性もある。

そこで、本講義の運営者は、`*.start.coins.tsukuba.ac.jp` という「ワイルドカード証明書」を、定期的に取得・更新し、その証明書ファイルデータを常時共用の Web サーバに置くサービスを構築した。

`*.start.coins.tsukuba.ac.jp` という「ワイルドカード証明書」は、`◎.start.coins.tsukuba.ac.jp` の `◎` の部分が任意のすべてのサーバで、共通的に使用することができる。

証明書ファイルおよび秘密鍵ファイル (この 2 つはセットで利用する) は、以下のコマンドで取得できる。

```
curl https://user_cert:pass_●●●●●●●●@ssl-cert-server.start.coins.tsukuba.ac.jp/wildcard_cert_files/start.coins.tsukuba.ac.jp/latest/cert.cer -o /etc/nginx/ssl_cert.cer

curl https://user_cert:pass_●●●●●●●●@ssl-cert-server.start.coins.tsukuba.ac.jp/wildcard_cert_files/start.coins.tsukuba.ac.jp/latest/cert.key -o /etc/nginx/ssl_cert.key
```

ただし、上記のうち `pass_●●●●●●●●` は、この証明書サーバにアクセスするための必要なパスワードを意味し、●●●●●●●● の部分は伏せ字となっている。この ●●●●●●●● の部分は、毎度おなじみの `start-maintepc1` (3C206 計算機室) に、シールで記載されている。それを目視して、置換すること。

上記を実行すると、証明書ファイルは `/etc/nginx/ssl_cert.cer` に、秘密鍵ファイルは `/etc/nginx/ssl_cert.key` に、それぞれ保存されるはずである。

```
cat /etc/nginx/ssl_cert.cer
```

```
cat /etc/nginx/ssl_cert.key
```

をそれぞれ実行し、証明書ファイルと思われるような、たくさんの文字が書かれたファイルが保存されていることを確認してから、次のステップに進むこと。


## 10.4. nginx の設定ファイルの記述

まず、既存の nginx の設定ファイル (サンプル) を一応バックアップする。

```
cp -n /etc/nginx/nginx.conf /etc/nginx/nginx.conf.original1
```

次に、既存の nginx の設定ファイルの中身を空にする。

```
truncate -s 0 filename /etc/nginx/nginx.conf
```

最後に、nano (テキストエディタ) を用いて、nginx の設定ファイルを編集する。
```
nano /etc/nginx/nginx.conf
```

nano の編集ボックスに、以下をコピー＆ペーストして、SSH 経由で貼り付けるとよい。

```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 4096;
    multi_accept on;
    use epoll;
}

http {
    server_tokens off;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    types {
        application/manifest+json webmanifest;
    }
    
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers "AES128-SHA:ALL:!EXPORT:!LOW:!aNULL:!eNULL:!SSLv2";
    
    log_format main '[$time_local] Client=[$remote_addr]:$remote_port Server=[$server_addr]:$server_port Host=$host Proto=$server_protocol Request="$request" Status=$status Size=$body_bytes_sent Time=$request_time Referer="$http_referer" UserAgent="$http_user_agent" Username=$remote_user Ssl=$ssl_protocol Cipher=$ssl_cipher Sni=$ssl_server_name';
    
    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log;
    
    gzip off;
    
    map $remote_addr $remote_endpoint_v4v6 {
        ~^[0-9.]+$          "$remote_addr:$remote_port";
        ~^[0-9A-Fa-f:.]+$   "[$remote_addr]:$remote_port";
        default             "1.0.0.1:1234";
    }
    
    server {
        listen 80 default_server;
        listen [::]:80 default_server;
        listen 443 ssl default_server;
        listen [::]:443 ssl default_server;
        
        server_name _;
        ssl_certificate /etc/nginx/ssl_cert.cer;
        ssl_certificate_key /etc/nginx/ssl_cert.key;
        
        location / {
            root /data1/web_server/default/;
        }
    }
}
```

上記を貼り付けて `/etc/nginx/nginx.conf` に上書き保存し、nano を終了したら、次に、

```
nginx -t
```

を実行する。その結果、

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

と表示されれば、`/etc/nginx/nginx.conf` の内容は一応文法問題はないということになる。エラーが出たら、文法間違いがあることになる。

エラーがなければ
```
systemctl stop nginx
```
として nginx を一度停止し、
```
systemctl start nginx
```
として nginx を再開する。


そして、手元の作業用 PC などの Web ブラウザで
```
https://★.start.coins.tsukuba.ac.jp/
```
(★ 部分は置換すること)

にアクセスしてみよう。その結果、以下の画面写真のように、先ほど `/data1/web_server/default/index.html` に置いたテキストデータが Web ブラウザに表示され、SSL 証明書エラーも表示されなければ、成功である。

(もし、上記のいずれかのステップに抜け漏れがあれば、エラーになってしまうはずである。)

![alt text](image-35.png)

次のステップとしては、

```
/data1/web_server/default/
```

というディレクトリにリッチなコンテンツを置けば、その内容がインターネット上に配信され、インターネット上の誰もが、筑波大学のサブドメイン `https://★.start.coins.tsukuba.ac.jp/` でアクセス可能である。


**これは、次のことを意味する。** あなたの自作サーバは、今、インターネットに対して、大学の公式ドメインを用いて、自己の責任で、直接に Web サイトを公開・配信している状態となっている。PC だけでなく、スマートフォンからもアクセスできるし、学内 LAN だけでなく、自宅や友達の PC からでもアクセスできる。この `https://★.start.coins.tsukuba.ac.jp/` で Web サイトを公開できたことを記念して、友達などに SNS でその旨を周知してもよい (大学の品格を低下させる公序良俗に反するメッセージは、掲載しないこと)。

**このような行為は、1990 年代末に、米国の多数の名門大学で、学生たちが、たとえば akebono.stanford.edu や google.stanford.edu のような大学サブドメインを登録し、そこで遊びの実験サーバーを立ち上げたことと全く同じ性質を有する。** 前述のとおり、これらの米国大学の例は、後の Yahoo! 社や Google 社などの全世界的プラットフォーマー企業に発展した。あなたも、この活動を発展させることにより、場合によっては、Yahoo! 社や Google 社を超える全世界的プラットフォーマー企業を立ち上げることが可能かも知れない。また、そこまで行かなくても、生涯の収入を得るための事業の基となる程度の技術研究の成功は、十分に可能である。



# 11. 発展
上記の Web サーバには、実は、少なくとも、以下の 2 つの問題がある。

1. 上述の「curl」コマンドで「ssl-cert-server.start.coins.tsukuba.ac.jp」という証明書サーバから取得した証明書は、最長で、取得時から約 3 ヶ月 (より短縮される可能性がある) 間の有効期限しかない。このまま放っておくと、約 3 ヶ月後には、期限切れのエラーになってしまう。  
   もちろん、再度 curl で同一のサーバから証明書を取得した上で、nginx の設定をリロードすれば、新しい有効期限を有するサーバ証明書に更新される。しかし、これを数週間ごとに手動で行なうのは、煩雑である。これを「cron」等を用いて自動化できないだろうか。
2. この Web サーバは、`http://` プロトコルでも、`https://` プロトコルでも、アクセス可能である。しかし、最近の慣習として、`http://` でアクセスした場合は、強制的に `https://` にリダイレクトする、という挙動が望ましいとされている。これを実装しないと、友達に URL を見せたときに馬鹿にされるかも知れないという点で不安である。nginx の設定ファイルだけで、これを実現できないだろうか。

いずれも、小さな問題であり、すぐに解決しなくとも支障はない。余裕があれば、インターネットでその方法を調べて、解決に向けて色々と試してみるとよい。


# 12. 次のステップ
今回で、任意の Web サーバをインターネット経由で公開する方法を習得できたと思われる。

次のステップは、いよいよ、この Web サーバを経由して、同一の AI サーバマシン上で、ChatGPT のような AI サービスを構築することになる。








