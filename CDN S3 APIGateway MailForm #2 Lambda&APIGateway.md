# 会員制のメール登録フォーム作成 (CloudFront部分)
## S3バケット作成

バケット名は任意で適当に作成します。

![image](https://user-images.githubusercontent.com/18514297/88680631-adecd600-d12b-11ea-8d6f-f255b7d61c42.png)

![image](https://user-images.githubusercontent.com/18514297/88680792-d7a5fd00-d12b-11ea-8359-0e9e10ac42e7.png)

![image](https://user-images.githubusercontent.com/18514297/88680954-045a1480-d12c-11ea-9583-445891b213ab.png)

![image](https://user-images.githubusercontent.com/18514297/88681025-1936a800-d12c-11ea-8ada-49098a83b544.png)

![image](https://user-images.githubusercontent.com/18514297/88681259-5864f900-d12c-11ea-8faf-4e33032c5e31.png)

バケットポリシーに以下の値を入力せよとのこと。ひとまずは指示に従いましょうかね。


```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "ARNの値に更新/*"
        },
        {
            "Sid": "PublicReadListBucket",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:ListBucket",
            "Resource": "ARNの値に更新"
        }
    ]
}
```

![image](https://user-images.githubusercontent.com/18514297/88681577-ae39a100-d12c-11ea-859d-9afd3b0c37ec.png)

ところがどっこい、ポリシー変更しようとしたら、エラーになった。
パブリックアクセスを許可していないといけないようだ。

![image](https://user-images.githubusercontent.com/18514297/88682033-2bfdac80-d12d-11ea-802b-4e99ae0201fa.png)

![image](https://user-images.githubusercontent.com/18514297/88682095-3e77e600-d12d-11ea-8b57-f4d74c7992d8.png)

![image](https://user-images.githubusercontent.com/18514297/88682290-7aab4680-d12d-11ea-902d-347bc83c4f4f.png)

さぁ、これでどうなるかな？

![image](https://user-images.githubusercontent.com/18514297/88682431-a0d0e680-d12d-11ea-8e52-b52994217044.png)

おっと、結果が変わって今度は正常に登録が完了しました。

![image](https://user-images.githubusercontent.com/18514297/88682537-ba722e00-d12d-11ea-8b3f-26778a031896.png)


