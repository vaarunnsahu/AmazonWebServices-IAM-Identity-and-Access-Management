# AWS IAM Basics: Identity & Access Management Deep Dive

This module covers the core fundamentals of **AWS IAM (Identity and Access Management)**. IAM is a **Global** and **FREE** service used centrally to manage access, authentication, and authorization across your entire cloud footprint.

---

## 1. Multi-Factor Authentication & Account Strategy

AWS utilizes a strict separation of privileges between account owners and operators.
* **Root User:** Created when the account is first built. Has absolute/full permissions. **Never use the Root account for daily operational work.**
* **IAM User:** Created by administrators with limited, scoped-down permissions for daily activities.
* **Account Alias:** Created at the root level to replace the generic 12-digit AWS Account ID with a user-friendly string (e.g., `https://reyacloud.signin.aws.amazon.com/console`), which remains identical for all IAM users under that account.

### Multi-Factor Authentication (MFA)
Setting up MFA (via apps like Google Authenticator) is **highly recommended** for the Root account and **mandatory** for every individual IAM user accessing the console.

![AWS Account Login & Access Methods](1_3.png)
![IAM Sign-in Custom URL Alias](7_3.png)

---

## 2. The 2 Ways to Access AWS

| Access Type | Primary Interface | Authentication Method | MFA Requirement |
| :--- | :--- | :--- | :--- |
| **Console Access** | Graphical User Interface (GUI) via Web Browser | Email/Password (Root) or Username/Password (IAM) | Required |
| **Programmatic Access** | AWS CLI (Command Line), SDKs (Java, .NET, Python), and Developer Tools | **Access Key** & **Secret Access Key** (1 Set) | No MFA |

### Critical Rules for Programmatic Access Keys:
1. **Visibility:** The Secret Access Key is visible **only once** at the time of creation. If lost, it cannot be recovered and must be re-generated.
2. **Security Risk:** Never store keys locally on a public-facing instance or commit them to source control. **Do not create or use access keys for the Root account.**
3. **Configuration:** Run the `aws configure` command to securely initialize your CLI credentials locally on Windows CMD or Linux.
4. **Limits:** A single IAM User can hold a **maximum of 2 sets of Keys** at any time. Rotate keys and passwords regularly.

---

## 3. IAM Entities: Users, Groups, and Policies

### IAM Groups
An IAM Group is a collection of IAM users. 
* Groups are used to assign the same permissions to a bunch of users simultaneously.
* **Nested Groups (Group under Group) are not possible.**
* An IAM User can belong to multiple IAM Groups concurrently. When added to a group, users inherit group-level permissions *without* losing their individual user-level permissions.

### IAM Policies
Permissions are defined using **JSON (JavaScript Object Notation)** files. New IAM users have **no policies attached by default**. You can attach a maximum of 10 policies to a single user or group.

* **Managed Policies:** Predefined policies created and maintained directly by AWS (e.g., `EC2FullAccess`).
* **Inline Policies:** Custom policies created and managed directly by the customer for granular, deeper resource-level permissions.

### Amazon Resource Name (ARN)
Every asset in AWS is mapped to a unique identifier called an ARN, which is heavily utilized inside JSON policy blocks to specify resource constraints.
* *Example IAM User ARN format:* `arn:aws:iam::9787875:user/pooja`

![IAM Groups, Policies, and JSON Structures](2_3.png)

### Core JSON Policy Structure
An IAM policy document contains the following standard structure:
* **Version:** The policy language version.
* **ID:** An optional policy identifier.
* **Statement:** An array containing one or more individual rule blocks:
    * **Sid (Statement ID):** An optional identifier for the specific statement.
    * **Effect:** Explicitly set to either `Allow` or `Deny`.
    * **Principal:** The specific Account, User, or Role to which this policy applies.
    * **Action:** The list of API actions/calls this policy permits or blocks.
    * **Resource:** The list of explicit ARNs to which the actions apply.
    * **Conditions:** Optional constraints defining exactly when the policy is in effect.

![IAM Policy Architecture and Tooling](6_3.png)

---

## 4. IAM Roles & Cross-Account Access

### Why Use Roles?
**AWS services cannot talk to each other by default.** Hardcoding permanent Access/Secret Keys inside application code or storing them locally on an EC2 instance is unsafe. Instead, use **IAM Roles**, which grant **temporary security credentials without explicit keys**.

### Key Behaviors:
* An IAM Role can be attached to *any* AWS service.
* A single EC2 instance can only have **1 IAM Role attached at a time**, but a single role can be attached across multiple EC2 instances.
* Roles use a **Trusted Entity (TE)** declaration to specify *who* can assume the role, combined with a standard policy declaring *what* they are allowed to do.

### Standard Automation Scenarios:
* **Scenario 1 (Service-to-Service):** An EC2 instance running AWS CLI or an application needs to pull/push objects to an Amazon S3 bucket. You attach an IAM Role to the EC2 instance with a Trusted Entity of `EC2` and a permission policy for `S3`.
* **Scenario 2 (Event-Driven Automation):** An AWS Lambda function needs to execute a script to stop target instances. You create an IAM Role with a Trusted Entity of `Lambda` and an `EC2` management permission policy.
* **Scenario 3 (Cross-Account / Switch Role):** To access resources safely in another corporate account (e.g., accessing an S3 bucket in *Reyaz's AWS Account* from *Sriram's AWS Account*), users leverage cross-account IAM Roles rather than creating duplicate IAM user accounts. A target `Switch Role` URL is generated, which issues a temporary 1-hour active session.

![IAM Roles Architecture](3_3.png)
![IAM Role Implementation Scenarios](8_3.png)

---

## 5. Enterprise Management: Federation, SSO, and Organizations

### Identity Federation & SAML 2.0
In an enterprise environment, creating local IAM users for every employee is a massive operational headache. Instead, companies connect their on-premises directory (like Active Directory via LDAP) to AWS using **SAML 2.0 (Security Assertion Markup Language)**. 

* **Identity Provider (IdP):** Your corporate directory system (Active Directory, Okta, Ping Identity). Holds user credentials.
* **Service Provider (SP):** AWS.
* **Mechanism:** The IdP authenticates the user ("Who are you?") and passes authorization details ("What can you do?") to AWS using an encrypted XML document, mapping the corporate user directly to an AWS IAM Role without utilizing local IAM users.

![Identity Provider & Active Directory Mapping](4_3.png)
![SAML 2.0 Authentication Workflow](5_3.png)

### AWS Organizations & IAM Identity Center
When dealing with multiple corporate AWS accounts, companies combine **AWS Organizations** and **AWS IAM Identity Center** to establish centralized Single Sign-On (SSO).

1. Enabling Organizations on a primary account turns it into the **Management Account**.
2. External developer or production accounts are invited to join as **Member Accounts**.
3. **Service Control Policies (SCPs):** Centrally applied from the Management Account to restrict maximum boundaries across Member Accounts (e.g., ensuring a Member Account cannot delete critical logs, even if their local user has admin rights).
4. Centralized users are created once inside the Identity Center, assigned custom permission sets, and given a single **AWS access portal URL** to log seamlessly into assigned Member Accounts without needing local credentials.

![AWS Organizations and Identity Center Portal Architecture](9_3.png)

---

## 6. Audit, Tracking, and Resource Tagging

### Tracking Tools
* **IAM Credentials Report:** A downloadable CSV report listing all account users and the exact status of their credentials (passwords, MFA, key age).
* **IAM Access Advisor:** Displays service-level permissions granted to a user versus exactly when those permissions were last accessed.
* **IAM Access Analyzer:** Scans your cloud blueprint to identify and flag resources exposed to external or unused access boundaries.

### IAM Tags
Tags are **Key-Value pairs** used globally across AWS for resource identification, metadata organization, billing allocation tracking, and advanced access control automation (e.g., executing an automated Lambda function that forces a `STOP` command across all EC2 instances containing the tag `Key = Name` / `Value = Testers`).
* *Maximum Limit:* Up to 50 tags per individual AWS resource.
