## gofins

> Use "go run . db sql -q <query>" in gofins/ to execute sql queries. The gofins binary knows best how to connect to the database, which is running inside a docker.


**Database**

Use "go run . db sql -q <query>" in gofins/ to execute sql queries. The gofins binary knows best how to connect to the database, which is running inside a docker.

Use development/db_export_schema.sh to export db schema.

We use sqlc to generate go code from sql queries:

gofins/sqlc.sh generates go code from gofins/pkg/db/schema.sql, gofins/sqlc.yaml contains the configuration.

# Architecture

Always pass through context.Context - as "ctx". We want all worker threads to react to shutdown signals.

use the TODO.md in project root to see what's to be done - you can also write into it, but don't be verbose, keep it short - it's the users list,  not yours.

# Seperation of Concerns / Architecture

gofins/pkg/fmp - adapter to fmp api, doesn't do any business logic, just API access, retries, conversion to internal format - it's a simple adapter.
gofins/pkg/calculator - functional layer, math and business logic, doesn't access db, doesn't access fmp, doesn't access any other package, just pure math 
gofins/pkg/db - database access, just database access (with a minimum of business logic, like aggregation), retries, conversion to internal format - it's a simple adapter.
gofins/pkg/types - structs, just structs, no methods, no functions 
gofins/pkg/forex - tool functions for forex conversion, optimized for speed, uses memory caching
gofins/pkg/f - a toolbox of generic pure functions, no module specific logic, no side effects
gofins/pkg/updates - update logic for fetching data from data providers (FMP) and updating the memory and database

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/flocko-motion) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
