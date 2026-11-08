+++
title = 'Saeed Afshari'
date = 2026-11-08
draft = false
layout = 'cv'
pdf = '/cv/Saeed_Afshari_CV.pdf'
photo = '/cv/saeed-afshari.jpg'
description = 'Staff Software Engineer · Distributed Systems · Agentic SDLC · Security & Platform Engineering — Munich, Germany · German citizen · saeed.afshari8@gmail.com · linkedin.com/in/saeed-afshari-22724a68'
+++

## Professional Summary

Staff software engineer with 15+ years of experience designing, building and operating distributed systems, financial workflows and enterprise security solutions. I work end to end: I sit with the people who own the problem, turn it into an architecture and a roadmap, and ship it to production, with AI agents built into how the work gets done.

I build the agentic software development lifecycle myself. I designed and shipped a catalogue of GitLab CI components in which Claude-powered agents review merge requests, resolve review feedback, fix security vulnerabilities test-first, vet dependency upgrades and implement Jira tickets from specifications. Each runs inside guardrails, with human approval where it matters. I also lead teams through large modernisations and teach engineers new skills, such as building their own Kafka-based solutions.

Daily hands-on experience running production workloads on Kubernetes, with Java, Kotlin, Python, TypeScript, AWS, GCP, Azure, Kafka and Terraform. I define SLIs, SLOs and error budgets and track them in Datadog.

## Core Expertise

* **Agentic engineering & AI-assisted SDLC:** Claude Code, AI code review, TDD-driven coding agents, guardrails, LLM integration, LangChain, agent-based systems
* **Architecture:** distributed systems, event-driven architecture, Domain-Driven Design, ADRs, technical strategy, legacy modernisation
* **Languages & frameworks:** Java, Kotlin, Python, Spring Boot, Angular, TypeScript
* **Cloud & platform:** AWS, Azure, GCP, Kubernetes, Docker, Terraform, GitLab CI/CD components
* **Data & messaging:** Kafka, Kafka Streams, RabbitMQ, PostgreSQL, MongoDB
* **Reliability:** observability, SLIs/SLOs/SLAs, error budgets, Datadog
* **Identity & security:** Microsoft Entra ID, OAuth 2.0, OpenID Connect, identity governance, SAST and vulnerability remediation

## Experience

### Staff Software Engineer — Flix
*Sep 2022 – Present · Munich, Germany*

**AI-assisted SDLC platform (flix-sdlc)**

* Created `flix-sdlc`, a versioned catalogue of GitLab CI components that puts AI agents into the delivery pipeline. Teams adopt a component with a few lines of configuration. Each component runs on either AWS Bedrock or the Claude CLI.
* **Code review agents** review merge request diffs and post inline findings. They apply configurable guardrails (clean code, architecture, security, testing, API design, SOLID, data integrity, error handling), which teams can override or share organisation-wide. Each run writes a metrics artifact with token usage, cost and finding counts.
* **Feedback resolver** works through open review discussions and decides per comment whether to implement the fix, push back with reasoning or ask for clarification. It can apply the fix, run the test suite and commit to the branch.
* **Security vulnerability resolver** turns open GitLab security findings into Jira tickets, then writes a failing test and a fix (red/green TDD) and opens a merge request. It deduplicates against existing MRs.
* **Dependency updater** runs after a passing Renovate pipeline. It files a Jira ticket with the upgrade's impact, analyses the codebase read-only, and marks the MR safe or breaking. It never approves or merges, and reports "breaking" when in doubt.
* **Coding agent** turns a Jira ticket plus a committed specification into an implementation plan, then into code, using TDD. It checks the baseline tests first and iterates on failing tests while the failure count keeps shrinking. It pushes the result and reports on both the Jira ticket and the MR. If the ticket is unclear, it asks clarifying questions on the ticket and writes no code. A Spring Boot toolchain image ships with it.
* **Dev-environment cleaner** removes per-branch feature environments once the Jira ticket is done. It is hard-limited to the dev namespace and starts as a dry run.

**Monolith modernisation: Partner Settlement**

* Led a team of engineers to modernise a 12-year-old Spring Boot monolith into a multi-project Gradle build with **10 bounded contexts**, covering the complexity of the Flix Partner Settlement business logic.
* Reviewed and learned the entire code base. Ran workshops to align the team on the problems with the existing solution: maintainability, extensibility, security vulnerabilities, and complexity hidden in twisted SQL stored procedures and virtual views.
* Extracted and documented the 10 bounded contexts and their execution flows, and wrote an ADR for each. Turned each context's challenges into a detailed software specification with a delivery roadmap, and wrote **350 Jira tickets** from that roadmap.

**Framework migration**

* Led a team of 5 engineers and a QA to migrate a legacy Spring Boot and Axon solution to the latest Spring Boot version, replacing Axon with Javers.

**Event-driven enablement**

* Led ERP engineers and trained them to design and ship their own software solutions that integrate with Apache Kafka.

**Identity and security**

* Led a company-wide Microsoft Entra governance initiative covering application identity, authentication and authorization standards. Defined organisation-wide guidelines and best practices.
* Designed reusable Terraform modules that let teams provision Entra applications and service principals with standard governance and security controls.
* Led identification and remediation of security risks across more than 2,000 enterprise applications. Found and fixed 1,957 orphaned Entra applications by establishing ownership with at least three accountable owners per application.

**Architecture and technical leadership**

* Provided technical leadership across multiple teams of Staff and Senior engineers on financial systems, security, platform engineering and data architecture, while staying hands-on in production code.
* Authored and drove adoption of ADRs on security strategy, platform modernisation, CI/CD standardisation, anomaly detection, data platform integration, taxation support and settlement architecture.
* Architected and delivered critical capabilities for the CommissionR and Settlr financial settlement platforms. Introduced consumer-driven contract testing across domain teams.
* Led architecture initiatives including Snowflake integration, anomaly detection, SAST adoption, taxation support and cloud migration strategy.
* Defined SLIs, SLOs, SLAs and error budgets for production products, tracked in Datadog. Owned deployment, configuration, observability and troubleshooting on Kubernetes.

### Senior Full-Stack Software Engineer — Flix
*Sep 2021 – Sep 2022*

* Designed and implemented the CommissionR settlement platform, delivering large-scale financial workflows and key architectural decisions.

### Senior Full-Stack Software Engineer — Flix
*Mar 2018 – Sep 2021*

* Developed the Flix Charter platform, led its migration from a monolith to microservices, and modernised the Angular architecture.

### Tech Lead — BRTel, Tehran
*Dec 2014 – Mar 2018*

* Led two Agile teams building a large-scale fan engagement platform. Defined the system architecture and technical roadmap.
* Designed distributed Java microservices integrated over REST and RabbitMQ, and standardised CI/CD on GitLab CI.
* Managed the VMware ESXi infrastructure, mentored engineers and ran architecture reviews.

### Senior Java Backend Developer — Rayvarz, Tehran
*Mar 2011 – Dec 2014*

* Built carrier-grade telecom network analysis platforms, including SS7 protocol analysers and packet and session components for SIP, RTP and SCTP.
* Developed a custom ORM that translates expression trees into optimised SQL.

### Software Consultant — TDMMO, Tehran
*Mar 2010 – Mar 2013*

* Worked with JICA experts and government stakeholders on a national earthquake early warning and disaster management platform: monitoring, simulation and reporting systems.

### Full-Stack C# Developer — Banicomm, Tehran
*Mar 2009 – Mar 2011*

* Built financial and telecom software on C# and .NET, including ISO 8583 payment message processing and SIP/RTP call-recording tools.

### PHP/Python Developer — Radfa, Tehran
*Jun 2004 – Mar 2009*

* Built e-commerce and business management platforms, Asterisk AGI telecom automation and XMPP-based CRM messaging integrations.

## Education

Bachelor's degree in Computer Engineering (Software), Payamenoor University

## Languages

English (advanced) · German (B2) · Persian (native) · Turkish (intermediate)

## Interests

Professional judoka · Reading · Movies · Hiking
