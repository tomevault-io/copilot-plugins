## site-mapper-agents

> |


# site-mapper-agents skill

`site-mapper-agents` is a Python library that builds typed extractors for
undocumented browser portals from a burst of CDP network traffic, and
auto-heals them when the portal's API changes.

## When to use

Trigger this skill when the user describes any of:

- "I want to scrape data out of `<some SaaS portal>` but they don't have an API."
- "Our extractor keeps breaking when the vendor updates their backend."
- "I have a Chrome extension capturing CDP traffic — how do I turn it into
  a structured schema?"
- "Build a self-healing connector for `<CRM/portal/dashboard>`."
- "Discover the API of `<X>` by watching network traffic."
- "Use an LLM to figure out which endpoints carry the data I want."

## Core concept

Three agents:

1. **Architect** — runs once during onboarding. Takes a burst of
   `CDPNetworkEvent` objects, classifies endpoints using a
   caller-supplied vocabulary, produces a `SiteSchema` and
   `NetworkSignature` list.
2. **Eavesdropper** — runs on every live event. Pure Pydantic validation,
   sub-millisecond. Extracts data using the saved signatures. No LLM.
3. **Healer** — runs only when the Eavesdropper reports a mismatch.
   Re-maps stale keys via structural + fuzzy + LLM semantic matching.

## Minimal usage

```python
from site_mapper_agents import (
    Architect, Eavesdropper, Healer,
    CDPNetworkEvent, TargetField, UserIntent,
)
from pydantic_ai.models.anthropic import AnthropicModel

intent = UserIntent(
    description="Account details",
    target_fields=[TargetField(name="account_id"), TargetField(name="email")],
)

# 1. Onboard
arch = Architect(model=AnthropicModel("claude-sonnet-4-5"))
for ev in cdp_burst:
    arch.record_traffic(ev)
proposal = await arch.propose(target_url=URL, user_intent=intent)
site = arch.build_mapped_site(proposal=proposal, target_url=URL, user_intent=intent)

# 2. Extract
eaves = Eavesdropper()
result, _ = eaves.ingest(live_event, sites=[site])

# 3. Heal (only fires on failure)
healer = Healer(model=AnthropicModel("claude-sonnet-4-5"))
if isinstance(_, ExtractionFailed):
    patch = await healer.diagnose(site=site, failed_event=_, new_response_body=live_event.body)
    healer.apply_patch(site, patch)
```

## Custom vocabularies

If the user is mapping a site whose endpoints don't fit generic CRUD
shapes (e.g. invoicing, ticketing, healthcare claims), define a
custom vocabulary:

```python
from site_mapper_agents import default_vocabulary, define_endpoint_type, merge_vocabularies

vocab = merge_vocabularies(
    default_vocabulary(),
    [define_endpoint_type(
        name="claim_status",
        description="Returns insurance claim adjudication status",
        expected_fields=["claim_id", "status", "denied_reason"],
    )],
)

arch = Architect(model=my_model, vocabulary=vocab)
```

## CDP capture

The library does not capture CDP traffic. Pair it with
`axumquant/cdp-network-interceptor` or any Chrome extension /
Puppeteer / Playwright session that emits `CDPNetworkEvent` shaped objects.

## Common pitfalls

- The Architect is the only expensive part — call it once per site.
- The Eavesdropper has no LLM calls and is sub-millisecond.
- Always wire the Healer if you care about the extractor staying
  alive when the upstream API changes shape.
- Iframe traffic requires the CDP forwarder to populate `frame_origin`
  on the event.

## Reference

See the [project README](README.md) for the full API reference, sequence
diagram, and provider integration guide.

---
> Source: [axumquant/site-mapper-agents](https://github.com/axumquant/site-mapper-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
