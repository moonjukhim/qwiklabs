![](https://cdn.qwiklabs.com/l0sFLZS%2BO9dGHcecgA2MrRf7u2BRusBkSVPzUDf8AUg%3D)
# Engineer Data in Google Cloud: Challenge Lab

**Overview**

할당 된 시간 내에 일련의 작업을 완료해야 합니다. 단계별 지침을 따르는 대신 시나리오와 일련의 작업이 제공됩니다. 직접 완료하는 방법을 알아내야 합니다!

Challenge Lab을 수강하더라도 BigQuery 또는 데이터 엔지니어링 개념을 배우지 않습니다. 제시된 과제에 대한 솔루션을 구축하려면 과제 랩이 속한 퀘스트에서 배운 기술을 사용해야 합니다. 

이 실습은 데이터 엔지니어링 퀘스트의 실습을 완료한 학생에게만 권장됩니다.

**Objectives**

Topics tested:

- 기존의 데이터로부터 BigQuery 테이블 생성
- BigQuery, Dataprep 혹은 Dataflow를 사용하여 ML Model 생성을 위한 데이터 정제
- BQML에서 모델 생성 및 최적화
- BQML에서 예측된 데이터 생성

---

## 도전 과제 시나리오

책상에 앉자마자 첫 번째 과제를 받게 됩니다. 요금을 예측하기 위한 BQML을 생성하려고 합니다. 데이터를 가져오고, 정제한 다음 새 데이터로 모델을 생성하고 배치 예측을 수행하여 모델 성능을 검토하고 애플리케이션 배포에 대한 진행 여부 결정을 내릴 수 있도록 합니다.

## Task 1: 훈련 데이터의 정제

랩이 시작될 때 데이터 집합을 만들고 데이터를 `taxirides`와 `historical_taxi_rides_raw` 테이블에 가져왔습니다. 

이 작업을 마무리 하기 위해 다음을 수행합니다.

- `historical_taxi_rides_raw` 데이터를 정제하고 `taxi_training_data`를 같은 데이터셋에 생성합니다. BigQuery, DataPrep, DataFlow등을 사용하여 데이터를 정제할 수 있습니다. 학습하고 하는 대상 속성은 `fare_amount`입니다.

힌트

- BQ UI에서 소스 데이터 집합을 볼 수 있으며, 먼저 서스 스키마를 숙지합니다.
- 예측 시 확인할 수 있는 데이터에 대한 힌트로서, 예측 시간에 도착한다는 것을 보여주는 `taxirides.report_prediction_data` 데이터를 숙지합니다.

데이터 정제 작업:

- `trip_distance`가 0보다 커야 합니다.
- `fare_amount` 금액이 작은 데이터는 제거합니다($2.5 미만).
- 위도와 경도가 사용 사례에 적합한지 확인합니다.
- `passenger_count`가 0보다 커야 합니다.
- `total_amount`에는 팁이 포함되므로 `tolls_amount` 및 `fare_amount`를 대상 변수로 추가합니다.
- 소스 데이터 집합이 크기 때문에(>1억행), 데이터 집합을 100만건 미만으로 샘플링합니다.
- 모델에서 사용할 필드만 복사합니다(`report_prediction_data`는 좋은 예제입니다).


## Start Lab

1. [1]At the top of your screen, launch your lab by clicking <span style="background-color:#34A853; font-family:Google Sans; font-weight:bold; font-size:90%; color:white; border-color:#34A853; border-radius:4px; border-width:2px; border-style:solid; outline-color:#ffffff; padding-top:5px; padding-bottom:5px; padding-left:10px; padding-right:10px">Start Lab</span>

This will start the process of provisioning your lab resources. An estimated amount of time to provision your lab resources will be displayed. You must wait for your resources to be provisioned before continuing.

<i class="fas fa-info-circle"></i> If you are prompted for a token, use the one distributed to you (or credits you have purchased). 

2. [2]Open your lab by clicking <span style="background-color:white; font-family:Google Sans; font-weight:bold; font-size:90%; color:#1a73e8; border-color:#dadce0; border-radius:4px; border-width:2px; border-style:solid; outline-color:#ffffff; padding-top:5px; padding-bottom:5px; padding-left:10px; padding-right:10px">Open Console</span>
This will open an AWS Management Console sign-in page.

3. [3]On the Sign-in page, configure:

* **IAM user name:** `awsstudent`
* **Password:** Paste the value of **Password** located to the left of these instructions.
* Click <span style="background-color:#257ACF; font-weight:bold; font-size:90%; color:white; position:relative; top:-1px; border-radius:5px; padding-top:3px; padding-bottom:3px; padding-left:10px; padding-right:10px;white-space: nowrap">Sign In</span>

<i class="fas fa-exclamation-triangle"></i> **Please do not change the Region unless instructed**.

---------------------------------------

### Common login errors

**Error: You must first log out**

![](https://s3-us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/sts-sign-in-images/media/logouterror.png)

If you see the message, **You must first log out before logging into a different AWS account:** 

   * Click **click here**
   * Close your browser tab to return to your initial Qwiklabs window
   * Click <span style="background-color:white; font-family:Google Sans; font-weight:bold; font-size:90%; color:#1a73e8; border-color:#dadce0; border-radius:4px; border-width:2px; border-style:solid; outline-color:#ffffff; padding-top:5px; padding-bottom:5px; padding-left:10px; padding-right:10px">Open Console</span> again

---------------------------------------

## Task 1: Review your Amazon S3 access policies

4. [4]In the AWS Management Console, click **Services**, and then click **S3**.

Several Amazon S3 buckets were created for you. Two buckets are listed as public with the names **privatebucket** and **publicbucket**. There are other buckets, but these are the two you will work with.

In the **Access** column on this screen, notice that both buckets are open to the public. As its name implies, the **publicbucket** is supposed to be publicly accessible, while the **privatebucket** is not supposed to have public access. A custom tag, **CanBePublic**, indicates if the bucket should or should not be publicly accessible. Follow the next steps to inspect the bucket tags to understand how they are set up.

5. [5]Click the bucket with **privatebucket** in its name.

6. [6]Click the **Properties** tab.

7. [7]Click the **Tags** card.

There are several tags applied to the bucket. Notice the **CanBePublic** tag has a value of _0_. This indicates that the bucket should **not** be public. Repeat the same steps for the public bucket to see how it is configured.

In this lab, you will build a solution that can correct violations and create notifications whenever a violation occurs. To do this, you will set up AWS Config rules to trigger a CloudWatch Events rule that triggers a Lambda function to automatically remove public access from the bucket that should not be public.

## Task 2: Connect to your Amazon Linux EC2 instance

In this section, you will sign in to an Amazon Elastic Compute Cloud (Amazon EC2) work instance so that you can use the AWS Command Line Interface (AWS CLI).

8. [8]Copy the **CommandHostSessionManagementUrl** value from the left side of the lab page, and paste it in a new browser tab. The command host terminal opens.

<i class="fas fa-exclamation-triangle" style="color:orange"></i> If you encounter issues connecting to Session Manager, <a href="#ssh-instructions">click here</a> for help connecting to an Amazon EC2 instance using an SSH client.

<a id='ssh-after'></a>

9. [9]To change your working directory to **home**, run the following command:

```bash
cd
```

<i class="fas fa-comment"></i> **Suggestion** To help differentiate executed commands from output in the AWS CLI, run the following command. This adds a blank line before any output to the screen:

```bash
trap 'printf "\n"' DEBUG
```

You can also alter your command prompt to make command output easier to read by exporting the PS1 variable. To do this, run the following command:

```bash
export PS1="\n[\u@\h \W] $ "
```

<a id='task3'></a>

## Task 3: Enable AWS Config to monitor your Amazon S3 buckets

First, you will create a configuration recorder, which is necessary to detect changes in your resource configurations and capture these changes as configuration items. In this lab, you are only monitoring Amazon S3 buckets, so you will scope down the resources that AWS Config monitors for changes.

10. [10]In your SSH session, run the following command. Replace `<ConfigRoleARN>` with the **ConfigRoleARN** value from the left side of the lab page:

```bash
aws configservice put-configuration-recorder \
--recording-group allSupported=false,includeGlobalResourceTypes=\
false,resourceTypes=AWS::S3::Bucket \
--configuration-recorder name=default,roleARN=<ConfigRoleARN>
```

<i class="fas fa-comment"></i> **Suggestion** This lab has several commands for you to copy, edit, and then paste into the AWS CLI. It is recommended that you copy such commands to a text editor, replace the necessary text, copy the updated text from the editor, and then paste the text into the AWS CLI. This is an easier way to insert values into a command rather than editing on the command line itself.

Next, you will create the delivery channel. AWS Config sends notifications and updated configuration states through the delivery channel. You can manage the delivery channel to control where AWS Config sends configuration updates. In this case, you will send notifications to Amazon Simple Notification Service (Amazon SNS), which sends emails to subscribed users when a bucket is out of compliance.

11. [11]Run the following command. Replace `<ConfigS3BucketName>` and `<ConfigSNSTopic>` with the appropriate values from the left side of the lab page.

```bash
aws configservice put-delivery-channel \
--delivery-channel configSnapshotDeliveryProperties=\
{deliveryFrequency=Twelve_Hours},name=default,\
s3BucketName=<ConfigS3BucketName>,\
snsTopicARN=<ConfigSNSTopic>
```

12. [12]To start the configuration recorder, run the following command:

```bash
aws configservice start-configuration-recorder --configuration-recorder-name default
```

## Task 4: Create and configure CloudWatch Events rules

AWS Config is now monitoring resources in this account and will record if a policy change has been applied. Now you want to ensure that those changes comply with company rules. When a change does not comply, you want to correct the policy to ensure compliance. AWS Config will send a message to CloudWatch Events when there is a resource that is not compliant with your rule.

To do this, you will create two JSON files that define the rules, which you will then import into AWS Config. One rule is to block public read access (*S3ProhibitPublicReadAccess*), and the other is to block public write access (*S3ProhibitPublicWriteAccess*).

13. [13]In your SSH session, to create the *S3ProhibitPublicReadAccess.json* file, run the following command:

```bash
cat <<EOF > S3ProhibitPublicReadAccess.json
{
  "ConfigRuleName": "S3PublicReadProhibited",
  "Description": "Checks that your S3 buckets do not allow public read access. If an S3 bucket policy or bucket ACL allows public read access, the bucket is noncompliant.",
  "Scope": {
    "ComplianceResourceTypes": [
      "AWS::S3::Bucket"
    ]
  },
  "Source": {
    "Owner": "AWS",
    "SourceIdentifier": "S3_BUCKET_PUBLIC_READ_PROHIBITED"
  }
}
EOF
```

14. [14]To create the *S3ProhibitPublicWriteAccess.json* file, run the following command:

```bash
cat <<EOF > S3ProhibitPublicWriteAccess.json
{
  "ConfigRuleName": "S3PublicWriteProhibited",
  "Description": "Checks that your S3 buckets do not allow public write access. If an S3 bucket policy or bucket ACL allows public write access, the bucket is noncompliant.",
  "Scope": {
    "ComplianceResourceTypes": [
      "AWS::S3::Bucket"
    ]
  },
  "Source": {
    "Owner": "AWS",
    "SourceIdentifier": "S3_BUCKET_PUBLIC_WRITE_PROHIBITED"
  }
}
EOF
```

15. [15]To add the two new rules to AWS Config, run the following commands:

```bash
aws configservice put-config-rule --config-rule file://S3ProhibitPublicReadAccess.json
```

```bash
aws configservice put-config-rule --config-rule file://S3ProhibitPublicWriteAccess.json
```

16. [16]In the AWS Management Console, click **Services**, and then click **Config**.

17. [17]In the left navigation pane, click **Rules** (toward the top of the list).

Notice the rules you just created: *S3PublicReadProhibited* and *S3PublicWriteProhibited*.

## Task 5: Create a Lambda function

Now that you have built the rules, you need to create a Lambda function to fix any issues and report problems to Amazon SNS.

18. [18]In your SSH session, run the following command:

```bash
cat lambda_function.py
```

The output is the Python code you will use to create your Lambda function. Take a moment to inspect the code to see what the Lambda function will do.

The Lambda function will scan all your buckets and look for the custom tag **CanBePublic**. If any bucket is assigned public access and does not have the tag **CanBePublic** set to **1**, the Lambda function will treat it as a policy violation. When this happens it will set the bucket ACL to private and send a message to SNS.

The next steps will publish the Lambda function based on this code.

19. [19]In the AWS Management Console, click **Services**, and then click **Lambda**.

You should not see the Lambda function called **RemoveS3PublicAccessDemo** since it has not been uploaded yet.

20. [20]In your SSH session, to create a .zip file from the Python code, run the following command:

```bash
zip lambda_function.zip lambda_function.py
```

The Lambda function needs to assume a role in your account to be able to describe the Amazon S3 buckets and look at the policies and tags that are applied. The lab environment includes a role with the correct permissions and trust policy for this function. For a copy of the security policy, see the <a href="#lambdapolicy">Appendix</a>.

21. [21]To create the Lambda function, run the following command. Replace `<LambdaRoleARN>` and `<ConfigSNSTopic>` with the appropriate values from the left side of the lab page:

```bash
aws lambda create-function --function-name RemoveS3PublicAccessDemo \
--runtime "python3.6" --handler lambda_function.lambda_handler \
--zip-file fileb://lambda_function.zip \
--environment Variables={TOPIC_ARN=<ConfigSNSTopic>} \
--role <LambdaRoleARN>
```

22. [22]Copy the **FunctionArn** value from the output and save it to use later.

23. [23]In the AWS Management Console, click **Services**, and then click **Lambda**.

You should see the Lambda function you just created called *RemoveS3PublicAccessDemo*.

The lab includes an Amazon SNS topic to notify subscribed users of any violations.

24. [24]In your SSH session, to subscribe to the notifications, run the following command. Replace `<ConfigSNSTopic>` with the appropriate value from the left side of the lab page, and replace `<your-email-address>` with an email address where you would like to receive notifications.

**Note** You can skip this step if you do not want to receive notifications. At the conclusion of the lab, the Amazon SNS topic will be deleted, and you will not receive emails from this lab in the future.

```bash
aws sns subscribe --topic-arn <ConfigSNSTopic> \
--protocol email --notification-endpoint <your-email-address>
```

## Task 6: Create a CloudWatch event

Now that you have created a Lambda function, you need to define how that function is going to be invoked. In this case, you can tie it to the managed rule you defined in the previous task.

25. [25]In your SSH session, to create a new file named *CloudWatchEventPattern.json*, run the following command:

```bash
cat <<EOF > CloudWatchEventPattern.json
{
  "source": [
    "aws.config"
  ],
  "detail": {
    "requestParameters": {
      "evaluations": {
        "complianceType": [
          "NON_COMPLIANT"
        ]
      }
    },
    "additionalEventData": {
      "managedRuleIdentifier": [
        "S3_BUCKET_PUBLIC_READ_PROHIBITED",
        "S3_BUCKET_PUBLIC_WRITE_PROHIBITED"
      ]
    }
  }
}
EOF
```

26. [26]To create the rule, run the following command:

```bash
aws events put-rule --name ConfigNonCompliantS3Event --event-pattern file://CloudWatchEventPattern.json
```

27. [27]Copy the **RuleArn** value from the output and save it to use later.

You should now have a Lambda function and a CloudWatch Events rule, but you need to connect them together.

28. [28]To add the Lambda function as a target for the rule, run the following command. Replace `<LambdaFunctionARN>` with the value you copied previously, and replace `<ConfigSNSTopic>` with the appropriate value from the left side of the lab page.

```bash
aws events put-targets --rule ConfigNonCompliantS3Event \
--targets "Id"="Target1","Arn"="<LambdaFunctionARN>" "Id"="Target2","Arn"="<ConfigSNSTopic>"
```

29. [29]To give the Lambda function permission to trigger from CloudWatch Events, run the following command. Replace `<rule-arn>` with the value you copied previously:

```bash
aws lambda add-permission --function-name RemoveS3PublicAccessDemo \
--statement-id my-scheduled-event --action 'lambda:InvokeFunction' \
--principal events.amazonaws.com --source-arn <rule-arn>
```

Now your CloudWatch Events rule can properly trigger your Lambda function.

30. [30]In the AWS Management Console, click **Services**, and then click **CloudWatch**.

31. [31]In the left navigation pane, under **Events**, click **Rules**. You should see your rule called *ConfigNonCompliantS3Event*.

32. [32]Click **ConfigNonCompliantS3Event**, and scroll down.

You should see the two targets you have defined: the Lambda function and an Amazon SNS topic. When a policy violation is detected, the Lambda function will fix the issue and push a message to Amazon SNS, which will send a notification to the subscribed users.

## Task 7: View the results

Now it is time to put everything together.

33. [33]In your SSH session, to start scanning for issues and fixing policy violations, run the following command:

```bash
aws configservice start-config-rules-evaluation \
--config-rule-names S3PublicReadProhibited S3PublicWriteProhibited
```

The Lambda function begins to review your Amazon S3 buckets and automatically fix the problems you saw at the beginning of this lab.

34. [34]In the AWS Management Console, click **Services**, and then click **S3**. In a few minutes, you should see that **privatebucket** is no longer publicly accessible.

## Lab complete

<i class="icon-flag-checkered"></i> Congratulations! You have completed the lab.

## End Lab

Follow these steps to close the console, end your lab, and evaluate the experience.

35. [35]Return to the AWS Management Console.

36. [36]On the navigation bar, click **awsstudent@&lt;AccountNumber&gt;**, and then click **Sign Out**.

37. [37]Click <span style="background-color:#D93025; font-family:Google Sans; font-weight:bold; font-size:90%; color:white; border-color:#D93025; border-radius:4px; border-width:2px; border-style:solid; outline-color:#ffffff; padding-top:5px; padding-bottom:5px; padding-left:10px; padding-right:10px">End Lab</span>

38. [38]Click <span style="background-color:#DEDEDE; font-family:Google Sans; font-weight:bold; font-size:90%; color:#444; position:relative; top:-1px; border-width:1px; border-style:solid; border-color:#444; padding-top:3px; padding-bottom:3px; padding-left:10px; padding-right:10px">OK</span>

39. [39]\(Optional\):

* Select the applicable number of stars <i class="far fa-star"></i>
* Type a comment
* Click **Submit**

 * 1 star = Very dissatisfied
 * 2 stars = Dissatisfied
 * 3 stars = Neutral
 * 4 stars = Satisfied
 * 5 stars = Very satisfied

You may close the dialog if you don't want to provide feedback.

## Additional Resources
For more information about AWS Training and Certification, see [*http://aws.amazon.com/training/*](http://aws.amazon.com/training/).

*Your feedback is welcome and appreciated.*
If you would like to share any feedback, suggestions, or corrections, please provide the details in our [*AWS Training and Certification Contact Form*](https://support.aws.amazon.com/#/contacts/aws-training).

## Appendix

<a id='ssh-instructions'></a>

Access to an Amazon EC2 Linux instance requires a secure connection using an SSH client. The following directions walk you through the process of connecting to your Amazon EC2 Linux instance. Choose one of the following guides:

- Connect to a Linux instance from <a href="#ssh-windows">Windows using PuTTY</a>
- Connect to a Linux instance from <a href="#ssh-macOS-Linux">macOS or Linux</a>

<a id='ssh-windows'></a>

### Connect to a Linux instance from Windows using PuTTY

**Note** Only perform the following steps if you are connecting from a Windows machine. If you are connecting from a macOS or Linux machine, <a href="#ssh-macOS-Linux">click here</a> for instructions.

40. [40]On the left side of the lab page, click <i class="fas fa-download"></i> **Download PPK**. Save the PPK file to the directory of your choice.

41. [41]Open PuTTY (from the **Start** menu, choose **PuTTY** > **PuTTY**).

**Note** If PuTTY is not already installed on your computer, download and install it from the following URL: [https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html). If you already have an older version of PuTTY installed, we recommend that you download the latest version.

42. [42]In the **Category** pane, choose **Session** and configure the following:

- For **Host Name**, enter `<user_name>@<public_dns_name>`, where `<public_dns_name>` is the **CommandHost** value on the left side of the lab page

    **Note** For Amazon Linux 2 or the Amazon Linux AMI, the user name is `ec2-user`. For an Ubuntu AMI, the user name is `ubuntu`.

- For **Connection type**, select **SSH**
- Ensure that the **Port** value is **22**

![PuTTY Connection Screen Example](https://ap-northeast-1-tcprod.s3.amazonaws.com/courses/ILT-TF-200-SISECO/v2.6.2/lab-3-config/instructions/en_us/images/putty-session-config.png)

43. [43]\(Optional\) You can configure PuTTY to automatically send 'keepalive' data at regular intervals to keep the session active. This is useful to avoid disconnecting from your instance due to session inactivity. In the **Category** pane, choose **Connection**, and then enter the required interval in the **Seconds between keepalives** field. For example, if your session disconnects after 10 minutes of inactivity, enter 180 to configure PuTTY to send keepalive data every 3 minutes.

44. [44]In the **Category** pane, expand **Connection**, expand **SSH**, and then choose **Auth**. Complete the following:

- Choose **Browse**.
- Select the .ppk file that you downloaded earlier, and choose **Open**.

    **Note** This .ppk file is usually located in the **Downloads** folder on your PC.

- \(Optional\) If you plan to start this session again later, you can save the session information for future use. Under **Category**, choose **Session**, enter a name for the session in **Saved Sessions**, and then choose **Save**.
- Choose **Open**.

45. [45]If this is the first time you have connected to this instance, PuTTY displays a security alert dialog box that asks whether you trust the host to which you are connecting. Choose **Yes**. A window opens and you are connected to your instance.

**Note** If you receive an error while attempting to connect to your instance, see [Troubleshooting Connecting to Your Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstancesConnecting.html).

<a id='ssh-macOS-Linux'></a>

### Connect to a Linux instance from macOS or Linux

**Note** Only perform the following steps if you are connecting from a macOS or Linux machine. If you are connecting from a Windows machine, <a href="#ssh-windows">click here</a> for instructions.

46. [46]On the left side of the lab page, click <i class="fas fa-download"></i> **Download PEM**. Save the file to the directory of your choice.

47. [47]Open terminal window on your local computer.

**Note** Your local computer most likely has an SSH client installed by default. You can check for an SSH client by typing `ssh` at the command line. If your local computer doesn't recognize the command, you can install an SSH client. For information about installing an SSH client on Linux or macOS X, see [https://www.openssh.com](https://www.openssh.com).

Complete the remaining connection steps in the terminal window.

48. [48]Change the directory to the folder where you downloaded the PEM file.

**Note** The PEM file is usually located in the **Downloads** folder on your computer. Access this directory by typing `cd ~/Downloads`

49. [49]Your key must not be publicly viewable for SSH to work. Change the permissions on the PEM file by running the following command. Replace `<PEM_FILE>` with the name of the PEM file you downloaded:

```bash
chmod 400 <PEM_FILE>
```

50. [50]Log in to the remote instance by running the following command. Replace `<PEM_FILE>` with the name of the PEM file you downloaded, `<user_name>` with the user name for the instance type you are connecting to, and `<public_dns_name>` with the **CommandHost** value from the left side of the lab page:

```bash
ssh -i <PEM_FILE> <user_name>@<public_dns_name>
```

**Note** For Amazon Linux 2 or the Amazon Linux AMI, the user name is `ec2-user`. For an Ubuntu AMI, the user name is `ubuntu`.

51. [51]If this is the first time you have connected to this instance, you see a response similar to the following:

```plaintext
The authenticity of host 'ec2-192-0-2-111.compute-1.amazonaws.com (192.0.2.111)'
can't be established.
RSA key fingerprint is 1f:51:ae:28:bf:89:e9:d8:1f:25:5d:37:2d:7d:b8:ca:9f:f5:f1:6f.
Are you sure you want to continue connecting (yes/no)?
```

52. [52]When prompted, enter `yes`

You are now connected to your instance.

**Note** If you receive an error while attempting to connect to your instance, see [Troubleshooting Connecting to Your Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstancesConnecting.html).

<a id='lambdapolicy'></a>

### Lambda function policy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "sns:Publish",
                "s3:GetBucketAcl",
                "s3:PutBucketAcl",
                "s3:GetBucketPolicy",
                "s3:GetBucketTagging",
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
```

### AWS Config policy

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:DeleteObject",
                "s3:DeleteObjectTagging",
                "s3:DeleteObjectVersionTagging",
                "s3:DescribeJob",
                "s3:Get*",
                "s3:HeadBucket",
                "s3:List*",
                "s3:PutBucketTagging",
                "s3:PutObject",
                "s3:PutObjectTagging",
                "s3:PutObjectVersionTagging",
                "s3:ReplicateTags",
                "sns:CheckIfPhoneNumberIsOptedOut",
                "sns:ConfirmSubscription",
                "sns:CreateTopic",
                "sns:DeleteTopic",
                "sns:Get*",
                "sns:List*",
                "sns:Publish",
                "sns:SetSMSAttributes",
                "sns:SetSubscriptionAttributes",
                "sns:SetTopicAttributes",
                "sns:Subscribe",
                "sns:Unsubscribe"
            ],
            "Resource": "*",
            "Effect": "Allow"
        }
    ]
}
```