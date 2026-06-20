## value-realization

> Analyze whether end users will discover clear value in product ideas. Use when: discussing product concepts, evaluating features, providing product improvement directions, planning marketing strategies, analyzing adoption or retention problems, assessing whether copy communicates value, mapping features to usage scenarios or use cases, or when the user is uncertain about product direction, positioning, or whether end-user demand exists (e.g., 'is this a good idea?', 'what do you think of this product?', 'will users want this?', 'what is this feature useful for?', 'how should I explain this feature's value?', 'what do you think of this copy?', 'help me write a few usage scenarios', 'why aren't users retaining?', 'how should we position this?').


# Value Realization Philosophy

**Status**: Production Ready ✅
**Version**: 1.2.5
**Last Updated**: 2026-05-12
**Type**: Analytical Framework

## Overview

This skill provides a philosophical framework and analytical methods for evaluating whether end users can "know" what value they can achieve through a product. It guides analysis from a value discovery perspective, rather than providing checklists.

**What this skill provides**:
- Framework to evaluate product ideas when certainty is lacking
- Analysis methods for assessing end user value discovery
- Patterns from real product successes and failures
- Analysis methods for product design, positioning, and value communication

**Core question**: Can end users clearly understand what value they'll achieve through the product - even if that value takes time to achieve?

**Key terminology**:
- **User**: The person using this skill (product creator, PM, designer, entrepreneur, etc.)
- **End user**: The person who will use the product being discussed
- **Value**: A beneficial relational property between subject and object. In product analysis, judgments about whether value holds must be specified in terms of outcomes that end users can understand, verify, and perceive in a concrete value scenario (such as identity, financial gain, capability enhancement, time savings, etc.)
- **Value Relation Position**: An analytical element within a value scenario that indicates where value confirmation or outcome realization occurs in the value relation. Analysis must distinguish between: end users confirming that an external object, content, feature, or product is valuable to them; end users achieving their own outcome through the product; and end users themselves, their actions, or their outputs being confirmed as valuable through feedback from other people, groups, organizations, or systems. The three can coexist, but they cannot substitute for one another
- **Features**: The product's technical capabilities
- **Value Scenario**: A concrete context in which end users use the product with a specific task or goal and obtain an outcome that can be understood, verified, and perceived. A value scenario should specify the end-user task, product role, value relation position, and outcome end users achieve
- **Usage Scenario**: A concrete context in which the product may be used
- **Use Case**: A specific task or workflow end users complete with the product in a given context

**Core distinction**:
- Features are not value
- Features are what the product can do, value is the outcomes end users gain
- Analysis must translate features into specific end user outcomes
- Value analysis should place value in a concrete scenario
- Value scenario analysis should specify the value relation position to avoid conflating external objects being considered valuable, end users achieving their own outcomes, and end users themselves, their actions, or their outputs being confirmed as valuable
- Usage scenarios explain where the product may be used, use cases explain how end users use it to complete tasks, and value scenarios explain what outcome end users get in that context

## Core Insight

End users adopt products when they **know** what value they'll get. This "knowing" is critical:

- If end users know they'll achieve something valuable (even long-term), they'll use it
- If end users don't know what they'll achieve, they won't use it - no matter how good the product is

**What "knowing" means**:
- End users can explain to themselves or others why they're using the product
- End users can describe what they'll achieve (not just what features exist)
- End users understand the outcome, even if it takes time to achieve

**Observed patterns**:
- When end users can articulate clear value → higher adoption rates
- When end users cannot articulate value → adoption challenges, even with innovative features
- Some end users adopt without full clarity, then discover value through use (progressive discovery)

**Value types end users seek** (but aren't limited to):
- Identity and belonging
- Financial gain
- Short-term benefits
- Long-term benefits
- Status and recognition
- Capability enhancement
- Time savings
- Problem resolution

**Role of value scenarios**:
Product attributes themselves are not end-user value. Analysis should use scenarios to connect product attributes to end-user outcomes, define the context where value occurs, and provide a basis for judging whether the value holds.

Attributes still matter. Attributes provide evidence; scenarios help establish the relationship between attributes and end-user outcomes.

Value scenarios should also specify the value relation position: what external object end users confirm as valuable to them, what outcome they achieve for themselves through the product, and whether they themselves, their actions, or their outputs are confirmed as valuable through feedback.

## The Challenge

Most product creators face a hidden problem: **end users often don't know what they actually want, and how they articulate it may be wrong**.

The job isn't just to build what end users ask for - it's to help end users discover what value they're actually seeking.

## How to Engage with This Skill

This skill operates through conversational analysis. When the user presents a product idea, feature, copy, usage scenario, or use case:

1. **Identify the end users** - Determine who will use the product
2. **Identify the value scenario** - Determine the concrete context where value occurs and specify the value relation position
3. **Evaluate through four dimensions** - Value clarity, timeline, perception, discovery
4. **Adjust output to the request** - Full analysis, copy, usage scenarios, use cases, or diagnostic assessment
5. **Consider context** - Each product, market, and end user group differs

**This framework guides thinking. It does not prescribe solutions.**

**Analysis approach:**
- Evaluate through four dimensions around the current value scenario
- Adjust output format to the current request:
  - When evaluating a product idea, fully analyze all four dimensions
  - When writing copy, usage scenarios, or use cases, first judge whether value is clear, when value occurs, whether the outcome is perceivable, and whether end users need to discover value through use
  - When diagnosing existing copy or a value proposition, use the four dimensions to assess effectiveness
  - When the user asks to optimize a product or provide improvement directions, first use the value scenario and four dimensions to explain how the current object of analysis affects how end users obtain, understand, verify, or perceive outcomes in a concrete value scenario, then propose adjustments related to value realization or value communication; adjustments should be derived from the value scenario and analysis conclusions, and should distinguish judgments supported by current evidence from hypotheses requiring verification
- Before outputting status indicators in the four-dimension analysis, read `references/scoring-rubric.md`
- Analysis process for each dimension:
  1. Explain the analytical reasoning for this dimension (why this dimension matters for this product)
  2. Systematically apply the dimension's analytical methods to the current product idea, feature, copy, usage scenario, or use case, stating the preconditions and applicability boundaries the judgment depends on (cannot skip the analysis and jump directly to questions)
  3. After completing the dimension analysis, provide status assessment using status indicators (🔴🟡🟢) with specific description of current state (not vague generalizations). Check the status against `references/scoring-rubric.md`
  4. When citing product cases, base on verifiable information and explain relevance to current product (case applicability assessment in "Research Methodology" section)
  5. Pose sharp questions that directly challenge product necessity or require comparison with existing solutions
- After completing all four dimensions, provide summary
- Avoid logical gaps, show complete reasoning chain
- Guide users to make decisions based on analysis

## Analysis Framework

Analyze these four dimensions around the current value scenario to evaluate whether end users will discover value:

### 1. Value Clarity

**Examine**:
- Can end users articulate what they'll achieve?
- Is the value proposition clear or vague to end users?
- Do end users understand the outcome, not just the features?
- Can end users explain the relationship between the product and their task in a concrete scenario?

**Why this matters**:
End users won't adopt a product if they can't explain to themselves (or others) why they're using it.

**Real example - Dropbox** (see `references/real-cases.md` for detailed data):
- Clear value to end users: "I can access my files from any device"
- End users immediately understood what they'd achieve
- Not about "cloud storage" (technical) but about "access anywhere" (value)
- Insight: Translate technical features into user-facing value

**Real example - Google Wave** (see `references/real-cases.md` for detailed analysis):
- Vague value to end users: "Unified communication"
- End users couldn't explain what they'd achieve
- Shut down 14 months after launch despite innovative features
- Lesson: Features without clear value = no adoption

**Analysis method**:
Ask: What would an end user say when asked "Why are you using this?" If the answer is unclear or feature-focused ("because it has X"), dig deeper into the actual value proposition. Then check whether the answer maps to a concrete scenario: under what conditions, to complete what task, and to obtain what result.

### 2. Value Timeline

**Examine**:
- Is the value immediate or delayed for end users?
- If delayed, do end users know it's coming?
- What keeps end users engaged during the journey?
- In the value scenario, does value occur immediately, later, or through sustained accumulation?

**Why this matters**:
Both short-term and long-term value are valid approaches. The choice depends on the product's nature, specific scenarios, and end user context. Neither is inherently superior.

**Short-term value products** (end users see results in minutes/hours):
- Dropbox: Upload → see file on other device (< 5 minutes)
- Zoom: Click link → join meeting (< 30 seconds)
- Stripe: Run test payment → see it work (< 1 minute)
- Key consideration: Immediate value is the complete product

**Long-term value products** (end users see results in weeks/months):
- Duolingo: Language fluency (6-12 months)
- Fitness apps: Body transformation (3-6 months)
- Investment apps: Wealth building (years)
- Key consideration: End users commit to the journey

**Design approaches available**:
- Pure short-term: Deliver immediate value, that's the complete product
- Pure long-term: End users are committed to the journey, no short-term touchpoints needed
- Hybrid: Long-term goal with optional short-term touchpoints (XP, streaks, milestones)
- All three approaches are valid - choose based on product nature and end user context

**Analysis method**:
Identify the primary value timeline. Assess whether the approach matches the product's nature, the current value scenario, and target end users' expectations. Don't force short-term mechanisms if end users are already committed to long-term goals.

### 3. Value Perception

**Examine**:
- Can end users see/feel what they achieved?
- Is progress tangible or abstract to end users?
- Can end users show others what they've achieved?
- In the value scenario, what concrete result can end users point to and say "I achieved this"?

**Why this matters**:
Invisible value feels like no value to end users. Progress must be perceivable.

**Note**: "Perceivable" takes different forms across product types:
- Consumer products: Immediate visual feedback in UI (file appears, photo enhanced)
- Enterprise software: Reports, dashboards, metrics, analytics
- Developer tools: Build outputs, test results, performance metrics
- The key is that end users can point to something concrete that shows value was delivered

**Visible outcomes for end users**:
- Dropbox: File appears on other device (tangible)
- Instagram: Beautiful photo with likes (tangible)
- GitHub: Contribution graph (tangible)
- Duolingo: Streak counter (tangible)
- Observation: These products make achievements visible and shareable

**Invisible outcomes** (problematic for end users):
- "Your data is synced" (abstract, can't see it)
- "Security improved" (no visible change)
- "Algorithm optimized" (nothing looks different)
- Observation: Technical improvements are difficult for end users to perceive without visible manifestations

**Analysis method**:
Identify what end users can point to and say "I achieved this". If the value is invisible, explore ways to make it tangible through UI, notifications, progress indicators, result comparisons, or scenario feedback.

### 4. Value Discovery

**Examine**:
- Do end users already know they want this?
- Or will end users discover the value after using it?
- How to help end users discover value they don't yet recognize?
- Do end users know the value before entering the scenario, or do they need to experience the scenario before recognizing it?

**Why this matters**:
Sometimes end users don't know what they want until they experience it. The product must help them discover it quickly.

**Discovery pattern - Instagram** (see `references/real-cases.md` for growth data):
- End users thought they wanted: "Share photos"
- End users discovered they valued: "Become a photographer" (identity)
- Instagram helped discovery through filters, likes, and social validation
- Insight: Instagram's success came from enabling identity transformation, not just photo sharing utility

**Discovery pattern - Notion**:
- End users thought they wanted: "Take notes"
- End users discovered they valued: "Become organized" (identity)
- Notion helped discovery through flexible databases and templates

**Analysis method**:
Determine whether end users already know what they want, or need to discover it. If discovery is needed, identify the fastest path to the "aha" moment through onboarding, tutorials, example scenarios, or progressive feature revelation.

## Patterns from Real Products

These aren't rules to follow - they're observed patterns to consider when analyzing specific situations.

For detailed case studies with real data, see `references/real-cases.md` (English) or `references/real-cases-zh.md` (中文).

### Pattern: Value Communication

**Products using concrete outcome descriptions**:
- Dropbox: "Access files from any device"
- Instagram: "Become a photographer" (identity transformation)
- Observation: These products use concrete, achievable outcome descriptions

**Products using technical or feature descriptions**:
- Google Wave: "Unified communication" (technical concept)
- Some products: "Cloud storage with 2GB free" (feature list)
- Some products: "Distributed file synchronization" (technical jargon)
- Observation: These descriptions make it harder for end users to understand what they'll achieve

## Real Examples

For complete case studies with metrics and data sources, see `references/real-cases.md`.

## When This Framework Applies

**Most applicable for**:
- Consumer products (B2C)
- Competitive markets (end users have alternatives)
- Products requiring adoption and retention
- New product categories (end users don't know what to expect)
- Situations where value propositions, usage scenarios, or feature-to-value explanations need to be expressed clearly

**Less applicable for**:
- Enterprise software (decision makers ≠ end users, switching costs high)
- Monopoly products (end users have no choice)
- Products where value is inherently delayed (investing, insurance)

## Common Pitfalls

### Pitfall 1: Assuming End Users Know What They Want

**The trap**: Building exactly what end users ask for
**The reality**: End users often don't know what they actually need
**The approach**: Help end users discover the real value through conversation and exploration

### Pitfall 2: Focusing on Features Instead of Value

**The trap**: "Our product has X, Y, Z features"
**The reality**: End users don't care about features, they care about what they'll achieve
**The approach**: Always translate features into value: "Feature X helps end users achieve Y"

### Pitfall 3: Mechanically Copying Patterns

**The trap**: "Duolingo uses streaks, so we should too"
**The reality**: Treating other products' features or patterns as value itself, but not understanding under what conditions and through what mechanisms these features generate value for their end users
**The approach**: Understand why these features or patterns generate value for their end users: in what scenarios, solving what problems, achieving what outcomes, then judge whether they apply to current end users

### Pitfall 4: Invisible Value

**The trap**: "Our algorithm is 10x better"
**The reality**: If end users can't see/feel the improvement, it doesn't matter
**The approach**: Make value tangible and visible to end users

### Pitfall 5: Cross-Context Misuse

**The trap**: "Dropbox succeeded with clear value, so the same conclusion can be applied directly"
**The reality**: Conclusions only hold under specific preconditions; when original and current contexts differ, the original conclusion may fail
**The approach**: Restore the preconditions behind the conclusion and verify whether the current situation satisfies them

### Pitfall 6: Cross-Level Misuse

**The trap**: "A judgment holds at the feature level, so the whole product is already validated"
**The reality**: Local, tactical, short-term conclusions do not equal whole-product, strategic, long-term conclusions
**The approach**: Determine which level the conclusion belongs to, then judge whether it can be generalized upward

### Pitfall 7: Discussing Value Without Scenarios

**The trap**: "This feature improves efficiency, reduces cost, and improves experience"
**The reality**: If the scenario where value occurs is not specified, end users will have difficulty judging whether the value applies to them
**The approach**: Map abstract value to a concrete scenario: who completes what task under what conditions, and what result they get through the product

### Pitfall 8: Confusing Different Value Relation Positions

**The trap**: Because end users find content interesting, information useful, the product easy to use, or experience short-term enjoyment, the product is judged to already support the claimed end-user outcome
**The reality**: These phenomena may correspond to external-object value confirmation or a short-term end-user outcome, but they do not mean that other outcomes such as capability, identity, relationships, contribution, task completion, or being needed have occurred
**The approach**: Specify the value relation position and judge whether the position that has occurred is sufficient to support the product claim; do not extrapolate evidence from one position into a result in another position

### Pitfall 9: Treating Contextual Certainty as Universal Certainty

**The trap**: "This judgment holds in another context, so it can be applied directly in the current context"
**The reality**: Judgments often depend on specific conditions. When the current context does not satisfy those conditions, whether the original judgment holds remains a hypothesis requiring verification
**The approach**: Restore the key conditions the original judgment depends on and check whether the current context satisfies them; if key conditions do not hold or have changed, downgrade the original judgment to a hypothesis requiring verification and identify what evidence is needed

## Research Methodology

### Verify Information Accuracy

When citing real product cases, base on verifiable information and explain relevance to current product.

**Tool Availability**:
- WebFetch and WebSearch available for verifying information
- When research fails, proceed with analysis based on framework and clearly indicate which information needs verification

### Verify Value Scenarios

Value scenarios can be proposed as analytical hypotheses, but they cannot be treated directly as facts.

**Distinguish between**:
- **Hypothetical value scenarios**: Possible use contexts, value relation positions, and outcomes inferred from the product, end users, and features
- **Validated value scenarios**: Use contexts, value relation positions, and outcomes supported by user interviews, behavior data, real cases, market materials, or product usage evidence

When a value scenario is unvalidated, state clearly that it is an analytical hypothesis and identify what evidence would be needed to verify its context, value relation position, and outcome.

### Condition Archaeology

**Why it matters**:
Theories and cases are distilled from concrete experience by stripping away contextual factors for easier transmission. What gets stripped away is often not noise, but necessary constraints for the conclusion to hold.

When you see "Dropbox succeeded with clear value," you receive the conclusion itself, but not the preconditions that made it work: competitive market, voluntary choice, individual users, immediate value delivery.

**Common problem**:
Users believe they've grasped the "law" and apply it directly to new contexts, but the new context may lack the necessary constraints. When it fails, they blame execution or luck rather than checking whether conditions match.

**How to apply**:
Before citing any theory, pattern, or case conclusion:
1. Restore the necessary constraints the original conclusion depends on
2. Verify whether those conditions hold in the current context
3. Determine the range and boundaries where the conclusion applies

**Key considerations**:
- Conditions change. Judgments that hold at one stage may fail after conditions change
- Don't seek to exhaust all conditions, but identify decisive conditions, main failure conditions, and mismatch risks
- Theory holds under specific conditions and may fail beyond those conditions

**Check during analysis**:
- Which preconditions and constraints the original case or conclusion depended on
- Whether those conditions hold in the current situation
- The range within which the conclusion holds, and beyond which it stops holding
- Whether the current discussion concerns a local judgment, a whole-product judgment, a short-term judgment, or a long-term judgment
- Whether the conclusion still holds after conditions change

**Common misuses**:
- **Cross-context misuse**: Taking a conclusion that holds under one set of preconditions and transferring it directly into a situation with different preconditions
- **Cross-level misuse**: Treating a local judgment, tactical judgment, or short-term judgment as if it were a whole-product judgment, strategic judgment, or long-term judgment

### Condition-Certainty Mismatch

**Definition**:
Condition-certainty mismatch describes a reasoning bias: when forming a current conclusion, a prior judgment that holds only under specific conditions is treated as if it also holds in the current context, turning a hypothesis that still requires verification into a premise for judgment. Here, certainty does not mean an unconditionally valid proposition; it means a judgment's status as a usable premise under specific conditions.

**How to apply**:
When a prior judgment needs to be used in the current context:
1. Restore the key conditions the judgment depends on
2. Check whether those conditions hold or have changed in the current context
3. Determine whether condition differences affect the judgment's applicability in the current context
4. Downgrade any part not supported by current conditions to a hypothesis requiring verification

**Key considerations**:
- When key conditions do not hold, have changed, or lack sufficient evidence, the prior judgment should not be used directly as a premise for the current conclusion
- Condition differences do not mean the prior judgment is invalid; explain how the differences affect the current judgment
- The result alone cannot establish the cause; attribution should state the reasoning basis and identify what additional evidence is needed

### Evaluating Case Study Applicability

The cases in `references/real-cases.md` (Dropbox, Instagram, Duolingo, WeChat, Google Wave, Quibi) illustrate patterns, rather than universal rules. Before using them, restore the preconditions behind those cases and then judge whether they transfer.

**Assess applicability**:
- **Product type match**: B2C consumer apps vs B2B developer tools vs enterprise software
- **Market context match**: Competitive markets vs niche markets vs monopoly situations
- **User behavior match**: Daily use vs episodic use vs one-time transactions
- **Value delivery match**: Immediate utility vs long-term transformation vs hybrid approaches

**When cases don't apply**:
If the user's product differs significantly from reference cases (e.g., B2B infrastructure tool vs C2C social app), search for comparable products in the same domain. Analyze those domain-specific examples instead of forcing consumer app patterns onto different contexts.

If you still need to borrow a cross-domain example, first restore the preconditions the original case depends on, then judge which of those conditions are shared and which are different in the current product, and distinguish whether the comparison concerns a local judgment, a whole-product judgment, a short-term judgment, or a long-term judgment.

**Example**:
- User discusses: Developer infrastructure tool (like Temporal, Kubernetes)
- Reference cases: Consumer apps (Dropbox, Instagram)
- Action: Search for similar developer tools, analyze their value propositions, adoption patterns
- Avoid: Applying Instagram's identity transformation pattern to infrastructure software

### Balancing Exploration and Evidence

**Exploratory thinking** (appropriate when):
- Identifying potential value types end users might seek
- Brainstorming ways to make value visible or tangible
- Considering multiple positioning approaches
- Exploring hypothetical value scenarios for product direction

**Evidence-based analysis** (required when):
- Claiming specific adoption patterns or metrics
- Comparing to real products or market examples
- Stating what "works" or "doesn't work" in practice
- Conducting analysis based on industry precedents

**Process**:
1. Explore possibilities through discussion and brainstorming
2. When specific claims or comparisons arise, verify with research
3. Conduct analysis based on verified patterns, not assumptions
4. Acknowledge when evidence is limited or context differs from known cases

### Research Sources

**Primary sources** (preferred):
- Official product websites and documentation
- Company blog posts or announcements
- Published metrics, user counts, or growth data
- Academic research or industry reports

**Secondary sources** (use with caution):
- Tech news articles or analysis pieces
- User reviews or community discussions
- Third-party market research or estimates

**Avoid**:
- Relying solely on memory or general knowledge
- Assuming patterns from one domain apply universally
- Making claims without verifiable sources
- Treating reference cases as prescriptive templates

## Guiding Principles

### Core Distinctions

**User vs End user**:
- User: The person using this skill (product creator, PM, designer, entrepreneur, etc.)
- End user: The person who will use the product being discussed
- These are distinct roles with different perspectives

**Features vs Value**:
- Features: What the product does (technical capabilities)
- Value: What result end users obtain or achieve through the product in a concrete value scenario
- End users adopt products based on value, not features

**Value perception timing**:
- Immediate perception: End users perceive they gained something during or right after use
- Delayed perception: End users perceive they gained something after sustained use over time
- These are not mutually exclusive; products can provide both
- Neither is inherently superior; each addresses different end user needs

### Research Approach

**When encountering unfamiliar concepts**:
- Research mentioned products, technologies, or domain-specific terms
- Use WebFetch or WebSearch to gather current information
- Seek official documentation, published metrics, and verified sources

**Balancing exploration and evidence**:
- Exploratory thinking: Appropriate when identifying potential value types or brainstorming approaches
- Evidence-based analysis: Required when claiming specific patterns, comparing to real products, or stating what works in practice

**Evaluating case applicability**:
- Reference cases illustrate patterns, not universal rules
- Assess whether product type, market context, user behavior, and value delivery match
- When cases do not apply, research comparable products in the relevant domain

## How to Use This Skill

This skill works best in conversation. When the user discusses a product idea, feature, copy, usage scenario, or use case:

1. **Identify the value scenario**: In what context do end users use the product, what is the value relation position, and what result do they get?
2. **Explore value clarity**: Can end users articulate what they'll achieve?
3. **Examine the timeline**: Is value immediate or delayed for end users? What's appropriate for this product?
4. **Assess perception**: Can end users see/feel their progress?
5. **Discover hidden value**: What value might end users not yet recognize?

**This isn't a checklist** - it's a way of thinking. Each product is different. Each market is different. The goal is to think clearly about whether end users will know what value they'll achieve in a concrete scenario.

**Research during analysis**: When the user mentions specific products, technologies, or concepts, this skill may research them via WebFetch or WebSearch to provide context-appropriate analysis based on current information rather than assumptions.

## Key Principles

1. **End users must "know" what value they'll achieve** - even if it takes time
2. **Value types are diverse** - identity, money, benefits, status, capability, and more
3. **End users often don't know what they want** - help them discover it
4. **Perception matters to end users** - invisible value feels like no value
5. **Context is everything** - patterns from one product may not apply to others
6. **Test with real end users, don't assume** - validate the context, value relation position, and outcome in a concrete value scenario
7. **Both short-term and long-term are valid** - neither is superior, choose based on product nature

## Additional Resources

### Reference Files

Case studies include quantitative data and data sources:
- **`references/real-cases.md`** - Dropbox, Instagram, Duolingo, WeChat, Google Wave, Quibi case studies (English)
- **`references/real-cases-zh.md`** - Dropbox、Instagram、Duolingo、微信、Google Wave、Quibi 的案例分析（中文）

Status indicator reference criteria:
- **`references/scoring-rubric.md`** - Reference criteria for status indicators (🔴🟡🟢) across four dimensions: value clarity, timeline, perception, discovery (English)
- **`references/scoring-rubric-zh.md`** - 价值清晰度、价值时间线、价值感知、价值发现四个维度的状态指示符（🔴🟡🟢）参考标准（中文）

Methodology research documents:
- **`references/learning-methodology.md`** - Analysis of operating mechanisms, risks, manifestations in different environments, and integration methods of foundation-first and problem-driven learning approaches (English)
- **`references/learning-methodology-zh.md`** - 基础优先型和问题驱动型学习方式的运作机制、风险、环境表现及结合方式分析（中文）

## Remember

This skill helps think about value, not prescribe solutions. Every product is unique. Every market is different. The goal is to discover whether end users will clearly understand what they'll achieve - because that understanding is what drives adoption.

---
> Source: [Done-0/value-realization](https://github.com/Done-0/value-realization) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
