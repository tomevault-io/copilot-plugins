## knowledge-base-system

> - name: Categoriseer kennisbestanden

version: 1.0

rules:
  - name: Categoriseer kennisbestanden
    matcher:
      extension: md
    actions:
      - action: move
        path: >-
          knowledge/{category}/{subcategory}/{filename}

variables:
  category:
    - Technologie
    - Wetenschap
    - Persoonlijke ontwikkeling
    - Projecten
    - Referenties

  subcategory:
    Technologie:
      - Programmeren
      - Netwerken
      - Databases
      - AI en Machine Learning
    Wetenschap:
      - Natuurkunde
      - Scheikunde
      - Biologie
      - Astronomie
    Persoonlijke ontwikkeling:
      - Productiviteit
      - Gezondheid
      - Financiën
      - Vaardigheden
    Projecten:
      - Actief
      - Afgerond
      - Ideeën
    Referenties:
      - Boeken
      - Artikelen
      - Podcasts
      - Video's

templates:
  - name: Standaard kennisbestand
    path: knowledge/Technologie/Programmeren/{filename}.md
    content: |
      # 📚 {title}

      ## 📝 Samenvatting
      
      [Korte samenvatting van het onderwerp]

      ## 🎯 Belangrijkste punten
      
      - <span style="color: #22c55e;">✓</span> Voordeel 1
      - <span style="color: #22c55e;">✓</span> Voordeel 2
      - <span style="color: #ef4444;">⚠</span> Nadeel 1

      ## 🔍 Details
      
      > [!NOTE]+ Belangrijke informatie
      > Hier komt belangrijke informatie

      > [!WARNING]+ Let op
      > Hier komen waarschuwingen

      ## 📊 Visualisaties

      ### Proces Flow
      ```mermaid
      graph TD
          A[Start] -->|Input| B[Proces]
          B --> C[Output]
          C -->|Feedback| B
      ```

      ### Sequence Diagram
      ```mermaid
      sequenceDiagram
          participant A as Component A
          participant B as Component B
          A->>B: Request
          B-->>A: Response
      ```

      ## 📚 Bronnen
      
      - [Bron 1]
      - [Bron 2]

      ## 🔗 Gerelateerde onderwerpen
      
      - [[Gerelateerd onderwerp 1]]
      - [[Gerelateerd onderwerp 2]]

      ## 📝 Notities
      
      Status: <span style="color: #22c55e;">●</span> Actief
      Prioriteit: <span style="color: #eab308;">●</span> Medium

      ## 🏷️ Tags
      
      #tag1 #tag2 #tag3

Je update altijd de betreffende kennisbestanden als er vragen over gesteld worden.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sjaaak777) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
