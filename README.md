# Comfy Cloud – Pipeline Health Monitoring (Module 2)

Module 2 demo: turn a **BUILD_FAILED** log line into a metric, add a dashboard & anomaly-detection alarm, and send alerts to e-mail or Slack. 

---

## Prerequisites

| Tool | Version tested |
|------|----------------|
| **AWS CLI** | ≥ 2.13 |
| **kubectl** | ≥ 1.29 |

1. **Module 1 pipeline is running** — CodeBuild project `hoodie-build` has succeeded at least once.  
2. **SNS topic `ComfyPipelineAlerts` exists and you are subscribed. Do this in CloudShell**

   ~~~bash
   # create the topic if it doesn’t exist
   aws sns create-topic --name ComfyPipelineAlerts

   # subscribe your e-mail
   aws sns subscribe \
     --topic-arn $(aws sns list-topics \
       --query "Topics[?contains(TopicArn,'ComfyPipelineAlerts')].TopicArn" \
       --output text) \
     --protocol email \
     --notification-endpoint you@example.com
   ~~~

3. *(Optional)* Slack alerts — console path **Amazon Q Developer in chat applications (previously AWS Chatbot) → Slack → Configure client**, pick `ComfyPipelineAlerts`, then authorize the workspace.  
4. Clone (or pull) this repo and stay on branch **main**

   ~~~bash
   git clone https://github.com/MichaelCassidyPS/comfy-cloud-monitoring.git
   cd comfy-cloud-monitoring
   ~~~


---


