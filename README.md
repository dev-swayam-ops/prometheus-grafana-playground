# Prometheus & Grafana Learning Repository 📊

A comprehensive, hands-on training program for mastering Prometheus and Grafana monitoring stack. This repository contains **60+ learning files** organized into **14 progressive modules** - from basics to production-ready monitoring architecture.

## 🎯 What You'll Learn

Master the complete monitoring ecosystem:
- **Prometheus**: Time-series database, scraping, querying, alerting
- **Grafana**: Dashboards, visualization, alerting, multi-datasource
- **AlertManager**: Alert routing, grouping, notifications
- **Kubernetes Monitoring**: Pod/node metrics, kube-state-metrics, Helm
- **Best Practices**: SLOs, cardinality management, incident response
- **Real-world Scenarios**: Multi-tier apps, incident response workflows

## 📚 Course Structure

### Learning Path: 14 Progressive Modules

#### **Foundation (Modules 00-02)**
Get Prometheus and Grafana running locally with hands-on labs.

| Module | Topic | Files | Focus |
|--------|-------|-------|-------|
| **00** | Setup & Prerequisites | 4 | Docker, Git, system requirements |
| **01** | Prometheus Basics | 4 | Architecture, metrics, TSDB concepts |
| **02** | Metrics & Exporters | 4 | Node Exporter, metric types, collection |

#### **Core Skills (Modules 03-05)**
Learn querying, alerting, and notification systems.

| Module | Topic | Files | Focus |
|--------|-------|-------|-------|
| **03** | PromQL Queries | 4 | Query language, operators, functions |
| **04** | Recording Rules & Alerting | 4 | Pre-computed rules, alert states |
| **05** | AlertManager & Notifications | 4 | Routing, grouping, multiple channels |

#### **Visualization (Modules 06-07)**
Master Grafana dashboards and panels.

| Module | Topic | Files | Focus |
|--------|-------|-------|-------|
| **06** | Grafana Basics | 4 | UI, data sources, first dashboard |
| **07** | Dashboards & Panels | 4 | Panel types, templating, variables |

#### **Advanced Grafana (Modules 08-09)**
Grafana alerting and Kubernetes-specific monitoring.

| Module | Topic | Files | Focus |
|--------|-------|-------|-------|
| **08** | Grafana Alerting | 4 | Alert rules, conditions, notifications |
| **09** | Kubernetes Monitoring | 4 | Helm, kube-state-metrics, kubectl |

#### **Architecture (Modules 10-11)**
Service discovery and reliability metrics.

| Module | Topic | Files | Focus |
|--------|-------|-------|-------|
| **10** | Service Discovery & Scraping | 4 | Static/DNS/file-based discovery |
| **11** | SLO/SLI & Error Budgets | 4 | Reliability targets, burn rates |

#### **Production Ready (Modules 12-14)**
Best practices, troubleshooting, and real-world scenarios.

| Module | Topic | Files | Focus |
|--------|-------|-------|-------|
| **12** | Observability Best Practices | 4 | Naming conventions, cardinality, runbooks |
| **13** | Troubleshooting & Debugging | 4 | Common issues, log analysis, incident response |
| **14** | Real-world Monitoring Labs | 4 | Multi-tier app, SLO implementation, incident sims |

## 📖 What's in Each Module?

Every module includes **4 learning files**:

### 1️⃣ **README.md** - Theory & Hands-on Lab
- What You'll Learn
- Prerequisites
- Key Concepts (with examples)
- Hands-on Lab with step-by-step commands
- Expected outputs
- Validation checklist
- Common mistakes & fixes
- Troubleshooting guide

**Example**: Module 03 README teaches PromQL with:
- Operator types (arithmetic, comparison, logical)
- Aggregation functions (sum, avg, rate, histogram_quantile)
- Template variables for dashboards
- Time range syntax
- Complete lab: 6 steps with curl examples

### 2️⃣ **exercises.md** - Progressive Practice
- 10 exercises (Easy → Medium difficulty)
- Hands-on tasks matching module content
- Clear validation criteria
- Builds skills progressively

**Example**: Module 10 exercises:
- E1 (Easy): View Prometheus targets
- E5 (Medium): Set scrape intervals
- E10 (Medium): Dynamic target lists

### 3️⃣ **solutions.md** - Complete Answers
- Full solutions for all 10 exercises
- Actual commands to run
- Expected outputs shown
- Brief explanations of why/how

**Example**: Module 04 solutions:
```promql
# Solution 5: Create recording rule
- record: job:request_rate:5m
  expr: sum(rate(http_requests_total[5m])) by (job)
```

### 4️⃣ **cheatsheet.md** - Quick Reference
- Command tables (Command | Purpose | Example)
- Syntax reference
- Common patterns
- Best practices
- Troubleshooting quick fixes

**Example**: Module 06 cheatsheet includes:
- Grafana API endpoints
- Default credentials
- Environment variables
- Common queries
- Dashboard JSON examples

## 🚀 How to Use This Repository

### Option 1: Linear Learning (Recommended for Beginners)
```
Start with Module 00 → Follow modules in order → Complete Module 14
Time: 40-50 hours of hands-on learning
```

### Option 2: Topic-Based Learning (For Experienced Users)
```
Jump to specific modules:
- "I know Prometheus, teach me Grafana" → Modules 06-08
- "I need Kubernetes monitoring" → Module 09
- "I need production readiness" → Modules 12-14
```

### Option 3: Just the Cheatsheets (Quick Reference)
```
Access module/cheatsheet.md for any topic
No prerequisites needed
Great for refresher/lookup
```

## 📋 Quick Start

### 1. Clone Repository
```bash
git clone https://github.com/YOUR_REPO/prometheus-grafana-playground.git
cd prometheus-grafana-playground
```

### 2. Start with Module 00
```bash
cd 00-setup-and-prerequisites
# Read README.md for system requirements
# Follow the exercises
```

### 3. Progress Through Modules
Each module is self-contained:
```bash
cd 01-prometheus-basics
cat README.md        # Learn theory & run hands-on lab
cat exercises.md     # Do 10 exercises
cat solutions.md     # Check your work
cat cheatsheet.md    # Quick reference
```

### 4. Hands-on Labs
```bash
# Most modules include docker-compose
docker-compose up -d
# Follow lab steps in README.md
docker-compose down
```

## 🎓 Learning Outcomes by Module

### After Module 01
✅ Understand Prometheus architecture
✅ Know how TSDB works
✅ Run Prometheus locally in Docker
✅ Query basic metrics

### After Module 03
✅ Write complex PromQL queries
✅ Understand aggregations and functions
✅ Use histogram_quantile for latency
✅ Create dashboard queries

### After Module 06
✅ Navigate Grafana UI
✅ Add data sources
✅ Create your first dashboard
✅ Work with multiple visualizations

### After Module 11
✅ Define SLOs for services
✅ Calculate error budgets
✅ Setup burn rate alerts
✅ Track SLO compliance

### After Module 13
✅ Debug common issues
✅ Troubleshoot missing metrics
✅ Analyze performance problems
✅ Follow incident response playbook

### After Module 14
✅ Monitor multi-tier applications
✅ Implement role-based dashboards
✅ Run incident simulations
✅ Design production monitoring

## 💡 Key Features

### 🎯 Practical Focus
- Every concept has hands-on exercises
- Real docker-compose examples
- Actual curl/PromQL commands
- Expected outputs shown

### 📚 Progressive Difficulty
- Module 00-02: Basics (run services)
- Module 03-07: Core skills (query, alert, visualize)
- Module 08-11: Advanced (Kubernetes, SLOs)
- Module 12-14: Production (best practices, incident response)

### 🔄 Multi-Format Learning
- **README**: Deep dive with theory
- **Exercises**: Hands-on practice
- **Solutions**: Verify your work
- **Cheatsheet**: Quick lookup

### 👥 Multiple Roles Covered
- **Beginners**: Start with Module 00
- **SREs**: Focus on Modules 11-14
- **Developers**: Modules 03-08
- **DevOps**: All modules in order

## 📊 Repository Statistics

```
Total Modules:        14
Total Files:          60 (4 per module)
Learning Hours:       40-50 (estimated)
Exercises:            140 (10 per module)
Code Examples:        500+
Commands Shown:       1000+
Docker Configs:       20+
PromQL Queries:       200+
Alert Rules:          50+
Grafana Dashboards:   30+
```

## 🛠️ Tech Stack Covered

| Technology | Modules | Purpose |
|-----------|---------|---------|
| **Prometheus** | 01, 03, 04, 10 | Metrics, querying, alerting |
| **Grafana** | 06, 07, 08 | Visualization, dashboards |
| **AlertManager** | 05 | Alert routing, notifications |
| **Kubernetes** | 09 | Pod/node monitoring |
| **Docker** | All | Lab environment |
| **PromQL** | 03, 04, 11, 12 | Querying language |
| **YAML** | 04, 05, 08 | Configuration |

## 📝 Example Learning Journey

**Day 1-2: Foundations**
```
Module 00: Setup (2 hours)
Module 01: Prometheus Basics (3 hours)
Module 02: Metrics & Exporters (3 hours)
```

**Day 3-4: Core Skills**
```
Module 03: PromQL (4 hours)
Module 04: Recording Rules & Alerting (4 hours)
Module 05: AlertManager (3 hours)
```

**Day 5-6: Visualization**
```
Module 06: Grafana Basics (3 hours)
Module 07: Dashboards & Panels (4 hours)
```

**Day 7-8: Advanced**
```
Module 08: Grafana Alerting (3 hours)
Module 09: Kubernetes (4 hours)
```

**Day 9-10: Production Ready**
```
Module 10: Service Discovery (3 hours)
Module 11: SLOs & Error Budgets (4 hours)
Module 12: Best Practices (3 hours)
Module 13: Troubleshooting (3 hours)
Module 14: Real-world Labs (5 hours)
```

## ✅ Prerequisites

### System Requirements
- Linux, macOS, or Windows (WSL2)
- 8GB RAM minimum
- 20GB disk space
- Internet connection

### Software Required
- Docker & Docker Compose
- Git
- curl (for API testing)
- Browser (Chrome/Firefox)

### Knowledge Prerequisites
- Basic Linux/command line
- YAML syntax familiarity
- Basic understanding of HTTP/networking
- No prior Prometheus/Grafana experience needed!

## 🤝 How to Contribute

Found an issue or want to improve?
```
1. Create an issue describing the problem
2. Fork the repository
3. Make changes to relevant module files
4. Ensure consistency with other modules
5. Submit a pull request
```

## 📞 Getting Help

### Need Help?
- Check the module's **Troubleshooting** section
- Review the **Cheatsheet** for common commands
- Look at **Solutions** for worked examples
- Check **Common Mistakes** section

### Want to Go Deeper?
- Official docs: https://prometheus.io/docs
- Grafana docs: https://grafana.com/docs
- Kubernetes monitoring: https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/

## 📄 License

This repository is open source and available under the MIT License.

## 🎉 What You'll Build

By completing this course, you'll be able to:

✅ Deploy Prometheus and Grafana in production
✅ Write complex PromQL queries and aggregations
✅ Create dashboards for different audiences
✅ Setup comprehensive alerting systems
✅ Monitor Kubernetes clusters
✅ Implement SLOs and error budgets
✅ Respond to incidents following playbooks
✅ Troubleshoot common monitoring issues
✅ Optimize cardinality and performance
✅ Design observability for complex systems

## 🚀 Start Learning Now

```bash
# Clone the repo
git clone https://github.com/YOUR_REPO/prometheus-grafana-playground.git

# Navigate to first module
cd 00-setup-and-prerequisites

# Read and follow the README
cat README.md

# Start hands-on exercises
cat exercises.md
```

---

**Happy Learning! 🎓**

Start with [Module 00: Setup and Prerequisites](./00-setup-and-prerequisites/README.md)

For questions, check the [Module 13 Troubleshooting Guide](./13-troubleshooting-and-debugging/README.md)
