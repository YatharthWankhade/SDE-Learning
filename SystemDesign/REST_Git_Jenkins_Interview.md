# REST Web Services, Git & Jenkins – Interview Notes (IBKR JD)

## Table of Contents
1. [REST API Principles](#1-rest-api-principles)
2. [HTTP Methods & Status Codes](#2-http-methods--status-codes)
3. [REST API Design Best Practices](#3-rest-api-design-best-practices)
4. [Authentication & Security](#4-authentication--security)
5. [REST Client in Java](#5-rest-client-in-java)
6. [Git – Version Control](#6-git--version-control)
7. [Jenkins – CI/CD](#7-jenkins--cicd)
8. [Common Interview Questions](#8-common-interview-questions)

---

## 1. REST API Principles

### REST Constraints (Richardson Maturity Model)
| Level | Description |
|---|---|
| **0** | Plain HTTP (no REST, just RPC over HTTP) |
| **1** | Resources (unique URIs per resource) |
| **2** | HTTP Methods correctly used (GET, POST, PUT, DELETE) |
| **3** | HATEOAS (links in response for discoverability) |

### REST vs SOAP
| | REST | SOAP |
|---|---|---|
| Protocol | HTTP | HTTP, SMTP, TCP |
| Format | JSON / XML | XML only |
| Style | Stateless, resource-oriented | Strict standard (WSDL) |
| Performance | Faster, lightweight | Overhead of XML envelope |
| Use Case | Web/Mobile APIs | Enterprise (banking, legacy) |

> **IBKR context:** IBKR's own Client Portal API is REST-based; integrations with legacy systems may use SOAP

---

## 2. HTTP Methods & Status Codes

### Methods (Safe & Idempotent)
| Method | Purpose | Safe? | Idempotent? |
|---|---|---|---|
| `GET` | Read resource | ✓ | ✓ |
| `POST` | Create resource | ✗ | ✗ |
| `PUT` | Replace resource | ✗ | ✓ |
| `PATCH` | Partial update | ✗ | ✗ |
| `DELETE` | Remove resource | ✗ | ✓ |
| `HEAD` | GET without body | ✓ | ✓ |
| `OPTIONS` | What methods supported | ✓ | ✓ |

### Status Codes
| Code | Meaning | Common Use |
|---|---|---|
| 200 | OK | Successful GET, PUT |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Validation error |
| 401 | Unauthorized | No/invalid credentials |
| 403 | Forbidden | Authenticated but no permission |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Duplicate resource |
| 422 | Unprocessable Entity | Semantic validation error |
| 429 | Too Many Requests | Rate limiting |
| 500 | Internal Server Error | Server bug |
| 503 | Service Unavailable | Server overloaded / down |

---

## 3. REST API Design Best Practices

### URL Design
```
# Nouns, not verbs (method IS the verb)
GET    /api/v1/instruments          # list all instruments
GET    /api/v1/instruments/AAPL     # get specific instrument
POST   /api/v1/instruments          # create new instrument
PUT    /api/v1/instruments/AAPL     # replace instrument
PATCH  /api/v1/instruments/AAPL     # partial update
DELETE /api/v1/instruments/AAPL     # delete instrument

# Nested resources for relationships
GET /api/v1/accounts/12345/positions      # positions for account 12345
GET /api/v1/accounts/12345/trades?date=2024-03-10  # filtered by date

# Bad patterns to AVOID
GET /api/getInstrument?symbol=AAPL   # verb in URL, query param for ID
POST /api/deleteInstrument           # wrong method
```

### Pagination (essential for large IBKR datasets)
```json
GET /api/v1/trades?page=2&size=100&sort=tradeDate,desc

Response:
{
  "data": [...],
  "pagination": {
    "page": 2,
    "size": 100,
    "totalElements": 15243,
    "totalPages": 153,
    "hasNext": true
  }
}
```

### Versioning Strategies
```
URI versioning:     /api/v1/trades  (most common, visible)
Header versioning:  Accept: application/vnd.ibkr.v2+json
Query param:        /api/trades?version=2
```

### Response Format (standard)
```json
{
  "status": "success",
  "data": {
    "tradeId": "T-001",
    "symbol": "AAPL",
    "quantity": 100,
    "price": 175.50,
    "timestamp": "2024-03-10T09:30:00Z"
  },
  "errors": null
}

// Error response
{
  "status": "error",
  "data": null,
  "errors": [
    {"code": "INVALID_SYMBOL", "message": "Symbol 'XXX' not found", "field": "symbol"}
  ]
}
```

---

## 4. Authentication & Security

### Bearer Token (JWT)
```bash
# Login
curl -X POST /api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"trader1","password":"secret"}'

# Response: {"token": "eyJhbGciOiJIUzI1NiJ9..."}

# Use token
curl /api/v1/accounts/12345 \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiJ9..."
```

### JWT Structure
```
Header.Payload.Signature
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0cmFkZXIxIiwiZXhwIjoxNzA...

Header:  {"alg":"HS256","typ":"JWT"}
Payload: {"sub":"trader1","roles":["TRADE","VIEW"],"exp":1710000000,"iat":1709996400}
Signature: HMAC_SHA256(base64(header)+"."+base64(payload), secret)
```

### Rate Limiting
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 950
X-RateLimit-Reset: 1710000000
Retry-After: 60   (if 429)
```

---

## 5. REST Client in Java

### HttpClient (Java 11+)
```java
HttpClient client = HttpClient.newHttpClient();

// GET request
HttpRequest getReq = HttpRequest.newBuilder()
    .uri(URI.create("https://api.ibkr.com/v1/portal/account/summary"))
    .header("Authorization", "Bearer " + token)
    .timeout(Duration.ofSeconds(10))
    .GET()
    .build();

HttpResponse<String> response = client.send(getReq, HttpResponse.BodyHandlers.ofString());
if (response.statusCode() == 200) {
    ObjectMapper mapper = new ObjectMapper();
    AccountSummary summary = mapper.readValue(response.body(), AccountSummary.class);
}

// POST request
String payload = """
    {"symbol": "AAPL", "quantity": 100, "side": "BUY", "orderType": "MARKET"}
    """;
HttpRequest postReq = HttpRequest.newBuilder()
    .uri(URI.create("https://api.ibkr.com/v1/portal/iserver/order"))
    .header("Content-Type", "application/json")
    .header("Authorization", "Bearer " + token)
    .POST(HttpRequest.BodyPublishers.ofString(payload))
    .build();

HttpResponse<String> postResp = client.send(postReq, HttpResponse.BodyHandlers.ofString());
```

---

## 6. Git – Version Control

### Core Concepts
```
Working Directory → Staging Area → Local Repo → Remote Repo
    git add          git add       git commit    git push
                                                 git pull (fetch + merge)
```

### Essential Commands
```bash
# Setup
git config --global user.name "Yatharth Wankhade"
git config --global user.email "yatharth@ibkr.com"

# Start
git init                          # new repo
git clone https://github.com/org/repo.git

# Staging and committing
git status                        # see what changed
git add file.java                 # stage specific file
git add -p                        # interactive stage (review hunks)
git commit -m "IBKR-1234: Add Oracle ref data loader"

# Branches
git branch feature/trade-loader   # create branch
git checkout -b feature/trade-loader  # create and switch
git switch feature/trade-loader   # switch (modern)
git merge feature/trade-loader    # merge into current
git rebase main                   # rebase onto main (cleaner history)

# Remote
git remote -v                     # list remotes
git fetch origin                  # download without merging
git pull origin main              # fetch + merge
git push origin feature/trade-loader

# Log & Diff
git log --oneline --graph --all   # pretty branch graph
git diff HEAD~1 HEAD              # last commit diff
git show <commit-hash>            # show specific commit

# Undo
git restore file.java             # discard working dir changes
git restore --staged file.java    # unstage
git revert <commit>               # create reverse commit (safe for shared branches)
git reset --hard HEAD~1           # DANGER: discard last commit (local only)
```

### Git Workflows
```
# GitFlow (common in enterprise like IBKR)
main       → production-ready code
develop    → integration branch
feature/*  → new features (branch from develop)
release/*  → release prep
hotfix/*   → urgent prod fixes (branch from main)

# Trunk-Based Development (modern CI/CD)
main       → always deployable
feature/*  → short-lived; merged quickly via PR
```

### Conflict Resolution
```bash
git merge feature/order-processor
# CONFLICT: auto-merge failed in OrderLoader.java
# Open file → look for <<<<<<< HEAD, =======, >>>>>>> sections
# Edit file to resolve
git add OrderLoader.java
git commit -m "Resolve merge conflict in OrderLoader"
```

### Advanced Git
```bash
# Stash (save work in progress)
git stash                         # save current changes
git stash pop                     # restore latest stash
git stash list                    # list stashes

# Cherry-pick (apply specific commit to another branch)
git cherry-pick abc1234            # apply commit to current branch

# Bisect (find bug-introducing commit)
git bisect start
git bisect bad HEAD               # current is broken
git bisect good v1.2.0            # this was working
# git bisect runs binary search, you mark each as good/bad

# Tags (release markers)
git tag -a v2.3.0 -m "Release 2.3.0"
git push origin v2.3.0
```

---

## 7. Jenkins – CI/CD

### Core Concepts
```
Pipeline → series of stages executed to build/test/deploy
Stage   → logical step (e.g., Build, Test, Deploy)
Step    → single task within a stage (sh, bat, echo)
Node/Agent → machine where pipeline runs
```

### Declarative Pipeline (Jenkinsfile)
```groovy
pipeline {
    agent { label 'java-11' }     // run on agent with Java 11

    environment {
        ORACLE_URL  = credentials('oracle-prod-url')
        JAVA_OPTS   = '-Xmx2g'
        APP_VERSION = "${GIT_COMMIT[0..7]}"
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()   // prevent parallel builds on same branch
    }

    triggers {
        cron('H 2 * * 1-5')        // nightly build on weekdays
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                    jacoco(execPattern: 'target/jacoco.exec')
                }
            }
        }

        stage('Integration Tests') {
            when { branch 'develop' }
            steps {
                sh 'mvn verify -P integration-test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Deploy to UAT') {
            when { branch 'develop' }
            steps {
                sh '''
                    ssh deploy@uat-server "
                        cd /opt/ibkr/app &&
                        java -jar trade-loader-${APP_VERSION}.jar --env=uat
                    "
                '''
            }
        }

        stage('Deploy to Production') {
            when { branch 'main' }
            input {
                message "Deploy to Production?"
                ok "Deploy"
                submitter "team-leads"
            }
            steps {
                sh 'ansible-playbook deploy-prod.yml -e version=${APP_VERSION}'
            }
        }
    }

    post {
        failure {
            emailext to: 'team@ibkr.com',
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build failed. Check: ${env.BUILD_URL}"
            slackSend channel: '#build-alerts', color: 'danger',
                message: "Build FAILED: ${env.JOB_NAME}"
        }
        success {
            slackSend channel: '#deployments', color: 'good',
                message: "Deployed: ${env.JOB_NAME} v${APP_VERSION}"
        }
    }
}
```

### Key Jenkins Plugins
| Plugin | Purpose |
|---|---|
| Maven Integration | Java builds |
| Git Plugin | SCM checkout |
| SonarQube Scanner | Code quality |
| JUnit | Test reports |
| JaCoCo | Code coverage |
| Credentials Binding | Secure secrets injection |
| Pipeline | Declarative/Scripted pipelines |
| Slack Notification | Team notifications |

---

## 8. Common Interview Questions

### REST
| Question | Answer |
|---|---|
| **REST vs RPC?** | REST: resource-oriented, stateless; RPC: function call over network (gRPC, SOAP) |
| **What is idempotency?** | Same request multiple times = same result; PUT/DELETE are idempotent, POST is not |
| **What is statelessness?** | Server stores no session; each request has all info needed; enables horizontal scaling |
| **How to design for high traffic?** | Rate limiting, caching (Redis), pagination, async processing, CDN for static resources |

### Git
| Question | Answer |
|---|---|
| **`git merge` vs `git rebase`?** | Merge: creates merge commit, preserves history; Rebase: replays commits, linear history |
| **`git fetch` vs `git pull`?** | Fetch: downloads remote changes without merging; Pull: fetch + merge |
| **What is a pull request / merge request?** | Code review mechanism before merging feature branch into main |
| **How to undo a pushed commit?** | `git revert <sha>` (safe, adds reverse commit); never `reset --hard` on shared branches |

### Jenkins
| Question | Answer |
|---|---|
| **What is CI/CD?** | CI: auto build+test on every commit; CD: auto deploy to staging/prod |
| **What is a Jenkinsfile?** | Pipeline-as-code stored in repo alongside source code |
| **How to manage secrets in Jenkins?** | Jenkins Credentials store + `withCredentials` or `credentials()` binding; never hard-code |
| **What is a parallel stage?** | Multiple stages running simultaneously in same pipeline to save time |
