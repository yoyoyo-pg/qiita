---
title: UbuntuのEC2にJMeterをインストールし、CLI上で負荷試験を実行するまで
tags:
  - JMeter
private: false
updated_at: '2024-04-04T19:46:52+09:00'
id: f6673bf9b6997ae84518
organization_url_name: null
slide: false
ignorePublish: false
---
## この記事について

- JMeterを利用する機会があったので、必要最低限の実施事項を記載します
- EC2の立ち上げや、シナリオの作成については省略します

## 利用するAMI

- Ubuntu 22.04を利用します

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/411902/c88c1eab-b284-34ea-a843-a8aec679cca8.png)

## 起動後

### JDKのインストール

```bash
cd /home/ubuntu
# install JDK
wget -O - https://apt.corretto.aws/corretto.key | sudo gpg --dearmor -o /usr/share/keyrings/corretto-keyring.gpg && \
echo "deb [signed-by=/usr/share/keyrings/corretto-keyring.gpg] https://apt.corretto.aws stable main" | sudo tee /etc/apt/sources.list.d/corretto.list
sudo apt-get update; sudo apt-get install -y java-17-amazon-corretto-jdk
```

参考：[Amazon Corretto 17 Installation Instructions for Debian-Based, RPM-Based and Alpine Linux Distributions](https://docs.aws.amazon.com/ja_jp/corretto/latest/corretto-17-ug/generic-linux-install.html)  

### JMeterのインストール

```bash
# install JMeter
wget https://downloads.apache.org//jmeter/binaries/apache-jmeter-5.6.tgz
tar -xvzf apache-jmeter-5.6.tgz
```

参考：[Download Apache JMeter](https://jmeter.apache.org/download_jmeter.cgi)

## 任意の実施事項

### ヒープメモリの設定

- `Uncaught Exception java.lang.OutOfMemoryError`となる事があるので、場合によっては調整が必要
- jmeterインストールディレクトリ（今回だと`/home/ubuntu/apache-jmeter-5.6/bin`）内の`jmeter`ファイルの以下を編集
  
```bash
# system's memory availability:
: "${HEAP:="-Xms1g -Xmx1g -XX:MaxMetaspaceSize=256m"}"
```

:::note info
Windowsの場合は`jmeter.bat`、Linuxの場合は`jmeter`ファイルの修正が必要です。
:::

参考：[1.4 Running JMeter](https://jmeter.apache.org/usermanual/get-started.html#running)

※実際にJMeter起動中に`top`コマンドを叩くと、`VIRT`が増えている事が確認できます

### テスト用ディレクトリの作成

- GUIで作成した`test.jmx`ファイルと、ログファイル用にディレクトリを作成
- スレッド数など、適宜`test.jmx`ファイル内の記載は調整

```bash
cd /home/ubuntu
mkdir load-test
```

## 負荷試験を実行

```bash
cd /home/ubuntu/apache-jmeter-5.6/bin
./jmeter -n -t /home/ubuntu/load-test/test.jmx -l /home/ubuntu/load-test/test-log.jtl
```

### CLIモードのオプション

- `-n`
  - JMeterがcliモードで実行されることを指定
- `-t`
  - 実行ファイル名を指定
- `-l`
  - サンプル結果を記録するファイルを指定

参考：[1.4.4 CLI Mode (Command Line mode was called NON GUI mode)](https://jmeter.apache.org/usermanual/get-started.html#non_gui)
