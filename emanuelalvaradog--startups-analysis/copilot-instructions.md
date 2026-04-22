## startups-analysis

> Startup success prediction.

Startup success prediction.

Primary Research Question:
"What are the most significant predictors of startup success, and can we accurately predict whether a startup will succeed (IPO/Acquisition) or fail (Closed) based on its early-stage characteristics?"

Specific Sub-Questions to Answer:

1. Feature Importance & Impact

- Which factors have the strongest correlation with startup success?
- How much does funding amount vs. funding source matter?
- Do geographic location and industry sector significantly impact outcomes?

2. Predictive Performance

- Can we predict startup outcomes with >70% accuracy?
- Which machine learning model performs best for this multi-class problem?
- How early can we reliably predict a startup's eventual outcome?

3. Business Insights

- What funding strategy leads to the highest success rates?
- Which investor types (VC, Angel, Corporate) provide the most value beyond capital?
- Are there optimal timing patterns for funding rounds?

4. Practical Applications

- Can we create a risk assessment tool for investors?
- What recommendations can we provide to early-stage startups?
- Which warning signs indicate a higher probability of failure?

The Ultimate Question to Answer:

"Given a startup's current characteristics (funding history, industry, location, team, investor profile), what is the probability it will achieve each outcome (Operating/Acquired/IPO/Closed), and what actions could improve its chances of success?"

This question combines:

Prediction: Probability estimates for each outcome
Explanation: Why certain outcomes are more likely
Actionability: What can be done to improve success chances

### File analysis

#### acquisitions:
Contains information about startups that have been bought.

Columns:
- id
- acquisition_id
- acquiring_object_id
- acquired_object_id
- term_code
- price_amount
- price_currency_code
- acquired_at
- source_url
- source_description
- created_at
- updated_at


#### degrees:
Contains the education backgrounds of individuals involved in the startup world.

Columns:
- id
- object_id
- degree_type
- subject
- institution
- graduated_at
- created_at
- updated_at

#### founding_rounds:
Contains information about startup funding rounds.

Columns:
- id
- funding_round_id
- object_id
- funded_at
- funding_round_type
- funding_round_code
- raised_amount_usd
- raised_amount
- raised_currency_code
- pre_money_valuation_usd
- pre_money_valuation
- pre_money_currency_code
- post_money_valuation_usd
- post_money_valuation
- post_money_currency_code
- participants
- is_first_round
- is_last_round
- source_url
- source_description
- created_by
- created_at
- updated_at


#### Funds
Contains data on the venture capital funds that make investments.

Columns:
- id
- fund_id
- object_id
- name
- funded_at
- raised_amount
- raised_currency_code
- source_url
- source_description
- created_at
- updated_at


#### Investments
Contains data on the various different investments made by venture capitalists.

Columns:
- id
- funding_round_id
- funded_object_id
- investor_object_id
- created_at
- updated_at

#### IPOs
Contains data on initial public offerings.

Columns:
- id
- ipo_id
- object_id
- valuation_amount
- valuation_currency_code
- raised_amount
- raised_currency_code
- public_at
- stock_symbol
- source_url
- source_description
- created_at
- updated_at

#### Milestones
Contains significant events within the startup ecosystem.

Columns:
- id
- object_id
- milestone_at
- milestone_code
- description
- source_url
- source_description
- created_at
- updated_at

#### Objects
Main file containing base information for each entity type (person, company, financial org or product)

- id
- entity_type
- entity_id
- parent_id
- name
- normalized_name
- permalink
- category_code
- status
- founded_at
- closed_at
- domain
- homepage_url
- twitter_username
- logo_url
- logo_width
- logo_height
- short_description
- description
- overview
- tag_list
- country_code
- state_code
- city
- region
- first_investment_at
- last_investment_at
- investment_rounds
- invested_companies
- first_funding_at
- last_funding_at
- funding_rounds
- funding_total_usd
- first_milestone_at
- last_milestone_at
- milestones
- relationships
- created_by
- created_at
- updated_at


#### Offices
Contains information about startup company offices.

Columns:
- id
- object_id
- office_id
- description
- region
- address1
- address2
- city
- zip_code
- state_code
- country_code
- latitude
- longitude
- created_at
- updated_at


#### People
Contains information about individuals in the startup world.

Columns:
- id
- object_id
- first_name
- last_name
- birthplace
- affiliation_name

#### Relationships
Contains relationship data that links companies to individuals and their positions.

Columns:
- id
- relationship_id
- person_object_id
- relationship_object_id
- start_at
- end_at
- is_past
- sequence
- title
- created_at
- updated_at

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/emanuelalvaradog) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
