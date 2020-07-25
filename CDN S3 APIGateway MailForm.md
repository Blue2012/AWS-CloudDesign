## CDNとS3とAPIGatewayを組み合わせて、会員制のメール登録フォーム作成

下のページを参考にして作成してみた。
https://hacknote.jp/archives/40978/

上記、ページの案内に沿って、作ってみる。

## ・メール送信用Lambda関数を用意する

以下のような感じで、まずはカスタムロールを用意する
![image](https://user-images.githubusercontent.com/18514297/88448938-e1a2d400-ce7d-11ea-917b-7c916c46b026.png)

![image](https://user-images.githubusercontent.com/18514297/88448956-026b2980-ce7e-11ea-8067-d56fa14416a5.png)
