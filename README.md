# README

## 目次

- LICENSE
- Playbook の動作概要
- 免責事項
- Playbook 動作側環境の要件
- 実行対象側確認済情報
- Playbook の実行方法
- Playbook の実行内容
- 動作環境の削除方法
    - 仮想環境の削除
    - モジュールの削除

## LICENSE

This software is released under the MIT License, see LICENSE.txt.

## Playbook の動作概要

本 Playbook は snmposter を実行する環境を作成する Playbook です。  
現状 MIRACLE LINUX 8.6 のみ動作確認をしていますが、気分が乗れば別の OS 版も作成してみたいと思います。  

## 免責事項

本 Playbook を使用して発生した障害、損失に対しては当方では責任を負いません。  
ご使用は自由にしていただいて構いませんが、事前によくご確認をお願いします。  

## Playbook 動作側環境の要件

- 動作確認済 OS：
    - MIRACLE LINUX release 9.2 (Feige)
- ansible-core 2.14.2 以上が動作している環境
    - ↑ この環境でのみ動作検証を実施

## 実行対象側確認済情報

- OS：
    - MIRACLE LINUX release 8.6 (Peony)

## Playbook の実行方法

- リポジトリのクローン
- hostlist の作成
    - hostlist.org をコピーして hostlist を作成
    - `[snmposter_hosts]` 配下に実行対象となるホストの情報を記載する。
        - 先頭にホスト名、`ansible_host=` のあとに実行対象の IP アドレスを記載する。
    - `[all:vars]` 配下に SSH 接続に使用する設定情報を記載する。
    - hostlist を上手に書けば複数の snmposter 実行環境を作成できる。が、手元で複数ホスト準備するのが面倒だったので、ここではそこまでの検証は行っておらず、手順も記入しない。
- 事前ログイン
    - Playbook 実行側から実行対象に SSH でログインができることを試しておく。
        - この時使用するユーザー名/パスワードは hostlist に設定したユーザー名/パスワードを使用する。
- コマンドの実行
    - コンソールで以下のコマンドを実行する。
        - `ansible-playbook snmposter_setup.yml -i hostlist`

## Playbook の実行内容

本 Playbook は以下を順に実行する。  
  
- SSH 接続確認
- yum で以下モジュールのインストール
    - python2-devel
    - gcc
    - wget
    - tar
- pip2 で以下のモジュールをインストール
    - virtualenv
- tar.gz をダウンロードして以下のモジュールをインストール
    - TwistedSNMP (0.3.13)
    - pysnmp-se (3.5.2)
- snmposter 起動用サンプルファイルのコピー

## snmposter の実行方法

Playbook の実行が正常に完了すると、snmposter が使用できる環境になっている。  
本 Playbook は起動に snmposter の実行必要なサンプルファイルを含んでおり、サンプルを使用した起動方法について以下に記載する。  
  
- `/snmposter/snmp_files/agents_org.csv` をコピーして `/snmposter/snmp_files/agents.csv` を作成する。
- エディタで `/snmposter/snmp_files/agents.csv` を開き、`<agent_ip>` を snmposter で使用する IP アドレスに書き換える。
    - OS で使用している IP アドレスは指定できない。別途 IP アドレスを用意すること。
    - ここで書き換えた IP アドレスは snmposter を実行している PC のインターフェースに追加される。
- ターミナルを起動して以下のコマンドを実行し、snmposter 実行用仮想環境を有効にする。
    - `source /snmposter/bin/activate`
- 続いて以下のコマンドを実行して snmposter を実行する
    - `snmposter -f /snmposter/snmp_files/agents.csv`

上記の実行完了後、`/snmposter/snmp_files/agents.csv` に記述した IP アドレス宛てに snmpwalk を実行して、以下のように応答があれば正常に稼働している。  
  
```
amegon@test:~ $ snmpwalk -v 2c -c public 192.168.1.251 .1.3
iso.3.6.1.2.1.1.1.0 = STRING: "Sample description"
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1
iso.3.6.1.2.1.1.3.0 = Timeticks: (68576537) 7 days, 22:29:25.37
iso.3.6.1.2.1.1.4.0 = STRING: "Sample Contact"
iso.3.6.1.2.1.1.5.0 = STRING: "Sample Name"
iso.3.6.1.2.1.1.6.0 = STRING: "Sample Location"
iso.3.6.1.2.1.1.7.0 = INTEGER: 1
iso.3.6.1.2.1.1.8.0 = Timeticks: (107) 0:00:01.07
End of MIB
amegon@test:~ $
```
  
その他、snmposter の使用方法については本家の情報等を参照のこと。  

## 動作環境の削除方法

### 仮想環境の削除

以下の手順で削除  

- snmposter を停止する  
    - OS を再起動できる場合
        - OS を再起動する
    - OS が再起動できない場合：でもネットワークは瞬断するので注意
        - ps コマンドで snmposter の PID を調べ kill する
        - `systemctl restart network` コマンドでネットワークを再起動する
- 以下のコマンドで仮想環境を無効にする
    - `deactivate`
- 以下のディレクトリを配下を含めて削除する
    - /snmposter

### モジュールの削除

インストールした各種モジュールが不要であれば手動で削除する。  
インストールしたモジュールは `Playbook の実行内容` を参照。
