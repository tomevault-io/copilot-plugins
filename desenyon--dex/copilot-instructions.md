## dex

> Generates matching and non matching examples.

# AGENT.md

# Dex

Dex is a single binary developer command center built in Go.

It is not a basic utility collection. It is an interactive terminal operating layer for developers that combines networking, process intelligence, API tooling, JSON tooling, system diagnostics, regex engineering, benchmarking, files, clipboard, and terminal visualization.

Dex must feel fast, beautiful, local first, deeply useful, and powerful enough that developers install it on every machine.

## Core Principles

1. One binary.
2. No background daemon required by default.
3. Every command works in plain CLI mode.
4. Every major module also has an interactive TUI mode.
5. Output must support human, JSON, CSV, Markdown, and raw modes.
6. No AI features for now.
7. No logs module.
8. No debug module.
9. No dependency manager, pip environment, git, or code context features.
10. The CLI should feel premium, polished, and visually alive.

## Global Command Shape

```bash
dex <module> <command> [args] [flags]
```

Examples:

```bash
dex network ip
dex network ports
dex process inspect 1234
dex api replay session.dexapi
dex json lens data.json
dex system health
dex regex lab
```

## Global Flags

```bash
--json
--csv
--markdown
--raw
--watch
--interval 1s
--no-color
--theme dark
--profile default
--save
--export path
--copy
--quiet
--verbose
```

## Global Interactive Mode

```bash
dex
```

Launches the Dex dashboard.

Sections:

```text
Network
Processes
API
JSON
System
Regex
Benchmark
Files
Clipboard
Terminal
Settings
```

The TUI must support:

```text
vim navigation
fuzzy search
command palette
split panes
live charts
copy selected value
export current view
mouse support
keyboard first workflow
theme picker
session restore
```

Recommended Go stack:

```text
cobra
bubbletea
lipgloss
bubbles
sqlite
badger or boltdb
net/http
net
os/exec
gopsutil
regexp/syntax
encoding/json
```

# Module: Network

The network module should be the deepest part of Dex.

## Basic Identity

```bash
dex network ip
dex network ip --all
dex network public-ip
dex network interfaces
dex network interface en0
dex network mac
dex network hostname
dex network gateway
dex network routes
dex network dns-config
dex network proxy
dex network vpn
```

## DNS

```bash
dex network dns google.com
dex network dns google.com --type A
dex network dns google.com --type AAAA
dex network dns google.com --type MX
dex network dns google.com --type TXT
dex network dns google.com --type CNAME
dex network dns google.com --type NS
dex network dns google.com --resolver 1.1.1.1
dex network dns-trace google.com
dex network dns-bench google.com
dex network dns-compare google.com
dex network dns-cache
dex network dns-flush
dex network dns-watch google.com
```

Novel DNS features:

```bash
dex network dns-story google.com
```

Shows the full resolution path, resolver used, TTLs, record changes, and suspicious mismatches.

```bash
dex network dns-drift google.com
```

Watches DNS over time and reports record changes.

## Ports

```bash
dex network ports
dex network ports --listening
dex network ports --open
dex network ports --range 3000-9000
dex network port 3000
dex network port kill 3000
dex network port owner 3000
dex network port free
dex network port suggest
dex network port reserve 3000
dex network port watch
```

Novel port features:

```bash
dex network port-timeline
```

Shows when ports opened, closed, and which process owned them.

```bash
dex network port-map
```

Visual map of local ports grouped by process.

## Connections

```bash
dex network connections
dex network connections --process node
dex network connections --remote
dex network connections --local
dex network connections --established
dex network connections --listening
dex network connections --country
dex network connections --process-tree
dex network connection inspect <id>
dex network connection kill <id>
dex network connections watch
```

Novel connection features:

```bash
dex network connection-radar
```

Live visual radar of outbound connections.

```bash
dex network process-map
```

Maps each process to its sockets, remote hosts, ports, protocols, and byte counts.

## Latency

```bash
dex network ping google.com
dex network latency google.com
dex network latency google.com --tcp 443
dex network latency google.com --http
dex network latency google.com --tls
dex network latency google.com --dns
dex network latency-bench google.com
dex network latency-watch google.com
dex network jitter google.com
```

Novel latency features:

```bash
dex network latency-stack google.com
```

Breaks latency into DNS, TCP, TLS, first byte, download, and total time.

```bash
dex network latency-compare google.com cloudflare.com openai.com
```

Shows side by side performance.

## Tracing

```bash
dex network traceroute google.com
dex network mtr google.com
dex network path google.com
dex network hops google.com
dex network route-to google.com
dex network path-watch google.com
```

Novel tracing features:

```bash
dex network path-map google.com
```

Draws an interactive route map in the terminal.

```bash
dex network route-drift google.com
```

Detects when network path changes over time.

## HTTP

```bash
dex network http https://example.com
dex network headers https://example.com
dex network status https://example.com
dex network redirect https://example.com
dex network redirect-chain https://example.com
dex network cookies https://example.com
dex network compression https://example.com
dex network cache https://example.com
dex network cors https://example.com
dex network http2 https://example.com
dex network http3 https://example.com
```

Novel HTTP features:

```bash
dex network request-waterfall https://example.com
```

Shows DNS, TCP, TLS, redirect, server wait, body download.

```bash
dex network header-diff url1 url2
```

Compares headers across environments.

## TLS and Certificates

```bash
dex network ssl example.com
dex network cert example.com
dex network cert-chain example.com
dex network cert-expiry example.com
dex network cert-watch example.com
dex network tls-version example.com
dex network cipher example.com
dex network handshake example.com
```

Novel TLS features:

```bash
dex network tls-timeline example.com
```

Visualizes each TLS handshake step.

```bash
dex network cert-drift example.com
```

Detects certificate changes over time.

## Bandwidth

```bash
dex network bandwidth
dex network bandwidth --interface en0
dex network bandwidth-watch
dex network top
dex network top --process
dex network top --host
dex network usage
dex network usage --day
dex network usage --process
dex network usage --domain
```

Novel bandwidth features:

```bash
dex network heatmap
```

Terminal heatmap of network usage over time.

```bash
dex network noisy
```

Finds the noisiest local process by network usage.

## Local Network

```bash
dex network lan
dex network lan scan
dex network lan devices
dex network lan names
dex network lan vendors
dex network lan ports
dex network lan watch
dex network arp
dex network neighbors
dex network multicast
dex network mdns
dex network bonjour
```

Novel LAN features:

```bash
dex network device-timeline
```

Shows devices entering and leaving the network.

```bash
dex network lan-map
```

Interactive local network map.

## Security Oriented Network Diagnostics

```bash
dex network exposed
dex network open-services
dex network risky-ports
dex network suspicious-connections
dex network foreign-connections
dex network cert-audit example.com
dex network cors-audit https://example.com
dex network redirect-audit https://example.com
```

## Network Sessions

```bash
dex network snapshot
dex network snapshot save
dex network snapshot diff before after
dex network snapshot restore-view
dex network report
dex network report --markdown
dex network report --html
```

# Module: Process

The process module should not be a clone of top. It should explain process behavior visually.

## Core Commands

```bash
dex process list
dex process tree
dex process inspect <pid>
dex process search node
dex process kill <pid>
dex process kill-port 3000
dex process watch
dex process top
dex process children <pid>
dex process parent <pid>
dex process ancestry <pid>
```

## Resource Usage

```bash
dex process cpu
dex process memory
dex process disk
dex process network
dex process sockets <pid>
dex process files <pid>
dex process env <pid>
dex process cwd <pid>
dex process command <pid>
dex process limits <pid>
```

## Novel Process Features

```bash
dex process fingerprint <pid>
```

Creates a behavioral fingerprint from CPU, memory, sockets, files, child processes, and runtime duration.

```bash
dex process story <pid>
```

Shows a readable timeline of what the process has done since Dex started observing.

```bash
dex process family <pid>
```

Interactive tree with parent, children, siblings, ports, files, and network connections.

```bash
dex process heat
```

Heatmap of process resource intensity.

```bash
dex process drift <pid>
```

Detects behavior changes in a long running process.

```bash
dex process compare <pid1> <pid2>
```

Compares two processes by resources, files, sockets, arguments, and environment.

```bash
dex process ghost
```

Finds suspicious orphaned, zombie, detached, or hidden looking processes.

```bash
dex process explain-port 3000
```

Shows the full chain from port to process to parent command.

```bash
dex process sandbox-view <pid>
```

Shows what the process can access, including files, sockets, permissions, and child process capability.

```bash
dex process wakeups
```

Shows which processes wake most often and may drain battery.

```bash
dex process lifecycle
```

Live stream of process births, deaths, forks, and exits.

# Module: API

The API module should be a Postman alternative for terminal developers.

## Requests

```bash
dex api get https://api.example.com/users
dex api post https://api.example.com/users --body body.json
dex api put https://api.example.com/users/1 --body body.json
dex api patch https://api.example.com/users/1 --body body.json
dex api delete https://api.example.com/users/1
dex api request request.dex
dex api repeat last
dex api save last users-create
```

## Collections

```bash
dex api collection new my-api
dex api collection add users-list
dex api collection run my-api
dex api collection export my-api
dex api collection import postman.json
dex api collection tree
dex api collection env dev
```

## Recording and Replay

```bash
dex api record
dex api record --port 8080
dex api record --proxy
dex api replay session.dexapi
dex api replay session.dexapi --speed 2x
dex api replay session.dexapi --only failed
dex api timeline session.dexapi
```

Novel API features:

```bash
dex api time-machine session.dexapi
```

Step through every request and response like a debugger.

```bash
dex api contract-learn session.dexapi
```

Infers endpoints, schemas, status codes, headers, and examples from real traffic.

```bash
dex api behavior-map session.dexapi
```

Maps how endpoints relate based on request order and shared IDs.

## Schema

```bash
dex api schema response.json
dex api schema infer response.json
dex api schema diff old.json new.json
dex api schema validate response.json schema.json
dex api schema sample schema.json
dex api schema openapi traffic.dexapi
```

Novel schema features:

```bash
dex api schema-stability samples/
```

Detects fields that are stable, optional, nullable, volatile, or inconsistent.

```bash
dex api schema-risk old.json new.json
```

Ranks breaking change risk.

## Mocking

```bash
dex api mock openapi.yaml
dex api mock response.json
dex api mock collection.dex
dex api mock --latency 500ms
dex api mock --error-rate 5
dex api mock --port 9000
```

Novel mocking features:

```bash
dex api chaos openapi.yaml
```

Mock server that randomly returns latency, malformed responses, empty arrays, missing fields, and edge cases.

```bash
dex api scenario checkout.yaml
```

Runs multi step API scenarios.

## Testing

```bash
dex api test request.dex
dex api test collection.dex
dex api assert status 200
dex api assert json user.id exists
dex api assert header content-type application/json
dex api test-report
```

Novel testing features:

```bash
dex api invariant collection.dex
```

Checks rules like IDs stay consistent, timestamps increase, pagination is stable, and errors use the same shape.

```bash
dex api fuzz https://api.example.com/users
```

Sends safe malformed inputs to test validation.

## Comparison

```bash
dex api diff dev prod
dex api diff-response old.json new.json
dex api diff-headers dev prod
dex api diff-latency dev prod
dex api compare-env dev staging prod
```

Novel comparison features:

```bash
dex api drift dev prod
```

Detects behavioral differences between environments.

```bash
dex api shadow old new
```

Sends the same request to two endpoints and compares responses.

# Module: JSON

The JSON module should be more beautiful and powerful than jq for exploration.

## Core Commands

```bash
dex json view data.json
dex json format data.json
dex json minify data.json
dex json validate data.json
dex json query data.json 'users[0].name'
dex json flatten data.json
dex json unflatten data.json
dex json keys data.json
dex json paths data.json
dex json types data.json
```

## Exploration

```bash
dex json lens data.json
dex json tree data.json
dex json table data.json
dex json stats data.json
dex json search data.json "email"
dex json grep data.json "active"
dex json pick data.json users.0.name
dex json omit data.json users.0.password
```

Novel JSON features:

```bash
dex json map data.json
```

Interactive map of structure, repeated shapes, nested depth, arrays, and object clusters.

```bash
dex json shape data.json
```

Summarizes object shapes and repeated schemas.

```bash
dex json entropy data.json
```

Finds fields with high variation, low variation, IDs, enums, timestamps, and likely secrets.

```bash
dex json lens data.json
```

Interactive JSON microscope with search, collapse, type coloring, path copying, and value pinning.

```bash
dex json compare old.json new.json
```

Semantic diff with added, removed, changed, type changed, moved, and reordered fields.

```bash
dex json sample huge.json --count 50
```

Samples huge JSON safely without loading all data into memory.

```bash
dex json stream huge.json
```

Streams JSON events for massive files.

```bash
dex json pivot data.json users
```

Turns arrays of objects into terminal tables.

```bash
dex json redact data.json --emails --tokens --keys
```

Redacts sensitive looking values.

```bash
dex json fingerprint data.json
```

Creates a stable structural hash for schema comparison.

# Module: System

The system module should make Dex feel like a premium machine control panel.

## Overview

```bash
dex system info
dex system health
dex system dashboard
dex system uptime
dex system profile
dex system snapshot
dex system snapshot diff before after
dex system report
```

## CPU

```bash
dex system cpu
dex system cpu live
dex system cpu cores
dex system cpu freq
dex system cpu load
dex system cpu temp
dex system cpu pressure
dex system cpu timeline
```

Novel CPU features:

```bash
dex system cpu-bursts
```

Detects short CPU spikes usually missed by standard monitors.

```bash
dex system cpu-personality
```

Shows whether the machine is idle heavy, bursty, throttled, saturated, or background noisy.

## Memory

```bash
dex system memory
dex system memory live
dex system memory pressure
dex system memory swap
dex system memory top
dex system memory timeline
dex system memory leaks-watch
```

Novel memory features:

```bash
dex system memory-map
```

Visual breakdown of memory by process category.

```bash
dex system memory-drift
```

Finds processes with steady memory growth.

## Disk

```bash
dex system disk
dex system disk usage
dex system disk io
dex system disk top
dex system disk health
dex system disk timeline
dex system disk large
dex system disk duplicates
dex system disk temp
```

Novel disk features:

```bash
dex system disk-bursts
```

Finds sudden write storms.

```bash
dex system disk-sankey
```

Visualizes where space is going.

## Battery and Power

```bash
dex system battery
dex system battery live
dex system battery cycles
dex system battery health
dex system battery drain
dex system battery estimate
dex system power
dex system power top
dex system sleep-blockers
```

Novel power features:

```bash
dex system battery-story
```

Shows what caused drain during a session.

```bash
dex system power-fingerprint
```

Ranks processes by CPU wakeups, network use, disk activity, and energy impact.

## Thermal

```bash
dex system thermal
dex system thermal live
dex system thermal pressure
dex system thermal throttling
dex system fan
dex system fan live
```

Novel thermal features:

```bash
dex system heat-story
```

Shows what caused thermal pressure over time.

## Startup and Services

```bash
dex system startup
dex system services
dex system launch-agents
dex system launch-daemons
dex system service inspect <name>
dex system service disable <name>
dex system service enable <name>
```

## Permissions

```bash
dex system permissions
dex system permissions camera
dex system permissions microphone
dex system permissions location
dex system permissions files
dex system permissions automation
```

## Environment

```bash
dex system env
dex system env search PATH
dex system path
dex system shell
dex system shells
dex system terminal
```

## Health Scoring

```bash
dex system score
```

Returns a local health score from:

```text
CPU pressure
memory pressure
disk pressure
battery health
thermal state
network stability
startup load
background process noise
```

# Module: Regex

The regex module should be an interactive regex lab.

```bash
dex regex test pattern input.txt
dex regex lab
dex regex explain '^[a-z]+$'
dex regex build
dex regex benchmark pattern input.txt
dex regex find pattern input.txt
dex regex replace pattern replacement input.txt
dex regex groups pattern input.txt
dex regex cheatsheet
dex regex escape "hello.world"
dex regex unescape "\d+"
```

Novel regex features:

```bash
dex regex visual '^(user|admin)-\d+$'
```

Shows a visual parse tree.

```bash
dex regex examples '^\d{3}-\d{2}-\d{4}$'
```

Generates matching and non matching examples.

```bash
dex regex danger pattern
```

Detects risky expressions that may cause catastrophic backtracking.

```bash
dex regex compare p1 p2 input.txt
```

Compares match sets and speed.

```bash
dex regex narrow input.txt
```

Interactive mode where the user selects matches and Dex proposes a regex.

# Module: Benchmark

```bash
dex bench run "command"
dex bench compare a b
dex bench history
dex bench trend
dex bench export
dex bench arena "cmd1" "cmd2"
dex bench flame "command"
dex bench regression
```

Novel feature:

```bash
dex bench lab
```

Interactive benchmark lab with runs, variance, warmups, memory, CPU, and charts.

# Module: Files

```bash
dex files tree
dex files size
dex files largest
dex files duplicates
dex files search
dex files watch
dex files recent
dex files old
dex files entropy
dex files type-map
dex files clean-preview
```

Novel feature:

```bash
dex files galaxy
```

Interactive visual map of directories by size, age, type, and change frequency.

# Module: Clipboard

```bash
dex clipboard history
dex clipboard search
dex clipboard save
dex clipboard pin
dex clipboard clear
dex clipboard export
dex clipboard watch
```

# Module: Terminal

```bash
dex terminal record
dex terminal replay
dex terminal sessions
dex terminal history
dex terminal stats
dex terminal aliases
dex terminal profile
```

# UI Design Requirements

Dex must look elite.

## Visual Style

```text
dark mode first
sharp borders
smooth terminal animations
clear spacing
subtle gradients where supported
beautiful tables
sparkline charts
heatmaps
live counters
status pills
keyboard hints
```

## TUI Components

```text
dashboard cards
command palette
fuzzy finder
side navigation
split views
detail drawers
timeline panels
tree views
waterfall charts
sparklines
tables with sorting
copyable cells
filter bars
```

## Example Dashboard

```text
DEX

System        Healthy
Network       42 active connections
Processes     183 running
API           7 saved collections
JSON          12 recent files
Regex         Lab ready

[ Network ] [ Process ] [ API ] [ JSON ] [ System ] [ Regex ]
```

# Storage

Dex should use local storage for history and snapshots.

```text
~/.dex/config.toml
~/.dex/history.db
~/.dex/snapshots/
~/.dex/api/
~/.dex/themes/
~/.dex/benchmarks/
```

# Project Structure

```text
dex/
  cmd/
    root.go
    network.go
    process.go
    api.go
    json.go
    system.go
    regex.go
    bench.go
    files.go
    clipboard.go
    terminal.go

  internal/
    network/
    process/
    api/
    jsonx/
    system/
    regexx/
    bench/
    files/
    clipboard/
    terminal/
    tui/
    storage/
    output/
    config/
    theme/

  pkg/
    render/
    snapshot/
    table/
    charts/
```

# MVP Build Order

## Phase 1

```text
dex network ip
dex network ports
dex network latency
dex process list
dex process inspect
dex json view
dex json diff
dex system info
dex system health
dex regex test
```

## Phase 2

```text
dex network connections
dex network dns
dex network ssl
dex process family
dex process heat
dex api get
dex api collection
dex json lens
dex system dashboard
dex regex lab
```

## Phase 3

```text
dex network radar
dex network path-map
dex api record
dex api replay
dex api contract-learn
dex json entropy
dex system score
dex files galaxy
dex bench lab
```

# Non Negotiable Quality Bar

Dex must be:

```text
fast
beautiful
local first
keyboard native
scriptable
safe by default
cross platform where possible
Mac optimized first
useful without accounts
useful without AI
powerful in plain CLI
excellent in TUI
```

# Final Product Vision

Dex should become the one terminal tool a developer opens when something feels wrong, slow, unclear, broken, exposed, unstable, or hard to inspect.

It should answer:

```text
What is my machine doing?
What is my network doing?
What is this process doing?
What is this API doing?
What changed?
What is slow?
What is exposed?
What is using resources?
What does this data look like?
How do I inspect this faster?
```

Dex is the developer cockpit for the local machine.

---
> Source: [desenyon/dex](https://github.com/desenyon/dex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
