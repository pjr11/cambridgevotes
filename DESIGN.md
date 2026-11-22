# Cambridge Votes: design and implementation plan

## Purpose

Cambridge Votes will be a fast, accessible, non-partisan static website for exploring Cambridge election results and political history. It should make both the latest picture and the underlying detail easy to understand, while preserving the context and uncertainty in historical records.

The initial reference is [Cambridge Election Results](https://www.cambridgeelections.org.uk/index.htm), a substantial archive compiled by Colin Rosenstiel and maintained by Keith Edkins. Its index points to city results since 1935, county material, councillors, maps, candidates and Cambridge constituency parliamentary elections since 1832. Some links to complete annual result pages on its index are dated, so coverage must be established by auditing the underlying pages, not by assuming the index lists every year. The existing [introduction](https://www.cambridgeelections.org.uk/intro.htm) explains important historical qualifications.

This document is a proposal, not a claim that the data has already been acquired, verified or licensed.

## Goals and scope

- Make it easy to answer: who won, by how much, where, when, and how does that compare with previous elections?
- Show complete candidate-level results, not just winners, with turnout, rejected ballots and electorate where reliably available.
- Explain the difference between an election result and the composition or control of a council.
- Make data sources, methods, gaps and corrections visible. Offer reusable downloads when rights permit.
- Work well on phones, with accessible tables and meaningful content without JavaScript.

**First release:** Cambridge City Council elections: a verified, complete recent election as a vertical slice, then additional years and by-elections as the data is checked. Include only features supported by the records actually acquired. Clearly state the published coverage and latest verified election date; do not imply that a partial archive is complete or that an election result describes the current council.

**Later:** extend the same model to county divisions and Cambridge parliamentary elections, followed by historical membership, other offices, richer comparisons and maps as sources permit. The initial local scope is the city, not all of Cambridgeshire; parliamentary constituency boundaries are a separate geography. Avoid promising a complete historical archive until a source and verification audit supports it.

## Audience and principles

Residents should get an approachable overview; researchers and journalists should be able to inspect candidate-level figures and sources. The site should be factual and politically neutral: party colour helps identification, not editorial emphasis. Text labels must accompany colours, uncertainties must be explicit, and analysis must never silently fill gaps.

## Information architecture

Use descriptive, durable URLs and normal links. The examples below are proposed routes, not evidence that these pages already exist.

| Page | Purpose and contents |
| --- | --- |
| Home `/` | Latest **verified published election**, date and coverage; concise result highlights; links to elections, areas and methodology. If a separate, dated membership dataset exists, show council composition as of its stated date; otherwise omit it. |
| Elections `/elections/` | Browse by year and election type; distinguish scheduled elections and by-elections, including multiple elections in one year. |
| Election `/elections/city/2018-05-03/` | Date and election type, seats contested, results by ward, aggregate figures only where valid, and links to individual contests and sources. Group simultaneous by-elections explicitly rather than hiding them in a year total. |
| Contest `/elections/city/2018-05-03/trumpington/` | Candidates, party labels, votes, vote shares and elected status; seats, electorate, turnout, rejected ballots, majority and notes when known. Link to the area and source. |
| Areas `/areas/` and `/areas/city-wards/trumpington/` | Browse wards and their results over time. Identify boundary eras and warn when the same name represents a different geography; do not imply a continuous series is geographically identical. |
| About and methodology `/about/` | Coverage, sources and credits, data definitions, known gaps, calculation rules, corrections and contact/reporting mechanism. |

Candidate and party pages, county and parliamentary routes, search, downloadable data and visual maps are subsequent features once the data, identity matching and rights justify them. Avoid blank or speculative pages. Provide indexes, breadcrumbs and related links so tables remain navigable without search.

## Experience and visual direction

- A contemporary editorial style: warm off-white background, dark ink, restrained accent colour, generous spacing and clear typographic hierarchy. Use a simple Cambridge-inspired identity without relying on official council branding.
- Lead with a short, dated summary; place the full result table close by. Use small horizontal bars and carefully labelled seat diagrams or timelines only when their input and definitions are sound. Never rely on colour alone; include values and party names.
- On small screens prioritise readable candidate, party, votes and outcome columns. Allow labelled horizontal scrolling for wider tables rather than hiding important values. Use accessible HTML tables with captions and headers, and expose exact values as text.
- Ensure keyboard access, visible focus, sufficient contrast, sensible heading order, descriptive links and reduced-motion support. Charts must have an equivalent table or textual summary. Target WCAG 2.2 AA and test with keyboard and a screen reader.
- Build complete HTML at build time, with progressive enhancement for optional filters or charts. Use lightweight assets and avoid a mandatory client-side framework, external tracking or third-party requests for core content.

## Data design

Keep curated, version-controlled source data separate from presentation and generate pages and downloads at build time. Prefer simple structured JSON files, validated against a schema; use stable IDs and ISO dates, not display names or years alone, for relationships and URLs. A possible model:

- **Election:** ID, date (or documented date precision if unknown), authority, type, scheduled/by-election classification, source references and notes. A single date may contain multiple distinct elections.
- **Geography:** stable ID, name, geography type, authority and boundary version with validity dates and source. Separate city wards, county divisions and parliamentary constituencies; do not join them just because names match.
- **Contest:** election and geography IDs, seats available, contest status (contested or unopposed), electorate, ballot papers cast, valid and rejected ballots, and source notes. Store unknown values as unknown, never zero.
- **Candidacy/result:** contest, person and party IDs, source name and party label, votes if available, elected status and relevant notes. An unopposed candidate can be elected without a vote total. Support multiple winners, tied votes and exceptional outcomes without guessing.
- **Person and party:** stable IDs with recorded names and aliases; retain the party description used at the time. Do not merge people based solely on matching names or silently merge historical party labels.
- **Sources and provenance:** title, URL or archival reference, publisher/author, retrieval date, licence or permission status and record-level citations where practical. Keep original values and transcription notes alongside corrections.
- **Council composition snapshot (later):** authority, observed date or date precision, party seat counts, vacancies if known, source and coverage. Record a reported administration or control description separately from seat counts, with its source and definition. A snapshot can stand alone without a complete event history; it does not establish membership between snapshots.
- **Membership events (later):** appointments, elections, departures and party changes with effective dates and sources, separate from contest results. A composition or control snapshot needs a stated date and coverage, including vacancies and uncertainty.

### Boundary and map sources

Maps are a later feature, but the data model should be ready for dated geometries. Promising sources to investigate are:

| Source | Potential use and limitations |
| --- | --- |
| [ONS Open Geography Portal](https://geoportal.statistics.gov.uk/) | Dated, downloadable boundary datasets: for example, ward vector boundaries as at [December 2011](https://www.arcgis.com/home/item.html?id=3ca4d8430072442bb3dc1d673a670b40) and [December 2021](https://www.arcgis.com/home/item.html?id=1f3ee65f8ff942908f8bea51a3eb10d2), [May 2023 county electoral divisions](https://www.arcgis.com/home/item.html?id=710c5509752e437dbf2252bde16f289f), and [July 2024 parliamentary constituencies](https://www.arcgis.com/home/item.html?id=e489d4e5fe9f4f9caa6af161da5442af). These are national snapshots, not proof that each geometry applies to a particular Cambridge contest; check the relevant features, dates and boundary changes. ONS [licensing guidance](https://www.ons.gov.uk/methodology/geography/licences) specifies OGL and ONS/OS attribution for digital boundaries. |
| [Ordnance Survey Boundary-Line](https://www.ordnancesurvey.co.uk/products/boundary-line) | GB administrative and electoral boundary polygons in several GIS formats, updated twice yearly. OS [documentation](https://docs.os.uk/os-downloads/products/areas-and-zones-portfolio/boundary-line) describes it as OGL data. Useful for recent boundaries and a source for locally retained dated releases; check availability and terms of older releases individually. Its advertised late-19th-century historical data concerns **county** boundaries, not evidence of historic Cambridge ward coverage. |
| [mySociety MapIt](https://mapit.mysociety.org/docs/) | An API for discovering current and some historical areas, their codes and boundary polygons. Its Cambridge ward records and GeoJSON endpoints were available in a small check; see below for generations, access limits and unresolved reuse terms. |
| [Local Government Boundary Commission for England: Cambridge review](https://www.lgbce.org.uk/all-reviews/cambridge) | The order map, final report and link to previous reviews help establish boundary changes and names. The documents are evidence for validation, not automatically reusable vector polygons; check their rights. In particular, reconcile legal effective dates and delayed elections with the election actually contested before assigning a geometry. |
| [Cambridge Election Results historic maps](https://www.cambridgeelections.org.uk/wardmaps.htm) | A visual reference for city ward eras from 1935 onwards and the 2017 county divisions. These are images, not verified GIS polygons, and the page credits OS permission; do not copy, trace or publish derivatives without checking the necessary rights. Older machine-readable boundaries remain an open research question. |

For each geometry, record the source and licence, release and snapshot date, geography type, verified area code and name, coordinate system, and the election date range for which its use is evidenced. Maintain a reviewed crosswalk from contest geography to the matching geometry version; ONS/OS codes and Democracy Club ballot IDs are not interchangeable. Do not extrapolate a snapshot backwards or forwards merely because the ward name matches. Where geometry or its validity is uncertain, show the result in the accessible list/table without a map rather than using today's boundaries.

For a future map, extract the relevant areas from a pinned, licensed source; transform and, if needed, simplify into a static web-friendly format while preserving topology. Record processing and attribution, validate geometry and contest joins, and visibly distinguish unopposed, missing and contested results. Avoid treating a single winner as representing every seat in a multi-seat contest. Keep a linked results table and non-map navigation, with no external map service required for core use.

#### MapIt assessment

MapIt is a useful candidate for discovering and retrieving polygons without processing a GB-wide file. The [API documentation](https://mapit.mysociety.org/docs/) describes area lookups by MapIt ID or ONS code, child-area queries, area types and GeoJSON/KML/WKT polygon endpoints. The separate `/geometry` endpoint supplies centroid, extent and area, not the polygon. Some list and lookup endpoints accept a historical `generation`; area records expose generation ranges and external codes.

A check on 22 November 2026 resolved [Cambridge City Council](https://mapit.mysociety.org/area/2263.html) via GSS code `E07000008`. Its district-ward children query returned 14 wards for both [generation 33](https://mapit.mysociety.org/area/2263/children?type=DIW&generation=33) and [generation 43](https://mapit.mysociety.org/area/2263/children?type=DIW&generation=43). For example, [Arbury 2957](https://mapit.mysociety.org/area/2957.html) and [Arbury 164381](https://mapit.mysociety.org/area/164381.html) have different area and GSS IDs; both returned GeoJSON polygons. This is a feasibility check, not validation of all ward geometries or their use in particular elections. Area IDs are distinct from Democracy Club ballot IDs, and a shared name alone is not a join.

The [generations list](https://mapit.mysociety.org/generations) associates generation 12 with the October 2009 Boundary-Line release and generation 43 with May 2021. The documentation cautions that only generations 12 onwards actually exist, apart from some council names in generation 10. Consequently do not infer complete Cambridge polygon coverage before 2009 from lower `generation_low` values. Generation creation dates and labels are not themselves proof of the boundaries used at an election; verify the relevant ward/division and effective election date against boundary orders and other sources. Assess county and parliamentary coverage separately.

MapIt's [pricing](https://mapit.mysociety.org/pricing/) describes a keyless sandbox with 50 calls per day and subscription/charitable tiers; plans are subject to rate limits. Check current access, quotas and any applicable service terms before importing. The API documentation and pricing do not establish the terms for caching, redistribution or derived polygons: confirm the underlying data licences, required attribution and permission to store and publish extracted geometry before adopting it. If suitable, retrieve a reviewed, dated snapshot outside the site build and retain its area ID, generation, external codes, source and processing metadata. Do not expose an API key in static pages or make the live API a dependency of core pages. Postcode lookup could be explored later, but MapIt uses postcode centroids, which may fall across a boundary from some addresses; disclose that limitation.

### Open Council Data UK

[Open Council Data UK](https://opencouncildata.co.uk/) complements election results with council membership and composition. Its [Cambridge council page](https://opencouncildata.co.uk/council.php?c=176&y=0) lists councillors by ward and party and a current composition. Its [free downloads](https://opencouncildata.co.uk/productsAndDownloads.php) offer annual UK councillor snapshots from 2016, party codes and council-level composition files for 1973–2015 and 2016–2026; the historic composition CSV contains Cambridge entries. These are potential sources for dated membership and composition views and cross-checks, not candidate vote totals or complete individual election histories.

The downloads describe annual snapshots taken shortly after May elections. Preserve that precision and the source's coverage rather than inventing an exact snapshot date or presenting a historical snapshot as current. A series of snapshots does not establish when someone took office, left or changed party. The [changes log](https://opencouncildata.co.uk/changes.php?y=2026) says its dates are update dates, not necessarily event dates; verify actual dates elsewhere before creating membership events. Where the source groups parties into “other”, retain that grouping and do not infer its breakdown. Distinguish majority, plurality and administration/control, especially for councils without an overall majority.

During discovery, inspect the relevant Cambridge rows and columns, compare snapshots with council and election sources, and record discrepancies and date limitations. Map authority, ward and party identifiers only when verified; annual councillor CSVs contain names and party codes but do not by themselves establish person identity across years. Treat potential links to Democracy Club people and parties as reviewed crosswalks, not automatic name-based matches. Use a dated local snapshot and provenance if this source is adopted; separately assess the current website and any paid datasets rather than assuming the free annual files are live.

### Democracy Club compatibility

Design the model to interoperate with [Democracy Club's candidate and result data](https://democracyclub.org.uk/data_apis/data/), without forcing older Cambridge records into its coverage or losing local historical detail. Its [CSV builder](https://candidates.democracyclub.org.uk/data/) produces one row per candidacy, with selectable field groups; the [results view](https://candidates.democracyclub.org.uk/data/?election_date=2018-05-03&field_group=results) illustrates the result columns. Democracy Club describes its coverage as general elections since 2010, local and devolved elections since 2016, and by-elections since May 2017, excluding town and parish councils. It collects candidate vote counts for first-past-the-post elections; turnout and spoiled ballot data depend on availability. Audit actual Cambridge coverage and field completeness rather than assuming every value is present.

Keep local IDs and the richer local model, with optional, explicitly namespaced Democracy Club identifiers and a documented crosswalk:

| Democracy Club CSV field(s) | Cambridge Votes mapping |
| --- | --- |
| `election_id`, `election_date` | External election ID and date on an election. Democracy Club's election grouping can differ by election type; map explicitly rather than assuming it always matches our authority-level grouping. |
| `ballot_paper_id`, `post_label`, `seats_contested` | External ballot ID on a contest; area label and seats available, subject to source verification. A ballot is a contest, not an individual candidacy or a boundary-version identifier. |
| `person_id`, `person_name` | External person ID and recorded candidate name. One person may have multiple candidacies. |
| `party_id`, `party_name` | External party ID and candidacy's recorded party label. Keep historical labels and locally defined parties even when no external ID exists. |
| `votes_cast`, `elected`, `tied_vote_winner`, `rank` | Candidate-level result fields, preserving missing and exceptional outcomes. Do not equate `votes_cast` with ballots cast. |
| `total_electorate`, `turnout_reported`, `turnout_percentage`, `spoilt_ballots`, `results_source` | Source-reported contest figures and result citation. Confirm definitions and units in the selected export before mapping turnout fields to local concepts; do not infer ballot-paper counts from an ambiguous field. |
| `cancelled_poll` | Preserve a cancelled-poll flag or status separately from contested/unopposed outcomes. |

Use Democracy Club identifiers only when obtained from an authoritative record and matched with evidence; do not manufacture them for older elections, people or boundary versions. Preserve ID strings exactly, including party prefixes. Review identity and ballot matches, including same-name people, by-elections, simultaneous contests and boundary changes; leave uncertain mappings unresolved. Keep provenance for each crosswalk and retain source-reported values alongside local corrections, flagging conflicts for review. A Democracy Club ballot ID does not replace a dated geography record.

Plan an import adapter for a pinned, dated CSV snapshot and its selected field groups. Group rows by verified ballot ID, check repeated contest-level values for consistency, and preserve unknown values instead of filling them with zero or false. Document field mappings, types, nulls and any lossy conversions. For a compatible CSV export, flatten one candidacy per row using the verified Democracy Club column names and meanings, repeating contest fields as needed; include external IDs only where verified and document local extensions and omissions. Do not call an export fully compatible until tested against the actual selected format. Preserve the full local JSON model for provenance, historical geography, membership and other data that a flat export cannot represent.

An illustrative contest record might look like this; `null` means not known, not zero:

```json
{
  "id": "city-2018-05-03-trumpington",
  "electionId": "city-2018-05-03",
  "geographyId": "city-ward-trumpington-2004",
  "seatsAvailable": 1,
  "status": "contested",
  "electorate": 9264,
  "ballotPapersCast": null,
  "rejectedBallots": 9,
  "results": [
    { "personId": "katie-thornburrow", "partyId": "labour", "votes": 1302, "elected": true }
  ],
  "sourceIds": ["cambridge-elections-2018"]
}
```

This is a partial example of the **local** record's shape, not Democracy Club JSON, a complete contest or a dataset to publish. No external IDs are shown because they have not been verified. Confirm the meaning of each source field during ingestion; in particular, do not assume that a source's displayed total includes or excludes rejected ballots without checking.

### Calculation and historical safeguards

- Retain source-reported figures and state the denominator and rounding for any calculated percentage. Turnout is ballot papers cast divided by electorate only when both are known and defined consistently; identify whether rejected ballots are included. Never substitute total candidate votes for ballot papers in a multi-seat contest.
- In multi-seat contests, voters may cast multiple votes: candidate shares based on ballot papers can sum to more than 100%. The archive notes that its multiple-vacancy percentage conventions differ across periods. Label measures and do not compare incompatible series as if they were equivalent.
- Show an unopposed election as such, with vote counts and turnout unavailable unless evidenced. Distinguish no recorded data from a genuine zero.
- For a single-seat contested election, majority is winner's votes minus runner-up's votes when both are known. For multiple seats, define any margin against the highest unelected candidate and do not invent one where no comparable loser exists. Preserve ties and source-reported exceptional outcomes.
- Election seats won are not council seats held. Gains and losses require a defensible comparison with the relevant outgoing seat or seats, including changes of party, by-elections and boundary changes; otherwise omit the claim or attribute a source's claim with its definition. Do not infer council control from one round of elections.
- Keep boundary changes visible in area histories and visualisations. Do not calculate ward-level swings across incompatible boundaries or compare city wards directly with differently defined county divisions. Party-level swing requires an explicit definition and comparable contests; omit it when those conditions are not met.
- Cite historical notes, retain ambiguity and distinguish source transcription from later corrections. Avoid automated identity matches unless reviewed.

## Sources, rights and maintenance

1. Inventory source pages and coverage, including ward-level history, recent results, by-elections and source-specific notes. Consult official declarations and other primary records when available; identify discrepancies rather than silently choosing a value.
2. The reference site's footer asserts copyright. Its [introduction](https://www.cambridgeelections.org.uk/intro.htm) credits different compilers for different periods, and its [maps page](https://www.cambridgeelections.org.uk/wardmaps.htm) states that its images are reproduced with Ordnance Survey permission. The repository's MIT licence does **not** establish rights to third-party data, text or images. Before copying, scraping, republishing, distributing downloads or reusing maps, establish applicable permissions and licences; credit sources as required. Check robots and access terms and use polite, low-volume fetching if permitted.
3. Democracy Club's [data reuse terms](https://candidates.democracyclub.org.uk/api/docs/terms/) state CC BY 4.0, with party logos and candidate photos excluded, and request a linked Democracy Club credit on each page using its data. Record the licence, attribution, changes and source snapshot for reused data, and check current terms at import and publication. Do not assume its licence covers independently sourced material or the excluded assets.
4. Open Council Data's [about page](https://opencouncildata.co.uk/aboutthissite.php) says its free CSV downloads are CC0 and its website and contents are CC BY-SA 4.0; it also links separate terms of use. Check the precise source, current licence and applicable terms before reuse, including share-alike obligations for website material and any separate paid dataset terms. Record provenance and credit the source even for CC0 data.
5. If reuse is not authorised or is unclear, use appropriately licensed official or other authorised sources instead; do not treat attribution as permission. Keep a rights and provenance register including licence terms, attribution, verification status and limitations for each source and geographic dataset.
6. Define an update workflow: obtain declarations, transcribe/import, validate, review against the source, publish with an as-of date, and record subsequent corrections with evidence. Provide a maintained contact route for corrections; do not publish personal contact details without consent.

## Proposed technical approach

- Use **Astro with TypeScript** for static generation: server-rendered HTML, file-based routes and optional small interactive islands. This is a starting recommendation, not an installed dependency. Keep the content and validation pipeline independent enough to change the presentation stack later.
- Validate structured records at build time and generate a static page per election, contest and area, plus indexes and a sitemap. Use canonical URLs under `https://www.cambridgevotes.uk/`, meaningful page titles, descriptions and dates; serve normal 404 pages. Keep source links and table data in the generated HTML.
- Keep a documented import pipeline separate from checked-in curated data. Imports should be repeatable, identify their source and report conflicts for review; a build should not fetch live third-party pages or silently overwrite corrections.
- Deploy the built output to a static host supporting the custom domain, HTTPS and predictable redirects. GitHub Pages is a candidate, subject to confirming hosting and DNS access; no backend is needed initially. Test a preview deployment, the custom domain, HTTPS, deep links and 404 behaviour before switching traffic.
- If suitable, licensed, versioned boundary geometry becomes available, add a progressively enhanced map with a non-map area list and accessible results. Otherwise use the list alone. Do not reuse the existing archive's map images on the assumption that its permission extends to this project.

## Implementation plan

Each phase ends in a reviewable, working increment. Dates and coverage are intentionally not fixed until source access and verification are known.

1. **Discovery and rights.** Audit the archive, official sources, relevant Democracy Club CSV exports and Open Council Data's Cambridge snapshots; record election types, dates, source structure, missing fields, licensing and boundary eras. Inventory available boundary datasets and gaps by election era and geography type, and inspect their licences. Inspect actual field groups, semantics and Cambridge coverage and document the proposed crosswalk. Select the most recent complete, authorised city election for the first slice, including any relevant simultaneous by-elections. Agree the initial coverage statement and identify a correction contact. **Done when:** every intended source and asset has documented reuse status, the slice has identifiable contest-level sources, and compatibility decisions and gaps are recorded.
2. **Data pipeline and vertical slice.** Define and validate the schema, source register and a small set of manually reviewed records; implement and test the Democracy Club CSV adapter against a dated sample where available, including reviewed ID mappings. Add other repeatable imports only where useful. Build the home, election, contest and area pages and methodology, with plain HTML tables first. **Done when:** every contest and candidate in the chosen slice is accounted for, displayed figures match their sources, the crosswalk and unresolved matches are documented, and the published coverage and date are explicit.
3. **Expand the city archive.** Import successive election years and by-elections in manageable batches, preserving source notes and boundary versions. Track coverage and unresolved discrepancies. Add navigation and, where justified, accessible charts and downloadable data. **Done when:** each published batch passes validation and source review, with gaps and exclusions visible.
4. **Richer political history.** Review and verify Open Council Data snapshots against other sources for dated composition views; show snapshot date precision and limitations. Acquire and verify membership events before showing event-based histories or inferring intervening states, and evidence any administration/control claims separately. Add candidate/party histories only after identity review. Add comparisons and maps only where the methods, boundaries and rights support them. **Done when:** each feature has documented definitions, dated provenance and a usable non-visual alternative.
5. **Other elections.** Add county divisions and parliamentary contests as separately scoped datasets and interfaces, including historical geography and voting rules. **Done when:** coverage is explicit and no cross-authority or cross-boundary comparison is implied without justification.

## Verification and release checklist

- Validate IDs and references, required fields, dates, non-negative counts, seats and elected statuses; flag duplicates, missing sources and conflicting records. Check turnout, totals and percentages only against their correctly defined denominators; report discrepancies for review rather than silently changing source values. Allow documented exceptions and historical unknowns.
- Add focused tests for multi-seat percentages, unopposed contests, ties, multiple elections on a date, missing values, boundary changes and composition versus seats won. Independently spot-check the first slice and each subsequent batch against declarations or cited archive pages, including every result in the initial slice.
- Test CSV import and any compatible export against a pinned Democracy Club sample: multiple candidacies per ballot, repeated contest fields, missing values, cancelled polls and conflicting rows. Check ID crosswalks, prevent unsupported identity merges and verify mapped values survive a round trip where the formats permit; document fields that cannot round-trip.
- For any Open Council Data import, check a pinned Cambridge sample against the source, preserve snapshot precision and grouped parties, and flag conflicts with other sources. Test that snapshot-only data cannot produce invented event dates or imply a current composition.
- Before publishing a map, check every contest-to-boundary join and election-date applicability against the source and boundary review; test missing or mismatched geometries, multiple winners and accessible non-map alternatives. Verify geometries after conversion and simplification, and include the required source and attribution.
- If MapIt is used, test the generation and area-code crosswalk against reviewed Cambridge examples on both sides of a boundary change, validate retrieved polygons and document unavailable eras, API limits and reuse permission.
- Check generated links and routes, metadata, source citations, download contents and keyboard/screen-reader usability on desktop and mobile. Test without JavaScript, and measure performance with a tool such as Lighthouse while treating accessibility audits as a supplement to manual testing.
- Automate validation, tests and static builds in CI once the project exists. Publish only verified data; document the dataset's coverage, last update and known limitations on the site.
- After deployment, verify the live custom-domain HTTPS site, representative deep links, mobile layout and not-found page. Retain a repeatable rollback to a previous static build.

## Decisions to confirm before implementation

- Which authorised sources can be used and redistributed, and what attribution or restrictions apply?
- Which recent complete city election should be the first verified release, and how far back should the first public release go?
- Who will maintain data and handle corrections, and what public contact route should be used?
- Which static host and DNS setup should serve the domain?
