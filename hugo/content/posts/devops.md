---
title: "DevOps and SRE"
date: 2023-08-25T16:57:15+10:00
# weight: 1
# aliases: ["/first"]
tags: ["DevOps", "SRE"]
author: "Colin Loh"
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: false
cover:
    image: "<image path/url>" # image path/url
    alt: "<alt text>" # alt text
    caption: "<text>" # display caption under cover
    relative: false # when using page bundles set this to true
    hidden: true # only hide on current single page
---
## Introduction:

Test

When I embarked on my journey as a Cloud Engineer, I found myself surrounded by buzzwords. One question that frequently crossed everyone's mind was:

> *"What is the difference between SRE and DevOps?"*

This question led me down a rabbit hole in search of answers, prompting me to delve deeper into the concepts of **SRE** and **DevOps**.

- - -

## What is DevOps?
DevOps, as the name suggests is a unification of “Development” and “Operations”. In both Amazon and Google’s definition:

> *“DevOps is set of **practices**, **guidelines** and **culture** to break down silo and ability to deliver application”* 

The term DevOps emphasises collaboration between Developers and their Operation team. It is a different way of working where Devs and Ops work can be done in the same team to increase productivity and efficiency. 

As Werner Vogels, Amazon CTO puts it: 
> *“You build it, you run it”* 

&nbsp;

## What is SRE? 

Ben Treynor Sloss, a Google Engineer introduces Site Reliability Engineering to be responsible for: 

> *“availability, latency, performance, efficiency, change management, monitoring, emergency response, and capacity planning of their service(s)”* 

This can be achieved by adhering to the following specific metrics such as: 

- SLO - Service Level Objective
  - Performance metrics and a threshold that should be met. 
- SLA - Service Level Agreement
  - Framework that ensures accountability and transparency. 
- SLI - Service Level Indicator 
  - Specific metrics such as response time, error rate, availability, latency.

Site Reliability Engineering investigates the root cause and devise strategies to prevent them such as doing backups, testing, disaster recovery and budgeting. 

- - -

## Key Differences between DevOps and SRE?

```
class SRE implements DevOps
```

DevOps is an abstract class while SRE is a human object who implements DevOps.
Another analogy that I like to use is DevOps being a religion while SRE is a clergy. 


DevOps is a culture, not a designated position which was introduced to break silos between the teams and SRE is a role that adopts DevOps practices. However, this is far from the reality as you can tell from the countless listing of DevOps role in LinkedIn. 

&nbsp;

### DevOps became a role but what is the difference with SRE? 

|                   | **DevOps Engineer**                                          | **Site Reliability Engineer**                                |
|-------------------|--------------------------------------------------------------|--------------------------------------------------------------|
| Goals             | Faster software delivery, increased frequency of releases, and improved collaboration. | Ensuring service uptime, minimizing incidents, budgeting and automating operational tasks. |
| Responsibility    | Creating a streamlined release process to deploy code quickly using CI/CD tools and automation.  | Focuses on operational tasks, monitoring, incident response, and automation to enhance reliability. |
| Automation        | Automation of development, testing, and deployment processes (CI/CD). | Automation of operational tasks, incident management, and toil reduction. |
| Tools             | CI/CD pipelines, infrastructure as code (IaC), version control systems (Git). | Monitoring tools, observability solutions, configuration management, chaos testing tools. |
| Collaboration     | Collaboration between development and operations teams to break down silos. | Collaboration between sysadmins and developers to bridge the gap and ensure reliability. |
| Metrics           | Emphasizes speed of delivery, lead time, deployment frequency. | Focuses on reliability metrics such as SLO, SLA and SLI.     |
| Incident Response | Involvement in incident response, but not necessarily the primary focus. | Core focus on incident management, post-incident analysis, and learning from failures. |

The table above reflects my own understanding of DevOps and SRE. 

In reality, DevOps Engineer focuses on reducing silo with Software Development teams. This is achievable by integrating development process into workflows, automate and managing releases by setting up CI/CD pipelines. 

DevOps aims to deploy code quickly but at the expense of its **stability** and **reliability**. This introduce the problem which Site Reliability Engineering role aims to tackle. SRE focuses on monitoring production systems, ways to enhance its reliability and uptime using availability zones, scaling and automation. 

- - -

## DevOps Concepts

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*h5Zs-8nFcTrgR1UceyKYXA.png)

The traditional application release process includes Developers developing code and then pass the package to the Operations team *“over the wall”* to deploy it. This also includes a long process from checklists, documentations, application testing, security, and approvals which slows down the deployment. 

DevOps aim to solve this with the following concepts: 

### 1. Collaboration and Communication:
DevOps emphasises the importance of open communication and collaboration between development and operations teams. By sharing knowledge, understanding each other's challenges, and working together, teams can overcome bottlenecks and achieve common goals.

### 2. Automation:
Automation is at the heart of DevOps. It involves automating repetitive and manual tasks, such as code deployment, testing, and provisioning infrastructure. Automation accelerates processes, reduces human errors, and ensures consistency.

### 3. Continuous Integration (CI):
CI involves integrating code changes from multiple developers into a shared repository multiple times a day. Automated tests are run to validate changes, ensuring that the codebase remains stable and functional.

### 4. Continuous Delivery (CD):
CD builds on CI by automating the deployment process. It ensures that code changes can be rapidly and reliably deployed to production environments. Automated pipelines push code through various stages, from testing to production, while maintaining quality.

### 5. Infrastructure as Code (IaC):
IaC treats infrastructure provisioning and management as software development. It involves using code to define and configure infrastructure, enabling version control, repeatability, and consistency.

### 6. Monitoring and Feedback Loops:
DevOps encourages continuous monitoring of applications and infrastructure to gather data on performance, user experience, and other key metrics. Feedback loops provide insights into system behaviour, helping teams make informed decisions for improvements.

### 7. Security as Code (DevSecOps):
Security is integrated into the development process, known as "Security as Code." Security checks, tests, and configurations are automated, ensuring that security is not an afterthought but an integral part of the pipeline.

DevOps transforms the software development process by fostering a culture of collaboration, continuous improvement, and automation. By aligning development and operations teams and embracing these core concepts, organisations can deliver high-quality software more frequently with fewer errors and respond to market demands effectively.