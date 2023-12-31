---
title: Google Cloud Error Reportingでエラーと上手く付き合う
date: '2022-11-01'
---

Error Reporting が最近色々進化していて、通知から記録まで隙のないサービスになっていたので全体像をまとめてみます。

## Error Reporting へエラーを送信する

GCP のインフラを利用している場合は自動で拾ってくれます。
ソレ以外の場合もいろいろライブラリがあるので調べてみてください。

## エラーのステータスを管理する

Error Reporting では、エラーのステータスを管理できます。

- **未解決**
- **確認済み**
- **解決済み**

で、確認済みにする場合は、**Github の Issue などをリンクする**ことが出来ます。

## Slack やメールでエラーを通知する

Error Reporting ではデフォルトで Slack やメールでの通知に対応しています。
通知の条件は、

- **新しいエラーが**発生した場合
- 解決済みのエラーが**再発した**場合
- **5 回以上**通知されたら、**6 時間はエラーが通知されなくなる**

なので、エラーのステータスをきちんと管理すれば、

- **対応が必要な（未知の/再発した）エラーのみ**を通知する
- 対応が必要でない（対応中の/避けられない）エラーの通知を**除外する**
- **通知が頻繁に来続けることを避ける**

が実現できます。

また、**Cloud Logging**（自動でエラーの分のログが記録されています）と**Cloud Monitoring**と組み合わせれば、

- **普段は対応が必要なさそうなエラーが頻発した場合にインシデントとして通知する**

も実現できます。
**例えば ポリシー "Too meny timeout" など、普段はタイムアウトは仕方ないので無視したいけど、めっちゃ多いと心配になる、という場合にも適切な通知を設定できると思います。**

## エラーの分析をする

Error Reporting ではトレースを見ることが出来ます。ここでいろいろ分析できますね。
ただし、リクエストの状態は見られれないので、そこは別の方法を検討する必要があります。

## エラーを記録する

Error Reporting の情報は同時に Cloud Logging に溜まっています。これを Cloud Storage にシンクすることで、永続的にエラーの情報を保存することが出来ます。
