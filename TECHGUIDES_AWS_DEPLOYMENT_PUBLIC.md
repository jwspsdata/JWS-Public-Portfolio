# AWS Deployment

Reference for the AWS deployment of TechGuides — written against what was actually built, not a proposal. This is a self-funded, cost-conscious **demonstration** of deploying a Dockerized version of the app to AWS — not a production system. Part 2 of this document separately describes what a real production deployment would add.

## Part 1 — What was actually deployed

### Architecture

A single EC2 instance runs a container built from a multi-stage Dockerfile, pulled from a private ECR repository. The image bakes the app's retrieval index and source documents directly into the build, rather than mounting them as a volume or rebuilding them at container startup — the query-time embedding model has to match exactly what built the index in the first place, so keeping both in one versioned image artifact turns a runtime coordination risk into a build-time guarantee. No load balancer, no auto-scaling, no multi-AZ — appropriate tradeoffs for a single-instance demo started/stopped by hand, not a service meant to stay up.

| Component | Choice | Why |
|---|---|---|
| Compute | EC2, single instance, `t3.medium` | Cheapest to control for a stop-when-not-demoing usage pattern — $0 compute while stopped |
| OS | Amazon Linux 2023 | Matches the Dockerfile's Linux base directly; ~half the hourly cost of the Windows Server equivalent |
| Registry | Amazon ECR | AWS-native auth (IAM, not a separate login), automatic vulnerability scanning on push |
| Secret storage | SSM Parameter Store (SecureString) | The Anthropic API key never appears in EC2 user-data or a config file |
| Instance access | SSM Session Manager | No open SSH port, no key pair to manage — access is IAM-gated, not network-gated |
| Public IP | Ephemeral (no Elastic IP) | AWS bills hourly for all public IPv4s now, attached or not — the free ephemeral IP is cheaper for an instance that's mostly stopped |
| Cost safety net | AWS Budget, $10/month, 50%/100% email alerts | Catches an accidentally-left-running instance |

### What was created

- ECR repository `techguides` (private AWS account, `us-east-1`), scan-on-push enabled, lifecycle policy capping stored images at the 3 most recent.
- IAM role `techguides-ec2-role` / instance profile `techguides-ec2-profile`: ECR read access, SSM managed instance access, and an inline policy scoping `ssm:GetParameter` to exactly one parameter path.
- SSM parameter `/techguides/anthropic-api-key` (SecureString).
- Security group `techguides-demo-sg`: inbound 8501/tcp restricted to a single IP (the deployer's own, at creation time) — no port 22.
- EC2 instance (Amazon Linux 2023, `t3.medium`, 20GB gp3), launched with a user-data boot script that installs Docker, authenticates to ECR, pulls the image, fetches the API key from SSM, and runs the container with `--restart unless-stopped`.
- AWS Budget `techguides-demo-budget`.

### Build and push

```bash
docker build -t techguides:local .

# Auth (see "PowerShell + ECR" note below if this fails with a 400 on Windows)
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account-id>.dkr.ecr.us-east-1.amazonaws.com

docker tag techguides:local <account-id>.dkr.ecr.us-east-1.amazonaws.com/techguides:0.1.0
docker tag techguides:local <account-id>.dkr.ecr.us-east-1.amazonaws.com/techguides:latest
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/techguides:0.1.0
docker push <account-id>.dkr.ecr.us-east-1.amazonaws.com/techguides:latest
```

**PowerShell + ECR note:** `aws ecr get-login-password | docker login --password-stdin` can fail with a `400 Bad Request` on Windows PowerShell 5.1 — not an auth error, an encoding one. Piping a string into a native process's stdin through PowerShell's pipeline can corrupt it. Fix: write the token to a no-BOM UTF-8 file and redirect it in via `cmd /c "docker login ... < file"` instead of piping directly.

### Access

The app is reachable at `http://<instance-public-ip>:8501` — only from the IP address the security group was created for. Since the instance doesn't hold an Elastic IP, the public IP changes on every stop/start; re-fetch it after starting:

```bash
aws ec2 describe-instances --instance-ids <instance-id> --query "Reservations[0].Instances[0].PublicIpAddress" --output text
```

To widen access temporarily (e.g. to demo to someone else), add a second ingress rule for their IP, or change the existing one to `0.0.0.0/0` and revert it afterward — don't leave 8501 open to the world by default.

### Cost

| State | Approx. cost |
|---|---|
| Instance stopped | ~$1.60-2.40/month (20GB gp3 EBS only) + trivial ECR storage (~$0.30-0.50/month for a few GB) |
| Instance running | ~$0.042/hr (`t3.medium`, us-east-1, on-demand) on top of the above |

The dominant lever is simply **not leaving the instance running** — stop it between demo sessions. The Budget alarm is a backstop, not the primary control.

### Stop / start / teardown

```bash
# Stop (between demos)
aws ec2 stop-instances --instance-ids <instance-id>

# Start (re-fetch the public IP afterward — see above)
aws ec2 start-instances --instance-ids <instance-id>

# Full teardown — nothing left billing
aws ec2 terminate-instances --instance-ids <instance-id>
aws ec2 delete-security-group --group-id <sg-id>
aws ecr batch-delete-image --repository-name techguides --image-ids imageTag=latest imageTag=0.1.0
aws ecr delete-repository --repository-name techguides --force
aws ssm delete-parameter --name /techguides/anthropic-api-key
aws iam remove-role-from-instance-profile --instance-profile-name techguides-ec2-profile --role-name techguides-ec2-role
aws iam delete-instance-profile --instance-profile-name techguides-ec2-profile
aws iam detach-role-policy --role-name techguides-ec2-role --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly
aws iam detach-role-policy --role-name techguides-ec2-role --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
aws iam delete-role-policy --role-name techguides-ec2-role --policy-name techguides-ssm-param-read
aws iam delete-role --role-name techguides-ec2-role
aws budgets delete-budget --account-id <account-id> --budget-name techguides-demo-budget
```

## Part 2 — What a production deployment would look like

This demo intentionally skips the things that make a deployment *operable by a team* rather than *runnable by one person who built it*. For a real production deployment of this app (or a similar containerized service), the gap looks like this:

| Area | Demo (this doc) | Production |
|---|---|---|
| Compute | Single EC2 instance, manually started/stopped | ECS Fargate (or EKS) service with a defined task count and auto-scaling policy — no single point of failure, no manual start/stop |
| Load balancing / TLS | None — direct instance IP | Application Load Balancer with an ACM-issued TLS certificate and a Route 53 custom domain |
| Networking | Default VPC, public subnet, security group scoped to one IP | Purpose-built VPC: compute in private subnets behind a NAT gateway, only the ALB in a public subnet |
| Secrets | SSM Parameter Store (SecureString) | AWS Secrets Manager — automatic rotation, finer-grained audit trail via CloudTrail |
| Deploys | Manual `docker build` / `push` / instance boot script | CI/CD pipeline (e.g. GitHub Actions): build → push to ECR → trigger an ECS rolling deployment, with automated tests gating the build |
| Observability | `docker logs`, manual health checks | CloudWatch dashboards and alarms (latency, error rate, container health, external API error rate), structured logging shipped to CloudWatch Logs or an aggregator |
| IAM | One instance role covering ECR + one SSM parameter | Least-privilege per-service roles — e.g. a separate execution role vs. task role in ECS, scoped narrowly to only what each service needs |
| Edge protection | None | AWS WAF in front of the ALB (rate limiting, common exploit rule sets) |
| Data versioning | Retrieval index baked into the image, rebuilt and redeployed manually when the corpus changes | A defined reindex pipeline with the resulting artifacts versioned (e.g. in S3) and the image build pulling a pinned version, so index changes are reproducible and rollback-able independent of code changes |
| Availability | Single instance, single AZ | Multiple tasks across at least two AZs behind the ALB |

None of this is implemented here — it's the explicit target this demo's architecture would evolve toward for a real workload, kept separate so the demo itself stays legible and cheap.

## VS Code extensions used

- **Docker** (`ms-azuretools.vscode-docker`) — Dockerfile linting, build/run from the editor, image/container explorer.
- **YAML** (`redhat.vscode-yaml`) — schema validation for `docker-compose.yml`.

(Amazon's "AWS Toolkit" functionality now lives inside the **Amazon Q** extension, but new Amazon Q Developer accounts closed May 15, 2026 and the plugin itself loses support April 30, 2027 — not a durable recommendation. The AWS Console covers the same resource-browsing need with no such expiry.)

## Related

- Live demo: [jws-ai-portfolio.streamlit.app](https://jws-ai-portfolio.streamlit.app/), under "TechGuides"
