# Full-Stack Analytics Dashboard — Replication Prompt
> Paste this entire document into a new Claude session as your first message.
> Claude will use it to guide you through building the full application.

---

## How to Use This Prompt

You are helping me build a **full-stack analytics reporting dashboard** from scratch,
modeled on a proven internal architecture. The architecture has already been battle-tested
in production. Your job is to replicate the same structural patterns, adapted to my
environment (AWS + Redshift).

You do NOT have access to the original codebase. Everything you need to know about the
architecture is described in this document. Where I haven't made a decision yet, ask me
and then propose options based on what is described here as the reference approach.

**Before writing any code, work through Section 0 (Decision Checkpoints) with me.**
Ask me each question, one at a time. Use my answers to tailor everything that follows.
If I say "I don't know" or "you decide," fall back to the reference approach described
in each section and explain why it was chosen.

---

## 0. Decision Checkpoints — Ask Me These First

Before generating any code, ask me these questions sequentially. Wait for each answer
before moving to the next.

1. **Project name and domain** — What is this dashboard called, and what business domain
   does it report on? (e.g., "pharmacy claims reporting", "member enrollment analytics")
   This determines naming conventions throughout the project.

2. **Redshift tables** — List the Redshift tables this dashboard will query, with their
   schemas (column names, types, partition/date columns). This is required to write
   repository and query layers.

3. **Primary date/partition column** — Which column(s) do users filter by date on?
   (e.g., `transaction_date`, `created_at`). Every query must filter on this column
   for performance.

4. **AWS deployment target** — Where will this run?
   - **ECS Fargate** (simpler, serverless containers, recommended if no K8s expertise)
   - **EKS** (Kubernetes, more control, needed if you have existing K8s infrastructure)
   - **App Runner** (simplest, good for low-traffic internal tools)
   - If unsure, recommend ECS Fargate and explain why.

5. **Redshift authentication** — How does the app authenticate to Redshift?
   - **IAM role** assumed by the ECS task/EC2 instance (recommended, no credentials
     in code)
   - **Username + password** stored in AWS Secrets Manager (acceptable fallback)
   - If unsure, recommend IAM + Secrets Manager for the connection string and explain
     the difference.

6. **Frontend authentication** — How do users log in?
   - Existing corporate SSO/OIDC provider (give me the issuer URL and client ID)
   - **Amazon Cognito** user pool
   - No auth (internal tool, network-restricted access only)
   - If unsure, ask whether the organization has a standard SSO and recommend that.

7. **What pages/views does the dashboard need?** — Describe each page or report view.
   For each, note what metrics it shows, what filters it has, and what chart types
   (trend lines, bar charts, tables, KPI numbers, maps). This drives the frontend
   component plan.

8. **KPI metrics** — List the key metrics users care about (counts, rates, averages,
   percentiles). These become the KPI cards at the top of each page.

---

## 1. Architecture Overview

The application is a **monorepo containing one backend and one frontend**, shipped as a
single Docker image. The FastAPI backend serves both the REST API and the compiled React
static files.

```
Browser (React SPA)
       │  HTTP/REST
       ▼
FastAPI (Python)   ← serves /api/* and static frontend files
       │
       ├── API Layer       (async route handlers, no business logic)
       ├── Service Layer   (async orchestration, parallel queries)
       ├── Repository Layer (sync data access, returns list[dict])
       ├── Query Constants (SQL strings as Python constants)
       │
       ▼
Redshift (read-only)
```

**Key principles baked into this architecture:**
- The backend is **async-first** (FastAPI + asyncio) but Redshift queries are **sync**
  (run in a thread pool to avoid blocking the event loop)
- All SQL is **parameterized** — never string-interpolated with user input
- Every query **must filter on the partition/date column** for performance
- Results are **cached in memory** for 24 hours; a background task pre-warms the cache
  at startup and once daily
- The frontend is a **compiled static SPA** — no SSR, no Next.js. It is served by
  FastAPI's `StaticFiles` mount in production.

---

## 2. Recommended Tech Stack

Use this stack unless you have a strong reason not to. Each choice was made
deliberately:

### Backend
| Package | Version | Why |
|---|---|---|
| `fastapi` | `>= 0.104` | Async-first, automatic OpenAPI docs, dependency injection |
| `uvicorn` | `>= 0.24` | ASGI server for local dev |
| `gunicorn` | `>= 21.2` | Production process manager (1 worker + UvicornWorker) |
| `pydantic-settings` | `>= 2.1` | Type-safe config from env vars |
| `cachetools` | `>= 5.3` | TTLCache for query result memoization |
| `redshift-connector` | `>= 2.1` | AWS-native Redshift driver (IAM-auth aware) |
| `psycopg2-binary` | `>= 2.9` | Alternative if using standard PostgreSQL wire protocol |
| `httpx` | `>= 0.27` | Async HTTP client for OAuth or external API calls |

> **`redshift-connector` vs `psycopg2`:** Prefer `redshift-connector` if using IAM
> authentication (no password, role-based). It handles IAM token exchange natively.
> Use `psycopg2` if you have a username/password from Secrets Manager — it is more
> widely supported and battle-tested.

### Frontend
| Package | Version | Why |
|---|---|---|
| `react` | `18.2` | Concurrent rendering, stable ecosystem |
| `typescript` | `5.3` | Type safety across all components and API contracts |
| `vite` | `5` | Fast HMR, optimized production builds |
| `tailwindcss` | `3.4` | Utility CSS — no separate CSS files, consistent spacing |
| `recharts` | `2.10` | Composable charting on top of D3, works well with React |
| `react-router-dom` | `6.21` | Client-side routing |
| `axios` | `1.15` | HTTP client with interceptors for auth headers |
| `date-fns` | `3.2` | Date manipulation without Moment.js bloat |
| `lucide-react` | `0.300` | Icon set |
| `react-simple-maps` | `3` | Geographic choropleth maps (optional, only if needed) |

---

## 3. Project Directory Structure

```
my-dashboard/
├── backend/
│   ├── app/
│   │   ├── main.py                  FastAPI app factory
│   │   ├── api/                     Route handlers (one file per domain)
│   │   │   └── overview.py
│   │   │   └── detail.py
│   │   │   └── health.py
│   │   ├── config/
│   │   │   ├── settings.py          Pydantic BaseSettings
│   │   │   └── queries/             SQL constant strings (one file per domain)
│   │   │       └── overview_queries.py
│   │   ├── database/
│   │   │   └── redshift_client.py   Singleton connection pool + cache
│   │   ├── repository/              Sync data access layer
│   │   │   └── overview_repository.py
│   │   ├── services/                Async orchestration layer
│   │   │   └── overview_service.py
│   │   ├── schemas/                 Pydantic request/response models
│   │   │   └── overview_schemas.py
│   │   ├── core/
│   │   │   └── cache_warmer.py      Background pre-warm task
│   │   └── exception/
│   │       └── handlers.py          Custom errors + FastAPI exception handlers
│   ├── Dockerfile
│   ├── requirements.txt
│   └── pytest.ini
├── frontend/
│   ├── src/
│   │   ├── api/
│   │   │   └── client.ts            Axios instance
│   │   ├── context/
│   │   │   └── AuthContext.tsx      SSO session state
│   │   ├── components/              Shared, reusable UI components
│   │   │   ├── FilterBar.tsx
│   │   │   ├── KPICard.tsx
│   │   │   ├── DataTable.tsx
│   │   │   ├── DrilldownPanel.tsx
│   │   │   └── charts/
│   │   │       ├── TrendChart.tsx
│   │   │       ├── BarChart.tsx
│   │   │       └── ChoroplethMap.tsx
│   │   ├── pages/                   One file per route/view
│   │   │   └── Overview.tsx
│   │   ├── router/
│   │   │   └── index.tsx
│   │   └── main.tsx
│   ├── public/
│   ├── index.html
│   ├── vite.config.ts
│   ├── tailwind.config.ts
│   └── tsconfig.json
├── deploy-configs/                  AWS deployment manifests
│   ├── ecs/
│   │   ├── task-definition.json
│   │   └── service.json
│   └── docker-compose.yml           Local dev only
├── .env.example
└── CLAUDE.md                        Architecture conventions for this project
```

---

## 4. Backend — Detailed Implementation

### 4.1 Configuration (`backend/app/config/settings.py`)

Use Pydantic `BaseSettings` with an app-specific env var prefix. All config comes from
environment variables — never hardcode values.

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # Redshift connection
    redshift_host: str
    redshift_port: int = 5439
    redshift_database: str
    redshift_schema: str = "public"
    redshift_user: str = ""           # empty when using IAM auth
    redshift_password: str = ""       # empty when using IAM auth
    redshift_iam: bool = True         # True = IAM role auth, False = user/password
    redshift_cluster_id: str = ""     # required for IAM auth

    # Table paths (fully qualified: schema.table_name)
    # Add one entry per table your dashboard queries
    my_main_table: str = "public.my_main_table"

    # Cache settings
    cache_ttl_seconds: int = 86400    # 24 hours
    cache_maxsize: int = 2048         # max number of cached query results

    # App settings
    cors_extra_origins: str = ""

    model_config = {"env_prefix": "APP_", "env_file": ".env"}

settings = Settings()
```

> **Why the prefix?** Using `APP_` (or your app name) avoids clashing with system
> env vars. All env vars in `.env.example` should start with this prefix.

---

### 4.2 Database Client (`backend/app/database/redshift_client.py`)

This is the most critical infrastructure file. The reference implementation uses a
**singleton pattern with a TTL cache and a concurrency semaphore**. Here is the pattern,
adapted for Redshift:

**The problem it solves:**
- Redshift queries are synchronous (blocking I/O). If you call them directly in an
  async FastAPI handler, you block the event loop.
- Repeated identical queries (e.g., the dashboard loading the same 30-day trend on
  every page visit) waste Redshift slots.
- Too many simultaneous queries can exhaust Redshift's WLM (Workload Management) queue.

**The solution:**
1. A **thread pool** (`ThreadPoolExecutor`) — sync queries run in worker threads, not
   the event loop.
2. A **TTLCache** (`cachetools`) — query results are cached by an MD5 key of
   `(sql, params)` for 24 hours.
3. A **semaphore** — limits how many concurrent queries can run at once.

```python
import hashlib
import threading
from concurrent.futures import ThreadPoolExecutor
from typing import Any

import redshift_connector
from cachetools import TTLCache

from app.config.settings import settings

_client: redshift_connector.Connection | None = None
_lock = threading.Lock()
_cache: TTLCache = TTLCache(
    maxsize=settings.cache_maxsize,
    ttl=settings.cache_ttl_seconds,
)
_cache_lock = threading.Lock()

# Semaphore: max concurrent Redshift queries. Tune this to your WLM queue slot count.
import asyncio
_semaphore: asyncio.Semaphore | None = None

def get_semaphore() -> asyncio.Semaphore:
    global _semaphore
    if _semaphore is None:
        _semaphore = asyncio.Semaphore(10)   # tune for your Redshift WLM config
    return _semaphore


def _get_client() -> redshift_connector.Connection:
    """Thread-safe singleton Redshift connection."""
    global _client
    if _client is None:
        with _lock:
            if _client is None:
                if settings.redshift_iam:
                    # IAM role authentication — no password needed
                    _client = redshift_connector.connect(
                        iam=True,
                        host=settings.redshift_host,
                        database=settings.redshift_database,
                        cluster_identifier=settings.redshift_cluster_id,
                        db_user=settings.redshift_user or "admin",
                        port=settings.redshift_port,
                    )
                else:
                    # Username + password (from Secrets Manager)
                    _client = redshift_connector.connect(
                        host=settings.redshift_host,
                        database=settings.redshift_database,
                        user=settings.redshift_user,
                        password=settings.redshift_password,
                        port=settings.redshift_port,
                    )
    return _client


def _cache_key(sql: str, params: dict) -> str:
    raw = sql + str(sorted(params.items()))
    return hashlib.md5(raw.encode()).hexdigest()


def run_query(sql: str, params: dict[str, Any] | None = None) -> list[dict[str, Any]]:
    """Sync. Checks cache first. Runs query in calling thread."""
    params = params or {}
    key = _cache_key(sql, params)

    with _cache_lock:
        if key in _cache:
            return _cache[key]

    conn = _get_client()
    cursor = conn.cursor()
    cursor.execute(sql, params)
    columns = [desc[0] for desc in cursor.description]
    rows = [dict(zip(columns, row)) for row in cursor.fetchall()]
    cursor.close()

    with _cache_lock:
        _cache[key] = rows

    return rows


async def run_query_async(
    sql: str,
    params: dict[str, Any] | None = None,
    executor: ThreadPoolExecutor | None = None,
) -> list[dict[str, Any]]:
    """Async wrapper. Respects semaphore, runs sync query in thread pool."""
    import asyncio
    async with get_semaphore():
        loop = asyncio.get_event_loop()
        return await loop.run_in_executor(executor, run_query, sql, params)
```

> **Note on connection pooling:** The above uses a single connection. For production
> with concurrent users, consider using `redshift_connector`'s built-in connection
> pooling or `SQLAlchemy` with a connection pool. Ask if you want to implement pooling.

> **Note on `psycopg2` alternative:** If you use `psycopg2`, replace
> `redshift_connector.connect(...)` with `psycopg2.connect(host=..., dbname=...,
> user=..., password=..., port=5439)`. The rest of the pattern is identical.
> Use `%s` placeholders instead of named `%(param)s` style depending on your
> parameterization approach.

---

### 4.3 Query Constants (`backend/app/config/queries/`)

Store every SQL string as a Python module-level constant. **No SQL anywhere else in the
codebase.** This makes queries reviewable, testable, and easy to find.

```python
# backend/app/config/queries/overview_queries.py

DAILY_SUMMARY = """
    SELECT
        DATE_TRUNC('day', {date_col}) AS period,
        COUNT(*) AS total_records,
        SUM(your_metric_col) AS total_metric,
        AVG(your_numeric_col) AS avg_value
    FROM {schema}.{table}
    WHERE {date_col} >= DATEADD(day, -%(days)s, CURRENT_DATE)
      AND {date_col} < CURRENT_DATE
    GROUP BY 1
    ORDER BY 1
"""

SUMMARY_BY_CATEGORY = """
    SELECT
        category_col,
        COUNT(*) AS record_count,
        SUM(metric_col) AS total
    FROM {schema}.{table}
    WHERE {date_col} >= DATEADD(day, -%(days)s, CURRENT_DATE)
    GROUP BY 1
    ORDER BY 2 DESC
    LIMIT %(limit)s
"""
```

> **Rules for every query:**
> 1. Always filter on the partition/date column (required for performance)
> 2. Always use `%(param_name)s` placeholders — never f-string user input
> 3. Table names (from settings) can be interpolated with `.format()` at module level
>    since they come from config, not user input
> 4. Keep queries in the query file; never build SQL dynamically in repository or
>    service layers

---

### 4.4 Repository Layer (`backend/app/repository/`)

Pure sync functions. They call `run_query()` and return `list[dict[str, Any]]`.
**No async, no domain logic, no transformation** — just raw row dicts.

```python
# backend/app/repository/overview_repository.py
from app.database.redshift_client import run_query
from app.config.queries.overview_queries import DAILY_SUMMARY
from app.config.settings import settings

def daily_summary(days: int) -> list[dict]:
    sql = DAILY_SUMMARY.format(
        schema=settings.redshift_schema,
        table=settings.my_main_table,
        date_col="your_date_column",
    )
    return run_query(sql, {"days": days})
```

---

### 4.5 Service Layer (`backend/app/services/`)

Async functions. Orchestrate one or more repository calls (often in parallel using
`asyncio.gather`). Apply business logic, compute derived metrics, shape the response.

```python
# backend/app/services/overview_service.py
import asyncio
from app.repository import overview_repository as repo

async def get_summary(days: int) -> dict:
    # Run multiple queries in parallel
    summary, by_category = await asyncio.gather(
        asyncio.to_thread(repo.daily_summary, days),
        asyncio.to_thread(repo.summary_by_category, days, limit=10),
    )
    return {
        "summary": summary,
        "by_category": by_category,
    }
```

> **`asyncio.to_thread` vs `run_in_executor`:** Use `asyncio.to_thread` (Python 3.9+)
> for simple cases. Use `run_in_executor` with a named `ThreadPoolExecutor` if you need
> fine-grained control over the thread pool size (e.g., capping to 40 workers).

---

### 4.6 API Layer (`backend/app/api/`)

Async route handlers. Zero business logic — delegate to service or directly to
`asyncio.to_thread(repo.method)` for simple queries.

```python
# backend/app/api/overview.py
from fastapi import APIRouter, Query
from app.services.overview_service import get_summary

router = APIRouter(prefix="/api/overview", tags=["overview"])

@router.get("/summary")
async def summary(days: int = Query(default=30, ge=1, le=365)):
    return await get_summary(days)
```

---

### 4.7 App Entry Point (`backend/app/main.py`)

```python
from contextlib import asynccontextmanager
from concurrent.futures import ThreadPoolExecutor

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from fastapi.staticfiles import StaticFiles
from fastapi.responses import FileResponse

from app.api import overview, health
from app.config.settings import settings
from app.core.cache_warmer import start_cache_warmer

_executor: ThreadPoolExecutor | None = None

@asynccontextmanager
async def lifespan(app: FastAPI):
    global _executor
    _executor = ThreadPoolExecutor(max_workers=40)
    # Store executor so it can be used in run_query_async
    app.state.executor = _executor
    # Start cache pre-warmer background task
    warmer_task = asyncio.create_task(start_cache_warmer())
    yield
    warmer_task.cancel()
    _executor.shutdown(wait=False)

app = FastAPI(title="My Dashboard API", lifespan=lifespan)

# CORS — restrict to your actual frontend origin in production
origins = ["http://localhost:5173"]
if settings.cors_extra_origins:
    origins += settings.cors_extra_origins.split(",")

app.add_middleware(CORSMiddleware, allow_origins=origins, allow_methods=["*"],
                   allow_headers=["*"])

# API routers
app.include_router(overview.router)
app.include_router(health.router)

# Serve compiled React SPA from /static (populated by Docker multi-stage build)
app.mount("/static", StaticFiles(directory="static"), name="static")

@app.get("/{full_path:path}")
async def spa_fallback(full_path: str):
    return FileResponse("static/index.html")
```

---

### 4.8 Cache Warmer (`backend/app/core/cache_warmer.py`)

**Why it exists:** Dashboard users expect near-instant page loads. If every user
triggers a cold Redshift query, the first person each morning waits 5–15 seconds.
The warmer fires HTTP GET requests to all API endpoints at startup and at 09:00 UTC
daily, so the cache is always warm.

```python
import asyncio
import logging
from datetime import datetime, timezone

import httpx

logger = logging.getLogger(__name__)

# All endpoint URL patterns to pre-warm. Add every dashboard endpoint here.
# These must match your actual API routes.
WARM_URLS = [
    "http://localhost:7072/api/overview/summary?days=7",
    "http://localhost:7072/api/overview/summary?days=30",
    "http://localhost:7072/api/overview/summary?days=90",
    # Add all day/filter permutations for each endpoint
]

async def _warm_batch(client: httpx.AsyncClient, urls: list[str], batch_size: int = 20):
    semaphore = asyncio.Semaphore(batch_size)
    async def fetch(url: str):
        async with semaphore:
            try:
                await client.get(url, timeout=60.0)
            except Exception as e:
                logger.warning("Cache warm failed for %s: %s", url, e)
    await asyncio.gather(*[fetch(u) for u in urls])


async def start_cache_warmer():
    await asyncio.sleep(10)   # let server fully start
    while True:
        logger.info("Cache warmer starting")
        async with httpx.AsyncClient() as client:
            await _warm_batch(client, WARM_URLS)
        logger.info("Cache warmer done")
        # Sleep until next 09:00 UTC
        now = datetime.now(timezone.utc)
        next_run = now.replace(hour=9, minute=0, second=0, microsecond=0)
        if next_run <= now:
            next_run = next_run.replace(day=next_run.day + 1)
        await asyncio.sleep((next_run - now).total_seconds())
```

> **Sizing `cache_maxsize`:** Count your total URL combinations (endpoints × days
> options × filter permutations). If you have 20 endpoints × 4 day options × 3 filter
> values = 240 combinations, set `cache_maxsize` to at least 512 to leave headroom.
> Setting it too low causes older entries to evict before daily re-warm runs.

---

### 4.9 Exception Handling (`backend/app/exception/handlers.py`)

```python
from fastapi import Request
from fastapi.responses import JSONResponse

class DatabaseError(Exception):
    def __init__(self, message: str):
        self.message = message

class ExternalAPIError(Exception):
    def __init__(self, message: str, status_code: int = 502):
        self.message = message
        self.status_code = status_code

async def database_error_handler(request: Request, exc: DatabaseError):
    return JSONResponse(status_code=500, content={"detail": exc.message})

async def external_api_error_handler(request: Request, exc: ExternalAPIError):
    return JSONResponse(status_code=exc.status_code, content={"detail": exc.message})

# Register in main.py:
# app.add_exception_handler(DatabaseError, database_error_handler)
```

---

## 5. Frontend — Detailed Implementation

### 5.1 Project Setup

```bash
npm create vite@latest frontend -- --template react-ts
cd frontend
npm install axios react-router-dom recharts date-fns lucide-react
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

`tailwind.config.ts`:
```ts
export default {
  content: ["./index.html", "./src/**/*.{ts,tsx}"],
  theme: { extend: {} },
  plugins: [],
}
```

`src/main.tsx`:
```tsx
import React from "react"
import ReactDOM from "react-dom/client"
import { RouterProvider } from "react-router-dom"
import { router } from "./router"
import "./index.css"

ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>
)
```

---

### 5.2 API Client (`src/api/client.ts`)

```ts
import axios from "axios"

const client = axios.create({
  baseURL: import.meta.env.VITE_API_URL ?? "",
  timeout: 30_000,
})

// Attach auth token from session storage if using SSO
client.interceptors.request.use((config) => {
  const token = sessionStorage.getItem("access_token")
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})

export default client
```

Create typed API functions per domain:
```ts
// src/api/overview.ts
import client from "./client"

export interface DailySummaryRow {
  period: string
  total_records: number
  total_metric: number
  avg_value: number
}

export const fetchDailySummary = (days: number) =>
  client.get<DailySummaryRow[]>(`/api/overview/summary?days=${days}`)
    .then(r => r.data)
```

---

### 5.3 Authentication (`src/context/AuthContext.tsx`)

> **Ask the developer:** What SSO/auth system are they using?
>
> - **Corporate OIDC (e.g., Okta, Ping, Azure AD):** Use `oidc-client-ts` or
>   `@auth0/auth0-react`. Store the access token in `sessionStorage`. Attach it
>   as a `Bearer` token on every API call. The reference app uses this approach —
>   redirect to the SSO login page if no session exists, then redirect back.
> - **Amazon Cognito:** Use `@aws-amplify/auth` or `amazon-cognito-identity-js`.
>   Same pattern — redirect to Cognito Hosted UI, receive token on callback.
> - **No auth (network-restricted):** Remove `AuthContext` and `ProtectedRoute`
>   entirely. Only do this if the app is only accessible on a VPN or private network.

Minimal OIDC context (adapt to your provider):
```tsx
import React, { createContext, useContext, useState, useEffect } from "react"

interface AuthState {
  token: string | null
  isAuthenticated: boolean
  login: () => void
  logout: () => void
}

const AuthContext = createContext<AuthState | null>(null)

export const AuthProvider = ({ children }: { children: React.ReactNode }) => {
  const [token, setToken] = useState<string | null>(
    sessionStorage.getItem("access_token")
  )

  const login = () => {
    // Redirect to your SSO provider's authorization URL
    // Replace with your OIDC issuer URL + client_id + redirect_uri
    window.location.href = `${import.meta.env.VITE_SSO_URL}/authorize?...`
  }

  const logout = () => {
    sessionStorage.removeItem("access_token")
    setToken(null)
  }

  return (
    <AuthContext.Provider value={{ token, isAuthenticated: !!token, login, logout }}>
      {children}
    </AuthContext.Provider>
  )
}

export const useAuth = () => {
  const ctx = useContext(AuthContext)
  if (!ctx) throw new Error("useAuth must be inside AuthProvider")
  return ctx
}
```

---

### 5.4 Component Pattern Library

These are the reusable building blocks. Build these first — all pages compose from them.

---

#### `FilterBar.tsx`

The global filter bar appears at the top of every page. It controls the date range and
any domain-specific filters (category, region, status, etc.). It passes values down
via props or a shared context.

```tsx
// src/components/FilterBar.tsx
interface FilterBarProps {
  days: number
  onDaysChange: (days: number) => void
  granularity: "daily" | "weekly" | "monthly"
  onGranularityChange: (g: "daily" | "weekly" | "monthly") => void
  // Add domain-specific filter props here as you discover them
}

const DAY_OPTIONS = [7, 14, 30, 90]
const GRANULARITY_OPTIONS = ["daily", "weekly", "monthly"] as const

export const FilterBar = ({
  days, onDaysChange, granularity, onGranularityChange
}: FilterBarProps) => (
  <div className="flex items-center gap-4 p-4 bg-white border-b border-gray-200">
    <div className="flex items-center gap-2">
      <span className="text-sm text-gray-600 font-medium">Period:</span>
      {DAY_OPTIONS.map(d => (
        <button
          key={d}
          onClick={() => onDaysChange(d)}
          className={`px-3 py-1 rounded text-sm ${
            days === d
              ? "bg-blue-600 text-white"
              : "bg-gray-100 text-gray-700 hover:bg-gray-200"
          }`}
        >
          {d}d
        </button>
      ))}
    </div>
    <div className="flex items-center gap-2">
      <span className="text-sm text-gray-600 font-medium">View:</span>
      {GRANULARITY_OPTIONS.map(g => (
        <button
          key={g}
          onClick={() => onGranularityChange(g)}
          className={`px-3 py-1 rounded text-sm capitalize ${
            granularity === g
              ? "bg-blue-600 text-white"
              : "bg-gray-100 text-gray-700 hover:bg-gray-200"
          }`}
        >
          {g}
        </button>
      ))}
    </div>
  </div>
)
```

---

#### `KPICard.tsx`

Displays a single key metric with an optional week-over-week delta indicator. Used in a
grid row at the top of every page.

```tsx
// src/components/KPICard.tsx
import { TrendingUp, TrendingDown, Minus } from "lucide-react"

interface KPICardProps {
  title: string
  value: string | number       // formatted: "1,234" or "98.5%"
  delta?: number               // percentage change: +5.2 or -3.1
  deltaLabel?: string          // e.g., "vs last week"
  subtitle?: string            // optional context line
}

export const KPICard = ({ title, value, delta, deltaLabel, subtitle }: KPICardProps) => {
  const trendColor = delta == null ? "gray"
    : delta > 0 ? "green"
    : delta < 0 ? "red"
    : "gray"

  const TrendIcon = delta == null ? Minus : delta > 0 ? TrendingUp : TrendingDown

  return (
    <div className="bg-white rounded-lg border border-gray-200 p-5 shadow-sm">
      <p className="text-sm text-gray-500 font-medium">{title}</p>
      <p className="text-3xl font-bold text-gray-900 mt-1">{value}</p>
      {delta != null && (
        <div className={`flex items-center gap-1 mt-2 text-sm text-${trendColor}-600`}>
          <TrendIcon size={14} />
          <span>{Math.abs(delta).toFixed(1)}% {deltaLabel ?? ""}</span>
        </div>
      )}
      {subtitle && <p className="text-xs text-gray-400 mt-1">{subtitle}</p>}
    </div>
  )
}
```

Usage on any page:
```tsx
<div className="grid grid-cols-4 gap-4 p-4">
  <KPICard title="Total Records" value="12,345" delta={+3.2} deltaLabel="vs last week" />
  <KPICard title="Success Rate" value="98.5%" delta={-0.3} deltaLabel="vs last week" />
</div>
```

---

#### `TrendChart.tsx`

Recharts line chart for time-series data. Accepts generic `data` and configurable
`lines` so it works for any metric without code changes.

```tsx
// src/components/charts/TrendChart.tsx
import {
  LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer
} from "recharts"

export interface LineConfig {
  key: string        // matches a key in your data objects
  label: string      // shown in legend and tooltip
  color: string      // hex or tailwind color
}

interface TrendChartProps {
  data: Record<string, unknown>[]
  lines: LineConfig[]
  xKey: string       // key for the x-axis (usually a date string)
  height?: number
  yFormatter?: (v: number) => string
}

export const TrendChart = ({
  data, lines, xKey, height = 300, yFormatter
}: TrendChartProps) => (
  <ResponsiveContainer width="100%" height={height}>
    <LineChart data={data} margin={{ top: 5, right: 20, left: 0, bottom: 5 }}>
      <CartesianGrid strokeDasharray="3 3" stroke="#f0f0f0" />
      <XAxis dataKey={xKey} tick={{ fontSize: 12 }} />
      <YAxis tickFormatter={yFormatter} tick={{ fontSize: 12 }} />
      <Tooltip formatter={(v, name) => [yFormatter ? yFormatter(Number(v)) : v, name]} />
      <Legend />
      {lines.map(l => (
        <Line
          key={l.key}
          type="monotone"
          dataKey={l.key}
          name={l.label}
          stroke={l.color}
          dot={false}
          strokeWidth={2}
        />
      ))}
    </LineChart>
  </ResponsiveContainer>
)
```

Usage:
```tsx
<TrendChart
  data={summaryRows}
  xKey="period"
  lines={[
    { key: "total_records", label: "Total Records", color: "#3b82f6" },
    { key: "total_metric",  label: "Metric",        color: "#10b981" },
  ]}
  yFormatter={(v) => v.toLocaleString()}
/>
```

---

#### `DataTable.tsx`

Generic sortable table. Pass column definitions and row data — it handles rendering,
sorting, and optional row click.

```tsx
// src/components/DataTable.tsx
import { useState } from "react"
import { ChevronUp, ChevronDown } from "lucide-react"

export interface Column<T> {
  key: keyof T
  label: string
  render?: (value: T[keyof T], row: T) => React.ReactNode
  sortable?: boolean
  align?: "left" | "right" | "center"
}

interface DataTableProps<T> {
  columns: Column<T>[]
  data: T[]
  onRowClick?: (row: T) => void
  emptyMessage?: string
}

export function DataTable<T extends Record<string, unknown>>({
  columns, data, onRowClick, emptyMessage = "No data"
}: DataTableProps<T>) {
  const [sortKey, setSortKey] = useState<keyof T | null>(null)
  const [sortDir, setSortDir] = useState<"asc" | "desc">("desc")

  const sorted = sortKey
    ? [...data].sort((a, b) => {
        const av = a[sortKey], bv = b[sortKey]
        return sortDir === "asc"
          ? av < bv ? -1 : av > bv ? 1 : 0
          : av > bv ? -1 : av < bv ? 1 : 0
      })
    : data

  const handleSort = (key: keyof T) => {
    if (sortKey === key) setSortDir(d => d === "asc" ? "desc" : "asc")
    else { setSortKey(key); setSortDir("desc") }
  }

  return (
    <div className="overflow-x-auto rounded-lg border border-gray-200">
      <table className="w-full text-sm">
        <thead className="bg-gray-50 text-gray-600 uppercase text-xs">
          <tr>
            {columns.map(col => (
              <th
                key={String(col.key)}
                className={`px-4 py-3 font-medium text-${col.align ?? "left"} ${
                  col.sortable ? "cursor-pointer hover:bg-gray-100 select-none" : ""
                }`}
                onClick={() => col.sortable && handleSort(col.key)}
              >
                <div className="flex items-center gap-1">
                  {col.label}
                  {col.sortable && sortKey === col.key && (
                    sortDir === "asc" ? <ChevronUp size={14} /> : <ChevronDown size={14} />
                  )}
                </div>
              </th>
            ))}
          </tr>
        </thead>
        <tbody className="divide-y divide-gray-100">
          {sorted.length === 0 ? (
            <tr><td colSpan={columns.length} className="px-4 py-8 text-center text-gray-400">{emptyMessage}</td></tr>
          ) : sorted.map((row, i) => (
            <tr
              key={i}
              className={`bg-white ${onRowClick ? "cursor-pointer hover:bg-blue-50" : ""}`}
              onClick={() => onRowClick?.(row)}
            >
              {columns.map(col => (
                <td key={String(col.key)} className={`px-4 py-3 text-${col.align ?? "left"}`}>
                  {col.render ? col.render(row[col.key], row) : String(row[col.key] ?? "")}
                </td>
              ))}
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  )
}
```

---

#### `DrilldownPanel.tsx`

A slide-in side panel for showing detail when a user clicks a row. Keeps the main page
visible in the background.

```tsx
// src/components/DrilldownPanel.tsx
import { X } from "lucide-react"

interface DrilldownPanelProps {
  isOpen: boolean
  onClose: () => void
  title: string
  children: React.ReactNode
  width?: string   // tailwind width class, default "w-96"
}

export const DrilldownPanel = ({
  isOpen, onClose, title, children, width = "w-96"
}: DrilldownPanelProps) => (
  <>
    {/* Backdrop */}
    {isOpen && (
      <div
        className="fixed inset-0 bg-black/20 z-40"
        onClick={onClose}
      />
    )}
    {/* Panel */}
    <div className={`fixed right-0 top-0 h-full ${width} bg-white shadow-xl z-50
      transform transition-transform duration-300 ${isOpen ? "translate-x-0" : "translate-x-full"}`}>
      <div className="flex items-center justify-between p-4 border-b border-gray-200">
        <h2 className="font-semibold text-gray-900">{title}</h2>
        <button onClick={onClose} className="text-gray-400 hover:text-gray-600">
          <X size={20} />
        </button>
      </div>
      <div className="p-4 overflow-y-auto h-[calc(100%-57px)]">
        {children}
      </div>
    </div>
  </>
)
```

---

#### `ChoroplethMap.tsx` (optional — geographic data only)

> **Ask the developer:** Do any reports show geographic data (by state, region,
> territory)? If yes, use this component. If no, skip it entirely.

```tsx
// src/components/charts/ChoroplethMap.tsx
import { useState } from "react"
import { ComposableMap, Geographies, Geography } from "react-simple-maps"
import { scaleSequential } from "d3-scale"
import { interpolateBlues } from "d3-scale-chromatic"

// US states topojson — host this file in your /public folder
const GEO_URL = "/us-states.json"

interface MapDataPoint {
  state: string        // two-letter state code
  value: number
}

interface ChoroplethMapProps {
  data: MapDataPoint[]
  metricLabel: string
}

export const ChoroplethMap = ({ data, metricLabel }: ChoroplethMapProps) => {
  const [tooltip, setTooltip] = useState<{ state: string; value: number } | null>(null)

  const valueMap = Object.fromEntries(data.map(d => [d.state, d.value]))
  const max = Math.max(...data.map(d => d.value), 1)
  const colorScale = scaleSequential(interpolateBlues).domain([0, max])

  return (
    <div className="relative">
      <ComposableMap projection="geoAlbersUsa">
        <Geographies geography={GEO_URL}>
          {({ geographies }) =>
            geographies.map(geo => {
              const stateCode = geo.properties.postal
              const value = valueMap[stateCode] ?? 0
              return (
                <Geography
                  key={geo.rsmKey}
                  geography={geo}
                  fill={colorScale(value)}
                  stroke="#fff"
                  strokeWidth={0.5}
                  onMouseEnter={() => setTooltip({ state: geo.properties.name, value })}
                  onMouseLeave={() => setTooltip(null)}
                  style={{ hover: { opacity: 0.8 } }}
                />
              )
            })
          }
        </Geographies>
      </ComposableMap>
      {tooltip && (
        <div className="absolute top-4 left-4 bg-white border border-gray-200 rounded p-2 shadow text-sm">
          <strong>{tooltip.state}</strong>: {tooltip.value.toLocaleString()} {metricLabel}
        </div>
      )}
    </div>
  )
}
```

---

#### `LoadingSpinner.tsx` + `ErrorMessage.tsx`

Every async data fetch needs loading and error states. Build these once:

```tsx
// src/components/LoadingSpinner.tsx
export const LoadingSpinner = () => (
  <div className="flex items-center justify-center py-12">
    <div className="w-8 h-8 border-4 border-blue-600 border-t-transparent rounded-full animate-spin" />
  </div>
)

// src/components/ErrorMessage.tsx
interface ErrorMessageProps { message: string; onRetry?: () => void }
export const ErrorMessage = ({ message, onRetry }: ErrorMessageProps) => (
  <div className="bg-red-50 border border-red-200 rounded-lg p-4 text-red-800">
    <p className="font-medium">Something went wrong</p>
    <p className="text-sm mt-1">{message}</p>
    {onRetry && (
      <button onClick={onRetry} className="mt-2 text-sm text-red-600 underline">
        Retry
      </button>
    )}
  </div>
)
```

---

### 5.5 Page Architecture Pattern

Every page follows the same structure. The developer should build one page end-to-end
first as a reference, then replicate the pattern.

```tsx
// src/pages/Overview.tsx  (template — rename and adapt for each page)
import { useState, useEffect } from "react"
import { FilterBar } from "../components/FilterBar"
import { KPICard } from "../components/KPICard"
import { TrendChart } from "../components/charts/TrendChart"
import { DataTable } from "../components/DataTable"
import { LoadingSpinner } from "../components/LoadingSpinner"
import { ErrorMessage } from "../components/ErrorMessage"
import { fetchDailySummary, type DailySummaryRow } from "../api/overview"

export const OverviewPage = () => {
  const [days, setDays] = useState(30)
  const [granularity, setGranularity] = useState<"daily"|"weekly"|"monthly">("daily")
  const [data, setData] = useState<DailySummaryRow[]>([])
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  const load = async () => {
    setLoading(true)
    setError(null)
    try {
      const rows = await fetchDailySummary(days)
      setData(rows)
    } catch (e) {
      setError(e instanceof Error ? e.message : "Failed to load data")
    } finally {
      setLoading(false)
    }
  }

  useEffect(() => { load() }, [days, granularity])

  const totalRecords = data.reduce((s, r) => s + r.total_records, 0)

  return (
    <div className="min-h-screen bg-gray-50">
      <FilterBar
        days={days} onDaysChange={setDays}
        granularity={granularity} onGranularityChange={setGranularity}
      />
      <div className="p-6 space-y-6">
        {/* KPI row */}
        <div className="grid grid-cols-4 gap-4">
          <KPICard title="Total Records" value={totalRecords.toLocaleString()} />
          {/* Add more KPIs here */}
        </div>

        {/* Chart */}
        {loading ? <LoadingSpinner /> : error ? <ErrorMessage message={error} onRetry={load} /> : (
          <div className="bg-white rounded-lg border border-gray-200 p-5">
            <h2 className="font-semibold text-gray-900 mb-4">Trend</h2>
            <TrendChart
              data={data}
              xKey="period"
              lines={[{ key: "total_records", label: "Records", color: "#3b82f6" }]}
            />
          </div>
        )}

        {/* Table */}
        {!loading && !error && (
          <div className="bg-white rounded-lg border border-gray-200 p-5">
            <h2 className="font-semibold text-gray-900 mb-4">Detail</h2>
            <DataTable
              columns={[
                { key: "period",        label: "Date",    sortable: true },
                { key: "total_records", label: "Records", sortable: true, align: "right",
                  render: v => Number(v).toLocaleString() },
              ]}
              data={data}
            />
          </div>
        )}
      </div>
    </div>
  )
}
```

---

### 5.6 Router (`src/router/index.tsx`)

```tsx
import { createBrowserRouter } from "react-router-dom"
import { OverviewPage } from "../pages/Overview"
// Import other pages as you add them

export const router = createBrowserRouter([
  { path: "/",         element: <OverviewPage /> },
  // { path: "/detail", element: <DetailPage /> },
])
```

---

## 6. Deployment

### 6.1 Multi-Stage Dockerfile

The reference approach produces a **single Docker image** that:
1. Builds the React frontend
2. Installs Python dependencies
3. Copies the compiled frontend into `backend/static/`
4. Runs FastAPI which serves both the API and the static files

```dockerfile
# Stage 1 — Build React frontend
FROM node:20-slim AS frontend-build
WORKDIR /frontend
COPY frontend/package*.json ./
RUN npm ci
COPY frontend/ ./
RUN npm run build
# Output: /frontend/dist/

# Stage 2 — Install Python dependencies
FROM python:3.11-slim AS py-build
WORKDIR /build
COPY backend/requirements.txt ./
RUN pip install --no-cache-dir --prefix=/deps -r requirements.txt

# Stage 3 — Production runtime
FROM python:3.11-slim AS runtime
WORKDIR /app

# Non-root user
RUN useradd -m -u 1000 app
USER app

# Copy Python deps
COPY --from=py-build /deps /usr/local
# Copy backend source
COPY --chown=app:app backend/app/ ./app/
# Copy compiled frontend into static/
COPY --from=frontend-build --chown=app:app /frontend/dist/ ./static/

EXPOSE 7072
CMD ["gunicorn", "app.main:app", "-w", "1", "-k", "uvicorn.workers.UvicornWorker",
     "--bind", "0.0.0.0:7072", "--timeout", "120"]
```

---

### 6.2 Local Development (`docker-compose.yml`)

```yaml
version: "3.9"
services:
  backend:
    build:
      context: .
      dockerfile: backend/Dockerfile
    ports:
      - "8000:7072"
    env_file: .env
    volumes:
      - ./backend/app:/app/app   # hot reload in dev

  frontend:
    build:
      context: frontend
      dockerfile: Dockerfile     # separate lightweight nginx image for local dev
    ports:
      - "5173:80"
    environment:
      - VITE_API_URL=http://localhost:8000
```

For local dev without Docker:
```bash
# Terminal 1 — backend
cd backend && uvicorn app.main:app --reload --port 8000

# Terminal 2 — frontend
cd frontend && VITE_API_URL=http://localhost:8000 npm run dev
```

---

### 6.3 AWS Deployment Options

> **Ask the developer which platform they are targeting.** Based on their answer, follow
> the matching section below.

---

#### Option A — ECS Fargate (Recommended if no K8s)

**Why:** No infrastructure to manage. AWS handles cluster, networking, and scaling.
Good for internal dashboards with moderate traffic.

**Task Definition (`deploy-configs/ecs/task-definition.json`):**
```json
{
  "family": "my-dashboard",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::ACCOUNT:role/ecsTaskExecutionRole",
  "taskRoleArn":      "arn:aws:iam::ACCOUNT:role/my-dashboard-task-role",
  "containerDefinitions": [{
    "name": "app",
    "image": "ACCOUNT.dkr.ecr.REGION.amazonaws.com/my-dashboard:latest",
    "portMappings": [{ "containerPort": 7072, "protocol": "tcp" }],
    "environment": [
      { "name": "APP_REDSHIFT_HOST",       "value": "your-cluster.region.redshift.amazonaws.com" },
      { "name": "APP_REDSHIFT_DATABASE",   "value": "your_db" },
      { "name": "APP_REDSHIFT_IAM",        "value": "true" },
      { "name": "APP_REDSHIFT_CLUSTER_ID", "value": "your-cluster-id" }
    ],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/my-dashboard",
        "awslogs-region": "us-east-1",
        "awslogs-stream-prefix": "ecs"
      }
    },
    "healthCheck": {
      "command": ["CMD-SHELL", "curl -f http://localhost:7072/health/live || exit 1"],
      "interval": 30,
      "timeout": 5,
      "retries": 3
    }
  }]
}
```

**IAM task role** needs:
- `redshift:GetClusterCredentials`
- `redshift:DescribeClusters`
- `redshift-data:ExecuteStatement` (if using Redshift Data API instead of direct
  connection)

Put the task role ARN in `taskRoleArn`. The app inherits the role — no credentials
in code or environment variables.

Add an **Application Load Balancer** in front of ECS for HTTPS termination and routing.

---

#### Option B — EKS (Kubernetes)

Use this if your organization already runs EKS. The architecture is equivalent to the
GKE reference — map GKE concepts to EKS:

| GKE (reference) | EKS (your target) |
|---|---|
| Workload Identity | EKS IRSA (IAM Roles for Service Accounts) |
| Istio ingress | AWS Load Balancer Controller + Ingress |
| Artifact Registry | ECR |
| ArgoCD | ArgoCD (same tool, works on EKS) |
| GKE HPA | K8s HPA (identical) |

**Deployment (`deploy-configs/eks/deployment.yaml`):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-dashboard
spec:
  replicas: 2
  template:
    spec:
      serviceAccountName: my-dashboard-sa   # annotated with IRSA role ARN
      containers:
        - name: app
          image: ACCOUNT.dkr.ecr.REGION.amazonaws.com/my-dashboard:latest
          ports:
            - containerPort: 7072
          envFrom:
            - configMapRef:
                name: my-dashboard-config
          livenessProbe:
            httpGet: { path: /health/live, port: 7072 }
            initialDelaySeconds: 30
            periodSeconds: 30
          resources:
            requests: { cpu: "250m", memory: "512Mi" }
            limits:   { cpu: "1000m", memory: "1Gi" }
```

---

#### Option C — AWS App Runner (Simplest)

**Use when:** Small team, low traffic, no infra expertise, just want it running fast.

App Runner pulls directly from ECR and runs the container. Add env vars in the App
Runner service configuration console. No task definitions, no load balancers.
Limitation: no fine-grained IAM task roles — use Secrets Manager for Redshift credentials.

---

### 6.4 Health Check Endpoint

Add this to your API layer. Required by all AWS deployment targets:

```python
# backend/app/api/health.py
from fastapi import APIRouter
router = APIRouter()

@router.get("/health/live")
async def liveness():
    return {"status": "ok"}

@router.get("/health/ready")
async def readiness():
    # Optionally: try a lightweight Redshift query here
    return {"status": "ok"}
```

---

### 6.5 Environment Variables Reference

Document all env vars in `.env.example`. Every secret goes in **AWS Secrets Manager**
(not in the Docker image, not in version control):

```bash
# .env.example
# Redshift
APP_REDSHIFT_HOST=my-cluster.abc123.us-east-1.redshift.amazonaws.com
APP_REDSHIFT_PORT=5439
APP_REDSHIFT_DATABASE=analytics
APP_REDSHIFT_SCHEMA=public
APP_REDSHIFT_IAM=true
APP_REDSHIFT_CLUSTER_ID=my-cluster
# Used only if APP_REDSHIFT_IAM=false:
# APP_REDSHIFT_USER=
# APP_REDSHIFT_PASSWORD=

# Tables (fully qualified schema.table)
APP_MY_MAIN_TABLE=public.my_main_table

# Frontend
VITE_API_URL=https://my-dashboard.internal.company.com

# Auth
VITE_SSO_URL=https://your-sso-provider.com
```

Sensitive values (passwords, client secrets) must never appear in `.env.example`.
Reference them by name only with an empty value.

---

## 7. CLAUDE.md — Architectural Conventions

Create this file at the repo root. It documents the rules Claude and developers must
follow when working in this codebase:

```markdown
# Architecture Conventions

## Layer rules
- SQL lives ONLY in `config/queries/`. Never write SQL in repository, service, or API files.
- Repository functions are ALWAYS sync (`def`, not `async def`). Return `list[dict]` only.
- Service functions are ALWAYS async. Call repos via `asyncio.to_thread()`.
- API functions are ALWAYS async. No business logic — delegate to service or repo.

## Query rules
- Every query MUST filter on the partition/date column. No full table scans.
- NEVER interpolate user input into SQL strings. Use parameterized queries only.
- Table names from settings may be `.format()`-ted in query constants.

## Cache rules
- Never bypass the cache. All queries go through `run_query()` in `redshift_client.py`.
- If a query must skip the cache (live data), add a `skip_cache: bool = False` param
  to `run_query()` and document why.

## Frontend rules
- All API calls go through `src/api/client.ts`. No direct `fetch()` calls.
- All page-level data fetching uses `useState` + `useEffect`. No global state manager.
- Every async call must handle loading and error states.
- Never hardcode API URLs. Always use `import.meta.env.VITE_API_URL`.
```

---

## 8. Getting Started — First Steps

After you have answered the questions in Section 0, implement in this order:

1. **Backend scaffold** — `settings.py`, `redshift_client.py`, `main.py`, health endpoint
2. **One domain end-to-end** — pick your most important table → write queries → repository → service → API endpoint → test with `curl`
3. **Frontend scaffold** — Vite setup, `client.ts`, routing, `FilterBar`, `KPICard`
4. **First page** — wire the one domain from step 2 to a page with KPI cards and a trend chart
5. **Remaining domains** — repeat steps 2–4 for each table/report
6. **Cache warmer** — add after all endpoints exist; populate `WARM_URLS` with every endpoint + filter permutation
7. **Dockerfile** — multi-stage build, verify frontend is served correctly by FastAPI
8. **Deployment** — choose ECS/EKS/App Runner; write task definition or manifests
9. **Auth** — add `AuthContext` and `ProtectedRoute` last, after the app is working
