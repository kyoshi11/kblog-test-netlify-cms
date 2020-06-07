---
templateKey: 'blog-post'
title: 'Goole Home-Ifttt-Irkitの連携復活'
date: 2020-05-17T18:28:10.000Z
featuredpost: false
featuredimage: img/cover.jpg
description: >-
  jamstackでブログ構築
tags:
  - ict
  - googlehome
  - ifttt
  - irkit
---

![Alt Text](img/2020/05-17-post/ksblog.png)

### Google Homeでの家電コントロールが復活

以前、Iftttに登録した設定が無効になってしまったものとずっと思っていたが、
そうではなかった様です（素直に嬉しい）

どうも、スマホのGoogle Homeの登録アカウントをいつぞのタイミングで変えてしまったらしく、その影響でデバイスコントロールができなくなってしまったようでした。

ただ、単純にIfttt側でリンクしていたGmailアカウントを変更しただけではダメでした。

以下のように、個々に登録したApplet(Iftttではそう呼ぶ＝AlexaだとSkill的な物)
で設定変更する
1.対象のAppletで「Settings」ボタンをクリック
2.「Check Now」ボタンをクリック
3.「Get notifications when this connection is active」ボタンスイッチをクリック
4.「Save」ボタンをクリック


## irkit
### IRkitのIPを調べる

$ dns-sd -B _irkit._tcp
Browsing for _irkit._tcp
DATE: ---Sun 17 May 2020---
12:14:44.005  ...STARTING...
Timestamp     A/R    Flags  if Domain               Service Type         Instance Name
12:14:44.313  Add        2   5 local.               _irkit._tcp.         iRKitA17B
$ dns-sd -G v4 iRKitA17B.local
DATE: ---Sun 17 May 2020---
12:16:20.916  ...STARTING...
Timestamp     A/R    Flags if Hostname                               Address                                      TTL
12:16:21.125  Add        2  5 irkita17b.local.                       192.168.35.107                               10

#### clientトークンを取得
$ curl -i "http://192.168.35.107/keys" -d '' -H "X-Requested-With: curl"
{"clienttoken":"7D30EE58C1B94EE2AEDB22530F92BDC4"}

#### デバイス🆔及びclientキーを取得
$ curl -i -d "clienttoken=7D30EE58C1B94EE2AEDB22530F92BDC4" "https://api.getirkit.com/1/keys"
{"deviceid":"A9AD7648D8194B06B1277F39791C62D8","clientkey":"1A1C1E925740421C85C8E8B5D32AF61B"}

#### リモコンボタンの押下待ち状態にする
$ curl -i "https://api.getirkit.com/1/messages?clientkey=1A1C1E925740421C85C8E8B5D32AF61B&clear=1"
HTTP/1.1 200 OK
Server: openresty
Date: Sun, 17 May 2020 08:13:40 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 634
Connection: keep-alive
Access-Control-Allow-Origin: *
Access-Control-Allow-Headers: X-Requested-With
ETag: "-1592554034"
X-Content-Type-Options: nosniff

{"message":{"format":"raw","freq":38,"data":[4713,1150,2211,1319,1190,1190,2211,1190,1190,1190,2211,1190,1190,1190,1190,1190,2211,1319,1002,1319,1002,1319,1002,1319,1111,50610,4713,1275,2211,1275,1037,1319,2288,1232,1037,1366,2288,1190,1037,1319,1037,1319,2288,1319,1150,1150,1002,1319,1111,1319,1111,50610,4713,1275,2211,1275,1037,1319,2211,1319,1037,1319,2288,1319,1002,1366,1111,1232,2288,1150,1150,1150,1037,1319,1037,1319,1037,50610,4554,1319,2288,1319,1111,1111,2211,1319,1111,1319,2288,1232,1037,1275,1037,1275,2288,1232,1232,1232,1037,1232,1232,1232,1232]},"hostname":"IRKitA17B","deviceid":"A9AD7648D8194B06B1277F39791C62D8"}

#### リモコンボタンの送信確認する
curl -i ‘https://api.getirkit.com/1/messages?clientkey=1A1C1E925740421C85C8E8B5D32AF61B&deviceid=A9AD7648D8194B06B1277F39791C62D8&message={"format":"raw","freq":38,"data":[4713,1150,2211,1319,1190,1190,2211,1190,1190,1190,2211,1190,1190,1190,1190,1190,2211,1319,1002,1319,1002,1319,1002,1319,1111,50610,4713,1275,2211,1275,1037,1319,2288,1232,1037,1366,2288,1190,1037,1319,1037,1319,2288,1319,1150,1150,1002,1319,1111,1319,1111,50610,4713,1275,2211,1275,1037,1319,2211,1319,1037,1319,2288,1319,1002,1366,1111,1232,2288,1150,1150,1150,1037,1319,1037,1319,1037,50610,4554,1319,2288,1319,1111,1111,2211,1319,1111,1319,2288,1232,1037,1275,1037,1275,2288,1232,1232,1232,1037,1232,1232,1232,1232]}’

※この時に以下のようなエラーとなるが特に無視して構わない
curl: (3) nested brace in URL position 163:

#### メッセージの返答部分をIftttのアクションに貼り付ける
Iftttの選択
＜Google Assistant＞
Say a simple phrase を選択

What do you want to say?
電気消して

What's another way to say it? (optional)

And another way? (optional)

What do you want the Assistant to say in response?
ライトを消します

Language
日本語

URL
https://api.getirkit.com/1/messages

Method
POST

Content Type (optional)
application/x-www-form-urlencoded

■Body (optional)
clientkey=1A1C1E925740421C85C8E8B5D32AF61B&deviceid=A9AD7648D8194B06B1277F39791C62D8&message={"format":"raw","freq":38,"data":[4713,1150,2211,1319,1190,1190,2211,1190,1190,1190,2211,1190,1190,1190,1190,1190,2211,1319,1002,1319,1002,1319,1002,1319,1111,50610,4713,1275,2211,1275,1037,1319,2288,1232,1037,1366,2288,1190,1037,1319,1037,1319,2288,1319,1150,1150,1002,1319,1111,1319,1111,50610,4713,1275,2211,1275,1037,1319,2211,1319,1037,1319,2288,1319,1002,1366,1111,1232,2288,1150,1150,1150,1037,1319,1037,1319,1037,50610,4554,1319,2288,1319,1111,1111,2211,1319,1111,1319,2288,1232,1037,1275,1037,1275,2288,1232,1232,1232,1037,1232,1232,1232,1232]}


#### 複数トリガーの実行
いろいろなやり方があるようだが、少し無骨なやり方だが、複数のトリガーを同じ内容にし、それぞれに別のアクションを定義するのが良さそう。

A
行ってきます
行ってらっしゃい
→　電気を消す

B
行ってきます
行ってらっしゃい
→　エアコンを止める

C
行ってきます
行ってらっしゃい
→　TVを消す


「行ってきます」と話すと、上記ABCの処理が順次行われる