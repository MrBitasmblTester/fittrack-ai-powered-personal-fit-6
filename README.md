# FitTrack – AI-Powered Personal Fit 6 — title

Description

FitTrack is a web and mobile platform that uses AI to generate personalized workout and nutrition plans based on user goals, preferences, and real-time health tracking data. The system separates responsibilities across a Go-based AI/ML microservice (inference & data processing) and an ASP.NET Core GraphQL API (API orchestration, authentication, and client-facing endpoints).

## Tech Stack

- Go (AI/ML, inference microservice)
- GraphQL (API schema and client-facing API)
- ASP.NET Core (GraphQL server, authentication, orchestration)

## Requirements

- Produce a full GitHub README.md following the provided structure.
- Refer specifically to the selected tech stack: Go, GraphQL, ASP.NET Core.
- Do not invent unrelated technologies.
- Do not generalize — always refer to the specific stack.
- Do not skip files normally required for the chosen stacks (e.g., go.mod, main.go, schema.graphql, Program.cs, appsettings.json).

## Installation

This project is composed of two components:
1) Go AI Engine (ai-engine)
2) ASP.NET Core GraphQL server (api)

Prerequisites

- Go 1.20+ installed (go command available)
- .NET 7.0 SDK or later (dotnet CLI available)
- Git

Setup Go AI Engine

1. Create project directory and initialize module:

   cd ai-engine
   go mod init github.com/yourorg/fittrack/ai-engine

2. Create main entry file (main.go). Ensure required files exist: go.mod, main.go.

3. Build and run locally:

   go build -o ai-engine ./
   ./ai-engine

Environment variables for Go AI Engine (examples)

- AI_ENGINE_PORT=8081
- AI_MODEL_PATH=/path/to/model (if using local model files)
- AI_LOG_LEVEL=info

Setup ASP.NET Core GraphQL Server

1. Create the ASP.NET Core project:

   mkdir api && cd api
   dotnet new web -n FitTrack.Api
   cd FitTrack.Api

2. Add a GraphQL server package (recommended: HotChocolate):

   dotnet add package HotChocolate.AspNetCore
   dotnet add package HotChocolate.Data

3. Ensure required files exist: Program.cs, appsettings.json, schema.graphql (or inline schema via HotChocolate).

4. Run the API locally:

   dotnet run

Environment variables for ASP.NET Core API (examples)

- ASPNETCORE_ENVIRONMENT=Development
- AI_ENGINE_URL=http://localhost:8081 (points to Go AI engine)
- JWT_SECRET=your_jwt_secret_here
- GRAPHQL_PATH=/graphql


## Usage

1. Start the Go AI Engine:

   cd ai-engine
   ./ai-engine

2. Start the ASP.NET Core GraphQL server:

   cd api/FitTrack.Api
   dotnet run

3. Open GraphQL playground (HotChocolate Banana Cake Pop or POST to GraphQL endpoint):

   POST http://localhost:5000/graphql

Example GraphQL query (generate plan):

query GeneratePlan($input: GeneratePlanInput!) {
  generatePlan(input: $input) {
    id
    createdAt
    workouts {
      day
      exercises {
        name
        sets
        reps
      }
    }
    nutrition {
      calories
      macros {
        protein
        carbs
        fat
      }
    }
  }
}

Variables (example):

{
  "input": {
    "userId": "user-123",
    "goals": "fat_loss",
    "preferences": { "diet": "vegetarian" },
    "healthMetrics": { "weight": 78.5, "restingHeartRate": 62 }
  }
}

The ASP.NET Core GraphQL resolvers will forward inference requests to the Go AI Engine (AI_ENGINE_URL) which returns structured plan data.

## Implementation Steps

1. Create repository layout with two top-level folders: /ai-engine (Go) and /api (ASP.NET Core).
2. Initialize Go module (go.mod) and implement main.go; expose a simple HTTP JSON API (e.g., POST /v1/infer) for plan generation and a health endpoint (GET /health).
3. Implement AI/ML inference logic inside the Go service. Provide a clear interface: accept user goals, preferences, and health-tracking payloads; return structured workout and nutrition plans. Add logging, config via environment variables, and graceful shutdown.
4. Create ASP.NET Core project (Program.cs, Startup configuration) and add HotChocolate for GraphQL. Define GraphQL schema (schema.graphql or code-first types).
5. Implement GraphQL types and resolvers in ASP.NET Core. Resolvers should validate requests, optionally apply auth checks (JWT), and call the Go AI Engine via HTTP (AI_ENGINE_URL) to get plan data.
6. Add authentication middleware in ASP.NET Core (JWT bearer) and secure the GraphQL endpoint. Configure JWT_SECRET and other secrets via environment variables or appsettings.json.
7. Implement integration tests for the GraphQL API that mock the Go AI Engine (or run an integration instance).
8. Add CI checks to ensure builds for both Go and .NET succeed, run unit tests, and run static analysis/linting for both stacks.
9. Provide example client queries, docs, and a Postman/HTTP example to call the GraphQL endpoint.
10. Prepare production configuration: set AI_ENGINE_URL to the production Go service, configure secure secrets storage for JWT_SECRET and model paths.

(Optional) Additional files normally required

- ai-engine/go.mod, ai-engine/main.go, ai-engine/handlers.go
- api/FitTrack.Api/Program.cs, api/FitTrack.Api/Startup.cs or equivalent, api/appsettings.json
- schema.graphql or C# schema definitions
- README.md (this file)


## API Endpoints

- GraphQL endpoint (ASP.NET Core):
  - POST /graphql  - main GraphQL entrypoint for queries and mutations
  - GET /graphql (Banana Cake Pop UI when enabled in Development)

- Go AI Engine endpoints:
  - POST /v1/infer  - accepts JSON payload { userId, goals, preferences, healthMetrics } and returns { plan }
  - GET /health  - returns 200 OK for health checks

Example Go inference request

POST http://localhost:8081/v1/infer
Content-Type: application/json

{
  "userId": "user-123",
  "goals": "muscle_gain",
  "preferences": { "equipment": "dumbbells" },
  "healthMetrics": { "weight": 72, "sleepHours": 7 }
}

Response (example):

{
  "planId": "plan-789",
  "workouts": [ ... ],
  "nutrition": { ... }
}


Notes

- Keep AI/ML model files and heavy inference artifacts outside the API project. The Go AI Engine is responsible for model loading and inference.
- Choose HotChocolate or another .NET GraphQL library per team preference; this README uses HotChocolate as an example.
- Store secrets securely in production (key vaults, environment variables on the host, or secret managers).