flowchart TD
  %% LEGEND (for quick reference)
  subgraph LEG[Legend]
  L1[[[L] Linked]]:::legend
  L2[[[R] Rollup/Lookup]]:::legend
  L3[[[F] Formula]]:::legend
  L4[[[S] Static/Manual]]:::legend
  L5[[[A] Airtable Automation (optional)]]:::legend
  end

  %% L0: REFERENCE DIMENSIONS
  subgraph L0[Reference Dimensions (stable)]
    PF[Patterns
      ---
      [S] Pattern
      [S] Intent Weight
      [S] Needs City?
      [S] Needs Season?
      [S] Template
      [F] Canonical Class]:::box

    CT[Service Areas (Cities)
      ---
      [S] City
      [S] Geo Weight]:::box

    SS[Seasons
      ---
      [S] Season/Holiday
      [S] Start
      [S] End
      [F] Season Window? (TODAY() between Start/End)
      [S] Base Season Weight]:::box

    SSW[Season × Service Weights (optional)
      ---
      [L] Season → Seasons
      [L] Family → Service Families
      [S] Service Season Weight]:::box
  end

  %% L1: ATOMIC ROWS
  subgraph L1[Atomic Rows]
    SVC[Services (SKUs)
      ---
      [S] Service Name
      [S] Price (or Unit Price)
      [S] Avg Units/Syringes/Vials (if per-unit)
      [F] Avg Ticket (Price × Units OR Price)
      [F] Price Weight (your installed formula)
      [F] Top Flag (Price Weight ≥ thresh)
      [L][A] Family → Service Families]:::box
  end

  %% L2: FAMILY AGGREGATES
  subgraph L2[Family Aggregates (rollups)]
    CF[Service Families
      ---
      [S] Family (e.g., Botox, Facials, Laser Rejuvenation)
      [R] Avg Price Weight (ROLLUP Services.Price Weight → AVERAGE)
      [R] Max Price Weight (ROLLUP Services.Price Weight → MAX)
      [R] Top Count (ROLLUP Services.Top Flag → SUM)
      [R] Service Count (COUNT of linked Services)
      [F] Top Share (Top Count / Service Count)
      [F] Family Priority
      [S] Gross Margin (0–1) (optional)
      [S] LTV/Repeat (0–1) (optional)]:::box
  end

  %% L3: KEYWORD ASSEMBLY
  subgraph L3[Keyword Assembly (Seeds)]
    CMB[Combos (Keyword Seeds)
      ---
      [L] Family → Service Families
      [L] Pattern → Patterns
      [L] City → Service Areas (optional)
      [L] Season → Seasons (optional)
      [R] Intent Weight (from Pattern)
      [R] Geo Weight (from City)
      [R] Family Priority (from Family)
      [R] Base/Service Season Weight (from Season/SSW)
      [F] Season Boost
      [F] Seed Keyword (from Pattern.Template via SWITCH())
      [F] Sense OK (Needs City/Season, not Blacklisted)
      [F] Canonical Key (normalized Service|Class|City|Season)
      [R] Published Keys (rollup from Published)
      [F] Cannibal Risk (FIND Canonical Key in Published Keys)
      [F] Priority Score
      [R] Last Similar Pub Date (rollup MAX Pub Date) (optional)
      [F] Freshness Weight (optional)]:::box
  end

  %% L4: PUBLICATION MEMORY
  subgraph L4[Publication Memory]
    PUB[Published / Drafts
      ---
      [S] Title/Slug
      [L] Family → Service Families
      [L] Pattern → Patterns
      [L] City → Service Areas (optional)
      [L] Season → Seasons (optional)
      [F] Canonical Key (same formula as Combos)
      [S] Pub Date]:::box

    BLK[Blacklist
      ---
      [L] Family
      [L] Pattern
      [L] City (optional)
      [F] Block Flag]:::box
  end

  %% L5: AUTOMATION
  subgraph L5[Automation]
    N8N[n8n
      ---
      Reads “Combos • Ready” (Sense OK & Cannibal OK)
      → Builds Title/Meta
      → Writes Draft/Scheduled → Published]:::auto
  end

  %% RELATIONSHIPS
  SVC -->|[L]| CF
  CF -->|[R] rollups| CMB
  PF -->|[R]| CMB
  CT -->|[R]| CMB
  SS -->|[R]| CMB
  SSW -->|[R] via Season/Family| SS
  PUB -->|[R] keys/dates| CMB
  BLK -->|[R] block lookups| CMB
  CMB -->|sorted by Priority Score| N8N
  N8N --> PUB

  classDef box fill:#0b0c10,stroke:#3a3f45,color:#eaecef;
  classDef auto fill:#111827,stroke:#3a3f45,color:#eaecef,stroke-dasharray: 3 3;
  classDef legend fill:#f3f4f6,stroke:#d1d5db,color:#111827;
