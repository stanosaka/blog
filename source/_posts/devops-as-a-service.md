---
title: Devops As A Service
date: 2019-07-04 09:54:43
tags: 
  - devops
  - aws
  - git
  - azure
---
![Imgur](https://i.imgur.com/5ci7g2C.png)
Business Requirements
- Start with business requirements
- DevOps is about doing the work right
- What about doing the right work? See
   - (Agile) Portfolio Management- making sure the work you are doing is funded and delivering business value, and
   - Lean Control - making sure you get early engagement with control functions in your organisation (Audit, Compliance, Security, Architecture,
     Accessibility, Marketing, etc.)
- Work/Requirements are comprised of
   - features of business value (2-3 months), divided into ...
   - sprints (2-3 weeks) and the sprints are made up of ...
   - tasks (2-3 days of work)

Tasks
- 2-3 days of work
- Developers pull tasks off of a sprint queue
- Sprint goals to demo working software at the end of each sprint

Code
- Integrated continuously, Build continuously
- All code is reviewed by another team member before committing
- Feature branching or trunk-based development?
- No long lived code branches

Continuous Integration
- Code built continuously (multiple times per day)
- Fast feedback -continous builds are very fast (< 5mins)
- Best practice build patterns/chains
   - Compile, unit test, integration test, deploy artefacts

Metrics
- Code quality is vital
- Code coverage measures test automation
- Gold/silver/bronze accreditation

Artifacts
- Green builds produce shippable artefacts (.jar, .dll, .exe, docker image)
- Single store for all internal and external artefacts and libraries
- Security policies around 3rd party libraries and external access

Infrastructure as Code
- Operations roles change
- Infrastructure provisioning and configuration is automated
- Orchestration tools to provision infrastructure (Terraform, Cloud Formation for AWS)
- Configuration management tools to install and manage software on provisioned infrastructure (Chef, Puppet, Ansible)
- IaC is stored, tested and versioned in source code control
- Organisational Change, move to Site Reliability Engineering (SRE)

Service Mangement
- Approvals and change are automated
- Products with higher levels of accreditation have lower change management overheads (more automation)

Continous Deployment
- Infrastructure provisioned automatically
- Configuration automated
- Change approvals automated
- Push button deployment to production

Monitoring
- Observability driven design
- Monitoring, logging, dash boarding early in the life-cycle
- Issues and observations feed back to developers

Security
- "Shift Left" security
- "average total cost of a breach ranges from $2.2 million to $6.9 million"
- Code vulnerability scanning in the build pipeline
- Build fail if major/critical issues
- Tools- CheckMarx, Fortify
- Artefact scanning for security vulnerabilities
- Firewalls to protect against 3rd party vulnerabilities
- Tools - Nexus Lifecycle/Fiewall, BackDuck
- Image scanning dof Docker images
- Tools- AquaSec, Twistlock, Tennable, OpenSCAP

Evolving DevOps @ Scale

Shadow DevOps -> Enterprise DevOps -> DevOps as a Service

10 years age -> 5 years ago  -> the future

![Enterprise Devops](https://i.imgur.com/6Ei88Zz.jpg)
![Devops as a Service](https://i.imgur.com/jRJvY69.jpg)
![GitOps](https://i.imgur.com/kQNGOfS.jpg)
![AWS DevOps](https://i.imgur.com/Vvmeafo.jpg)
![Azure DevOps](https://i.imgur.com/lwpUXVf.jpg)
