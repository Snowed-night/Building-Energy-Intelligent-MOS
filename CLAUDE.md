# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**建筑能源智能管理系统 (Building Energy Intelligent MOS)** — A web system for building energy consumption query, statistics, and intelligent operations. Competition project using Java Spring Boot + React Ant Design + PostgreSQL + Elasticsearch + DeepSeek API + MCP protocol.

Only the `Backend/` module is currently implemented. Frontend, MCP, and AI/RAG modules are planned per `../project-plan.md`.

## Backend Commands

All commands run from `Backend/`:

```bash
# Run the application
./mvnw spring-boot:run

# Build (skip tests)
./mvnw package -DskipTests

# Run all tests
./mvnw test

# Run a single test class
./mvnw test -Dtest=BackendApplicationTests

# Build JAR
./mvnw clean package
```

**Prerequisites:** PostgreSQL running at `localhost:5432/energy_mos`, user `postgres`, password `123456`.

After startup: API at `http://localhost:8888`, Swagger UI at `http://localhost:8888/swagger-ui.html`.

Database schema is auto-initialized via `src/main/resources/init_tables.sql`. JPA DDL is set to `update`.

## Architecture

### Backend Layer Structure

Standard Spring Boot MVC layers under `src/main/java/org/example/backend/`:

- **`controller/`** — REST endpoints; delegates to services with no business logic
- **`service/`** — All business logic
- **`repository/`** — Spring Data JPA repositories; `EnergyRecordRepository` extends `JpaSpecificationExecutor` for dynamic filtering
- **`entity/`** — JPA entities: `Building`, `MonitorDevice`, `EnergyRecord`
- **`dto/`** — Request/response DTOs: `EnergyRecordRequest`, `TimeSummaryDto`, `AnomalyDto`, `CopDto`
- **`common/`** — `ApiResponse<T>` (unified response wrapper with `code`/`message`/`data`) and `GlobalExceptionHandler`

### Three Controllers

| Controller | Responsibilities |
|---|---|
| `EnergyController` | Building/device list, energy record CRUD, CSV import |
| `StatisticsController` | Time-series aggregation, COP stats, anomaly detection |
| `ReportController` | CSV export via EasyExcel |

### Key Business Logic

- **Time-series aggregation** — SQL `date_trunc` via JPQL, supporting hourly/daily/monthly granularity
- **COP calculation** — `COP = 制冷量 / hvac_kwh`, cooling capacity estimated from HVAC supply/return temperature differential
- **Anomaly detection** — Z-Score method; records with `|z| > 3σ` flagged as anomalies
- **CSV import** — `EnergyService.importFromCsv()` uses Apache Commons CSV; maps column headers to entity fields
- **Dynamic filtering** — `EnergyService` builds JPA `Specification` predicates for buildingId, deviceId, time range, device status

### Database

Core table is `energy_record` with 12 measurement fields. Compound index on `(building_id, record_time)` is critical for time-range queries. See `src/main/resources/init_tables.sql` for full schema and sample data.

### Planned Modules (not yet implemented)

Per `../project-plan.md`:
- **`mcp/`** — Spring AI MCP Server exposing energy tools for LLM access
- **`ai/`** — DeepSeek API integration + Elasticsearch RAG knowledge base (hybrid BM25 + kNN retrieval)
- **Frontend** — React + Ant Design + ECharts, Zustand state management, SSE for streaming AI responses
- **`data-scripts/`** — Python scripts for dataset generation (≥1000 records)
- **`docker-compose.yml`** — One-command startup for all services

## API Protocol

See `../接口协议文档.md` for the complete API contract (Chinese). All responses follow `ApiResponse<T>` with structure: `{ code, message, data }`.
