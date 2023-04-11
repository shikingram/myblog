---
title: "GitOps"   
author: "Kingram"  
date: 2023-04-11   
lastmod: 2022-04-11

tags: [  
"cloud-native",
]
---

# GitOps

## What is GitOps?   
GitOps is an operational framework that takes DevOps best practices used for application development such as version control, collaboration, compliance, and CI/CD, and applies them to infrastructure automation.

While much of the software development lifecycle has been automated, infrastructure has remained a largely manual process that requires specialized teams. With the demands made on today’s infrastructure, it has become increasingly crucial to implement infrastructure automation. Modern infrastructure needs to be elastic so that it can effectively manage cloud resources that are needed for continuous deployments.

Modern and cloud native applications are developed with speed and scale in mind. Organizations with a mature DevOps culture can deploy code to production hundreds of times per day. DevOps teams can accomplish this through development best practices such as version control, code review, and CI/CD pipelines that automate testing and deployments.

GitOps is used to automate the process of provisioning infrastructure, especially modern cloud infrastructure. Similar to how teams use application source code, **operations teams that adopt GitOps use configuration files stored as code (infrastructure as code). GitOps configuration files generate the same infrastructure environment every time it’s deployed**, just as application source code generates the same application binaries every time it’s built.

## How do team put GitOps into practice?   
GitOps is not a single product, plugin, or platform. There is no one-size-fits-all answer to this question, as the best way for teams to put GitOps into practice will vary depending on the specific needs and goals of the team. However, some tips on how to get started with GitOps include using a dedicated GitOps repository for all team members to share configurations and code, automating the deployment of code changes, and setting up alerts to notify the team when changes occur.

GitOps requires three core components:

**IaC:**   
GitOps uses a Git repository as the single source of truth for infrastructure definitions. Git is an open source version control system that tracks code management changes, and a Git repository is a .git folder in a project that tracks all changes made to files in a project over time. **Infrastructure as code (IaC)** is the practice of keeping all infrastructure configuration stored as code. The actual desired state may or may not be not stored as code (e.g., number of replicas or pods).

**MRs:**    
GitOps uses merge requests (MRs) or pull requests (PRs) as the change mechanism for all infrastructure updates. The MR or PR is where teams can collaborate via reviews and comments and where formal approvals take place. A merge commits to your main (or trunk) branch and serves as an audit log or audit trail.

**CI/CD:**    
GitOps automates infrastructure updates using a Git workflow with continuous integration and continuous delivery (CI/CD). **When new code is merged, the CI/CD pipeline enacts the change in the environment. Any configuration drift, such as manual changes or errors, is overwritten by GitOps automation so the environment converges on the desired state defined in Git**. GitLab uses CI/CD pipelines to manage and implement GitOps automation, but other forms of automation, such as definitions operators, can be used as well.

## GitOps challenges
With any collaborative effort, change can be tricky and GitOps is no exception. GitOps is a process change that will require discipline from all participants and a commitment to doing things in a new way. It is vital for teams to write everything down.

GitOps allows for greater collaboration, but that is not necessarily something that comes naturally for some individuals or organizations. A GitOps approval process means that developers make changes to the code, create a merge request, an approver merges these changes, and the change is deployed. This sequence introduces a “change by committee” element to infrastructure, which can seem tedious and time-consuming to engineers used to making quick, manual changes.

It is important for everyone on the team to record what’s going on in merge requests and issues. The temptation to edit something directly in production or change something manually is going to be difficult to suppress, but the less “cowboy engineering” there is, the better GitOps will work.

## GitOps benefits   
There are many benefits of GitOps, including **improved efficiency and security, a better developer experience, reduced costs, and faster deployments.**

With GitOps, **organizations can manage their entire infrastructure and application development lifecycle using a single, unified tool.** This allows for greater collaboration and coordination between teams and results in fewer errors and faster problem resolution.

In addition, GitOps can help organizations take advantage of containers and microservices and maintain consistency across all their infrastructure — from Kubernetes cluster configurations and Docker images to cloud instances and anything on-prem.

## What is the difference between GitOps and DevOps?    
There are a few key differences between GitOps and DevOps. For one, GitOps relies heavily on automation and tooling to manage and deploy code changes, while DevOps focuses more on communication and collaboration between teams. Additionally, **GitOps is typically used in conjunction with containerization technologies like Kubernetes, while DevOps can be used with any type of application.**

GitOps is a branch of DevOps that focuses on using Git code repositories to manage infrastructure and application code deployments. **The main difference between the two is that in GitOps, the Git repository is the source of truth for the deployment state, while in DevOps, it is the application or server configuration files.**

## Key components of a GitOps workflow   
There are four key components to a GitOps workflow, a Git repository, a continuous delivery (CD) pipeline, an application deployment tool, and a monitoring system.
- The Git repository is the source of truth for the application configuration and code.
- The CD pipeline is responsible for building, testing, and deploying the application.
- The deployment tool is used to manage the application resources in the target environment.
- The monitoring system tracks the application performance and provides feedback to the development team.

source:https://about.gitlab.com/topics/gitops/