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
   git clone https://github.com/<your-handle>/comfy-cloud-monitoring.git
   cd comfy-cloud-monitoring
   ~~~

5. Resume the EKS node-group (if you scaled it to 0 after Module 1)

   ~~~bash
   # Bring the EKS node-group back online (scales to 2 nodes)
   NG=$(eksctl get nodegroup --cluster dev-eks -o json | jq -r '.[0].Name')

   eksctl scale nodegroup \
     --cluster dev-eks \
     --name "$NG" \
     --nodes 2 \
     --nodes-min 2 \
     --nodes-max 2 \
     --region us-east-1

   # Wait ~1 minute for nodes to register, then restore workloads
   kubectl scale deployment \
     --all \
     --replicas=2
   ~~~

---


