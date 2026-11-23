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
dotnet run --project AwesomeApi                      # http profile: http://localhost:5138
dotnet run --project AwesomeApi --launch-profile https   # https://localhost:7243
```

There are no test projects yet. When one is added, register it in the solution (`dotnet sln TestProject.slnx add <path>.csproj`) so CI picks it up. Run a single test with `dotnet test TestProject.slnx --filter "FullyQualifiedName~<TestName>"`.

`AwesomeApi/AwesomeApi.http` has sample requests for the VS Code / Visual Studio REST client.

## Architecture

- Minimal APIs: all setup lives in `AwesomeApi/Program.cs` (top-level statements). There are no controllers. Endpoints are mapped with `app.MapGet(...)`, and DTOs are records declared in the same file.
- OpenAPI uses the built-in `Microsoft.AspNetCore.OpenApi` (`AddOpenApi` / `MapOpenApi`), not Swashbuckle. The document is served at `/openapi/v1.json`, and only in the Development environment.
- Configuration is in `appsettings.json` plus `appsettings.Development.json`. Both launch profiles set `ASPNETCORE_ENVIRONMENT=Development`.

## CI

`.github/workflows/ci.yml` runs on pushes and PRs to `main` (ubuntu-latest). It restores, builds and tests `TestProject.slnx` in Release, then publishes `AwesomeApi` and uploads the output as the `AwesomeApi` artifact. The workflow token is limited to `contents: read`; add extra permissions per job only when a step needs them.
