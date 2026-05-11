# 🦅 Iota-Signa-Dev
**InfoSec Engineer | Agentic AI Architect | DevSecOps Lead**

> "Securing the evolution of autonomous systems through robust governance, immutable infrastructure, and high-performance engineering."

---

## 🎯 Profile Overview

I am a Cybersecurity specialist and Systems Engineer with over 15 years of experience building and securing complex IT ecosystems. My current focus lies at the intersection of **Agentic AI Governance** and **DevSecOps**, where I orchestrate hybrid environments (Humans + Autonomous Agents) with a strict "Security-First" approach.

- **🛡️ InfoSec & Hardening:** Top 5% Rank on **Hack The Box**. Expert in **Linux Kernel Hardening**, **RBAC/ACL** architecture, and **IDS/IPS** orchestration (CrowdSec, Fail2Ban).
- **🤖 Agentic AI Governance:** Specialist in designing secure, governed LLM-powered agents (LangChain, RAG) with robust audit trails.
- **🏗️ DevSecOps & Networking:** Architecting immutable CI/CD pipelines and complex networking resolutions (Netplan, systemd-networkd, Cloudflare Zero Trust).

---

## 🚀 Featured Project: OrderKill Ecosystem
**High-Performance B2B SaaS | Built in 33 Days**

OrderKill (v4.5.3 Golden State) is a multi-tenant platform for the retail and food industry, serving as a benchmark for modern, secure, and AI-augmented software development.

### Technical Highlights:
- **🔒 Advanced Multi-Tenancy:** Implemented **Row Level Security (RLS)** at the database level via Prisma extensions, ensuring cryptographic data isolation between tenants.
- **🧠 CoWorker AI Engine:** An autonomous agent layer powered by **LangChain** and **pgvector**, capable of real-time inventory management and contextual business logic.
- **🛠️ DevSecOps Pipeline:** Full "Shift-Left" security integration using **Semgrep** and **Gitleaks** within a **Turborepo** monorepo structure.
- **⚡ Modern Stack:** Built on **Next.js 16**, **React 19**, **Hono API**, and **PostgreSQL**, optimized for Edge compatibility and high concurrency.

---

## 🏗️ How I Implemented the Stack (Technical Strategy)

The **OrderKill** ecosystem was engineered for extreme scalability and security through a modular, automated approach:

- **Monorepo Orchestration:** Implemented a **Turborepo** structure to manage multiple high-concurrency apps (Customer Portal, Admin Dashboard, Edge API) while maintaining shared logic through internal packages (`@orderkill/security`, `@orderkill/db`).
- **Automated Multi-Tenancy:** Engineered a robust isolation layer using **Prisma Client Extensions**. This automatically injects `tenantId` context into every database operation, reinforced by **PostgreSQL Row Level Security (RLS)** for absolute data segregation.
- **Agentic AI Integration:** Built the "CoWorker" assistant using **LangChain** for complex tool-calling orchestration and **pgvector** for semantic search. This allows the AI to perform Retrieval-Augmented Generation (RAG) on specific tenant inventory and rules.
- **Shift-Left Security:** Hardened the development lifecycle by integrating **Semgrep** and **Gitleaks** directly into the CI pipeline, ensuring no vulnerability or secret reaches production.
- **Edge-Ready Performance:** Utilized **Hono** and **Jose** (JWT/JWE) to ensure the authentication and API layers are compatible with global Edge runtimes (Cloudflare Workers / Vercel Edge).

---

## 🔬 Technical Deep Dives & Security Hardening

Beyond standard development, I specialize in resolving complex architectural failures and implementing high-level security controls:

- **Advanced System Remediation:** Solved critical race conditions in **systemd** sockets and complex **NetworkManager/Netplan** integration issues to ensure 100% uptime for administrative interfaces.
- **IDS/IPS Orchestration:** Implemented and tuned **CrowdSec** security engines, developing custom detection scenarios for HTTP-CVEs and multi-port scanning defense.
- **Enterprise RBAC & ACLs:** Designed granular access control systems using **POSIX ACLs** and specialized **Sudoers** whitelisting to enforce the principle of least privilege in production environments.
- **Zero-Trust Perimeter:** Secured administrative panels (Cockpit, SSH) using **Cloudflare Zero Trust** tunnels, moving away from vulnerable public-facing ports.

---

## 🛠️ Core Technology Stack

| Domain | Tools & Technologies |
| :--- | :--- |
| **Artificial Intelligence** | LangChain, RAG Engines, Vector Databases (pgvector), Anthropic/OpenAI/Ollama |
| **Security & InfoSec** | IAM, JWT/JWE (Jose), Semgrep, Gitleaks, Ethical Hacking (HTB), Burp Suite, ZAP |
| **Infrastructure / DevOps** | Docker, Turborepo, PNPM, NGINX, Redis, PgBouncer, Terraform, Bash/Python |
| **Fullstack Development** | Next.js 16 (App Router), React 19, TypeScript 6.0, Prisma ORM, Hono, Zustand |
| **Compliance** | ISO/IEC 27001, NIST CSF, HIPAA, GDPR, CCPA |

---

## 📈 Professional Trajectory

My career is backed by global roles at industry leaders where I led critical IT security projects and infrastructure migrations:

- **General Motors:** Technical Security Consultant (IAM, Audit, Compliance).
- **Hewlett Packard / EDS:** System Analyst & IAM Specialist (SOX Compliance, UNIX/Linux Security).
- **Independent Consultant:** InfoSec Architect and Cybersecurity Advisor.

---

## 🤝 Let's Connect

I am open to collaborations in **Agentic AI Governance**, **DevSecOps automation**, and **Architectural Consulting**.

- **LinkedIn:** [Iota-Signa-Dev](https://www.linkedin.com/in/jose-i-carpanzano/)
- **Portfolio/Blog:** [iota-sigma.dev](https://iota-sigma.dev)
- **Contact:** Via GitHub / LinkedIn

---
*"Code mutates, architecture endures."*
