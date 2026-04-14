## openehr-module2-vibe-rules

> POST a flat openEHR composition

POST a flat openEHR composition

curl --location 'http://[baseUrl]/rest/openehr/v1/ehr/[ehr_id]/composition?templateId=[template_id]' \
--header 'Accept: application/openehr.wt.flat.schema+json' \
--header 'Content-Type: application/openehr.wt.flat.schema+json' \
--header 'Prefer: return=representation' \
--data '{
    [Flat composition]
}'

Update a flat composition

curl --location --request PUT 'https://[baseUrl]/rest/openehr/v1/ehr/[[ehr_id]]/composition/[[composition_id]]?templateId=[[template_id]]' \
--header 'Accept: application/openehr.wt.flat.schema+json' \
--header 'Content-Type: application/openehr.wt.flat.schema+json' \
--header 'Prefer: return=representation' \
--header 'If-Match: [versioned_composition_id]' \
--data '{
     [[FLAT COMPOSITION]]
}'


Listing the created compositions using AQL

curl --location 'https://[baseUrl]/rest/openehr/v1/query/aql' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--data '{"q": "SELECT c/uid/value from EHR e CONTAINS COMPOSITION c WHERE c/archetype_details/template_id/value ='\''[[template_id]]'\'' AND e/ehr_id/value='\''[[ehr_id]]'\''"}'

Deleting a compositon

curl --location --request DELETE 'https://[baseUrl]/rest/openehr/v1/ehr/[ehr_id]/composition/[versioned_composition_id]' \
--header 'Accept: application/json' \
--header 'Content-Type: application/json' \
--header 'Prefer: return=representation' \

Fetching a flat openEHR composition from ehrbase

curl --location 'https://[baseUrl]/rest/openehr/v1/ehr/[ehr_id]/composition/[composition_id]' \
--header 'Accept: application/openehr.wt.flat.schema+json'

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vipinsanthosh)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/vipinsanthosh)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
