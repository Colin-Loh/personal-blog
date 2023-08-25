---
title: "SRE and DevOps"
date: 2023-08-25T16:57:15+10:00
# weight: 1
# aliases: ["/first"]
tags: ["DevOps"]
author: "Colin Loh"
# author: ["Me", "You"] # multiple authors
showToc: false
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
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
editPost:
    URL: "https://github.com/<path_to_repo>/content"
    Text: " " # edit text
    appendFilePath: true # to append file path to Edit link
---

## Introduction:

A few years back, when I was starting our my career as a Platform Engineer. I found myself being surrounded with buzzwords with one question that seems to often crosses everyone minds. 

“What is the difference between SRE and DevOps?”.

This questions actually led me down a rabbit hole to look for answers and understand more about **SRE** and **DevOps** and its concept with that in mind. 

### What is DevOps?
DevOps as the name suggests, a unification of “Development” and “Operations”. In both Amazon and Google definition, DevOps is a “set of **practices**, **guidelines** and **culture** to break down silo and ability to deliver application” 

The word DevOps emphasises collaboration between Developers and their Operation team. It is a different way of working where Devs and Ops work can be done in the same team to increase productivity and efficiency. 

“You build it, you run it” by Werner Vogels, Amazon CTO

### What is SRE? 
Site Reliability Engineering is a **role** with a set of practices that was introduced by Ben Treynor Sloss, Google Engineer. SRE is responsible to ensure production services is up and running, reliable and adhering to the terms SLO, SLA and SLI.

- SLO - Service Level Objective
  - Specific performance metrics and a threshold that should be met. An example would be SLO might state an uptime of 99.9% uptime, services should be available and measures such as availability zones.
- SLA - Service Level Agreement
  - Contractual framework that ensures accountability and transparency between service provider and the users. 
- SLI - Service Level Indicator 
  - Specific metrics such as response time, error rate, availability, latency that is relevant to the service’s performance. SRE can gain insight to the behaviour of the service and make informed decision.

Site Reliability Engineering adopts an investigative mindset to analyse and identify what went wrong and how to avoid them such as doing backups, testing, disaster recovery and budgets. 

### Key Differences?

```
class SRE implements DevOps
```

DevOps Is a Abstract Class. SRE is an human object who implements DevOps. 

DevOps is a culture, not a designated position which was introduced to break silos between the teams. SRE is a role that adopts DevOps practices. I like to view it as DevOps being a religion while SRE is a clergy. However, this is not the reality from the countless listing of DevOps role on LinkedIn. 

DevOps became a role but what is the difference with SRE? That’s a great question. 

|                   | **DevOps Engineer**                                          | **Site Reliability Engineer**                                |
|-------------------|--------------------------------------------------------------|--------------------------------------------------------------|
| Goals             | Faster software delivery, increased frequency of releases, and improved collaboration. | Ensuring service uptime, minimizing incidents, budgeting and automating operational tasks. |
| Responsibility    | Creating a streamlined release process to deploy code quickly using CI/CD tools and automation.  | Focuses on operational tasks, monitoring, incident response, and automation to enhance reliability. |
| Automation        | Automation of development, testing, and deployment processes (CI/CD). | Automation of operational tasks, incident management, and toil reduction. |
| Tools             | CI/CD pipelines, infrastructure as code (IaC), version control systems (Git). | Monitoring tools, observability solutions, configuration management, chaos testing tools. |
| Collaboration     | Collaboration between development and operations teams to break down silos. | Collaboration between sysadmins and developers to bridge the gap and ensure reliability. |
| Metrics           | Emphasizes speed of delivery, lead time, deployment frequency. | Focuses on reliability metrics such as SLO, SLA and SLI.     |
| Incident Response | Involvement in incident response, but not necessarily the primary focus. | Core focus on incident management, post-incident analysis, and learning from failures. |

The table above is my own take on understanding the difference between DevOps and SRE. From the table you can see a pattern. 

The reality is that DevOps Engineer are essentially infrastructure engineers that focuses on reducing silo with software development teams. This is achievable by integrating development process into workflows, automate and managing releases by setting up CI/CD pipelines. 

DevOps aims to deploy code quickly but at the expense of its stability and reliability. This introduce the need of Site Reliability Engineering. SRE focuses on monitoring production systems, ways to enhance its reliability and uptime using availability zones, scaling and automation. 

### DevOps Concept

The traditional application release process includes Developers developing code and then pass the package to the Operations team “over the wall” to deploy it. This also includes a long process from checklists, documentations, application testing, security, and approvals which slows down the deployment. 

DevOps aim to solve this with the following concepts: 

**1. Collaboration and Communication:**
DevOps emphasises the importance of open communication and collaboration between development and operations teams. By sharing knowledge, understanding each other's challenges, and working together, teams can overcome bottlenecks and achieve common goals.

**2. Automation:**
Automation is at the heart of DevOps. It involves automating repetitive and manual tasks, such as code deployment, testing, and provisioning infrastructure. Automation accelerates processes, reduces human errors, and ensures consistency.

**3. Continuous Integration (CI):**
CI involves integrating code changes from multiple developers into a shared repository multiple times a day. Automated tests are run to validate changes, ensuring that the codebase remains stable and functional.

**4. Continuous Delivery (CD):**
CD builds on CI by automating the deployment process. It ensures that code changes can be rapidly and reliably deployed to production environments. Automated pipelines push code through various stages, from testing to production, while maintaining quality.

**5. Infrastructure as Code (IaC):**
IaC treats infrastructure provisioning and management as software development. It involves using code to define and configure infrastructure, enabling version control, repeatability, and consistency.

**6. Monitoring and Feedback Loops:**
DevOps encourages continuous monitoring of applications and infrastructure to gather data on performance, user experience, and other key metrics. Feedback loops provide insights into system behavior, helping teams make informed decisions for improvements.

**7. Culture of Learning and Improvement:**
DevOps promotes a culture of continuous learning and improvement. Teams embrace failures as opportunities to learn, identify root causes, and implement preventive measures to avoid similar issues in the future.

**8. Security as Code:**
Security is integrated into the development process, known as "Security as Code." Security checks, tests, and configurations are automated, ensuring that security is not an afterthought but an integral part of the pipeline.

DevOps transforms the software development process by fostering a culture of collaboration, continuous improvement, and automation. By aligning development and operations teams and embracing these core concepts, organisations can deliver high-quality software more frequently, with fewer errors, and respond to market demands effectively.