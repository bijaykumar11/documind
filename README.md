<h1 align="center">DocuMind - AI Document Q&A Service</h1>

<p align="center">
  <a href="https://openjdk.org/"><img src="https://img.shields.io/badge/Java_21-5382A1?style=flat&logo=openjdk&logoColor=white" alt="Java" /></a>
  <a href="https://spring.io/projects/spring-boot"><img src="https://img.shields.io/badge/Spring_Boot_3.2-6DB33F?style=flat&logo=springboot&logoColor=white" alt="Spring Boot" /></a>
  <a href="https://reactjs.org/"><img src="https://img.shields.io/badge/React_18-61DAFB?style=flat&logo=react&logoColor=black" alt="React" /></a>
  <a href="https://www.typescriptlang.org/"><img src="https://img.shields.io/badge/TypeScript-3178C6?style=flat&logo=typescript&logoColor=white" alt="TypeScript" /></a>
  <a href="https://aws.amazon.com/cdk/"><img src="https://img.shields.io/badge/AWS_CDK-232F3E?style=flat&logo=amazonaws&logoColor=white" alt="AWS CDK" /></a>
  <a href="https://docs.langchain4j.dev/"><img src="https://img.shields.io/badge/LangChain4j_0.35-1C3C3C?style=flat&logo=langchain&logoColor=white" alt="LangChain4j" /></a>
  <a href="https://www.postgresql.org/"><img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=flat&logo=postgresql&logoColor=white" alt="PostgreSQL" /></a>
  <a href="https://www.docker.com/"><img src="https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white" alt="Docker" /></a>
</p>

## Overview

Upload documents and ask questions — DocuMind uses RAG (Retrieval-Augmented Generation) to provide accurate, context-aware answers from your files.

A demo MVP showcasing Java, TypeScript, AWS CDK, LangChain4j, Spring Boot, and React.

**Why?** To demonstrate a production-ready AI application with modern infrastructure. Useful for teams needing quick answers from documentation, research papers, or internal knowledge bases.

*Built in Boston, USA — January 2026*

## Table of Contents

- [Live Demo](#-live-demo)
- [Screenshot](#screenshot)
- [Features](#features)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Local Development](#local-development)
- [Environment Variables](#environment-variables)
- [AWS Deployment](#aws-deployment)
- [Documentation](docs/)
  - [Architecture Deep Dive](docs/ARCHITECTURE.md)
  - [Deployment & Teardown Guide](docs/DEPLOYMENT.md)
- [License](#license)

## Live Demo

| Resource | URL |
|----------|-----|
| **Frontend** | [https://d3py6xai9gspab.cloudfront.net](https://d3py6xai9gspab.cloudfront.net) |
| **API** | [http://DocuMi-Backe-Mnku72pk6qQ2-534965933.us-east-1.elb.amazonaws.com](http://DocuMi-Backe-Mnku72pk6qQ2-534965933.us-east-1.elb.amazonaws.com) |

## Screenshot

![DocuMind Upload](docs/Upload.png)
![DocuMind Question & Answer](docs/Chat.png)

*Upload a document, ask questions, and get AI-powered answers with source citations.*

## Features

- PDF/Text document upload
- Automatic text extraction and chunking
- Vector embeddings with pgvector
- RAG-based Q&A with LangChain4j
- Conversation history
- Modern React UI

## Architecture

```
┌─────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  React Frontend │────▶│  Spring Boot API │────▶│  LangChain4j    │
│  (TypeScript)   │     │  (Java 21)       │     │  + OpenAI/Local │
└─────────────────┘     └──────────────────┘     └─────────────────┘
                               │
                    ┌──────────┴──────────┐
                    ▼                     ▼
              ┌──────────┐         ┌──────────────┐
              │    S3    │         │  PostgreSQL  │
              │  (Docs)  │         │  (pgvector)  │
              └──────────┘         └──────────────┘
```

### AI Architecture (RAG Pipeline)

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              DOCUMENT INGESTION                                     │
│                                                                                     │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────┐    ┌───────────────────────┐  │
│  │  Upload  │───▶│  Parse PDF/  │───▶│   Chunk      │───▶│  Generate Embeddings  │  │
│  │  Document│    │  Text Files  │    │   (1500 char)│    │  (AllMiniLmL6V2)      │  │
│  └──────────┘    └──────────────┘    └──────────────┘    └───────────┬───────────┘  │
│                                                                      │              │
│                                                                      ▼              │
│                                                          ┌───────────────────────┐  │
│                                                          │  Store in pgvector    │  │
│                                                          │  (PostgreSQL)         │  │
│                                                          └───────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              QUESTION ANSWERING                                     │
│                                                                                     │
│  ┌──────────┐    ┌──────────────┐    ┌──────────────┐    ┌───────────────────────┐  │
│  │  User    │───▶│  Embed       │───▶│  Vector      │───▶│  Retrieve Top 5       │  │
│  │  Question│    │  Question    │    │  Similarity  │    │  Relevant Chunks      │  │
│  └──────────┘    └──────────────┘    └──────────────┘    └───────────┬───────────┘  │
│                                                                      │              │
│                                                                      ▼              │
│  ┌──────────┐    ┌──────────────────────────────────────────────────────────────┐   │
│  │  Answer  │◀───│  OpenAI GPT-4o-mini generates answer with context + sources  │   │
│  └──────────┘    └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### AWS Infrastructure

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                    USERS                                            │
└─────────────────────────────────────┬───────────────────────────────────────────────┘
                                      │ HTTPS
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                              CLOUDFRONT CDN                                         │
│                    https://d3py6xai9gspab.cloudfront.net                            │
│                                                                                     │
│  ┌─────────────────┐                              ┌─────────────────┐               │
│  │   Static Assets │◄─────────────────────────────┤   API Routes    │               │
│  │   (/, /*, etc.) │                              │   (/api/*)      │               │
│  └─────────────────┘                              └─────────────────┘               │
└─────────────┬───────────────────────────────────────────────┬───────────────────────┘
              │                                               │
              ▼                                               ▼
┌─────────────────────────────────────┐    ┌─────────────────────────────────────────┐
│            S3 BUCKET                │    │        APPLICATION LOAD BALANCER        │
│   documind-frontend-884710187119    │    │              (Port 80)                  │
│         (Frontend Assets)           │    └─────────────┬───────────────────────────┘
└─────────────────────────────────────┘                  │ HTTP
                                                         ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                                    VPC                                              │
│                          (us-east-1a & us-east-1b)                                  │
│                                                                                     │
│  ┌─────────────────────────────────────────────────────────────────────────────┐    │
│  │                           PUBLIC SUBNETS                                    │    │
│  │  ┌─────────────────┐                    ┌─────────────────┐                 │    │
│  │  │ Internet Gateway│                    │   NAT Gateway   │                 │    │
│  │  └─────────────────┘                    └─────────────────┘                 │    │
│  └─────────────────────────────────────────────────────────────────────────────┘    │
│                                           │ Outbound Internet                       │
│                                           ▼                                         │
│  ┌─────────────────────────────────────────────────────────────────────────────┐    │
│  │                          PRIVATE SUBNETS                                    │    │
│  │                                                                             │    │
│  │  ┌─────────────────────────────────────────────────────────────────────┐    │    │
│  │  │                    ECS FARGATE CLUSTER                              │    │    │
│  │  │                                                                     │    │    │
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                  │    │    │
│  │  │  │ ECS Task    │  │ ECS Task    │  │ ECS Task    │                  │    │    │
│  │  │  │documind-api │  │documind-api │  │documind-api │                  │    │    │
│  │  │  └─────────────┘  └─────────────┘  └─────────────┘                  │    │    │
│  │  │              Auto Scaling: 50% - 200% capacity                      │    │    │
│  │  └─────────────────────────────────────────────────────────────────────┘    │    │
│  │                                           │ Database Connection             │    │
│  │                                           ▼                                 │    │
│  │  ┌─────────────────────────────────────────────────────────────────────┐    │    │
│  │  │                    RDS POSTGRESQL                                   │    │    │
│  │  │  Instance: db.t3.micro | Storage: 20GB (auto-scale to 100GB)        │    │    │
│  │  └─────────────────────────────────────────────────────────────────────┘    │    │
│  └─────────────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────────────┘
                                           │ Document Storage
                                           ▼
┌─────────────────────────────────────────────────────────────────────────────────────┐
│  S3 DOCUMENTS BUCKET: documind-documents-884710187119 (Encrypted, CORS enabled)     │
└─────────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────────┐
│  SECURITY: Security Groups │ IAM Roles │ Secrets Manager (DB creds, OpenAI key)     │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

**Key Features:**
- **High Availability**: Multi-AZ deployment across two availability zones
- **Scalability**: Auto-scaling ECS service and RDS storage
- **Security**: Private subnets for application and database tiers
- **Performance**: CloudFront CDN for global content delivery
- **AI Integration**: OpenAI API for intelligent document processing

## Project Structure

```
├── backend/          # Spring Boot + LangChain4j API
├── frontend/         # React + TypeScript UI
└── infra/            # AWS CDK Infrastructure
```

## Local Development

Run everything locally using Docker for PostgreSQL.

### Prerequisites
- Java 21+
- Node.js 20+
- Docker

### 1. Start PostgreSQL (with pgvector)
```bash
cd backend
docker compose up -d
```

### 2. Configure Environment
Create `backend/.env` with your OpenAI API key (see [Environment Variables](#environment-variables)).

### 3. Start Backend
```bash
cd backend
mvn spring-boot:run
```

### 4. Start Frontend
```bash
cd frontend
npm install
npm run dev
```

Access the app at **http://localhost:5173**

## Environment Variables

| Variable | Description | Required | Default |
|----------|-------------|----------|--------|
| `OPENAI_API_KEY` | OpenAI API key for GPT-4o-mini | Yes | - |
| `SPRING_DATASOURCE_URL` | PostgreSQL connection URL | No | `jdbc:postgresql://localhost:5432/documind` |
| `SPRING_DATASOURCE_USERNAME` | Database username | No | `documind` |
| `SPRING_DATASOURCE_PASSWORD` | Database password | No | `documind` |

**Local Development:** Create `backend/.env`:
```
OPENAI_API_KEY=sk-your-key-here
```

**AWS Deployment:** Store in Secrets Manager (see [Post-Deploy](#post-deploy-configure-openai-key)).

---

## AWS Deployment

The app is currently deployed on AWS. To deploy your own instance:

### Prerequisites
- AWS CLI configured (`aws configure`)
- AWS CDK installed (`npm install -g aws-cdk`)

### Deploy
```bash
cd infra
npm install
npx cdk bootstrap   # First time only
npx cdk deploy
```

### Post-Deploy: Configure OpenAI Key
```bash
aws secretsmanager put-secret-value \
  --secret-id documind/openai-api-key \
  --secret-string '{"apiKey":"sk-your-actual-key"}'
```

### Deploy Frontend Updates
```bash
cd frontend
npm run build
aws s3 sync dist/ s3://documind-frontend-<ACCOUNT_ID>-<REGION>/ --delete
aws cloudfront create-invalidation --distribution-id <DIST_ID> --paths "/*"
```


## License

MIT License - see [LICENSE](LICENSE) for details.

<!-- auto-update 2022-12-08 12:26:04 -->

<!-- auto-update 2022-12-10 12:26:04 -->

<!-- auto-update 2022-12-12 12:26:04 -->

<!-- auto-update 2022-12-14 12:26:04 -->

<!-- auto-update 2022-12-16 12:26:04 -->

<!-- auto-update 2022-12-18 12:26:04 -->

<!-- auto-update 2022-12-20 12:26:04 -->

<!-- auto-update 2022-12-22 12:26:04 -->

<!-- auto-update 2022-12-24 12:26:04 -->

<!-- auto-update 2022-12-26 12:26:04 -->

<!-- auto-update 2022-12-28 12:26:04 -->

<!-- auto-update 2022-12-30 12:26:04 -->

<!-- auto-update 2023-01-01 12:26:04 -->

<!-- auto-update 2023-01-03 12:26:04 -->

<!-- auto-update 2023-01-05 12:26:04 -->

<!-- auto-update 2023-01-07 12:26:04 -->

<!-- auto-update 2023-01-09 12:26:04 -->

<!-- auto-update 2023-01-11 12:26:04 -->

<!-- auto-update 2023-01-13 12:26:04 -->

<!-- auto-update 2023-01-15 12:26:04 -->

<!-- auto-update 2023-01-17 12:26:04 -->

<!-- auto-update 2023-01-19 12:26:04 -->

<!-- auto-update 2023-01-21 12:26:04 -->

<!-- auto-update 2023-01-23 12:26:04 -->

<!-- auto-update 2023-01-25 12:26:04 -->

<!-- auto-update 2023-01-27 12:26:04 -->

<!-- auto-update 2023-01-29 12:26:04 -->

<!-- auto-update 2023-01-31 12:26:04 -->

<!-- auto-update 2023-02-02 12:26:04 -->

<!-- auto-update 2023-02-04 12:26:04 -->

<!-- auto-update 2023-02-06 12:26:04 -->

<!-- auto-update 2023-02-08 12:26:04 -->

<!-- auto-update 2023-02-10 12:26:04 -->

<!-- auto-update 2023-02-12 12:26:04 -->

<!-- auto-update 2023-02-14 12:26:04 -->

<!-- auto-update 2023-02-16 12:26:04 -->

<!-- auto-update 2023-02-18 12:26:04 -->

<!-- auto-update 2023-02-20 12:26:04 -->

<!-- auto-update 2023-02-22 12:26:04 -->

<!-- auto-update 2023-02-24 12:26:04 -->

<!-- auto-update 2023-02-26 12:26:04 -->

<!-- auto-update 2023-02-28 12:26:04 -->

<!-- auto-update 2023-03-02 12:26:04 -->

<!-- auto-update 2023-03-04 12:26:04 -->

<!-- auto-update 2023-03-06 12:26:04 -->

<!-- auto-update 2022-10-05 11:12:13 -->

<!-- auto-update 2022-10-07 11:12:13 -->

<!-- auto-update 2022-10-09 11:12:13 -->

<!-- auto-update 2022-10-11 11:12:13 -->

<!-- auto-update 2022-10-13 11:12:13 -->

<!-- auto-update 2022-10-15 11:12:13 -->

<!-- auto-update 2022-10-17 11:12:13 -->

<!-- auto-update 2022-10-19 11:12:13 -->

<!-- auto-update 2022-10-21 11:12:13 -->

<!-- auto-update 2022-10-23 11:12:13 -->

<!-- auto-update 2022-10-25 11:12:13 -->

<!-- auto-update 2022-10-27 11:12:13 -->

<!-- auto-update 2022-10-29 11:12:13 -->

<!-- auto-update 2022-10-31 11:12:13 -->

<!-- auto-update 2022-11-02 11:12:13 -->

<!-- auto-update 2022-11-04 11:12:13 -->

<!-- auto-update 2022-11-06 11:12:13 -->

<!-- auto-update 2022-11-08 11:12:13 -->

<!-- auto-update 2022-11-10 11:12:13 -->

<!-- auto-update 2022-11-12 11:12:13 -->

<!-- auto-update 2022-11-14 11:12:13 -->

<!-- auto-update 2022-11-16 11:12:13 -->

<!-- auto-update 2022-11-18 11:12:13 -->

<!-- auto-update 2022-11-20 11:12:13 -->

<!-- auto-update 2022-11-22 11:12:13 -->

<!-- auto-update 2022-11-24 11:12:13 -->

<!-- auto-update 2022-11-26 11:12:13 -->

<!-- auto-update 2022-11-28 11:12:13 -->

<!-- auto-update 2022-11-30 11:12:13 -->

<!-- auto-update 2022-12-02 11:12:13 -->

<!-- auto-update 2022-12-04 11:12:13 -->

<!-- auto-update 2022-12-06 11:12:13 -->

<!-- auto-update 2022-12-08 11:12:13 -->

<!-- auto-update 2022-12-10 11:12:13 -->

<!-- auto-update 2022-12-12 11:12:13 -->

<!-- auto-update 2022-12-14 11:12:13 -->

<!-- auto-update 2022-12-16 11:12:13 -->

<!-- auto-update 2022-12-18 11:12:13 -->

<!-- auto-update 2022-12-20 11:12:13 -->

<!-- auto-update 2022-12-22 11:12:13 -->

<!-- auto-update 2022-12-24 11:12:13 -->

<!-- auto-update 2022-12-26 11:12:13 -->

<!-- auto-update 2022-12-28 11:12:13 -->

<!-- auto-update 2022-12-30 11:12:13 -->

<!-- auto-update 2023-01-01 11:12:13 -->

<!-- auto-update 2023-01-03 11:12:13 -->

<!-- auto-update 2023-01-05 11:12:13 -->

<!-- auto-update 2023-01-07 11:12:13 -->

<!-- auto-update 2023-01-09 11:12:13 -->

<!-- auto-update 2023-01-11 11:12:13 -->

<!-- auto-update 2023-01-13 11:12:13 -->

<!-- auto-update 2023-01-15 11:12:13 -->

<!-- auto-update 2023-01-17 11:12:13 -->

<!-- auto-update 2023-01-19 11:12:13 -->

<!-- auto-update 2023-01-21 11:12:13 -->

<!-- auto-update 2023-01-23 11:12:13 -->

<!-- auto-update 2023-01-25 11:12:13 -->

<!-- auto-update 2023-01-27 11:12:13 -->

<!-- auto-update 2023-01-29 11:12:13 -->

<!-- auto-update 2023-01-31 11:12:13 -->

<!-- auto-update 2023-02-02 11:12:13 -->

<!-- auto-update 2023-02-04 11:12:13 -->

<!-- auto-update 2023-02-06 11:12:13 -->

<!-- auto-update 2023-02-08 11:12:13 -->

<!-- auto-update 2023-02-10 11:12:13 -->

<!-- auto-update 2023-02-12 11:12:13 -->

<!-- auto-update 2023-02-14 11:12:13 -->

<!-- auto-update 2023-02-16 11:12:13 -->

<!-- auto-update 2023-02-18 11:12:13 -->

<!-- auto-update 2023-02-20 11:12:13 -->

<!-- auto-update 2023-02-22 11:12:13 -->

<!-- auto-update 2023-02-24 11:12:13 -->

<!-- auto-update 2023-02-26 11:12:13 -->

<!-- auto-update 2023-02-28 11:12:13 -->

<!-- auto-update 2023-03-02 11:12:13 -->

<!-- auto-update 2023-03-04 11:12:13 -->

<!-- auto-update 2023-03-06 11:12:13 -->

<!-- auto-update 2023-03-08 11:12:13 -->

<!-- auto-update 2023-03-10 11:12:13 -->

<!-- auto-update 2023-03-12 11:12:13 -->

<!-- auto-update 2023-03-14 11:12:13 -->

<!-- auto-update 2023-03-16 11:12:13 -->

<!-- auto-update 2023-03-18 11:12:13 -->

<!-- auto-update 2023-03-20 11:12:13 -->

<!-- auto-update 2023-03-22 11:12:13 -->

<!-- auto-update 2023-03-24 11:12:13 -->

<!-- auto-update 2023-03-26 11:12:13 -->

<!-- auto-update 2023-03-28 11:12:13 -->

<!-- auto-update 2023-03-30 11:12:13 -->

<!-- auto-update 2023-04-01 11:12:13 -->

<!-- auto-update 2023-04-03 11:12:13 -->

<!-- auto-update 2023-04-05 11:12:13 -->

<!-- auto-update 2023-04-07 11:12:13 -->

<!-- auto-update 2023-04-09 11:12:13 -->

<!-- auto-update 2023-04-11 11:12:13 -->

<!-- auto-update 2023-04-13 11:12:13 -->

<!-- auto-update 2023-04-15 11:12:13 -->

<!-- auto-update 2023-04-17 11:12:13 -->

<!-- auto-update 2023-04-19 11:12:13 -->

<!-- auto-update 2023-04-21 11:12:13 -->

<!-- auto-update 2023-04-23 11:12:13 -->

<!-- auto-update 2023-04-25 11:12:13 -->

<!-- auto-update 2023-04-27 11:12:13 -->

<!-- auto-update 2023-04-29 11:12:13 -->

<!-- auto-update 2023-05-01 11:12:13 -->

<!-- auto-update 2023-05-03 11:12:13 -->

<!-- auto-update 2023-05-05 11:12:13 -->

<!-- auto-update 2023-05-07 11:12:13 -->

<!-- auto-update 2023-05-09 11:12:13 -->

<!-- auto-update 2023-05-11 11:12:13 -->

<!-- auto-update 2023-05-13 11:12:13 -->

<!-- auto-update 2023-05-15 11:12:13 -->

<!-- auto-update 2023-05-17 11:12:13 -->

<!-- auto-update 2023-05-19 11:12:13 -->

<!-- auto-update 2023-05-21 11:12:13 -->

<!-- auto-update 2023-05-23 11:12:13 -->

<!-- auto-update 2023-05-25 11:12:13 -->

<!-- auto-update 2023-05-27 11:12:13 -->

<!-- auto-update 2023-05-29 11:12:13 -->

<!-- auto-update 2023-05-31 11:12:13 -->

<!-- auto-update 2023-06-02 11:12:13 -->

<!-- auto-update 2023-06-04 11:12:13 -->

<!-- auto-update 2023-06-06 11:12:13 -->

<!-- auto-update 2023-06-08 11:12:13 -->

<!-- auto-update 2023-06-10 11:12:13 -->

<!-- auto-update 2023-06-12 11:12:13 -->

<!-- auto-update 2023-06-14 11:12:13 -->

<!-- auto-update 2023-06-16 11:12:13 -->

<!-- auto-update 2023-06-18 11:12:13 -->

<!-- auto-update 2023-06-20 11:12:13 -->

<!-- auto-update 2023-06-22 11:12:13 -->

<!-- auto-update 2023-06-24 11:12:13 -->

<!-- auto-update 2023-06-26 11:12:13 -->

<!-- auto-update 2023-06-28 11:12:13 -->

<!-- auto-update 2023-06-30 11:12:13 -->

<!-- auto-update 2023-07-02 11:12:13 -->

<!-- auto-update 2023-07-04 11:12:13 -->

<!-- auto-update 2023-07-06 11:12:13 -->

<!-- auto-update 2023-07-08 11:12:13 -->

<!-- auto-update 2023-07-10 11:12:13 -->

<!-- auto-update 2023-07-12 11:12:13 -->

<!-- auto-update 2023-07-14 11:12:13 -->

<!-- auto-update 2023-07-16 11:12:13 -->

<!-- auto-update 2023-07-18 11:12:13 -->

<!-- auto-update 2023-07-20 11:12:13 -->

<!-- auto-update 2023-07-22 11:12:13 -->

<!-- auto-update 2023-07-24 11:12:13 -->

<!-- auto-update 2023-07-26 11:12:13 -->

<!-- auto-update 2023-07-28 11:12:13 -->

<!-- auto-update 2023-07-30 11:12:13 -->
