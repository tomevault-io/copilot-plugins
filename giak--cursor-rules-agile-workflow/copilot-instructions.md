## cursor-rules-agile-workflow

> IMPLEMENT when CREATING or MANAGING epics and stories to ENSURE consistent structure, effective splitting, and proper tracking


# Epic & Story Management Framework

↹ foundational•principles [p=1] {
  ⊕ structure•standardization => consistent•documentation
  ⊕ value•first•approach => user•centric•stories
  ⊕ appropriate•granularity => manageable•increments
  ⊕ traceable•relationships => connected•artifacts
  ⊕ continuous•refinement => evolving•quality
  ⊕ measurable•outcomes => verifiable•success
}
Σ core•guidelines

↹ document•relationships [p=1] {
  ⊕ hierarchy•position => PFD → PRD → Architecture → Epics → Stories
  
  ⊕ pfd•relationship {
    consumes: strategic•vision+business•goals,
    provides: high•level•context+project•boundaries,
    ensures: business•alignment+strategic•fit
  }
  
  ⊕ prd•relationship {
    consumes: functional+non-functional•requirements,
    provides: feature•specifications+acceptance•criteria,
    transforms: requirements → implementable•stories,
    requires: bi-directional•traceability
  }
  
  ⊕ architecture•relationship {
    consumes: technical•constraints+design•decisions,
    provides: implementation•guidance+technical•boundaries,
    ensures: technical•feasibility+architectural•compliance,
    requires: consistent•technical•approach
  }
  
  ⊕ epic•to•prd•mapping {
    requirement: every•epic•maps•to•prd•feature,
    traceability: explicit•references•to•prd•sections,
    validation: confirm•coverage•of•requirements
  }
  
  ⊕ story•to•architecture•mapping {
    requirement: implementation•conforms•to•architecture,
    traceability: references•to•architectural•components,
    validation: technical•alignment•verification
  }
  
  ⊕ visual•representation: ```mermaid
    flowchart TB
      PFD[Project Foundation Document] --> PRD[Product Requirements Document]
      PFD --> ARCH[Architecture Document]
      
      PRD --> EPICS[Epics]
      ARCH --> EPICS
      
      EPICS --> STORIES[Stories]
      STORIES --> TASKS[Implementation Tasks]
      
      classDef pfd fill:#ff9900,stroke:#333,stroke-width:2px
      classDef prd fill:#66cc99,stroke:#333,stroke-width:2px
      classDef arch fill:#6699ff,stroke:#333,stroke-width:2px
      classDef epic fill:#cc99ff,stroke:#333,stroke-width:2px
      classDef story fill:#ff99cc,stroke:#333,stroke-width:2px
      classDef task fill:#ffcc66,stroke:#333,stroke-width:1px
      
      class PFD pfd
      class PRD prd
      class ARCH arch
      class EPICS epic
      class STORIES story
      class TASKS task
  ```
}
Σ documentation•hierarchy

↹ epic•organization [p=1] {
  ⊕ directory•structure => .ai/epic-{n}/
  ⊕ epic•metadata => _epic.md
  ⊕ story•placement => {m}-{code}.story.md
  ⊕ sequential•numbering => continuous•across•project
  ⊕ descriptive•naming => kebab-case•convention
  ⊕ status•tracking => [not-started|in-progress|complete]
}
Σ storage•convention

↹ epic•template [p=1] -> [
  ⊕ header {
    title: "Epic-{N}: {Epic Title}",
    format: markdown•h1
  },
  
  ⊕ traceability {
    prd: "Reference to PRD section: {Section}.{Subsection}",
    requirements: list•of•requirement•ids•from•prd
  },
  
  ⊕ description {
    content: comprehensive•overview,
    scope: clearly•defined•boundaries
  },
  
  ⊕ objectives {
    format: bullet•list,
    focus: business•value,
    specificity: measurable•when•possible
  },
  
  ⊕ stories {
    format: checkbox•list,
    naming: "Story-{N}.{M}: {Title}",
    status: progress•tracking
  },
  
  ⊕ success•criteria {
    format: bullet•list,
    qualities: specific+measurable+achievable
  },
  
  ⊕ risks {
    format: table,
    columns: [risk+impact+probability+mitigation],
    completeness: all•identified•risks
  },
  
  ⊕ dependencies {
    internal: cross•epic•references,
    external: third•party•dependencies,
    architectural: references•to•architecture•components
  },
  
  ⊕ status {
    values: [not-started|in-progress|complete],
    visibility: prominently•displayed
  }
]
Σ epic•document•structure

↹ story•lifecycle [p=1] {
  ⊕ creation => draft•initial•structure
  ⊕ refinement => complete•details•estimate
  ⊕ validation => verify•INVEST•criteria
  ⊕ prd•compliance => verify•alignment•with•requirements
  ⊕ architecture•compliance => verify•technical•feasibility
  ⊕ approval => ready•for•development
  ⊕ implementation => in•progress•status
  ⊕ verification => acceptance•criteria•check
  ⊕ completion => documentation•updates
}
Σ story•progression

↹ story•template [p=1] -> [
  ⊕ header {
    title: "Story: {Story Title}",
    meta: "Epic-{N}: {Epic Title}\nStory-{M}: {Story Title}"
  },
  
  ⊕ traceability {
    prd: "Reference to PRD requirement: {Requirement ID}",
    architecture: "Reference to Architecture component: {Component Path}"
  },
  
  ⊕ description {
    format: "**En tant que** {role}\n**Je veux** {action}\n**afin de** {benefit}",
    focus: value•to•user
  },
  
  ⊕ status {
    values: [draft|in-progress|complete|cancelled],
    visibility: always•current
  },
  
  ⊕ context {
    epic•relationship: connection•to•parent,
    dependencies: related•stories,
    constraints: limitations•assumptions,
    architectural•considerations: relevant•design•patterns+constraints
  },
  
  ⊕ estimation {
    format: "Story Points: {number}",
    scale: relative•complexity
  },
  
  ⊕ acceptance•criteria {
    format: "Étant donné {context}, quand {action}, alors {expected}",
    numbered: sequential•list,
    completeness: all•scenarios•covered,
    prd•alignment: maps•to•prd•requirements
  },
  
  ⊕ tasks {
    structure: hierarchical•list,
    tracking: checkbox•status,
    granularity: implementation•level
  },
  
  ⊕ development•principles {
    focus: specific•to•story,
    sections: [to-follow, to-avoid],
    architecture•compliance: align•with•architectural•decisions
  },
  
  ⊕ notes {
    technical: implementation•details,
    decisions: architectural•choices,
    constraints: technical•limitations•from•architecture
  },
  
  ⊕ communication {
    format: conversation•history,
    content: decisions•questions•clarifications
  }
]
Σ story•document•structure

↹ validation•checks [p=1] {
  ⊕ document•alignment {
    epic•validation: [
      every•epic•references•prd,
      objectives•align•with•business•goals,
      scope•within•project•boundaries
    ],
    story•validation: [
      every•story•references•epic,
      story•implements•prd•requirement,
      implementation•respects•architectural•constraints,
      acceptance•criteria•validates•prd•requirements
    ]
  },
  
  ⊕ completeness•check {
    prd•to•epic: all•prd•features•covered•by•epics,
    epic•to•story: all•epic•objectives•implemented•in•stories,
    architecture•compliance: all•architectural•constraints•respected
  }
}
Σ traceability•verification

↹ INVEST•principles [p=1] {
  ⊕ independent => minimal•dependencies
  ⊕ negotiable => flexible•implementation
  ⊕ valuable => user•benefiting•outcome
  ⊕ estimable => predictable•effort
  ⊕ small => sprint•completable
  ⊕ testable => verifiable•criteria
}
Σ quality•criteria

↹ splitting•triggers [p=1] {
  ⊕ excessive•points => >8•story•points
  ⊕ too•many•criteria => >7•acceptance•criteria
  ⊕ multiple•functions => different•user•benefits
  ⊕ cross•cutting => spans•technical•layers
  ⊕ estimation•difficulty => team•uncertainty
  ⊕ linguistic•indicators => "and"+"or"+"also"
}
Σ discovery•indicators

↹ SPIDR•techniques [p=1] {
  ⊕ spike => research•investigation
  ⊕ path => workflow•variations
  ⊕ interface => interaction•methods
  ⊕ data => information•subsets
  ⊕ rules => business•logic•variations
}
Σ primary•splitting•patterns

↹ additional•techniques [p=1] {
  ⊕ workflow•steps => process•stages
  ⊕ CRUD•operations => data•manipulation•types
  ⊕ functional•variations => core+advanced
  ⊕ deferred•performance => function•then•optimize
  ⊕ effort•reduction => simple•parts•first
}
Σ complementary•splitting•strategies

↹ splitting•workflow [p=1] -> [
  ⊕ evaluation {
    confirm: excessive•size,
    understand: core•value,
    map: complete•user•flow
  },
  
  ⊕ technique•selection {
    analyze: story•nature,
    select: appropriate•pattern,
    apply: create•sub•stories
  },
  
  ⊕ validation {
    verify: INVEST•compliance,
    confirm: value•delivery,
    prioritize: implementation•order,
    maintain: traceability•to•prd+architecture
  }
]
Σ splitting•process

↹ anti•patterns [p=1] {
  ⊕ technical•stories => missing•user•value
  ⊕ horizontal•slicing => layer•based•splitting
  ⊕ valueless•increments => no•standalone•benefit
  ⊕ ambiguous•criteria => non•verifiable•outcomes
  ⊕ dependency•chains => sequential•requirements
  ⊕ arbitrary•estimation => non•consensus•based
  ⊕ stale•status => outdated•progress
  ⊕ orphaned•stories => epic•disconnected
  ⊕ documentation•gaps => missing•context
  ⊕ inconsistent•detail => varying•specificity
  ⊕ disconnected•documentation => missing•traceability•to•prd+architecture
  ⊕ scope•creep => implementing•features•not•in•prd
}
Σ practices•to•avoid

↹ best•practices [p=1] {
  ⊕ collaborative•creation => team•involvement
  ⊕ value•prioritization => business•impact•first
  ⊕ communication•logging => decision•tracking
  ⊕ regular•review => periodic•refinement
  ⊕ concrete•examples => specific•scenarios
  ⊕ post•validation•tasks => technical•breakdown
  ⊕ vertical•consistency => aligned•artifacts
  ⊕ immediate•updates => real•time•status
  ⊕ visual•mapping => story•relationships
  ⊕ pre•development•validation => quality•gate
  ⊕ bidirectional•traceability => maintain•references•both•ways
  ⊕ document•impact•analysis => evaluate•changes•across•all•documents
}
Σ recommended•approaches

↹ reference•links [p=2] {
  ⊕ guides: [
    SPIDR: "https://www.mountaingoatsoftware.com/blog/five-simple-but-powerful-ways-to-split-user-stories",
    INVEST: "https://www.agilealliance.org/glossary/invest",
    story•mapping: "https://www.jpattonassociates.com/user-story-mapping"
  ],
  
  ⊕ related•rules: [
    pfd: "mdc:5000-workflow-foundation-document-pfd",
    prd: "mdc:5002-workflow-product-requirements-document",
    architecture: "mdc:5003-workflow-architecture-document",
    workflow: "mdc:5020-workflow-agile-modern",
    tdd: "mdc:guides/test-driven-development.md",
    estimation: "mdc:guides/story-estimation.md"
  ]
}
Σ knowledge•sources

<requires>5000-workflow-foundation-document-pfd</requires>
<requires>5002-workflow-product-requirements-document</requires>
<requires>5003-workflow-architecture-document</requires>
<requires>5020-workflow-agile-modern</requires>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/giak) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
