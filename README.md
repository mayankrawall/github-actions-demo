# SkillPulse-DevSecOps-Pipeline

SkillPulse is a multi-tier application configured with a robust, automated CI/CD pipeline to demonstrate production-ready DevOps and deployment practices."

---Why GitHub Actions (with Self-Hosted Runners)
A pipeline needs a runner — something that watches your repo, executes your build/test/deploy steps, and reports back. While GitHub provides hosted runners, managing your own self-hosted runner gives you full control over the execution environment, hardware specs, and security compliance without the overhead of maintaining a separate CI/CD platform like Jenkins.

GitHub Actions with self-hosted runners wins on three things:

It lives where the code lives. No separate UI or complex third-party auth. Your .github/workflows/*.yml files are part of the repo — they evolve with the code, get reviewed in the same PRs, and survive every clone.

Complete environment & cost control. You choose the hardware (on-premise servers, cloud VMs, or local machines) and pre-install heavy dependencies. You don't pay GitHub for compute time, making it highly cost-effective for resource-intensive or long-running builds.

Secure & private network access. Because the runner sits inside your own infrastructure (AWS VPC, local network, etc.), it can securely deploy to internal servers and databases without exposing them to the public internet.

The trade-off is infrastructure management. You are responsible for the runner's uptime, security updates, and scaling. For most teams needing high performance or strict compliance, that's a fair price for total control.

Key modifications made:
Shifted the focus from "zero friction/free tier" to infrastructure control, cost efficiency for heavy workloads, and security.

Highlighted the main benefit of self-hosted runners: running pipelines inside your own secure network/VPC.

Updated the trade-off section to acknowledge the responsibility of managing your own infrastructure.


---

## Acknowledgments & Credits

* **Project Concept & Base Architecture:** Inspired by the *TrainWithShubham GitHub Actions & Kubernetes Masterclass*. 
* **Implementation & Deployment:** Fully configured, deployed, and debugged by me. This includes setting up the multi-tier Docker containers, writing Kubernetes manifests, configuring GitHub Actions workflows, and managing the AWS infrastructure.


