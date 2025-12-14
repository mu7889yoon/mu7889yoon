+++
date = '2025-12-13T00:00:00+09:00'
draft = false
tags = ['tech', 'tips', 'aws', 'step-functions']
title = 'Step Functionsはタスク毎にIAMのセッションが変わる(GenerateServiceLastAccessedDetails → GetServiceLastAccessedDetailsでお困りの方必見)'
+++

よ〜んです。

> 2022年にほぼ同じ内容の記事が公開されておりましたが、2025年も変わらないよ。と言う意味で記事化させていただきます。
> [AWS Step Functions ステートマシンで実行されるタスクはそれぞれセッションプリンシパルが異なる](https://dev.classmethod.jp/articles/aws-step-functions-statemachine-task-assumerole/)

AWSのサービス一覧を取得する方法として、以下のような手法があります。

```bash
JOB_ID=$(aws iam generate-service-last-accessed-details --arn arn:aws:iam::aws:policy/AdministratorAccess --output text)
aws iam get-service-last-accessed-details --job-id $JOB_ID

// output
{
    "JobStatus": "COMPLETED",
    "JobType": "SERVICE_LEVEL",
    "JobCreationDate": "2025-12-13T00:00:00.000000+09:00",
    "ServicesLastAccessed": [
        {
            "ServiceName": "AWS App2Container",
            "ServiceNamespace": "a2c",
            "TotalAuthenticatedEntities": 0
        },
        {
            "ServiceName": "Alexa for Business",
            "ServiceNamespace": "a4b",
            "TotalAuthenticatedEntities": 0
        },
```

これらはAWS APIを組み合わせたものですので、Step Functionsで実行可能ではないか？と思い、以下のようなState Machineを組んでみたところ、意図した動作をせず、ハマりまくったため、調査したことを記録します。

## 調査
### エラー内容

### Step Functionsの実行ログを見てみる

### CloudShellで生成した JobId を使ってStep FunctionsでGetService　してみる

### Step Functionsで生成した JobId を使ってCloudShellでGetServiceしてみる

### API Documentを読む

### CloudTrailを見る


## まとめ

Step Functionsの仕様を知りつつ、`GenerateServiceLastAccessedDetails`のハマりどころと言うような内容でした。

## 追記

ほぼ同じ内容の記事がありました。ググり不足ですね...

[AWS Step Functions ステートマシンで実行されるタスクはそれぞれセッションプリンシパルが異なる](https://dev.classmethod.jp/articles/aws-step-functions-statemachine-task-assumerole/)