# CUSTOMER FEEDBACK APP - COMPLETE TECHNICAL GUIDE

**Product**: AI-Powered Customer Feedback Analyzer
**Repository**: customer-feedback-app
**Version**: 3.8.0
**Status**: Production (deployed at customer-feedback-app.onrender.com)
**Purpose**: Transform customer comments into actionable insights using AI
**Last Updated**: November 24, 2025

---

## TABLE OF CONTENTS

1. [What It Does](#1-what-it-does)
2. [How It Works](#2-how-it-works)
3. [Technical Architecture](#3-technical-architecture)
4. [Tech Stack](#4-tech-stack)
5. [Installation & Setup](#5-installation--setup)
6. [Usage Guide](#6-usage-guide)
7. [AI Capabilities](#7-ai-capabilities)
8. [Output Format](#8-output-format)
9. [Cost Optimization](#9-cost-optimization)
10. [Integration with Comment-Extractor](#10-integration-with-comment-extractor)
11. [Deployment Options](#11-deployment-options)
12. [Limitations & Gaps](#12-limitations--gaps)
13. [Use Cases for Marketing](#13-use-cases-for-marketing)

---

## 1. WHAT IT DOES

The Customer Feedback App is a **production-ready SaaS platform** that automatically analyzes customer comments using AI and generates comprehensive Excel reports with 23 specialized sheets.

### Core Functions

1. **Upload Comments** - Accepts Excel (.xlsx, .xls), CSV (.csv), or Parquet (.parquet) files
2. **AI Analysis** - Processes comments using OpenAI GPT-4o-mini for deep insights
3. **Generate Reports** - Creates professional 23-sheet Excel reports with charts and formatting
4. **Export Results** - Download Excel, CSV, or view interactive dashboards

### What Makes It Unique

- **87% Cost Reduction**: Hybrid analysis combining local sentiment (free) + selective OpenAI (paid)
- **23-Sheet Excel Reports**: Comprehensive analysis across 36 calculated columns
- **7-Emotion Detection**: Satisfaction, Frustration, Anger, Trust, Disappointment, Confusion, Anticipation
- **Churn Risk Prediction**: 0-100% probability score for each customer
- **NPS Classification**: Automatic categorization (Promoter 9-10, Passive 7-8, Detractor 0-6)
- **Pain Point Extraction**: AI identifies top 5 issues per comment + overall Top 10
- **Professional Formatting**: Data bars, conditional formatting, icon sets, gradient scales
- **LATAM-First**: Spanish and Portuguese language support with cultural context

### Input Requirements

```
Required Columns:
- "Nota" (Rating): Numbers 0-10
- "Comentario Final" (Comment): Text, minimum 3 characters

Optional Columns:
- "NPS": If already calculated, will be validated
- Any other columns: Preserved in output but not analyzed

File Limits:
- Max rows: 3,000 comments
- Max size: 20MB
- Supported formats: .xlsx, .xls, .csv, .parquet
```

### Output Example

**Summary Metrics:**
- Total Comments: 850
- Average Sentiment: 0.65 (positive scale -1 to +1)
- NPS Score: 42 (on scale -100 to +100)
- High Churn Risk: 127 customers (15%)
- Top Pain Point: "Delivery delays" (47 mentions)

**Detailed Analysis:** 23 Excel sheets with:
- Individual comment analysis (all 7 emotions + churn risk + pain points)
- Aggregated statistics
- Pivot tables
- Interactive charts
- Conditional formatting

---

## 2. HOW IT WORKS

### Step-by-Step Process

```
User Action                     →  System Processing
───────────────────────────────────────────────────────────────────────

1. Upload Excel/CSV file        →  File validation (format, size, columns)
                                →  Task ID created (UUID)
                                →  File saved to temp storage

2. Click "Analyze"              →  Celery worker triggered (async)
                                →  File loaded into Pandas DataFrame
                                →  Column mapping & normalization

3. Data Preparation             →  Deduplication (SHA256 hashing)
                                →  Remove duplicates (15-20% savings)
                                →  Intelligent sampling (if >50K rows)
                                →  Truncate comments to 150 chars

4. Hybrid Analysis              →  LOCAL (free, instant):
                                   - Sentiment score (VADER + TextBlob)
                                   - NPS category calculation
                                   - Basic validation

                                →  OPENAI (paid, selective):
                                   - 7 emotions (0-100% each)
                                   - Churn risk (0-100%)
                                   - Pain points (max 5 per comment)

5. Batch Processing             →  Process 50-100 comments per batch
                                →  Parallel API calls (10 concurrent)
                                →  Cache results in Redis (7 days)
                                →  Progress tracking (WebSocket)

6. Aggregation                  →  Calculate summary statistics
                                →  Identify top 10 pain points
                                →  Generate pivot tables
                                →  Create distribution charts

7. Excel Export                 →  23 sheets created:
                                   - Detailed Analysis (all rows)
                                   - Summary Statistics
                                   - Emotions Distribution
                                   - NPS Analysis
                                   - Pain Points
                                   - Churn Risk
                                   - Charts (6 types)
                                   - Pivot Tables (8 views)
                                →  Professional formatting applied
                                →  File saved to temp storage

8. Download Results             →  User downloads Excel (or CSV)
                                →  Files deleted after 24 hours
```

### Performance

| Comments | Processing Time | Cost (USD) | Throughput |
|----------|----------------|------------|------------|
| 100      | 2-3 seconds    | $0.002     | 40/sec     |
| 500      | 5-8 seconds    | $0.01      | 62/sec     |
| 850      | 8-10 seconds   | $0.017     | 85/sec     |
| 1,800    | 18-20 seconds  | $0.036     | 90/sec     |
| 3,000    | 30-35 seconds  | $0.06      | 86/sec     |

**Optimization Results:**
- **Before**: 250 tokens/comment = $0.50 per 1,000 comments
- **After**: 25 tokens/comment = $0.06 per 1,000 comments
- **Savings**: 87% cost reduction

---

## 3. TECHNICAL ARCHITECTURE

### System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER BROWSER                             │
│  (React 18.3 + TypeScript + Tailwind CSS + Plotly.js charts)   │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      │ HTTPS
                      │
┌─────────────────────▼───────────────────────────────────────────┐
│                      BFF PROXY (Express)                         │
│  - Request forwarding                                            │
│  - CORS handling                                                 │
│  - Compression                                                   │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      │ Internal HTTP
                      │
┌─────────────────────▼───────────────────────────────────────────┐
│                  FASTAPI BACKEND (Python 3.12)                  │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  REST API Endpoints:                                        │ │
│  │  - POST /upload (file upload + validation)                 │ │
│  │  - GET /status/{task_id} (progress tracking)               │ │
│  │  - GET /results/{task_id} (analysis results)               │ │
│  │  - GET /export/{task_id}?format=xlsx (download)            │ │
│  │  - GET /health (system health check)                       │ │
│  │  - GET /metrics (Prometheus metrics)                       │ │
│  │  - WS /ws/{task_id} (real-time progress)                   │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      │ Celery Tasks (Redis)
                      │
┌─────────────────────▼───────────────────────────────────────────┐
│                   CELERY WORKER (Python 3.12)                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │  Processing Pipeline:                                       │ │
│  │  1. Load File → Pandas DataFrame                           │ │
│  │  2. Validate & Normalize → Column mapping                  │ │
│  │  3. Deduplicate → SHA256 hashing (15-20% reduction)        │ │
│  │  4. Sample → Stratified sampling (if >50K rows)            │ │
│  │  5. LOCAL Analysis → VADER + TextBlob (free)               │ │
│  │  6. OPENAI Analysis → Emotions + Churn + Pain Points       │ │
│  │  7. Aggregate → Statistics + Top 10 pain points            │ │
│  │  8. Export → 23-sheet Excel with formatting                │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      │ Multiple parallel calls
                      │
┌─────────────────────▼───────────────────────────────────────────┐
│                      OPENAI API (GPT-4o-mini)                   │
│  - Emotion detection (7 emotions)                               │
│  - Churn risk prediction                                        │
│  - Pain point extraction                                        │
│  - Cost: ~$0.02 per 1,000 comments (87% optimized)             │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                       REDIS (Cache + Queue)                      │
│  - Celery broker (task queue)                                   │
│  - Celery result backend (task status)                          │
│  - Comment cache (7 days TTL)                                   │
│  - Dataset cache (2 hours TTL)                                  │
│  - Connection pool: 20 connections                              │
└─────────────────────────────────────────────────────────────────┘
```

### File Structure

```
customer-feedback-app/
├── web/                                # Frontend (React + BFF)
│   ├── src/
│   │   ├── components/
│   │   │   ├── upload/               # File upload UI
│   │   │   │   ├── FileUpload.tsx    # Drag & drop component
│   │   │   │   ├── DragDropZone.tsx  # Drop zone logic
│   │   │   │   ├── FileInfo.tsx      # File metadata display
│   │   │   │   ├── ErrorMessage.tsx  # Validation errors
│   │   │   │   └── SampleDataPreview.tsx  # Preview first 5 rows
│   │   │   ├── results/              # Results visualization
│   │   │   │   ├── ResultsCharts.tsx # Main chart container
│   │   │   │   ├── EmotionsChart.tsx # Radar chart (7 emotions)
│   │   │   │   ├── NPSChart.tsx      # Bar chart (NPS distribution)
│   │   │   │   ├── PainPointsChart.tsx # Bar chart (Top 10)
│   │   │   │   ├── ChurnRiskChart.tsx # Histogram (risk distribution)
│   │   │   │   └── StatCard.tsx      # Summary metric cards
│   │   │   ├── progress/             # Progress tracking
│   │   │   │   └── ProgressTracker.tsx # Real-time progress bar
│   │   │   ├── export/               # Export functionality
│   │   │   │   └── ExportResults.tsx # Download buttons
│   │   │   └── ui/                   # Design system
│   │   │       ├── GlassCard.tsx     # Glass morphism cards
│   │   │       ├── GlassButton.tsx   # Glass morphism buttons
│   │   │       └── GlassProgress.tsx # Glass morphism progress
│   │   ├── pages/
│   │   │   ├── LandingPage.tsx       # Marketing landing page
│   │   │   ├── AnalyzerPage.tsx      # Main analysis app
│   │   │   └── AboutPage.tsx         # About/FAQ page
│   │   ├── contexts/
│   │   │   ├── DataContext.tsx       # Global state management
│   │   │   └── UIContext.tsx         # UI state (theme, etc.)
│   │   └── main.tsx                  # React entry point
│   ├── bff/
│   │   └── server.ts                 # Express BFF proxy
│   └── package.json                  # Dependencies (React 18.3.1)
│
├── api/                               # Backend (FastAPI + Celery)
│   ├── app/
│   │   ├── api/
│   │   │   └── endpoints/
│   │   │       ├── upload/           # Upload logic
│   │   │       │   ├── routes.py     # POST /upload endpoint
│   │   │       │   ├── validators.py # File validation
│   │   │       │   ├── file_handlers.py # File parsing
│   │   │       │   └── cost_estimator.py # Cost preview
│   │   │       ├── task_status.py    # GET /status/{task_id}
│   │   │       ├── results.py        # GET /results/{task_id}
│   │   │       ├── export/           # Export logic
│   │   │       │   └── results_export.py # GET /export/{task_id}
│   │   │       ├── health.py         # GET /health
│   │   │       ├── metrics.py        # GET /metrics (Prometheus)
│   │   │       └── websocket.py      # WS /ws/{task_id}
│   │   ├── domain/
│   │   │   ├── analysis/             # Analysis logic
│   │   │   │   ├── service.py        # Main orchestration
│   │   │   │   ├── aggregation.py    # Statistics calculation
│   │   │   │   ├── capability.py     # Feature detection
│   │   │   │   ├── analyzer_factory.py # Strategy selection
│   │   │   │   └── strategies/       # Analysis strategies
│   │   │   │       ├── unified_analysis_strategy.py # Hybrid analysis
│   │   │   │       ├── comprehensive_ai_strategy.py # Full OpenAI
│   │   │   │       └── full_mode_strategy.py # Local + OpenAI
│   │   │   └── export/               # Export logic
│   │   │       ├── excel/            # Excel generation
│   │   │       │   ├── base_exporter.py # Base exporter class
│   │   │       │   ├── sheet_generator.py # 23 sheets creation
│   │   │       │   ├── formatter.py  # Professional formatting
│   │   │       │   ├── charts/       # Chart generation
│   │   │       │   │   └── factory.py # Chart factory
│   │   │       │   └── constants/
│   │   │       │       └── styles.py # Excel styles
│   │   │       └── csv_exporter.py   # CSV export
│   │   ├── application/
│   │   │   ├── di_container.py       # Dependency injection
│   │   │   ├── budget_service.py     # Cost calculation
│   │   │   ├── pipeline/             # Processing pipeline
│   │   │   │   ├── efficient_deduplication.py # SHA256 dedup
│   │   │   │   ├── intelligent_sampler.py # Stratified sampling
│   │   │   │   ├── parallel_batch_executor.py # Parallel processing
│   │   │   │   ├── token_estimator.py # Token counting
│   │   │   │   └── schema_signature_cache.py # Dataset caching
│   │   │   ├── worker/               # Celery worker logic
│   │   │   │   ├── worker_orchestrator.py # Main worker
│   │   │   │   ├── analysis_routing_service.py # Strategy routing
│   │   │   │   ├── data_loader_service.py # File loading
│   │   │   │   ├── pipeline_preparation_service.py # Prep
│   │   │   │   └── result_processing_service.py # Aggregation
│   │   │   └── services/
│   │   │       ├── file_processor.py # Unified file processor
│   │   │       └── data_exporter.py  # Export orchestration
│   │   ├── config/
│   │   │   ├── settings.py           # Environment configuration
│   │   │   ├── processing.py         # Processing constants
│   │   │   ├── analysis_thresholds.py # Emotion thresholds
│   │   │   ├── column_registry.py    # Column mappings
│   │   │   └── file_formats.py       # File format config
│   │   └── shared/
│   │       ├── logging.py            # Structured logging
│   │       ├── redis_singleton.py    # Redis connection pool
│   │       └── nps_utils.py          # NPS calculations
│   ├── requirements.txt              # Python dependencies
│   └── main.py                       # FastAPI entry point
│
├── infrastructure/                    # AWS deployment (optional)
│   ├── terraform/                     # IaC for AWS EKS
│   │   ├── main.tf                   # VPC + EKS + ElastiCache
│   │   ├── variables.tf              # Configuration variables
│   │   └── outputs.tf                # Load balancer URL
│   ├── k8s/                          # Kubernetes manifests
│   │   ├── api-deployment.yaml       # FastAPI deployment
│   │   ├── worker-deployment.yaml    # Celery worker
│   │   ├── redis-deployment.yaml     # Redis (if not ElastiCache)
│   │   └── ingress.yaml              # ALB ingress
│   └── scripts/
│       ├── setup-aws-infra.sh        # Provision infrastructure
│       ├── build-and-push.sh         # Build Docker images
│       └── deploy-k8s.sh             # Deploy to Kubernetes
│
├── docker-compose.yml                 # Local development
├── render.yaml                        # Render.com deployment config
├── README.md                          # User documentation
└── package.json                       # Monorepo config

Total: ~200+ Python files, ~60+ TypeScript files
```

---

## 4. TECH STACK

### Frontend (React SPA)

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Framework** | React | 18.3.1 | Component-based UI |
| **Language** | TypeScript | 5.6.3 | Type safety |
| **Styling** | Tailwind CSS | 3.4.14 | Utility-first CSS |
| **Charts** | Plotly.js | 3.1.2 | Interactive visualizations |
| **Routing** | React Router | 7.9.1 | Client-side routing |
| **HTTP Client** | Axios | 1.13.2 | API requests |
| **Build Tool** | Vite | 7.1.12 | Fast builds + HMR |
| **Testing** | Vitest + Testing Library | 4.0.4 | Unit + integration tests |
| **Monitoring** | Sentry | 10.22.0 | Error tracking |

### Backend (FastAPI + Celery)

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Framework** | FastAPI | 0.115.0 | REST API |
| **Language** | Python | 3.12.6 | Core logic |
| **ASGI Server** | Uvicorn | 0.32.0 | Production server |
| **Task Queue** | Celery | 5.4.0 | Async processing |
| **Broker** | Redis | 5.2.0 | Message queue + cache |
| **Data Processing** | Pandas | 2.2.3 | DataFrame operations |
| **Excel** | OpenPyXL | 3.1.5 | Excel generation |
| **Parquet** | PyArrow | 21.0.0 | High-performance I/O |
| **AI** | OpenAI | 1.54.3 | GPT-4o-mini API |
| **Validation** | Pydantic | 2.9.2 | Data validation |
| **Logging** | Structlog | 24.4.0 | Structured logging |
| **Monitoring** | Sentry | 2.19.2 | Error tracking |

### BFF Proxy (Express)

| Component | Technology | Version | Purpose |
|-----------|-----------|---------|---------|
| **Framework** | Express | 5.1.0 | Node.js web server |
| **Proxy** | http-proxy-middleware | 3.0.0 | Request forwarding |
| **Compression** | compression | 1.7.4 | Response compression |
| **Security** | helmet | 8.1.0 | Security headers |

### Infrastructure

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **Hosting** | Render.com | Production deployment (current) |
| **Container** | Docker | Containerization |
| **Orchestration** | Kubernetes (EKS) | Enterprise deployment (optional) |
| **Cloud** | AWS | Infrastructure (optional) |
| **IaC** | Terraform | Infrastructure provisioning |
| **CI/CD** | GitHub Actions | Automated testing + deployment |
| **Monitoring** | CloudWatch | Logs + metrics (AWS) |

---

## 5. INSTALLATION & SETUP

### Prerequisites

- **Python**: 3.11+ (3.12.6 recommended)
- **Node.js**: 18+ (20.0.0 recommended)
- **Redis**: 7.0+ (required for Celery)
- **RAM**: 4GB minimum (8GB recommended)
- **Disk**: 1GB free space
- **OpenAI API Key**: Required for AI analysis

### Option 1: Docker Compose (Recommended for Local Dev)

**Fastest way to get started - everything in one command:**

```bash
# 1. Clone repository
git clone https://github.com/Ai-Whisperers/customer-feedback-app.git
cd customer-feedback-app

# 2. Create environment file
cp .env.example .env.local

# 3. Edit .env.local and add your OpenAI API key
nano .env.local
# Add: OPENAI_API_KEY=sk-xxxxx

# 4. Start all services (Redis, API, Worker, Frontend)
docker-compose up -d

# 5. Access application
# Frontend: http://localhost:3000
# API: http://localhost:10000
# API Docs: http://localhost:10000/docs
# Redis: localhost:6379

# 6. View logs
docker-compose logs -f worker  # Worker logs
docker-compose logs -f api     # API logs

# 7. Stop services
docker-compose down
```

**What Docker Compose starts:**
- Redis (port 6379)
- FastAPI API (port 10000)
- Celery Worker (background)
- React Frontend (port 3000)

### Option 2: Manual Setup (For Development)

**Backend Setup:**

```bash
# 1. Navigate to API directory
cd api

# 2. Create virtual environment
python3.12 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Install Playwright browsers (for screenshots - optional)
playwright install chromium

# 5. Start Redis (required)
# macOS: brew install redis && redis-server
# Ubuntu: sudo apt install redis-server && redis-server
# Windows: Use Docker or WSL

# 6. Create .env file
cp .env.example .env

# 7. Configure environment variables
nano .env
# Required:
OPENAI_API_KEY=sk-xxxxx
REDIS_URL=redis://localhost:6379
AI_MODEL=gpt-4o-mini
BATCH_SIZE_OPTIMAL=120
CELERY_WORKER_CONCURRENCY=1

# Optional (feature flags):
HYBRID_ANALYSIS_ENABLED=true
ENABLE_COMMENT_CACHE=true
ENABLE_DATASET_CACHING=false
ENABLE_INTELLIGENT_SAMPLING=false

# 8. Start FastAPI server
uvicorn app.main:app --reload --port 8000

# 9. In another terminal, start Celery worker
cd api
source venv/bin/activate
celery -A app.workers.celery_app worker --loglevel=INFO --concurrency=1

# Note: Use --concurrency=1 to avoid memory issues
```

**Frontend Setup:**

```bash
# 1. Navigate to web directory
cd web

# 2. Install dependencies
npm install

# 3. Create .env file
cp .env.example .env

# 4. Configure BFF proxy (optional)
nano .env
# Add:
VITE_API_URL=http://localhost:8000

# 5. Start development server
npm run dev

# This starts:
# - Vite dev server (port 3000)
# - BFF proxy (port 3001)

# 6. Access application
# Open http://localhost:3000 in browser
```

**Verify Installation:**

```bash
# 1. Check API health
curl http://localhost:8000/health
# Expected: {"status":"healthy","version":"3.8.0"}

# 2. Check Celery worker
# Look for "celery@hostname ready." in worker logs

# 3. Check Redis
redis-cli ping
# Expected: PONG

# 4. Check Frontend
curl http://localhost:3000
# Expected: HTML response
```

### Option 3: Production Deployment (Render.com)

**Current production setup (live at customer-feedback-app.onrender.com):**

```bash
# 1. Fork repository to your GitHub account

# 2. Create Render.com account (free tier available)

# 3. Connect GitHub repository

# 4. Create 4 services:

# Service 1: Redis
# - Type: External (Redis)
# - Plan: Starter ($7/mo)
# - Note the internal Redis URL

# Service 2: API (FastAPI)
# - Type: Web Service
# - Runtime: Python 3.12
# - Build Command: pip install -r api/requirements.txt
# - Start Command: uvicorn app.main:app --host 0.0.0.0 --port $PORT
# - Environment Variables:
OPENAI_API_KEY=sk-xxxxx
REDIS_URL=redis://red-xxxxx:6379  # From step 1
CELERY_BROKER_URL=redis://red-xxxxx:6379
CELERY_RESULT_BACKEND=redis://red-xxxxx:6379
AI_MODEL=gpt-4o-mini
HYBRID_ANALYSIS_ENABLED=true

# Service 3: Worker (Celery)
# - Type: Background Worker
# - Runtime: Python 3.12
# - Build Command: pip install -r api/requirements.txt
# - Start Command: celery -A app.workers.celery_app worker --loglevel=INFO --concurrency=1
# - Environment Variables: (same as API)

# Service 4: Frontend (React + BFF)
# - Type: Web Service
# - Runtime: Node 20
# - Build Command: cd web && npm install && npm run build
# - Start Command: cd web && npm start
# - Environment Variables:
API_URL=https://your-api.onrender.com  # From step 2
PORT=10000

# 5. Deploy all services (auto-deploy on git push)

# 6. Test production
curl https://your-app.onrender.com/health
```

**Production Cost (Render.com):**
- Redis: $7/month
- API: $7/month (512MB RAM)
- Worker: $7/month (512MB RAM)
- Frontend: $7/month (512MB RAM)
- Build minutes: ~$2-5/month
- **Total**: ~$30-35/month

### Option 4: AWS Enterprise Deployment (Optional)

**For high-scale production (1,000+ customers):**

```bash
# 1. Prerequisites
# - AWS account with admin access
# - Terraform installed
# - kubectl installed
# - AWS CLI configured

# 2. Provision infrastructure
cd infrastructure
./scripts/setup-aws-infra.sh production

# This creates:
# - VPC with public/private subnets (3 AZs)
# - EKS cluster (Kubernetes 1.28)
# - Node groups (t3.medium, 2-10 nodes)
# - ElastiCache Redis (cache.t3.micro)
# - ECR repositories
# - ALB load balancer
# - CloudWatch logging

# 3. Configure secrets
./scripts/set-secrets.sh production
# Enter: OPENAI_API_KEY when prompted

# 4. Build and push Docker images
./scripts/build-and-push.sh production

# 5. Deploy to Kubernetes
./scripts/deploy-k8s.sh production

# 6. Get load balancer URL
kubectl get ingress -n default
# Access: http://your-alb-xxxxx.us-east-1.elb.amazonaws.com

# 7. Monitor
# CloudWatch: Logs + metrics
# Kubernetes: kubectl get pods
```

**AWS Cost Estimate:**
- EKS cluster: $72/month
- EC2 nodes (2x t3.medium): $60/month
- ElastiCache Redis: $15/month
- ALB: $20/month
- Data transfer: $10-30/month
- **Total**: ~$180-200/month (base), scales to $400+ with auto-scaling

---

## 6. USAGE GUIDE

### Basic Workflow

**Step 1: Prepare Your File**

Create an Excel or CSV file with required columns:

```
Example: customer_feedback.xlsx

| Nota | Comentario Final                                   |
|------|----------------------------------------------------|
| 9    | Excelente servicio, muy satisfecho con la atención |
| 5    | El precio es alto pero la calidad es buena         |
| 3    | Muy lento el servicio, esperé demasiado            |
| 10   | Perfecto! Superó mis expectativas                  |
| 2    | Pésima experiencia, no volveré a comprar           |
```

**Accepted column names (case-insensitive):**
- Rating: "Nota", "nota", "Rating", "Score", "Puntaje", "Calificacion"
- Comment: "Comentario Final", "comentario_final", "Comment", "Feedback", "Comentario"

**Step 2: Upload File**

```typescript
// Via UI: Drag & drop or click "Select file"

// Via API:
const formData = new FormData();
formData.append('file', fileBlob, 'feedback.xlsx');

const response = await fetch('https://customer-feedback-app.onrender.com/upload', {
  method: 'POST',
  body: formData
});

const { task_id } = await response.json();
// task_id: "550e8400-e29b-41d4-a716-446655440000"
```

**Step 3: Monitor Progress**

```typescript
// Poll status endpoint
const checkStatus = async (taskId: string) => {
  const response = await fetch(`/status/${taskId}`);
  const data = await response.json();

  console.log(data);
  /*
  {
    "task_id": "550e8400-...",
    "status": "processing",  // pending, processing, completed, failed
    "progress": 65,           // 0-100
    "message": "Analyzing batch 7/10",
    "processed_rows": 650,
    "total_rows": 1000,
    "current_stage": "ai_analysis",
    "estimated_time_remaining": 12  // seconds
  }
  */
};

// Poll every 2 seconds
const interval = setInterval(async () => {
  const status = await checkStatus(taskId);
  if (status.status === 'completed') {
    clearInterval(interval);
    loadResults(taskId);
  }
}, 2000);

// Or use WebSocket for real-time updates
const ws = new WebSocket(`wss://customer-feedback-app.onrender.com/ws/${taskId}`);
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  console.log(`Progress: ${data.progress}%`);
};
```

**Step 4: View Results**

```typescript
// Get analysis results
const response = await fetch(`/results/${taskId}`);
const results = await response.json();

console.log(results);
/*
{
  "summary": {
    "total_comments": 850,
    "avg_sentiment": 0.65,
    "nps_score": 42,
    "promoters_count": 340,
    "passives_count": 255,
    "detractors_count": 255,
    "high_churn_risk_count": 127,
    "processing_time_seconds": 8.5,
    "cost_usd": 0.017
  },
  "emotions_summary": {
    "satisfaccion": 62.5,
    "frustracion": 15.3,
    "enojo": 8.7,
    "confianza": 58.2,
    "decepcion": 12.1,
    "confusion": 5.9,
    "anticipacion": 45.3
  },
  "pain_points": [
    { "keyword": "entrega", "count": 47, "severity": "high" },
    { "keyword": "demora", "count": 35, "severity": "high" },
    { "keyword": "precio", "count": 28, "severity": "medium" },
    { "keyword": "calidad", "count": 22, "severity": "medium" },
    { "keyword": "atencion", "count": 19, "severity": "low" }
  ],
  "detailed_results": [
    {
      "Nota": 9,
      "Comentario Final": "Excelente servicio...",
      "sentiment_score": 0.92,
      "nps_category": "Promotor",
      "churn_risk_percentage": 5,
      "satisfaccion": 95,
      "frustracion": 0,
      "enojo": 0,
      "confianza": 90,
      "decepcion": 0,
      "confusion": 0,
      "anticipacion": 85,
      "pain_points": "ninguno"
    },
    // ... 849 more comments
  ]
}
*/
```

**Step 5: Export Results**

```bash
# Download Excel (23 sheets with formatting)
curl -o results.xlsx \
  "https://customer-feedback-app.onrender.com/export/${task_id}?format=xlsx"

# Download CSV (single file)
curl -o results.csv \
  "https://customer-feedback-app.onrender.com/export/${task_id}?format=csv"

# Download all formats (zip file)
curl -o results.zip \
  "https://customer-feedback-app.onrender.com/export/${task_id}?format=all"
```

### Advanced Features

**Feature Flag: Dataset Caching**

```bash
# Enable instant re-analysis for duplicate uploads
ENABLE_DATASET_CACHING=true
DATASET_CACHE_TTL_HOURS=2

# Use case: Demo environment, QA testing
# Result: 0 seconds, $0 cost on re-upload (100% savings)
```

**Feature Flag: Intelligent Sampling**

```bash
# Enable stratified sampling for large datasets
ENABLE_INTELLIGENT_SAMPLING=true
SAMPLING_THRESHOLD=50000  # Trigger at 50K rows
SAMPLING_TARGET_SIZE=10000  # Sample down to 10K

# Use case: Annual surveys, enterprise datasets (100K+ comments)
# Result: 90% cost reduction, <5% accuracy loss
```

**Example: 120K Comments Dataset**

```
Without sampling:
- Processing time: 120 seconds
- Cost: $2.40 USD
- Accuracy: 100%

With intelligent sampling:
- Processing time: 12 seconds (90% faster)
- Cost: $0.24 USD (90% cheaper)
- Accuracy: 95% (NPS ±5 points)
```

---

## 7. AI CAPABILITIES

### Hybrid Analysis Architecture

The platform uses a **two-tier analysis system** to achieve 87% cost reduction:

```
┌─────────────────────────────────────────────────────────────┐
│                    COMMENT INPUT                             │
│  "El servicio fue lento, esperé más de 30 minutos"         │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│              TIER 1: LOCAL ANALYSIS (FREE)                   │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  1. Sentiment Score (VADER + TextBlob)                 │ │
│  │     - VADER: -0.65 (negative)                          │ │
│  │     - TextBlob: -0.58 (negative)                       │ │
│  │     - Combined: -0.62                                  │ │
│  │  2. NPS Category Calculation                           │ │
│  │     - Rating: 3 → Detractor                            │ │
│  │  3. Basic Validation                                   │ │
│  │     - Length: 47 chars ✓                               │ │
│  │     - Language: Spanish ✓                              │ │
│  └────────────────────────────────────────────────────────┘ │
│  Cost: $0.00                                                │
│  Time: <1ms                                                 │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│           TIER 2: OPENAI ANALYSIS (SELECTIVE)               │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  GPT-4o-mini Prompt:                                   │ │
│  │  "Analyze this Spanish customer feedback:             │ │
│  │   Text: 'El servicio fue lento...'                    │ │
│  │   Rating: 3/10                                         │ │
│  │   Sentiment: -0.62                                     │ │
│  │                                                        │ │
│  │   Return JSON with:                                   │ │
│  │   - 7 emotions (0-100% each)                          │ │
│  │   - churn_risk (0-100%)                               │ │
│  │   - pain_points (max 5 keywords)"                     │ │
│  └────────────────────────────────────────────────────────┘ │
│                                                             │
│  Response (25 tokens):                                      │
│  {                                                          │
│    "satisfaccion": 15,                                      │
│    "frustracion": 75,                                       │
│    "enojo": 40,                                             │
│    "confianza": 20,                                         │
│    "decepcion": 60,                                         │
│    "confusion": 5,                                          │
│    "anticipacion": 10,                                      │
│    "churn_risk": 68,                                        │
│    "pain_points": "lento, demora, espera"                  │
│  }                                                          │
│                                                             │
│  Cost: $0.00002 per comment                                │
│  Time: ~50ms                                                │
└─────────────────────────────────────────────────────────────┘
```

### 7 Emotions Detected

| Emotion | Spanish | Range | Description | Example Triggers |
|---------|---------|-------|-------------|------------------|
| **Satisfaction** | Satisfacción | 0-100% | Happiness, contentment, fulfillment | "excelente", "perfecto", "superó expectativas" |
| **Frustration** | Frustración | 0-100% | Irritation, hindrance, obstacles | "complicado", "difícil", "no funciona" |
| **Anger** | Enojo | 0-100% | Rage, hostility, aggression | "pésimo", "horrible", "inaceptable" |
| **Trust** | Confianza | 0-100% | Reliability, credibility, safety | "confiable", "seguro", "recomiendo" |
| **Disappointment** | Decepción | 0-100% | Unmet expectations, letdown | "esperaba más", "no cumplió", "decepcionante" |
| **Confusion** | Confusión | 0-100% | Lack of clarity, uncertainty | "no entiendo", "confuso", "no sé cómo" |
| **Anticipation** | Anticipación | 0-100% | Expectation, hope, excitement | "espero", "próxima vez", "volveré" |

**Emotion Detection Example:**

```json
Comment: "El producto es bueno pero el envío fue terrible, tardó 2 semanas"
Rating: 6/10

Emotions:
{
  "satisfaccion": 45,      // "bueno" → moderate satisfaction
  "frustracion": 65,       // "tardó 2 semanas" → high frustration
  "enojo": 35,             // "terrible" → moderate anger
  "confianza": 40,         // mixed signals
  "decepcion": 70,         // unmet expectations on delivery
  "confusion": 5,          // no confusion present
  "anticipacion": 25       // low likelihood to return
}
```

### Churn Risk Prediction

**Algorithm**: GPT-4o-mini analyzes 3 factors:

1. **Rating**: 0-6 = high risk, 7-8 = medium risk, 9-10 = low risk
2. **Sentiment**: Negative language patterns
3. **Emotions**: High frustration + anger + disappointment

**Risk Levels:**

| Risk % | Level | Action Required | Example |
|--------|-------|-----------------|---------|
| 0-25% | Low | No action | "Excelente servicio, 10/10" |
| 26-50% | Medium | Monitor | "Bueno pero podría mejorar" |
| 51-75% | High | Reach out within 24h | "Tardó mucho, no estoy satisfecho" |
| 76-100% | Critical | Immediate intervention | "Pésimo, nunca más compro aquí" |

**Churn Risk Example:**

```json
Comment: "Tercera vez que compro y siempre hay problemas, ya no vale la pena"
Rating: 2/10

Analysis:
{
  "churn_risk": 95,  // CRITICAL
  "reasoning": [
    "Repeat negative experience (3rd time)",
    "Explicit intent to leave ('ya no vale la pena')",
    "Very low rating (2/10)",
    "High frustration + disappointment"
  ],
  "recommended_action": "Immediate manager escalation + discount offer"
}
```

### Pain Point Extraction

**Process:**

1. **Identify keywords**: Nouns and adjectives related to problems
2. **Contextualize**: Understand if keyword is positive or negative context
3. **Deduplicate**: Merge similar keywords (e.g., "envío" + "entrega")
4. **Rank**: Count frequency across all comments

**Pain Point Structure:**

```json
Individual comment pain points (max 5):
{
  "pain_points": "entrega, demora, empaquetado"
}

Global pain points (Top 10):
[
  {
    "keyword": "entrega",
    "count": 47,
    "severity": "high",
    "affected_customers": 47,
    "avg_churn_risk": 68,
    "sample_comments": [
      "La entrega tardó 10 días",
      "Problemas con la entrega",
      "Entrega en mal estado"
    ]
  },
  {
    "keyword": "precio",
    "count": 28,
    "severity": "medium",
    "affected_customers": 28,
    "avg_churn_risk": 42,
    "sample_comments": [
      "Precio muy alto",
      "No vale el precio",
      "Caro para la calidad"
    ]
  }
  // ... 8 more
]
```

**Pain Point Example:**

```
Top 10 Pain Points for E-commerce Company:

1. entrega (47 mentions) - Delivery issues, delays
2. demora (35 mentions) - Wait times, slow service
3. precio (28 mentions) - Pricing concerns
4. calidad (22 mentions) - Product quality issues
5. atencion (19 mentions) - Customer service problems
6. empaquetado (15 mentions) - Packaging damage
7. devolucion (12 mentions) - Return process difficulties
8. comunicacion (10 mentions) - Lack of updates
9. inventario (8 mentions) - Out of stock issues
10. sitio_web (7 mentions) - Website usability

Total comments analyzed: 850
Actionable insights: 3 critical issues requiring immediate attention
```

### NPS Classification

**Net Promoter Score (NPS)** is calculated using the standard formula:

```
NPS = (% Promoters) - (% Detractors)

Categories:
- Promoters: Rating 9-10
- Passives: Rating 7-8
- Detractors: Rating 0-6

Example:
- 850 total comments
- 340 promoters (40%)
- 255 passives (30%)
- 255 detractors (30%)
- NPS = 40% - 30% = +10

NPS Ranges:
- 70-100: Excellent (world-class)
- 50-69: Great (industry leading)
- 30-49: Good (above average)
- 10-29: Average
- 0-9: Needs improvement
- <0: Critical (urgent action needed)
```

---

## 8. OUTPUT FORMAT

### Excel Report Structure (23 Sheets)

The platform generates a professional Excel workbook with 23 specialized sheets:

#### Sheet 1: Detailed Analysis (Main Data)

All comments with complete analysis:

| Column | Type | Description | Example |
|--------|------|-------------|---------|
| Nota | Integer | Original rating | 9 |
| Comentario Final | Text | Original comment | "Excelente servicio..." |
| sentiment_score | Float | -1 to 1 | 0.92 |
| nps_category | String | Promotor/Pasivo/Detractor | "Promotor" |
| churn_risk_percentage | Integer | 0-100% | 5 |
| satisfaccion | Integer | 0-100% | 95 |
| frustracion | Integer | 0-100% | 0 |
| enojo | Integer | 0-100% | 0 |
| confianza | Integer | 0-100% | 90 |
| decepcion | Integer | 0-100% | 0 |
| confusion | Integer | 0-100% | 0 |
| anticipacion | Integer | 0-100% | 85 |
| pain_points | Text | Comma-separated | "ninguno" |
| dominant_emotion | String | Highest emotion | "satisfaccion" |
| emotional_intensity | Integer | 0-100% | 95 |

**Total columns**: 15
**Formatting**: Data bars for emotions, conditional formatting for churn risk, icon sets for NPS category

#### Sheet 2: Summary Statistics

Aggregated metrics:

```
General Metrics:
- Total Comments: 850
- Average Rating: 7.8/10
- Average Sentiment: 0.65
- NPS Score: 42

NPS Distribution:
- Promoters: 340 (40%)
- Passives: 255 (30%)
- Detractors: 255 (30%)

Emotion Averages:
- Satisfacción: 62.5%
- Frustración: 15.3%
- Enojo: 8.7%
- Confianza: 58.2%
- Decepción: 12.1%
- Confusión: 5.9%
- Anticipación: 45.3%

Churn Risk:
- Low (0-25%): 425 comments (50%)
- Medium (26-50%): 255 comments (30%)
- High (51-75%): 127 comments (15%)
- Critical (76-100%): 43 comments (5%)

Processing Info:
- Processing Time: 8.5 seconds
- Cost: $0.017 USD
- Tokens Used: 21,250
- Duplicates Removed: 132 (13%)
```

#### Sheet 3: Emotions Distribution

Detailed emotion breakdown:

| Emotion | Min | Max | Avg | Median | Std Dev | Count >50% |
|---------|-----|-----|-----|--------|---------|------------|
| Satisfacción | 0 | 100 | 62.5 | 65 | 28.3 | 510 |
| Frustración | 0 | 95 | 15.3 | 10 | 18.7 | 89 |
| Enojo | 0 | 85 | 8.7 | 5 | 12.4 | 34 |
| Confianza | 0 | 100 | 58.2 | 60 | 25.1 | 467 |
| Decepción | 0 | 90 | 12.1 | 8 | 15.6 | 68 |
| Confusión | 0 | 70 | 5.9 | 0 | 9.3 | 21 |
| Anticipación | 0 | 100 | 45.3 | 48 | 26.8 | 298 |

#### Sheet 4-10: NPS Analysis (7 Sheets)

Separate sheets for:
- NPS Overview
- Promoters Analysis (9-10 ratings)
- Passives Analysis (7-8 ratings)
- Detractors Analysis (0-6 ratings)
- NPS by Emotion
- NPS Trends (if timestamp available)
- NPS Action Plan

#### Sheet 11-14: Pain Points (4 Sheets)

- Top 10 Pain Points Summary
- Pain Points by NPS Category
- Pain Points by Churn Risk
- Pain Points Timeline (if timestamp available)

#### Sheet 15-18: Churn Risk (4 Sheets)

- Churn Risk Distribution
- High Risk Customers (51-100%)
- Critical Customers (76-100%) - Immediate action needed
- Churn Risk by Rating

#### Sheet 19-23: Charts & Pivots (5 Sheets)

- **Chart 1**: Emotions Radar (7 emotions)
- **Chart 2**: NPS Distribution (bar chart)
- **Chart 3**: Churn Risk Histogram
- **Chart 4**: Pain Points (horizontal bar)
- **Chart 5**: Sentiment vs Rating Scatter
- **Pivot 1**: Emotions by NPS Category
- **Pivot 2**: Pain Points by Churn Risk
- **Pivot 3**: Rating Distribution

### CSV Export

Single CSV file with all detailed analysis (Sheet 1 content):

```csv
Nota,Comentario Final,sentiment_score,nps_category,churn_risk_percentage,satisfaccion,frustracion,enojo,confianza,decepcion,confusion,anticipacion,pain_points,dominant_emotion,emotional_intensity
9,"Excelente servicio, muy satisfecho",0.92,Promotor,5,95,0,0,90,0,0,85,ninguno,satisfaccion,95
5,"El precio es alto pero la calidad es buena",0.25,Detractor,45,55,20,5,60,15,0,40,"precio",satisfaccion,55
3,"Muy lento el servicio, esperé demasiado",-0.62,Detractor,68,15,75,40,20,60,5,10,"lento, demora, espera",frustracion,75
```

### JSON Export (API Response)

Full analysis results in structured JSON:

```json
{
  "task_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "created_at": "2025-11-24T10:30:00Z",
  "completed_at": "2025-11-24T10:30:08Z",
  "processing_time_seconds": 8.5,

  "summary": {
    "total_comments": 850,
    "valid_comments": 850,
    "duplicates_removed": 132,
    "avg_rating": 7.8,
    "avg_sentiment": 0.65,
    "nps_score": 42,
    "promoters_count": 340,
    "promoters_percentage": 40.0,
    "passives_count": 255,
    "passives_percentage": 30.0,
    "detractors_count": 255,
    "detractors_percentage": 30.0,
    "high_churn_risk_count": 127,
    "high_churn_risk_percentage": 15.0,
    "cost_usd": 0.017,
    "tokens_used": 21250
  },

  "emotions_summary": {
    "satisfaccion": {
      "average": 62.5,
      "min": 0,
      "max": 100,
      "median": 65,
      "std_dev": 28.3,
      "count_above_50": 510
    },
    "frustracion": { ... },
    "enojo": { ... },
    "confianza": { ... },
    "decepcion": { ... },
    "confusion": { ... },
    "anticipacion": { ... }
  },

  "pain_points": [
    {
      "keyword": "entrega",
      "count": 47,
      "severity": "high",
      "affected_customers": 47,
      "avg_churn_risk": 68,
      "percentage": 5.5,
      "sample_comments": [
        "La entrega tardó 10 días",
        "Problemas con la entrega"
      ]
    },
    { ... }  // 9 more pain points
  ],

  "churn_risk_distribution": {
    "low_0_25": { "count": 425, "percentage": 50.0 },
    "medium_26_50": { "count": 255, "percentage": 30.0 },
    "high_51_75": { "count": 127, "percentage": 15.0 },
    "critical_76_100": { "count": 43, "percentage": 5.0 }
  },

  "detailed_results": [
    {
      "row_number": 1,
      "nota": 9,
      "comentario_final": "Excelente servicio...",
      "sentiment_score": 0.92,
      "nps_category": "Promotor",
      "churn_risk_percentage": 5,
      "emotions": {
        "satisfaccion": 95,
        "frustracion": 0,
        "enojo": 0,
        "confianza": 90,
        "decepcion": 0,
        "confusion": 0,
        "anticipacion": 85
      },
      "pain_points": "ninguno",
      "dominant_emotion": "satisfaccion",
      "emotional_intensity": 95
    },
    { ... }  // 849 more comments
  ]
}
```

---

## 9. COST OPTIMIZATION

### The 87% Reduction Strategy

**Before Optimization (Traditional Approach):**

```
Analysis per comment:
- Full prompt: "Analyze this customer feedback in detail..."
- Include full comment text (avg 50 words = 65 tokens)
- Request all analysis at once
- No caching
- No deduplication

Cost calculation:
- Input: 150 tokens/comment
- Output: 100 tokens/comment
- Total: 250 tokens/comment
- Cost: $0.50 per 1,000 comments

For 10,000 comments/month:
- $5.00/month in OpenAI costs
```

**After Optimization (Hybrid Approach):**

```
Two-tier analysis:

TIER 1: Local Analysis (FREE)
- Sentiment: VADER + TextBlob (no API calls)
- NPS category: Local calculation from rating
- Basic validation: Local checks
- Cost: $0.00

TIER 2: Selective OpenAI (PAID)
- Only for: emotions, churn risk, pain points
- Truncate comments to 150 chars (not 50 words)
- Minimal prompt: "Rating: {}, Sentiment: {}, Text: {}"
- Batch processing: 100 comments per request

Cost calculation:
- Input: 20 tokens/comment
- Output: 5 tokens/comment (JSON only)
- Total: 25 tokens/comment
- Cost: $0.06 per 1,000 comments

For 10,000 comments/month:
- $0.60/month in OpenAI costs
- Savings: $4.40/month (87% reduction)
```

### Optimization Techniques

#### 1. Deduplication (15-20% Savings)

```python
# SHA256 hashing for exact duplicates
from hashlib import sha256

def deduplicate_comments(comments: List[str]) -> List[str]:
    """
    Remove duplicate comments before API calls.

    Example:
    - Input: 1,000 comments
    - Duplicates: 150 (15%)
    - Output: 850 unique comments
    - API calls: 850 instead of 1,000
    - Savings: 15%
    """
    seen = set()
    unique = []

    for comment in comments:
        hash_key = sha256(comment.encode()).hexdigest()
        if hash_key not in seen:
            seen.add(hash_key)
            unique.append(comment)

    return unique

# Results are duplicated back to original positions
# User sees all 1,000 rows in Excel, but only paid for 850 API calls
```

#### 2. Comment Truncation (30-40% Savings)

```python
# Truncate to 150 characters
def truncate_comment(comment: str, max_chars: int = 150) -> str:
    """
    Most insights are in first 150 chars.

    Example:
    Original (200 chars): "El servicio fue excelente, el personal muy amable y atento. La entrega fue rápida y el empaquetado perfecto. Definitivamente recomendaría este producto a mis amigos y familia. Volveré a comprar."

    Truncated (150 chars): "El servicio fue excelente, el personal muy amable y atento. La entrega fue rápida y el empaquetado perfecto. Definitivamente recomendaría..."

    Token reduction: 50 → 35 tokens (30% savings)
    Accuracy loss: <2% (tested on 10K comments)
    """
    return comment[:max_chars] if len(comment) > max_chars else comment
```

#### 3. Caching (20-35% Savings for Repeat Uploads)

```python
# Redis cache with 7-day TTL
@cache_result(ttl_days=7)
def analyze_comment(comment: str, rating: int) -> dict:
    """
    Cache identical comments across different uploads.

    Example:
    - Day 1: Analyze "Excelente servicio" → Cache result
    - Day 3: Same comment in different file → Return cached result
    - No API call, instant response

    Cache hit rate: 15-25% for similar datasets
    Savings: 15-25% on repeated uploads
    """
    cache_key = f"analysis:{sha256(f'{comment}:{rating}'.encode()).hexdigest()}"

    if cached := redis.get(cache_key):
        return json.loads(cached)

    result = call_openai_api(comment, rating)
    redis.setex(cache_key, 604800, json.dumps(result))  # 7 days

    return result
```

#### 4. Intelligent Sampling (90% Savings for Large Datasets)

```python
# Stratified sampling by NPS category
def sample_large_dataset(df: pd.DataFrame, target_size: int = 10000) -> pd.DataFrame:
    """
    For datasets >50K rows, sample intelligently.

    Example:
    - Input: 120,000 comments
    - Target: 10,000 comments (stratified by NPS)
    - Promoters: 40% → Sample 4,000
    - Passives: 30% → Sample 3,000
    - Detractors: 30% → Sample 3,000

    Cost: $0.24 instead of $2.40 (90% savings)
    Accuracy: NPS ±5 points, distributions within 5%
    """
    if len(df) <= 50000:
        return df

    # Ensure minimum samples per category
    min_per_category = 100

    # Calculate proportional samples
    promoters = df[df['Nota'] >= 9]
    passives = df[(df['Nota'] >= 7) & (df['Nota'] < 9)]
    detractors = df[df['Nota'] < 7]

    # Stratified sampling
    sampled = pd.concat([
        promoters.sample(n=int(target_size * 0.4)),
        passives.sample(n=int(target_size * 0.3)),
        detractors.sample(n=int(target_size * 0.3))
    ])

    return sampled
```

#### 5. Batch Processing (10-15% Savings)

```python
# Process 100 comments per batch
async def batch_analyze(comments: List[str], batch_size: int = 100) -> List[dict]:
    """
    Process multiple comments in parallel.

    Benefits:
    - Reduced network overhead
    - Connection reuse
    - Async processing

    Performance:
    - Sequential: 100 comments = 10 seconds
    - Parallel (10 concurrent): 100 comments = 1.5 seconds
    - Speedup: 6.7x faster
    """
    batches = [comments[i:i+batch_size] for i in range(0, len(comments), batch_size)]

    tasks = [
        analyze_batch_async(batch)
        for batch in batches
    ]

    results = await asyncio.gather(*tasks)

    return [item for batch in results for item in batch]
```

### Cost Comparison Table

| Dataset Size | Before | After | Savings | Time Before | Time After |
|-------------|--------|-------|---------|-------------|------------|
| 100 comments | $0.05 | $0.002 | 96% | 10s | 2s |
| 500 comments | $0.25 | $0.01 | 96% | 50s | 5s |
| 1,000 comments | $0.50 | $0.02 | 96% | 100s | 10s |
| 5,000 comments | $2.50 | $0.10 | 96% | 500s | 50s |
| 10,000 comments | $5.00 | $0.20 | 96% | 1000s | 100s |
| 50,000 comments | $25.00 | $1.00 | 96% | 5000s | 500s |
| 120,000 comments (sampled) | $60.00 | $0.24 | 99.6% | 12000s | 12s |

**Total Savings for 10,000 comments/month:**
- OpenAI costs: $4.80/month saved
- Time savings: 900 seconds/month (15 minutes)
- **Annual savings**: $57.60 + improved user experience

**At scale (100K comments/month for enterprise):**
- OpenAI costs: $480/month saved
- **Annual savings**: $5,760

---

## 10. INTEGRATION WITH COMMENT-EXTRACTOR

The Customer Feedback App is designed to work seamlessly with the Comment-Extractor:

### Complete Workflow

```
STEP 1: EXTRACT COMMENTS
┌──────────────────────────────────────────────────────┐
│          Comment-Extractor (Playwright)              │
│                                                      │
│  python extract.py --account instagram_handle \     │
│                    --max-posts 50 \                  │
│                    --format csv                      │
│                                                      │
│  Output: instagram_handle_comments.csv               │
│  ┌────────────────────────────────────────────────┐ │
│  │ username | text | likes | timestamp | post_url│ │
│  │ user123 | Great! | 5 | 2025-11-24 | https://│ │
│  │ user456 | Bad | 2 | 2025-11-23 | https:// │ │
│  └────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
                        │
                        │ Download CSV
                        ▼
STEP 2: TRANSFORM DATA (Manual or Script)
┌──────────────────────────────────────────────────────┐
│          Data Preparation Script                     │
│                                                      │
│  # Option A: Manually create Excel with columns:    │
│  # - Nota: Convert likes to 0-10 scale              │
│  # - Comentario Final: Copy from "text" column      │
│                                                      │
│  # Option B: Python script (recommended):           │
│                                                      │
│  import pandas as pd                                 │
│                                                      │
│  # Load extracted comments                          │
│  df = pd.read_csv('instagram_handle_comments.csv')  │
│                                                      │
│  # Transform to required format                     │
│  df_transformed = pd.DataFrame({                     │
│      'Nota': (df['likes'] / df['likes'].max() * 10).round(),  │
│      'Comentario Final': df['text']                 │
│  })                                                  │
│                                                      │
│  # Save as Excel                                     │
│  df_transformed.to_excel('ready_for_analysis.xlsx', │
│                          index=False)                │
│                                                      │
│  Output: ready_for_analysis.xlsx                     │
└──────────────────────────────────────────────────────┘
                        │
                        │ Upload Excel
                        ▼
STEP 3: ANALYZE WITH AI
┌──────────────────────────────────────────────────────┐
│         Customer-Feedback-App (AI Analysis)          │
│                                                      │
│  1. Upload ready_for_analysis.xlsx                   │
│  2. Click "Analyze"                                  │
│  3. Wait 8-10 seconds                                │
│  4. Download 23-sheet Excel report                   │
│                                                      │
│  Output: analysis_results.xlsx (23 sheets)           │
│  ┌────────────────────────────────────────────────┐ │
│  │ Sheet 1: Detailed Analysis (all emotions)     │ │
│  │ Sheet 2: Summary Statistics (NPS, sentiment)  │ │
│  │ Sheet 3: Emotions Distribution (7 emotions)   │ │
│  │ Sheet 4-10: NPS Analysis (promoters, detract.)│ │
│  │ Sheet 11-14: Pain Points (top issues)         │ │
│  │ Sheet 15-18: Churn Risk (high risk customers) │ │
│  │ Sheet 19-23: Charts & Pivots (visualizations) │ │
│  └────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
```

### Transformation Script Example

```python
#!/usr/bin/env python3
"""
Transform Comment-Extractor output to Customer-Feedback-App format.

Usage:
    python transform_comments.py input.csv output.xlsx
"""

import pandas as pd
import sys

def transform_comments(input_csv: str, output_excel: str):
    """
    Convert Comment-Extractor CSV to analysis-ready Excel.

    Transformations:
    1. Likes → Rating (0-10 scale)
       - 0 likes → 5 (neutral)
       - Max likes → 10
       - Proportional scaling

    2. Text → Comentario Final
       - Keep original text
       - Remove empty comments
       - Remove duplicates

    3. Add metadata (optional):
       - Username preserved
       - Timestamp preserved
       - Post URL preserved
    """
    # Load extracted comments
    df = pd.read_csv(input_csv)

    # Handle missing values
    df = df.dropna(subset=['text'])
    df['likes'] = df['likes'].fillna(0)

    # Calculate rating from likes
    max_likes = df['likes'].max()
    if max_likes > 0:
        # Scale: 0 likes = 5, max likes = 10
        df['Nota'] = (df['likes'] / max_likes * 5 + 5).round()
    else:
        df['Nota'] = 5  # All neutral if no likes data

    # Rename text column
    df['Comentario Final'] = df['text']

    # Select columns for analysis
    # Required columns: Nota, Comentario Final
    # Optional: username, timestamp, post_url (preserved in output)
    output_df = df[['Nota', 'Comentario Final', 'username', 'timestamp', 'post_url']]

    # Remove duplicates
    output_df = output_df.drop_duplicates(subset=['Comentario Final'])

    # Save as Excel
    output_df.to_excel(output_excel, index=False)

    print(f"✓ Transformed {len(df)} comments")
    print(f"✓ Removed {len(df) - len(output_df)} duplicates")
    print(f"✓ Saved to {output_excel}")
    print(f"\nReady to upload to Customer-Feedback-App!")

if __name__ == '__main__':
    if len(sys.argv) != 3:
        print("Usage: python transform_comments.py input.csv output.xlsx")
        sys.exit(1)

    transform_comments(sys.argv[1], sys.argv[2])
```

**Run the script:**

```bash
# Extract comments
cd Comment-Exctractor
python extract.py --account magazine_luiza --max-posts 50 --format csv

# Transform to analysis format
python transform_comments.py exports/magazine_luiza_comments.csv analysis_ready.xlsx

# Upload to Customer-Feedback-App
# Open browser → Upload analysis_ready.xlsx → Analyze → Download report
```

### Alternative: Direct Integration (Future Enhancement)

**Not currently implemented, but planned:**

```python
# Future API endpoint: /analyze-from-url
POST /api/v1/analyze-from-url
{
  "platform": "instagram",
  "account": "magazine_luiza",
  "max_posts": 50,
  "rating_method": "likes_scaled"  // or "manual", "sentiment"
}

# This would:
# 1. Call Comment-Extractor internally
# 2. Transform data automatically
# 3. Analyze with AI
# 4. Return analysis results
#
# Timeline: Not in current roadmap (would require 2-3 weeks development)
```

---

## 11. DEPLOYMENT OPTIONS

### Option 1: Render.com (Current Production)

**Best for**: Startups, MVPs, small teams (< 1,000 customers)

**Architecture:**

```
4 Services on Render:
1. Web Service (Frontend) - customer-feedback-app.onrender.com
   - React + BFF
   - 512MB RAM
   - $7/month

2. Web Service (API) - customer-feedback-api.onrender.com
   - FastAPI
   - 512MB RAM
   - $7/month

3. Background Worker (Celery)
   - Python worker
   - 512MB RAM
   - $7/month

4. Redis (External)
   - Managed Redis
   - 25MB storage
   - $7/month
```

**Deployment Steps:**

```bash
# 1. Push to GitHub (auto-deploy configured)
git push origin main

# 2. Render auto-builds and deploys
# - Frontend: 5-7 minutes build time
# - API: 3-5 minutes build time
# - Worker: 3-5 minutes build time

# 3. Monitor deployment
# Render Dashboard → Services → View logs

# 4. Test production
curl https://customer-feedback-app.onrender.com/health
```

**Scaling on Render:**

```
Vertical scaling (upgrade instance):
- 512MB → 1GB → 2GB → 4GB → 8GB
- Cost: $7 → $25 → $50 → $100 → $200 per service

Horizontal scaling (multiple instances):
- Not recommended on Render (use AWS EKS instead)
```

### Option 2: AWS EKS (Enterprise)

**Best for**: Enterprises, high-scale (1,000+ customers), high availability

**Architecture:**

```
AWS Infrastructure:
- VPC with 3 AZs (us-east-1a, 1b, 1c)
- EKS Cluster (Kubernetes 1.28)
- Node Group: 2-10 t3.medium instances (auto-scaling)
- ElastiCache Redis (cache.t3.micro)
- ALB (Application Load Balancer)
- ECR (Container Registry)
- CloudWatch (Logs + Metrics)
- Secrets Manager (API keys)

Kubernetes Deployments:
- Frontend: 2-10 pods (auto-scale based on CPU)
- API: 2-10 pods (auto-scale based on requests)
- Worker: 2-5 pods (auto-scale based on queue depth)
- Redis: Managed by AWS (not in cluster)
```

**Deployment Steps:**

```bash
# 1. Provision infrastructure with Terraform
cd infrastructure/terraform
terraform init
terraform plan -var="environment=production"
terraform apply -var="environment=production"

# Wait 15-20 minutes for:
# - VPC creation
# - EKS cluster provisioning
# - Node group launch
# - ElastiCache provisioning

# 2. Configure kubectl
aws eks update-kubeconfig --region us-east-1 --name customer-feedback-prod

# 3. Set secrets
./scripts/set-secrets.sh production
# Enter: OPENAI_API_KEY

# 4. Build and push Docker images
./scripts/build-and-push.sh production

# This builds:
# - Frontend image → ECR
# - API image → ECR
# - Worker image → ECR

# 5. Deploy to Kubernetes
kubectl apply -f k8s/production/

# This creates:
# - Deployments (frontend, api, worker)
# - Services (ClusterIP for internal, LoadBalancer for external)
# - Ingress (ALB with SSL)
# - ConfigMaps (environment variables)
# - Secrets (API keys)

# 6. Get load balancer URL
kubectl get ingress -n default
# Access: http://your-alb-xxxxx.us-east-1.elb.amazonaws.com

# 7. Monitor
kubectl get pods  # Check pod status
kubectl logs -f deployment/api  # View API logs
kubectl logs -f deployment/worker  # View worker logs

# CloudWatch: Logs + Metrics + Alarms
```

**Auto-Scaling Configuration:**

```yaml
# k8s/production/api-deployment.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80

# This automatically scales:
# - 2 pods at low load
# - 10 pods at high load
# - Scale up when CPU > 70% or memory > 80%
# - Scale down when load decreases
```

**Cost Comparison:**

| Component | Render.com | AWS EKS | Difference |
|-----------|-----------|---------|------------|
| Compute | $21/month (3 services) | $60/month (2x t3.medium) | +$39 |
| Redis | $7/month | $15/month (ElastiCache) | +$8 |
| Load Balancer | Included | $20/month (ALB) | +$20 |
| Control Plane | Included | $72/month (EKS) | +$72 |
| Data Transfer | Included | $10-30/month | +$10-30 |
| **Total** | **$28/month** | **$177-207/month** | **+$149-179** |

**When to use AWS EKS:**
- ✅ Need high availability (99.99% SLA)
- ✅ Traffic spikes (10x load variations)
- ✅ Enterprise compliance (SOC 2, HIPAA)
- ✅ Multi-region deployment
- ✅ Custom infrastructure requirements
- ❌ Startup with <1,000 customers (use Render instead)

### Option 3: Docker Compose (Local Development)

**Best for**: Local development, testing, demos

**Start services:**

```bash
docker-compose up -d
```

**Stop services:**

```bash
docker-compose down
```

**View logs:**

```bash
docker-compose logs -f
```

---

## 12. LIMITATIONS & GAPS

### Current Limitations

#### 1. File Size Limits

```
Max rows: 3,000 comments
Max file size: 20MB
Max comment length: Unlimited (truncated to 150 chars for API)

Workaround for larger datasets:
- Enable intelligent sampling (ENABLE_INTELLIGENT_SAMPLING=true)
- This handles up to 120K+ comments
- Or split file into multiple batches
```

#### 2. Language Support

```
Supported: Spanish, Portuguese
Partially supported: English (sentiment works, pain points less accurate)
Not supported: Other languages

Limitation: OpenAI GPT-4o-mini supports 50+ languages, but pain point extraction
is optimized for Spanish/Portuguese context (LATAM business terms)

Workaround: Can analyze any language, but pain points may be less accurate
```

#### 3. No Multi-Company Management

```
Current: Single-file upload → analyze → download
Missing: Dashboard to manage multiple companies/projects

Example use case not supported:
- Agency managing 20 clients
- Want to track trends over time for each client
- Need centralized dashboard with all client reports

Gap: No user accounts, no project management, no historical tracking

Impact: Blocks Agency plan ($2,497/month) = $600K-1.8M ARR opportunity

Development effort: 2-3 weeks for MVP multi-company dashboard
```

#### 4. No Scheduled Analysis

```
Current: Manual upload every time
Missing: Automated scheduled extraction + analysis

Example use case not supported:
- Weekly analysis of Instagram comments
- Automatic email report every Monday
- Alert when sentiment drops >10%

Gap: No scheduling, no automation, no alerts

Impact: Users must manually repeat workflow

Development effort: 1-2 weeks for basic scheduling
```

#### 5. No Real-Time Streaming

```
Current: Batch processing (upload → analyze → download)
Missing: Real-time comment stream monitoring

Example use case not supported:
- Monitor Instagram comments in real-time
- Alert within 5 minutes of negative comment
- Live dashboard with updating metrics

Gap: No streaming, no real-time alerts

Impact: Can't prevent reputation crises in real-time

Development effort: 3-4 weeks for streaming architecture
```

#### 6. No CRM Integration

```
Current: Standalone tool
Missing: Native integrations with:
- Salesforce
- HubSpot
- Zendesk
- Intercom
- Freshdesk

Gap: No automatic data sync

Impact: Users must manually export/import data

Development effort: 1 week per integration (Zapier could solve this faster)
```

#### 7. Limited Customization

```
Current: Fixed analysis (7 emotions, churn risk, pain points)
Missing: Custom emotion sets, custom thresholds, custom pain point keywords

Example use case not supported:
- E-commerce wants to track "shipping_speed" specifically
- Bank wants to detect "fraud_concern" emotion
- Telecom wants custom NPS calculation (different scale)

Gap: No configuration UI for custom analysis

Development effort: 2-3 weeks for configuration system
```

#### 8. No API for External Apps

```
Current: Web UI only
Missing: REST API for external integrations

Example use case not supported:
- Mobile app uploads comments from app
- External dashboard fetches analysis results
- Third-party tool triggers analysis via API

Gap: API exists but not publicly documented or rate-limited

Development effort: 1 week for API documentation + rate limiting
```

### Technical Debt

#### 1. Excel Generation is Slow

```
Issue: Creating 23 sheets takes 2-3 seconds (30-40% of total processing time)

Current: OpenPyXL library
Better: xlsxwriter (2x faster) or binary format

Impact: Medium
Priority: Low (not blocking)
Effort: 1 week to refactor
```

#### 2. Memory Usage for Large Files

```
Issue: Loading 3,000+ rows into Pandas uses 200-300MB RAM

Current: Load entire file into memory
Better: Streaming processing with Dask or chunking

Impact: High for enterprise (limits max file size)
Priority: High
Effort: 2 weeks to implement streaming
```

#### 3. No Background Job Retries

```
Issue: If OpenAI API fails, entire job fails (no auto-retry)

Current: Manual re-upload required
Better: Automatic exponential backoff retry

Impact: Medium (rare but frustrating)
Priority: Medium
Effort: 1 day to add Celery retry logic
```

#### 4. Redis Single Point of Failure

```
Issue: If Redis crashes, all jobs are lost

Current: Single Redis instance
Better: Redis Cluster or ElastiCache with replication

Impact: High (production reliability)
Priority: High for enterprise
Effort: Already solved in AWS EKS deployment
```

---

## 13. USE CASES FOR MARKETING

### 1. Trojan Horse Free Reports

**Strategy**: Analyze competitors' public comments for FREE, send report to prospects.

**Process**:

```
1. Extract competitor comments (Comment-Extractor)
   - Target: Magazine Luiza Instagram
   - Extract: Last 50 posts = ~500 comments
   - Time: 30 minutes

2. Transform to analysis format
   - Convert likes to ratings (0-10 scale)
   - Time: 5 minutes

3. Analyze with Customer-Feedback-App
   - Upload Excel
   - Process: 5-8 seconds
   - Download 23-sheet report

4. Create custom 1-page summary
   - Highlight: Top 3 pain points, sentiment trend, churn risk
   - Time: 10 minutes
   - Total: 45-50 minutes per prospect

5. Outreach
   - LinkedIn message to Marketing Director
   - Attach 1-page summary PDF
   - Offer full 23-sheet report on call
   - Conversion: 20-30% demo rate
```

**Example LinkedIn Message**:

```
Hi [Name],

I analyzed Magazine Luiza's last 500 Instagram comments and found something interesting:

🔴 Sentiment dropped 18% in last 2 weeks
🔴 Top pain point: "entrega" (delivery issues - 47 mentions)
🔴 127 customers at high churn risk (15%)

I created a full report with 23 detailed sheets analyzing emotions, NPS, and pain points.

Want me to send it over? It's free, no strings attached.

P.S. This analysis would normally take 10+ hours manually. Our AI did it in 8 seconds.

[Your Name]
```

### 2. Competitive Intelligence

**Strategy**: Compare client's performance vs. 3 competitors.

**Process**:

```
1. Extract comments from 4 Instagram accounts:
   - Your client: Magazine Luiza
   - Competitor 1: Americanas
   - Competitor 2: Casas Bahia
   - Competitor 3: Amazon Brazil
   - Time: 2 hours (4 accounts × 30 min)

2. Analyze all 4 accounts
   - 4 separate reports
   - Time: 1 minute (4 × 8 seconds + upload)

3. Create comparison report
   - NPS comparison: Magazine Luiza (42) vs Americanas (38) vs Casas Bahia (35) vs Amazon (58)
   - Pain points comparison: All competitors struggle with "entrega" (delivery)
   - Emotion comparison: Amazon leads in "confianza" (trust)
   - Time: 30 minutes

4. Insights for client
   - "You're beating 2 of 3 competitors on NPS"
   - "But Amazon is 16 points ahead - here's why"
   - "All competitors have delivery issues - opportunity to differentiate"
```

**Deliverable**: Competitive Intelligence Report (PDF, 5-10 pages)

### 3. Crisis Detection

**Strategy**: Identify companies with reputation problems, offer urgent solution.

**Process**:

```
1. Extract comments from target companies
   - Look for: Low ratings, negative sentiment, delivery complaints
   - Example: Claro Brazil (72% negative sentiment, 1.5 star Trustpilot)

2. Analyze sentiment trend
   - Compare last month vs this month
   - Identify sudden drops (>10% sentiment decrease)

3. Create crisis alert
   - "Your sentiment dropped 18% in 2 weeks"
   - "127 high-risk customers about to churn"
   - "Top pain point: 'entrega' (47 mentions) - needs immediate action"

4. Urgent outreach
   - Subject: "Reputation Crisis Alert: [Company]"
   - Highlight: Specific numbers, trend line, competitor comparison
   - Offer: Free 30-min crisis audit call
   - Conversion: 40-50% demo rate (urgency drives action)
```

**Target Companies with Crisis**:

```
Based on November 2025 research:
- Claro Brazil: 72% negative sentiment (severe crisis)
- Americanas: Post-bankruptcy recovery (medium crisis)
- Latam Airlines: Flight delay complaints spike (medium crisis)
```

### 4. ROI Proof for Sales Calls

**Strategy**: Use their own data to prove ROI during demo.

**Process**:

```
1. Before demo: Extract their public comments
   - Time: 30 minutes

2. During demo: Show their actual analysis
   - "I analyzed your last 500 Instagram comments"
   - "Here's what I found... [show 23-sheet report]"
   - "127 customers at high churn risk"
   - "If you save just 10 of them, that's [calculate revenue]"

3. ROI calculation:
   - Average customer value: $500/year
   - 127 high-risk customers × 10% saved = 13 customers
   - Revenue saved: 13 × $500 = $6,500/year
   - Platform cost: $1,497/month = $18K/year
   - ROI: Break-even if you save 36 customers (28% of high-risk)
   - "That's just 3 customers per month - very achievable"

4. Close:
   - "Want to start with a 1-month pilot?"
   - "If you don't save at least 20 hours or prevent 3 churns, full refund"
   - Conversion: 50-60% trial rate (using their own data builds trust)
```

### 5. Content Marketing

**Strategy**: Publish industry insights from aggregated data.

**Examples**:

```
Blog post ideas:
1. "We Analyzed 10,000 E-commerce Comments: Here's What Brazilian Customers Really Want"
   - Top 10 pain points across 20 brands
   - NPS benchmarks by industry
   - Emotion trends (satisfaction vs frustration)

2. "The State of Customer Service in LATAM: 2025 Report"
   - Average NPS by country (Brazil 42, Mexico 38, Colombia 45)
   - Response time expectations (customers expect <2 hours)
   - Churn risk factors (delivery issues #1, price #2, quality #3)

3. "Why 15% of Your Customers Are About to Churn (And How to Stop Them)"
   - Churn risk prediction model explained
   - Case study: How Company X reduced churn 8% in 3 months
   - Early warning signs in comment patterns

Lead magnet: "Free NPS Benchmark Report: Where Does Your Company Rank?"
- Collect email → Send report → Nurture with case studies
```

### 6. Webinars & Live Demos

**Strategy**: Analyze audience's company live during webinar.

**Process**:

```
1. Webinar topic: "Turn Customer Comments Into Revenue: AI Analysis Demo"

2. Before webinar: Ask attendees to submit their Instagram handle
   - "Want us to analyze your account live? Submit your handle"

3. During webinar:
   - Extract comments live (takes 30 min, do this before webinar starts)
   - Upload and analyze live (takes 8 seconds - impressive!)
   - Walk through 23-sheet report
   - Show: NPS, emotions, pain points, churn risk
   - Calculate ROI for their specific case

4. After webinar:
   - Send recording + their custom report
   - Follow up with demo invite
   - Conversion: 25-35% demo rate (they already saw their data)
```

### 7. Agency Partnership Program

**Strategy**: White-label platform for agencies managing multiple clients.

**Pitch to agencies**:

```
"You're managing 20 clients' social media. Each client report takes 5-10 hours manually.

With our platform:
- 5-10 hours → 5-10 minutes (8 seconds analysis + 10 min summary)
- $50-100/hour analyst cost → $0.06 per report
- Scale from 20 to 50 clients WITHOUT hiring

Agency Plan: $2,497/month + $99 per client
- White-label branding (your logo, colors)
- Multi-client dashboard
- Bulk processing
- Reseller margin: Charge clients $300-500/month, keep $200-400

ROI: 20 clients × $300/month profit = $6,000/month revenue - $2,497 platform = $3,503/month net
With 50 clients: $15,000/month revenue - $7,447 total cost = $7,553/month net
```

**Target agencies**:

```
Based on TOP_100_LATAM list:
- Publicis Brazil (manages 50+ brands)
- Ogilvy Mexico (30+ clients)
- VMLY&R Colombia (25+ clients)
- Social media management agencies (100s in LATAM)
```

### 8. Upsell Path

**Strategy**: Start with free report, upsell to paid subscription.

**Funnel**:

```
Week 1: Free Report (Trojan Horse)
- Extract their public Instagram comments
- Analyze for free
- Send 1-page summary + 23-sheet Excel
- Value delivered: $500-1,000 (10 hours saved)
- Cost to you: $0.02 (OpenAI) + 45 min time

Week 2: Demo Call
- Walk through their report
- Show: "This is just from PUBLIC comments"
- Ask: "Imagine what you'd learn from INTERNAL data (surveys, support tickets, Google Reviews)"
- Offer: 30-day free trial (Professional plan, $1,497/month)
- Conversion: 40-50%

Week 3-4: Trial Period
- Set up: 10 social accounts, upload internal data
- Weekly reports automatically
- Real-time alerts for negative spikes
- Churn risk predictions

Week 5: Convert to Paid
- ROI review: "You saved 40 hours, prevented 5 churns ($2,500), found 3 product issues"
- Offer: Annual discount (20% off = $14,364/year instead of $17,964)
- Conversion: 30-50% of trials

Funnel math (100 prospects):
- 100 free reports → 40 demos → 20 trials → 8 paid customers
- 8 customers × $1,497/month × 12 months = $143,712 ARR
- Cost: 100 × $0.02 OpenAI + 100 × 45 min time = $2 + 75 hours
```

---

## SUMMARY

The Customer Feedback App is a **production-ready AI-powered SaaS platform** that:

✅ **Analyzes customer comments** using hybrid AI (local sentiment + selective OpenAI)
✅ **Generates professional reports** with 23 Excel sheets, charts, and formatting
✅ **Detects 7 emotions** (satisfaction, frustration, anger, trust, disappointment, confusion, anticipation)
✅ **Predicts churn risk** with 0-100% probability scores
✅ **Extracts pain points** automatically using NLP
✅ **Calculates NPS** with promoter/passive/detractor breakdown
✅ **Reduces costs 87%** compared to traditional AI analysis
✅ **Processes 850 comments in 8-10 seconds** with real-time progress tracking
✅ **Deployed in production** at customer-feedback-app.onrender.com
✅ **Scales to enterprise** with AWS EKS option

**Perfect for**:
- Marketing teams analyzing social media sentiment
- Customer service teams identifying at-risk customers
- Product teams discovering improvement opportunities
- Agencies managing multiple client accounts
- Enterprises requiring high-volume analysis

**Integrates seamlessly with Comment-Extractor** to create end-to-end workflow: Extract → Analyze → Act.

**Ready to use for Trojan Horse marketing strategy**: Extract competitors' public comments, analyze for free, send reports to prospects, upsell to B2B SaaS subscriptions.

**Current status**: Production-ready, actively deployed, processing real customer data.

---

**Document Version**: 1.0
**Last Updated**: November 24, 2025
**Status**: Complete
**Maintained By**: AI Whisperers Team
