+++
date = '2025-12-14T11:21:46+09:00'
draft = false
tags = ['tech', 'tips', 'aws', 'cloudformation']
title = 'CloudformationのLaunch Stackボタンが便利'
+++

よ〜んです。

ハンズオンとかやる時に、ローカルにコードを落としてもらうほどじゃないけど、仕方なくローカルに落としてもらってるってことありますよね。

| Region | 1-Click URL |
| :-: | :-: |
| ap-northeast-3(大阪) | [![Cloudformation Launch Stack button](/images/launch-stack.svg)](https://ap-northeast-3.console.aws.amazon.com/cloudformation/home?region=ap-northeast-3#/stacks/quickcreate?stackName=SampleStack&templateURL=https://publicbucketa6745c15-n3bqc2binchm.s3.ap-northeast-1.amazonaws.com/LambdaSample.json) |
| ap-northeast-1(東京) | [![Cloudformation Launch Stack button](/images/launch-stack.svg)](https://ap-northeast-1.console.aws.amazon.com/cloudformation/home?region=ap-northeast-1#/stacks/quickcreate?stackName=SampleStack&templateURL=https://publicbucketa6745c15-n3bqc2binchm.s3.ap-northeast-1.amazonaws.com/LambdaSample.json) |
| us-east-1(バージニア北部) | [![Cloudformation Launch Stack button](/images/launch-stack.svg)](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?stackName=SampleStack&templateURL=https://publicbucketa6745c15-n3bqc2binchm.s3.ap-northeast-1.amazonaws.com/LambdaSample.json) |



```shell
AWS_REGION=
STACK_NAME=
TEMPLATE_S3_URL=
echo "https://${AWS_REGION}.console.aws.amazon.com/cloudformation/home?region=${AWS_REGION}#/stacks/quickcreate?stackName=${STACK_NAME}&templateURL=${TEMPLATE_S3_URL}"
```
