# Just-in-Time-JIT-Access-and-Cloud-Security-Hardening
Domain: Identity and Access Management (IAM) | Cloud Security Operations

## Project Overview
This project demonstrates the implementation of a secure, least-privilege architecture on Google Cloud Platform (GCP). It focuses on securing a Cloud Run application using Identity-Aware Proxy (IAP) and simulating Just-In-Time (JIT) administrative elevation.

## Technologies Used
* GCP Services: Cloud Run, Identity-Aware Proxy (IAP), Privileged Access Manager (PAM).
* Security Concepts: RBAC (Role-Based Access Control), Least Privilege, Context-Aware Access.
* Tools: Google Cloud Shell (CLI), YAML, Infrastructure as Code (IaC) principles.

## Key Implementation Steps

### 1. Application Hardening
* Deployed a Cloud Run service (`secret-memo-portal`) with "Internal Only" ingress.
* Configured Identity-Aware Proxy (IAP) to ensure only authenticated users with specific IAM conditions could access the portal.

### 2. PAM Implementation & Troubleshooting
* Attempted deployment of Privileged Access Manager (PAM) for automated JIT elevation.
* Engineering Challenge: Encountered API constraints requiring an "Organization" node for PAM entitlements.
* Solution: Navigated complex YAML configurations to satisfy API requirements for resource types and justification configurations.

### 3. JIT Simulation (The Pivot)
Due to the platform constraint (standalone project vs. organization node), I pivoted to a manual JIT workflow to maintain project continuity:
* Elevation: Used `gcloud` CLI to bind `roles/run.admin` to a specific user on-demand.
* Verification: Confirmed administrative access via the Cloud Run console.
* Revocation: Executed IAM policy removal and verified "Least Privilege" status using:
    `gcloud projects get-iam-policy [soc-analyst-489351] --filter="bindings.members:[chinonyestanleyokere@gmail.com]"`

## Security Observations
* Session Persistence: Observed a 60-90 second propagation delay between CLI revocation and UI updates in the GCP Console.
* Auditability: All elevation and revocation actions were tracked via Google Cloud Audit Logs, ensuring a full trail for compliance.

---

## Architecture & Implementation

### 1. Application Lockdown (IAP)
To prevent unauthorized access, I deployed a Cloud Run service and restricted ingress to "Internal + Load Balancing" only. I then activated Identity-Aware Proxy to act as a gatekeeper.
![IAP](Creation%20of%20new%20IAM%20policy.png)
![IAP](Removing%20public%20access%20to%20created%20link.png)
![IAP](Public%20access%20removed%20from%20URL.png)
![IAP](Principle%20and%20condition%20added.png)
![IAP](Turning%20on%20IAP%20for%20the%20URL.png)
Showing the IAP dashboard with the Cloud Run service toggled 'On' and the specific user permissions assigned.*

### 2. The PAM Configuration Challenge
I attempted to implement Privileged Access Manager (PAM) for automated elevation. This involved deep-level debugging of YAML configurations to satisfy the GCP Beta API requirements. During the implementation, I identified a critical platform constraint: PAM requires the project to be part of a Google Cloud Organization. 
![PAM](Creating%20entitlement%20with%20PAM%201.png)
![PAM](platform%20constraint%20analysis.png)
Navigating and resolving 'Invalid Argument' errors during the YAML-based entitlement creation. The specific error message identifying that PAM requires an Organization node—a key finding for infrastructure planning.

### 3. JIT Simulation & Verification
To maintain the security objective, I pivoted to a manual JIT simulation using the `gcloud` CLI. I successfully elevated my account to `Cloud Run Admin` and then verified the removal of access.
![JIT](simulate%20JIT%20acess.png)
![JIT](Grant%20of%20JIT%20successfull.png)
![JIT](Revoke%20grant%20of%20JIT.png)
![JIT](Grant%20successfully%20rekoved.png)
Running `get-iam-policy` to confirm that 'run.admin' privileges were successfully revoked, returning an empty result.*

---

## Key Cybersecurity Findings
* Infrastructure as Code (IaC): Defined security entitlements via YAML files, ensuring repeatable and auditable configurations.
* Least Privilege: Demonstrated the full lifecycle of a privilege—granting access only when needed and revoking it immediately after the task.
* Propagation Awareness: Documented the delay between IAM backend revocation and Frontend UI updates (Console ghosting), a critical factor in incident response.
