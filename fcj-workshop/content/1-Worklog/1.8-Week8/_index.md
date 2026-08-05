---
title: "Worklog Week 8"
date: 2026-06-23
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Objectives

Set up a resource and log monitoring system for the EC2 instance, and create an automated alerting mechanism to notify the development team (Developers & Operations) whenever an incident occurs.

---

### Completed Tasks

- [x] Install and configure the CloudWatch Agent on the EC2 instance.
- [x] Push Nginx and Docker container logs (via the awslogs driver) to CloudWatch Log groups.
- [x] Create a Metric Filter in CloudWatch to catch log lines containing the `ERROR` keyword.
- [x] Set up CloudWatch Alarms to monitor the `ERROR` metric.
- [x] Integrate Amazon SNS to send Email/SMS alerts whenever the Alarm is triggered.

---

### Implementation Details

**1. Install CloudWatch Agent**
- Ensure the EC2 IAM Role (configured in Week 4) has the `CloudWatchAgentServerPolicy` attached.
- Install the agent and configure the JSON file to collect logs from `/var/log/nginx/error.log` and push them to the `/aws/ec2/nginx-error` log group.

**2. Collect Docker Logs**
Instead of leaving Docker logs scattered on the virtual machine, configure `docker-compose.yml` to use the `awslogs` logging driver:
```yaml
    logging:
      driver: awslogs
      options:
        awslogs-region: ap-southeast-1
        awslogs-group: /aws/docker/spring-boot-backend
        awslogs-create-group: "true"
```
Application logs will now be streamed directly to CloudWatch.

**3. Create Metric Filter & Alarms**
- Go to CloudWatch Log groups and create a Metric Filter with the pattern `"?ERROR" "?Exception"`.
- Create an Alarm based on this newly created Metric, setting the trigger condition when the number of errors is greater than 0 within a 1-minute period.

**4. Set up Alerts via SNS**
- Create an Amazon SNS Topic named `AppErrorAlerts`.
- Add an **Email** (and SMS if needed) Subscription for the Developers team. Confirm the subscription via email.
- Go back to the Alarm configuration in step 3, under "Notification", and select sending notifications to the `AppErrorAlerts` Topic.

---

### Challenges & Solutions

| # | Challenge | Solution |
|---|-----------|----------|
| 1 | Permissions error when Docker pushes logs to CloudWatch | The EC2 IAM Role lacked permissions to create log groups. Updated the IAM Policy to grant `logs:CreateLogGroup` and `logs:PutLogEvents`. |
| 2 | Receiving too many spam emails from CloudWatch Alarms | Adjusted the Alarm to be less sensitive (e.g., > 5 errors in 5 minutes) or fine-tuned the Metric Filter to only catch critical business errors. |

---

### References

- [Collect Metrics and Logs with CloudWatch Agent](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Install-CloudWatch-Agent.html)
- [Use Amazon CloudWatch Logs with Docker](https://docs.docker.com/config/containers/logging/awslogs/)
- [Amazon SNS Getting Started](https://docs.aws.amazon.com/sns/latest/dg/sns-getting-started.html)
