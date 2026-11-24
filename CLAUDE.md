# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ASP.NET Core Web API on .NET 10 (SDK 10.0.x). The solution file is `TestProject.slnx` (XML `.slnx` format, not `.sln`) at the repo root; the only project is `AwesomeApi/AwesomeApi.csproj` (`Microsoft.NET.Sdk.Web`, nullable and implicit usings enabled).

## Commands

Run from the repo root:

```
dotnet restore TestProject.slnx
dotnet build TestProject.slnx -c Release
dotnet test TestProject.slnx -c Release
dotnet run --project AwesomeApi                          # http profile: http://localhost:5138
dotnet run --project AwesomeApi --launch-profile https   # https://localhost:7243
```

There are no test projects yet. When one is added, register it in the solution (`dotnet sln TestProject.slnx add <path>.csproj`) so CI picks it up. Run a single test with `dotnet test TestProject.slnx --filter "FullyQualifiedName~<TestName>"`.

`<UseAppHost>false</UseAppHost>` in the csproj is intentional and must stay. The dev machine has a Windows Application Control policy that blocks the unsigned `AwesomeApi.exe` apphost, so the app must start through `dotnet AwesomeApi.dll`. `dotnet run` does this automatically when no apphost exists.

Docker: the build context is the repo root (`.dockerignore` lives there), so run `docker build -f AwesomeApi/Dockerfile -t awesomeapi .` and then `docker run -p 8080:8080 awesomeapi`. The container listens on HTTP 8080, runs as the non-root `app` user, and uses the Production environment, so the OpenAPI document is not served unless you pass `-e ASPNETCORE_ENVIRONMENT=Development`.

## Architecture

- Endpoints are MVC controllers only. They live in `AwesomeApi/Controllers/`, are `[ApiController]` classes deriving from `ControllerBase`, and use explicit attribute routes such as `[Route("api/hello")]`. `Program.cs` (top-level statements) only registers services (`AddControllers`, `AddOpenApi`) and the pipeline (`UseHttpsRedirection`, `MapControllers`); no minimal API endpoints are mapped there.
- OpenAPI uses the built-in `Microsoft.AspNetCore.OpenApi` (`AddOpenApi` / `MapOpenApi`), not Swashbuckle. The document is served at `/openapi/v1.json`, and only in the Development environment.
- Configuration is in `appsettings.json` plus `appsettings.Development.json`. Both launch profiles set `ASPNETCORE_ENVIRONMENT=Development`.

## CI

`.github/workflows/ci.yml` runs on pushes and PRs to `main` (ubuntu-latest), and can also be started manually (`workflow_dispatch`). It restores, builds and tests `TestProject.slnx` in Release, then publishes `AwesomeApi` and uploads the output as the `AwesomeApi` artifact. The workflow token is limited to `contents: read`; add extra permissions per job only when a step needs them.
