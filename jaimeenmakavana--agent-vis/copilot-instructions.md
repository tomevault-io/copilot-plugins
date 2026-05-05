## folder-structure

> ├── tailwind.config.js


agentvis/
├── README.md
├── package.json
├── tsconfig.json
├── next.config.js
├── tailwind.config.js
├── .env.example
├── .gitignore
│
├── public/
│ ├── assets/
│ │ ├── icons/
│ │ └── images/
│ └── mock-data/
│ ├── agent-traces.json
│ ├── swarm-topology.json
│ ├── rag-vectors.json
│ └── performance-metrics.json
│
├── src/
│ ├── app/
│ │ ├── layout.tsx
│ │ ├── page.tsx
│ │ ├── globals.css
│ │ │
│ │ ├── traceability/
│ │ │ ├── page.tsx
│ │ │ ├── layout.tsx
│ │ │ ├── chain-of-thought/
│ │ │ │ └── page.tsx
│ │ │ ├── tree-of-thoughts/
│ │ │ │ └── page.tsx
│ │ │ └── async-timeline/
│ │ │ └── page.tsx
│ │ │
│ │ ├── orchestration/
│ │ │ ├── page.tsx
│ │ │ ├── layout.tsx
│ │ │ ├── swarm-network/
│ │ │ │ └── page.tsx
│ │ │ ├── conversation-sequence/
│ │ │ │ └── page.tsx
│ │ │ └── swimlanes/
│ │ │ └── page.tsx
│ │ │
│ │ ├── memory/
│ │ │ ├── page.tsx
│ │ │ ├── layout.tsx
│ │ │ ├── vector-space/
│ │ │ │ └── page.tsx
│ │ │ ├── context-window/
│ │ │ │ └── page.tsx
│ │ │ └── knowledge-graph/
│ │ │ └── page.tsx
│ │ │
│ │ └── observability/
│ │ ├── page.tsx
│ │ ├── layout.tsx
│ │ ├── latency-waterfall/
│ │ │ └── page.tsx
│ │ ├── risk-heatmap/
│ │ │ └── page.tsx
│ │ └── token-burndown/
│ │ └── page.tsx
│ │
│ ├── components/
│ │ ├── ui/
│ │ │ ├── button.tsx
│ │ │ ├── card.tsx
│ │ │ ├── tabs.tsx
│ │ │ ├── dialog.tsx
│ │ │ ├── tooltip.tsx
│ │ │ └── badge.tsx
│ │ │
│ │ ├── layout/
│ │ │ ├── Navbar.tsx
│ │ │ ├── Sidebar.tsx
│ │ │ ├── Footer.tsx
│ │ │ └── RouteGuard.tsx
│ │ │
│ │ ├── traceability/
│ │ │ ├── ChainOfThoughtDAG.tsx
│ │ │ ├── TreeOfThoughtsViz.tsx
│ │ │ ├── AsyncTimeline.tsx
│ │ │ ├── NodeInspector.tsx
│ │ │ └── ThoughtCard.tsx
│ │ │
│ │ ├── orchestration/
│ │ │ ├── SwarmNetworkGraph.tsx
│ │ │ ├── SequenceDiagram.tsx
│ │ │ ├── SwimlaneCanvas.tsx
│ │ │ ├── AgentNode.tsx
│ │ │ └── MessageFlow.tsx
│ │ │
│ │ ├── memory/
│ │ │ ├── VectorSpacePlot.tsx
│ │ │ ├── ContextTreemap.tsx
│ │ │ ├── KnowledgeGraphViz.tsx
│ │ │ ├── SemanticDistanceIndicator.tsx
│ │ │ └── EntityCard.tsx
│ │ │
│ │ ├── observability/
│ │ │ ├── LatencyWaterfallChart.tsx
│ │ │ ├── RiskHeatmap.tsx
│ │ │ ├── TokenBurndownChart.tsx
│ │ │ ├── MetricCard.tsx
│ │ │ └── AlertPanel.tsx
│ │ │
│ │ └── shared/
│ │ ├── LoadingSpinner.tsx
│ │ ├── ErrorBoundary.tsx
│ │ ├── JsonViewer.tsx
│ │ ├── SearchBar.tsx
│ │ └── FilterPanel.tsx
│ │
│ ├── lib/
│ │ ├── utils.ts
│ │ ├── cn.ts
│ │ ├── data-processing/
│ │ │ ├── trace-parser.ts
│ │ │ ├── graph-builder.ts
│ │ │ ├── vector-transformer.ts
│ │ │ └── metrics-calculator.ts
│ │ │
│ │ ├── visualization/
│ │ │ ├── d3-helpers.ts
│ │ │ ├── graph-layouts.ts
│ │ │ ├── color-schemes.ts
│ │ │ └── animation-configs.ts
│ │ │
│ │ └── mock-generators/
│ │ ├── generate-traces.ts
│ │ ├── generate-swarm-data.ts
│ │ ├── generate-vectors.ts
│ │ └── generate-metrics.ts
│ │
│ ├── hooks/
│ │ ├── useAgentTrace.ts
│ │ ├── useSwarmData.ts
│ │ ├── useVectorSpace.ts
│ │ ├── useMetrics.ts
│ │ ├── useResizeObserver.ts
│ │ └── useDebounce.ts
│ │
│ ├── types/
│ │ ├── agent.types.ts
│ │ ├── graph.types.ts
│ │ ├── metrics.types.ts
│ │ ├── visualization.types.ts
│ │ └── index.ts
│ │
│ ├── constants/
│ │ ├── routes.ts
│ │ ├── visualization-configs.ts
│ │ ├── color-palette.ts
│ │ └── mock-data-config.ts
│ │
│ └── styles/
│ ├── visualizations.css
│ └── themes.css
│
├── docs/
│ ├── ARCHITECTURE.md
│ ├── VISUALIZATION_GUIDE.md
│ ├── MOCK_DATA_STRUCTURE.md
│ └── DEPLOYMENT.md
│
└── tests/
├── unit/
│ ├── utils.test.ts
│ └── data-processing.test.ts
└── integration/
└── visualization-render.test.tsx

---
> Source: [JaimeenMakavana/agent-vis](https://github.com/JaimeenMakavana/agent-vis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
