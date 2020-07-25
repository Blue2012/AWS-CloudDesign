## CDNとS3とAPIGatewayを組み合わせて、会員制のメール登録フォーム作成

下のページを参考にして作成してみた。
https://hacknote.jp/archives/40978/

上記、ページの案内に沿って、作ってみる。

## １．メール送信用のLambda関数作成
### １－１．テンプレートの選択

以下のような感じで、まずはカスタムロールを用意する
![image](https://user-images.githubusercontent.com/18514297/88448938-e1a2d400-ce7d-11ea-917b-7c916c46b026.png)

![image](https://user-images.githubusercontent.com/18514297/88448956-026b2980-ce7e-11ea-8067-d56fa14416a5.png)

テンプレートから作成を選択し、ロールテンプレートに「SNS 発行ポリシー」を選択。
出来上がりは↓のような感じとなる。

![image](https://user-images.githubusercontent.com/18514297/88449146-8ffb4900-ce7f-11ea-977f-8ea16426abd8.png)

### １－２．トリガーの追加

トリガーにCloudwatchEventsを選択。

![image](https://user-images.githubusercontent.com/18514297/88449210-fd0ede80-ce7f-11ea-8468-ffa3214d831a.png)

新規ルールの作成で、5分間隔で実行するルール式を作成します。
![image](https://user-images.githubusercontent.com/18514297/88449232-27609c00-ce80-11ea-88dd-68c0420f8908.png)

ルール自体はcron式で記載されているとのこと。
記載式は以下の通り。

```cron(*/5 * ? * * *)```

![image](https://user-images.githubusercontent.com/18514297/88449278-8c1bf680-ce80-11ea-91e6-360a47389c4a.png)

### １－３．SNS送信連携用Lambda関数作成

上記までで、5分間おきにLambda関数が実行できる設定を有効化できたため、次はSNSで通知を行う仕組みを実装します。
まずはSNSでこの仕組みのために専用のSNSトピックを用意します。

![image](https://user-images.githubusercontent.com/18514297/88449375-6511f480-ce81-11ea-9f0f-b7e772ab3601.png)

では、トピックの作成を選択して、新規トピックを作成します。

![image](https://user-images.githubusercontent.com/18514297/88449383-7a871e80-ce81-11ea-8f9b-1748f31c8751.png)

今回の実装はシンプルに行うため、特に設定値のカスタマイズは実施しません。
ただし、配信ステータスのログ記録だけは処理がどうなったかのかを後から追えるように有効にしておきましょう。

![image](https://user-images.githubusercontent.com/18514297/88449400-a73b3600-ce81-11ea-9ccd-06b7ca773e24.png)
![image](https://user-images.githubusercontent.com/18514297/88449430-d5b91100-ce81-11ea-83ac-646046267925.png)
![image](https://user-images.githubusercontent.com/18514297/88449446-ecf7fe80-ce81-11ea-8ac4-423200d703ed.png)

私が利用している環境では既存のロールが存在しなかったため、新規作成という形にしました。
ロールが存在しない場合は、新規作成を選択することで、IAMの画面へガイドしてくれて、その先で作成することが可能です。

![image](https://user-images.githubusercontent.com/18514297/88449513-70195480-ce82-11ea-9849-7add359fe593.png)
![image](https://user-images.githubusercontent.com/18514297/88449493-4a8c4b00-ce82-11ea-8d8c-885c059e0b02.png)

タグは特にいじらず、そのまま、作成します。
![image](https://user-images.githubusercontent.com/18514297/88449520-8f17e680-ce82-11ea-93e2-01f5a01ae456.png)

無事にトピックの作成に成功しました。
![image](https://user-images.githubusercontent.com/18514297/88449538-ab1b8800-ce82-11ea-8b75-54b043ef70ae.png)

この作成したトピックを利用して、Lambdaからメール送信を行いますので、トピックのARNをメモっておきます。
そして、このARNをメール送信用Lambda関数の中にリテラルとして、組み込みます。

出来上がった関数のコードだけ貼っておきます。（ARNは隠し）

```
import boto3

sns = boto3.client('sns')
TOPIC_ARN = u'arn:aws:sns:ap-northeast-1:483799857078:MailSendTopic'
msg = u'message test hacknote'
subject = u'title test hacknote'

def lambda_handler(event, context):
    request = {
    'TopicArn': TOPIC_ARN,
    'Message': msg,
    'Subject': subject
    }
    response = sns.publish(**request)
    return 'Successfully mail processed records'
```

でテスト用のイベントについては、今回はAWSイベントであれば、何でもよいため、
ひとまずは凝った作りをせずに↓のような内容で作成しておきます。

![image](https://user-images.githubusercontent.com/18514297/88449747-52e58580-ce84-11ea-8c5a-11fa99cd1203.png)

このテストを実行してみます。
たしかに、以下の通り、成功しました。

![image](https://user-images.githubusercontent.com/18514297/88449874-870d7600-ce85-11ea-8b48-91fe45600378.png)

いちおう、番外編となりますが、トピックを作成しても、それに紐づく、サブスクリプションが無いと
通知先が存在しない状態となりますので、そちらも用意する方法を記載しておこうと思います。
※ただし、5分間隔で通知が来てはたまったもんではないので、やり方を記載しておくまでに留めておきます。

![image](https://user-images.githubusercontent.com/18514297/88449960-203c8c80-ce86-11ea-8055-17229cfad312.png)
![image](https://user-images.githubusercontent.com/18514297/88450016-8f19e580-ce86-11ea-83e9-99b53afafe86.png)

上記を設定すると指定したアドレス宛てにメール送信が行われるようになります。

## ２．APIGatewayの用意
### ２－１．API発行

今回はRestAPIを作成していきます。
いまは、APIと一口でいっても色々なタイプのものが出てきていますからね。

![image](https://user-images.githubusercontent.com/18514297/88450072-3139cd80-ce87-11ea-975a-e4cffc178eae.png)

用意するものについては特にカスタマイズしないので、名前だけ変えます。
今回、CloudFrontと連携を行うものの、ここでは直接、CloudFrontとの連携を行う訳ではないため、リージョンを選択します。

![image](https://user-images.githubusercontent.com/18514297/88450086-5af2f480-ce87-11ea-88ad-8569620056be.png)
