# comfy-cloud-monitoring

# Comfy Cloud Monitoring Demo (Module 2)

Follow this guide to reproduce the **“From Data to Action – Alerting on Failed Hoodie Builds”** demo.

---

## Prerequisites

| Requirement            | Notes                                                                                                                         |
|------------------------|--------------------------------------------------------------------------------------------------------------------------------|
| **AWS CLI v2**         | Configured for an IAM identity that can create CloudWatch, CloudWatch Logs, and SNS resources.                                |
| **SNS topic**          | Create **ComfyPipelineAlerts** with: `aws sns create-topic --name ComfyPipelineAlerts`                                        |
| **Module 1 pipeline**  | A working CodeBuild project and CodePipeline that build & deploy the Hoodie API.                                             |

---

## 1  Import the Dashboard

``aws cloudwatch put-dashboard --dashboard-name "ComfyCloud-Pipeline-Health" --dashboard-body file://cw-dashboard.json``

Open `cw-dashboard.json` beforehand and replace `${CODEBUILD_PROJECT}` with your actual project name.

---

## 2  Create the Metric Filter

``aws logs put-metric-filter --log-group-name "/aws/codebuild/<YOUR_PROJECT_NAME>" --cli-input-json file://metric-filter.json``

All **BUILD_FAILED** log lines become data points in the custom metric `ComfyCloud/BuildFailedEvents`.

---

## 3  Add the Anomaly-Detection Alarm

``aws cloudwatch put-anomaly-detector --namespace "ComfyCloud" --metric-name "BuildFailedEvents" --stat "Sum"``  
``aws cloudwatch put-metric-alarm --cli-input-json file://alarm.json``

Ensure the **AlarmActions** ARN in `alarm.json` points to **ComfyPipelineAlerts**.

---

## 4  Subscribe Destinations

``aws sns subscribe --topic-arn <TOPIC_ARN> --protocol email  --notification-endpoint alex@comfycloud.example``  
``aws sns subscribe --topic-arn <TOPIC_ARN> --protocol https --notification-endpoint <CHATBOT_WEBHOOK_URL>``

Confirm the email link and authorize the Slack channel in AWS Chatbot.

---

## 5  Trigger a Controlled Failure

1. Copy the helper script: `cp scripts/fail_build.sh .`  
2. In **buildspec.yml** (pre_build phase) add: `- ./fail_build.sh`  
3. Commit and push:  
* git add fail_build.sh buildspec.yml
* git commit -m "Intentional failure for monitoring demo"
* git push

---

## 6  Clean Up (optional)

``aws cloudwatch delete-alarms --alarm-names Comfy-BuildFail-Anomaly``  
``aws cloudwatch delete-anomaly-detector --namespace ComfyCloud --metric-name BuildFailedEvents --stat Sum``  
``aws logs delete-metric-filter --log-group-name "/aws/codebuild/<YOUR_PROJECT_NAME>" --filter-name BuildFailedFilter``  
``aws cloudwatch delete-dashboard --dashboard-name "ComfyCloud-Pipeline-Health"``

You now have a complete **metrics → dashboard → alarm → notification** pipeline for Comfy Cloud’s builds!
