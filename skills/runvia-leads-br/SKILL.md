---
name: runvia-leads-br
description: >-
  Builds Brazilian B2B lead lists with Runvia: finds businesses by segment and city on Google
  Maps or in the CNPJ registry, enriches each one with Receita Federal data (CNPJ, partners,
  activity, size, Simples Nacional status, state registration), reads the company's website for
  public contacts, and looks for buying signals in news and job posts. Output is a CSV with
  sources and the cost per row. Triggers: "lista de leads", "prospecção", "empresas de <setor>
  em <cidade>", "consulte o CNPJ", "sócios da empresa", "enriquecer CNPJ", "leads B2B".
license: Apache-2.0
metadata:
  cli-version: 0.2.0
---

# Brazilian leads with Runvia

Read the `runvia` skill first. CNPJá data comes from public registries (Receita Federal,
Simples Nacional, SINTEGRA); Google results come through Serper; websites through Firecrawl.

## 1. Brief

Ask in one message: the segment, the cities or states, company size, what makes a company a
good fit (and what rules one out), how many leads, and the budget. Then show the plan with the
price per step and the total.

## 2. Find companies

Two sources; use either or both.

- **By the registry (cleanest).** `cnpja.office.search` filters by activity code (CNAE,
  `mainActivity.id.in`), city (`address.municipality.in`, IBGE code) or state
  (`address.state.in`), size (`company.size.id.in`), Simples (`company.simples.optant.eq`),
  status (`status.id.in`, 2 = active), founding date and whether a phone or email exists
  (`phones.ex`, `emails.ex`). `limit` is required; it bills 7.965 credits per call plus 8.85 credits per 10 rows. Find the
  CNAE code first (search Google for "CNAE <segment>" with `serper.google.search`) and confirm it
  with the user.
- **By the map.** `serper.google.places` (`q` "<segment> em <cidade>", 1.1 credits per page) or
  `serper.google.maps` (3.3 credits) returns name, address, phone, rating, review count and
  website. Always put the city inside `q`. It has no CNPJ; match it in step 3.

## 3. Enrich

| Field | Tool | Price |
|---|---|---|
| Full company record by CNPJ | `cnpja.office.lookup` (`taxId`) | 8.85 cr |
| Record with extras (partners, registrations) | `cnpja.office.details` (`taxId`) | 53.1 cr |
| Simples Nacional and MEI | `cnpja.simples.lookup` (`taxId`) | 8.85 cr |
| State registration | `cnpja.ccc.lookup` (`taxId`, `states`) | 8.85 cr |
| Partners by name | `cnpja.person.search` (`name.in`, `limit`) | 7.965 cr per call plus 8.85 cr per 10 rows |
| Website text and public contacts | `firecrawl.scrape` (`url`) | 3 cr |
| Company's CNPJ from its name | `serper.google.search` (`q`) "<nome> CNPJ <cidade>" | 1.1 cr |

Filter on what you already have before you enrich: drop inactive companies, wrong sizes and
places with no website when the user needs one. Run the 53.1-credit details call only on the
final list.

## 4. Buying signals (optional)

- News in the last 30 days: `serper.google.news` with the company name.
- Hiring: `tikhub.linkedin.web-v2.search-jobs` (`keywords`, `geo_code`) or a Google search for
  "<empresa> vagas".
- Reviews that show a problem you solve: `serper.google.reviews` on the place.
Give each lead one line on why it fits now, with its source link.

## 5. Output

CSV columns: razão social, nome fantasia, CNPJ, cidade, UF, CNAE principal, porte, Simples,
telefone, e-mail, site, nota no Google, sinal, fonte, custo. Leave a field empty when no source
returned it and say how many rows have each field. Use only business contact data the sources
return; do not guess emails. Report the credits spent per step and per lead.
