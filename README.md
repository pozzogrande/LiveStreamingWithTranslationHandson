# 同時通訳付き映像配信システムを作ってみよう
このハンズオンではAWSから公開している下記のサンプルを元に実際の構築を行います。

[https://github.com/awslabs/live-streaming-with-automated-multi-language-subtitling](https://github.com/awslabs/live-streaming-with-automated-multi-language-subtitling)

所用時間 : 60 分程度

## アーキテクチャ

![Architecture](images/live-streaming-with-automated-multi-language-subtitling-architecture.png "アーキテクチャ")

### 利用するAWSのサービス
- AWS MediaLive
- AWS MediaPackage
- Amazon CloudFront
- Amazon CloudWatch Events
- Amazon Simple Notification Service (Amazon SNS)
- Amazon Simple Storage Service (Amazon S3)
- Amazon Transcribe
- Amazon Elastic Container Service (Amazon ECS)
- Amazon Translate
- AWS Lambda

## 手順概要と補足

### 1. 配信環境作成

1. マネジメントコンソールを開き、**AWS services** - **All services** 内の **CloudFormation** のリンク、もしくは検索窓で _CloudFormation_ と入力し、表示されたリンクをクリック

2. **Create stack** のプルダウンから **With new resources (standard)** をクリック

3. **Create stack** にて **Prerequisite - Prepare template** の **Template is ready** を選択し、 **Specify template** で **Upload a template file** を選択

4. **Choose file** のボタンをクリックし、_live-streaming-with-automated-multi-language-subtitling.yaml_ のファイルを選択する

5. **Next** をクリック。

6. **Specify stack details** にて下記の値の項目のみ設定変更し、 **Next** をクリック。

| 設定項目 | 設定値 |
| --- | --- |
| Stack name | _任意の名前 (アルファベット大小文字, 数字, "-")_ |
| Source Input Type | RTMP_PUSH |
| Input Security Group CIDR Block (REQUIRED) | 0.0.0.0/0 |

7. **Configure stack options** にて何も設定せずに **Next** をクリック

8. **Review _Stack name_** にて **Capabilities** の **I acknowledge that AWS CloudFormation might create IAM resources with custom names.** にチェックを入れて **Create stack** をクリック

![Capabilities](images/CFnTemplateReview.jfif)

9. CloudFormationのスタック画面にて進行状況を確認する

![StackProgress](images/stack-progress.jfif)

![refresh](images/refresh.jfif) アイコンを任意のタイミングでクリックすると進行状況を手動で更新できる

> **_NOTE_** : 正常に完了するまで10 ～ 15分ほど要します

10. 完了すると **CREATE_COMPLETE** にステータスが変更する

![CFnStackComplete](images/CFnStackComplete.jfif)

**Outputs** タブをクリックし、**CloudFrontHlsPlaybackUrl** と **MediaLivePrimaryEndpoint** の値をメモする

![CFnOutputs](images/CFNStackOutputs.jfif)

### 2. 配信側設定

> _**NOTE :**_ 事前にお配りした配信設定テストの資料をご確認ください

### 3. MediaLive への追加設定

AWSのロゴをクリックし、マネジメントコンソールのトップ画面に戻る

検索窓にて _MediaLive_ と入力し、表示された MediaLive のリンクをクリック

![MediaLiveLaunch](images/MediaLiveLaunch.png)

**Channels** -> **_'YOUR CLOUDFORMATION STACK NAME'_-livestream** のリンクをクリック、**Stop** をクリック

![MediaLiveChangingTranslatedLanguages](images/MediaLiveChangingTranslatedLanguages.jfif)

2 分ほど待つと **Channel state** が **Idle** に変わるので、**Channel Actions** プルダウンから **Edit channnel** をクリック

![EditChannel](images/EditChannel.png)

**Channel** - **Output groups** の **1.Live (HLS)** をクリック

**HLS outputs** で **Add output** をクリック

**Outpput 7** の **Name modifier** に _**\_ja**_ と入力し、**Setting** をクリック。

下記のとおり設定する

| 設定項目 | 設定値 |
| --- | --- |
| Output name | japanese |

**Stream settings** で **Remove video** と **Remove audio 1** をそれぞれクリックする

**Add caotion** プルダウンから **Create a new caption encode** をクリックする

下記のとおり設定する

| 設定項目 | 設定値 |
| --- | --- |
| Caption Selector Name | empty |
| Caption Settings | WebVTT destination |
| Language Code | ja |
| Kanguage Description | Generated Japanese |

**Update channel** をクリック

### 4. Lambda への追加設定

AWSのロゴをクリックし、マネジメントコンソールのトップ画面に戻る

検索窓にて _Lambda_ と入力し、表示された AWS Lambda 

**Functions** を選択し、検索窓にて **SNSTriggerAWSTranslateLambda** を検索して表示されたリンクをクリック

**Function name** が **_'YOUR CLOUDFORMATION STACK NAME'_-SNSTriggerAWSTranslateLambda** のリンクをクリック

**Configuration** タブを選択し、左のペインから **Environment variables** を選択

**Edit** をクリックし、**CAPTION_LANGUAGES** に _**ja**_ を追加して **Save** をクリックする

### 5. チャンネルの起動

MediaLive の画面に戻り、当該のチャンネルを選択して **Start** をクリック

**Channel state** が **Running** に変わることを確認

### 6. 配信の確認

https://hls-js.netlify.app/demo/ へアクセス

1でメモしたCloudFrontHlsPlaybackUrlを入力し、**Enter** を入力

おおよそ20秒程度の遅延で表示される