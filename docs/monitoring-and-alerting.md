# Automated Monitoring & Logging for Deployment Issues  
**AWS ECS (Fargate), CloudWatch, Application Load Balancer**

---

## Table of Contents

1. [Issue Overview](#issue-overview)
2. [Monitoring Design Philosophy](#monitoring-design-philosophy)
3. [Architecture Overview](#architecture-overview)
4. [STEP 1 — Centralized Logging (Foundation)](#step-1--centralized-logging-foundation)
5. [STEP 2 — ECS Monitoring & Alarms](#step-2--ecs-monitoring--alarms)

   * [6.1 Alarm: ECS Tasks Not Running](#61-alarm-ecs-tasks-not-running)
   * [6.2 Alarm: ECS Deployment Failure](#62-alarm-ecs-deployment-failure)
   * [6.3 Alarm: ECS High Memory Usage](#63-alarm-ecs-high-memory-usage)
   * [STEP 2 Final Checklist](#step-2-final-checklist)
6. [STEP 3 — ALB Monitoring (User Impact Detection)](#step-3--alb-monitoring-user-impact-detection)

   * [7.1 Alarm: ALB Unhealthy Targets](#71-alarm-alb-unhealthy-targets)
   * [7.2 Alarm: ALB Target 5XX Errors](#72-alarm-alb-target-5xx-errors)
   * [STEP 3 Final Checklist](#step-3-final-checklist)
7. [STEP 4 — SNS Alerts (Real-Time Notifications)](#step-4--sns-alerts-real-time-notifications)
8. [STEP 5 — Failure Simulation (Validation)](#step-5--failure-simulation-validation)
9. [STEP 6 — Restore Application & Final Wiring](#step-6--restore-application--final-wiring)
10. [Final Verification Checklist](#final-verification-checklist)
11. [Conclusion](#conclusion)

---

## Issue Overview

### What the Issue Was
The system lacked automated, reliable monitoring and alerting for ECS deployments.  
Deployments could fail at runtime (task crashes, failed health checks, bad releases) without any immediate signal, even when CI pipelines reported success.

### Why This Is Dangerous
In real-world production environments:
- Jenkins success only confirms that deployment commands ran.
- ECS and ALB determine whether the application is actually running and serving users.
- Silent failures lead to prolonged outages, delayed response, and loss of trust.

### What Success Looks Like After the Fix
- All ECS containers stream logs centrally to CloudWatch.
- ECS failures, deployment failures, and resource pressure are detected automatically.
- ALB detects real user impact (unhealthy targets, 5XX errors).
- Critical failures trigger real-time alerts.
- The system is validated through controlled failure simulation.

---

## Monitoring Design Philosophy

### Why Logging Is the Foundation
Monitoring answers *that* something failed.  
Logs explain *why* it failed.

Without centralized logs:
- Crashes cannot be diagnosed.
- Deployment failures lack root cause.
- Monitoring signals become noise.

### Why Jenkins Success ≠ Deployment Success
Jenkins:
- Builds images
- Pushes to ECR
- Calls `update-service`

ECS:
- Pulls the image
- Starts the container
- Runs health checks
- Decides if the service stabilizes

Only ECS can confirm success.

### Infrastructure Health vs User Impact
- **ECS metrics** tell you if containers are running.
- **ALB metrics** tell you if users can actually use the application.

Both are required.

### Mental Model: DesiredTaskCount vs RunningTaskCount
ECS continuously tries to enforce:
```

DesiredTaskCount == RunningTaskCount

```
Any sustained mismatch indicates downtime or instability.

---

## Architecture Overview
![Automated ECS Monitoring Architecture](./architecture_diagram/architecture_diagram.png)

## STEP 1 — Centralized Logging (Foundation)

### Objective
Ensure every ECS container streams logs to CloudWatch so failures always leave evidence.

### ECS Task Definition Log Configuration
Container definition must include:

```json
"logConfiguration": {
  "logDriver": "awslogs",
  "options": {
    "awslogs-group": "/ecs/<task-name>",
    "awslogs-region": "<region>",
    "awslogs-stream-prefix": "ecs"
  }
}
```

### Why Fargate Requires External Logs

* Fargate tasks are ephemeral.
* No SSH access is possible.
* Containers can disappear instantly on failure.
* Logs must be exported in real time.

### GUI Steps to Verify Logs

1. AWS Console → CloudWatch
2. Log groups → `/ecs/<task-name>`
3. Verify log streams exist:

   ```
   ecs/<container-name>/<random-id>
   ```
4. Confirm logs update when traffic hits the application.

### Completion Checklist

* Log group exists
* Log streams are created
* Logs appear on requests
* No permission errors

### Issue Mapping

| Issue Requirement   | Status |
| ------------------- | ------ |
| Centralized logging | Done   |
| Runtime visibility  | Done   |
| Debug capability    | Done   |

---

## STEP 2 — ECS Monitoring & Alarms

### Purpose

Detect runtime crashes and failed deployments automatically.

### Why ECS Metrics May Be Missing

ECS service-level metrics require **Container Insights**.
Without it, metrics like `RunningTaskCount` and `ServiceDeploymentFailures` do not exist.

### Enable Container Insights (GUI)

1. AWS Console → ECS → Clusters
2. Select cluster → Update cluster
3. Enable **Container Insights**
4. Update cluster (no downtime)

### Force Metric Emission

After enabling:

* ECS → Service → Update → Update service
* Wait 3–5 minutes for metrics to appear.

---

### 6.1 Alarm: ECS Tasks Not Running

**What It Detects**

* Container crashes
* Image pull failures
* OOM kills
* Health check failures

**Why It Is Critical**
This alarm answers:

> Is my application actually running right now?

**GUI Steps**

* CloudWatch → Alarms → Create alarm
* Namespace: ECS
* Metric: `RunningTaskCount`
* Dimensions: ClusterName, ServiceName

**Metric Configuration**

| Setting            | Value           |
| ------------------ | --------------- |
| Statistic          | Minimum         |
| Period             | 60s             |
| Evaluation periods | 2               |
| Threshold          | < Desired count |

**Alarm Name**

```
ecs-tasks-not-running
```

**Design Rationale**

* Minimum catches brief drops.
* Two datapoints avoid restart noise.

**Success Criteria**

* Alarm state: OK during normal operation.

---

### 6.2 Alarm: ECS Deployment Failure

**What It Detects**

* Failed rolling deployments
* Services that never stabilize

**Why Jenkins Cannot Detect This**
Jenkins cannot observe ECS stabilization or rollback behavior.

**Metric**
`ServiceDeploymentFailures`

**Configuration**

| Setting   | Value |
| --------- | ----- |
| Statistic | Sum   |
| Period    | 60s   |
| Threshold | > 0   |

**Alarm Name**

```
ecs-deployment-failed
```

**Success Criteria**

* Alarm remains OK during healthy deployments.

---

### 6.3 Alarm: ECS High Memory Usage

**Why This Is a Warning**
High memory does not mean downtime yet, but it predicts it.

**Metric**
`MemoryUtilization`

**Configuration**

| Setting    | Value   |
| ---------- | ------- |
| Statistic  | Average |
| Period     | 60s     |
| Evaluation | 3       |
| Threshold  | ≥ 80%   |

**Alarm Name**

```
ecs-high-memory-usage
```

**Success Criteria**

* Alarm only triggers on sustained pressure.

---

### STEP 2 Final Checklist

* Tasks not running alarm
* Deployment failure alarm
* High memory warning alarm

---

## STEP 3 — ALB Monitoring (User Impact Detection)

### Why ALB Monitoring Is Required

ECS may say “running” while users experience errors.

---

### 7.1 Alarm: ALB Unhealthy Targets

**What It Detects**

* Failed health checks
* Wrong ports
* Hung applications

**Metric**
`UnHealthyHostCount`

**Configuration**

| Setting   | Value   |
| --------- | ------- |
| Statistic | Maximum |
| Period    | 60s     |
| Threshold | > 0     |

**Alarm Name**

```
alb-unhealthy-targets
```

**Success Criteria**

* Alarm remains OK under normal traffic.

---

### 7.2 Alarm: ALB Target 5XX Errors

**ELB 5XX vs Target 5XX**

* ELB 5XX: load balancer failure
* Target 5XX: application failure

**Metric**
`HTTPCode_Target_5XX_Count`

**Configuration**

| Setting   | Value |
| --------- | ----- |
| Statistic | Sum   |
| Period    | 60s   |
| Threshold | ≥ 5   |

**Alarm Name**

```
alb-target-5xx-errors
```

**Success Criteria**

* No alerts during normal behavior.

---

## STEP 4 — SNS Alerts (Real-Time Notifications)

### Why Alarms Without Alerts Are Useless

Dashboards do not wake engineers. Alerts do.

### SNS Design Pattern

CloudWatch Alarm → SNS Topic → Notification endpoint

### Create SNS Topic (GUI)

* SNS → Topics → Create topic
* Type: Standard
* Name: `deployment-alerts`

### Create Email Subscription

* Protocol: Email
* Confirm subscription via email.

### Alert Hygiene

* Critical alarms send alerts.
* Warning alarms do not.

---

## STEP 5 — Failure Simulation (Validation)

### Why Simulation Is Mandatory

Monitoring is unproven until it detects real failure.

### Safe Failure Methods

* Break container startup
* Exit application immediately
* Fail health check

### Expected Results

* ECS alarms enter ALARM state
* SNS email received
* Logs show failure reason

---

## STEP 6 — Restore Application & Final Wiring

### Restore Application

* Revert failure change
* Redeploy service
* Verify recovery

### Attach SNS to All Critical Alarms

Attach `deployment-alerts` to:

* ecs-tasks-not-running
* ecs-deployment-failed
* alb-unhealthy-targets
* alb-target-5xx-errors

---

## Final Verification Checklist

* Logs visible in CloudWatch
* ECS alarms operational
* ALB alarms operational
* SNS notifications delivered
* Alarms return to OK after recovery

---

## Conclusion

This fix introduces a complete, production-grade monitoring and alerting system for ECS-based deployments.

It:

* Detects deployment and runtime failures automatically
* Surfaces real user impact
* Delivers actionable alerts in real time
* Is validated through controlled failure testing

The GitHub issue is fully resolved with a design that matches real-world operational standards.

