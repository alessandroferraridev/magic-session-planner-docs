# Formats and session configuration

Accepted on 27 November 2026: support the full range of Magic formats from the first release, rather than a Commander-first or five-format subset. This document owns format coverage, catalogue semantics, host configuration and player grouping. API/storage details belong to [Data and API](./data-and-api.md); verification belongs to [Testing strategy](./testing-strategy.md).

## Meaning of complete support

Support means users can describe, discover, join and organize a session with the appropriate card pool, preparation, game structure, materials and declared rules. It does not mean implementing Magic gameplay, checking decklists/card legality, maintaining a card-price service, operating digital matches or running tournament pairings. Those capabilities remain outside the session planner.

Cover every label in the current [Wizards formats hub](https://magic.wizards.com/en/formats), the additional multiplayer variants in the [Comprehensive Rules](https://magic.wizards.com/en/rules), community and historical formats, and unlisted/new house formats through a structured Custom entry. A finite list cannot enumerate every playgroup's invented rules: Custom is a first-release capability, not a deferred feature or a free-text database format code.

All real sessions remain physical gatherings discovered from the current device position. Digital formats are selectable for an in-person device-play gathering with explicit client/device/network requirements; they do not enable remote matchmaking, expose online handles or replace approval-gated venue privacy. Card/client/product purchases are participants' arrangements, not a new app payment flow or infrastructure expense.

The pilot admits 1-16 human player seats per session, including a playing host. One-seat sessions exist for explicitly solo/custom/cooperative practice profiles; ordinary multiplayer profiles still need a structurally valid layout. Larger events and automated tournaments remain outside pilot capacity. A bot, Horde deck, avatar or non-playing host is never a booked player seat.

## Independent concepts

| Concept | Meaning and examples |
| --- | --- |
| Format definition | Stable identity/card-pool family: Standard, Commander, Pauper, Duel Commander |
| Rules profile | Immutable version of authority, card-pool reference, bans/points reference, deck guidance and relevant exceptions |
| Preparation | Bring a deck, booster draft, pick-two, sealed, Cube, Jumpstart or shared supplied pool |
| Game structure | Duel rotation, one multiplayer pod, several pods, teams, asymmetric Archenemy or cooperative/solo |
| Optional modifiers | Planechase, Vanguard or an explicitly compatible extra rules module |
| Play medium | Physical cards or devices at the same venue; Arena and Magic Online remain distinct clients |
| Intent | Learning, casual, tuned or competitive; cEDH maps to Commander with competitive intent, not a new ban list |
| Session capacity | Total human seats across games, not the number in one match. Eight Modern attendees can play four duels |

A source's recommended player count is guidance, not a universal session cap. An advertised 60-card, 100-card or 40-card deck size is card information, never seat capacity. The app can validate structural configuration and required disclosures; it must not claim a deck is legal without examining that deck.

## Constructed catalogue

Canonical codes below are stable and independent of translated display labels. Deck guidance is a concise reference; governing rules, exceptions and banned lists apply. Standard/Pioneer/Modern/Legacy/Vintage/Pauper have different legal pools even when their usual deck minimum is 60.

| Code | Entry | Deck/pool guidance and host disclosures |
| --- | --- | --- |
| standard | [Standard](https://magic.wizards.com/en/formats/standard) | 60+ main deck, rotating pool; record current or explicitly frozen rules date and BO1/BO3 |
| pioneer | [Pioneer](https://magic.wizards.com/en/formats/pioneer) | 60+, nonrotating pool beginning with Return to Ravnica; platform and legal-pool reference matter |
| modern | [Modern](https://magic.wizards.com/en/formats/modern) | 60+, its own nonrotating pool and bans; do not infer legality from Pioneer |
| legacy | [Legacy](https://magic.wizards.com/en/formats/legacy) | 60+, historical card pool with format-specific bans |
| vintage | [Vintage](https://magic.wizards.com/en/formats/vintage) | 60+, banned and restricted lists; restricted cards differ from banned cards |
| pauper | [Pauper](https://magic.wizards.com/en/formats/pauper) | 60+, qualifying common printings and its own bans; Arena Historic Pauper is not this same legal pool |
| commander | [Commander / EDH](https://magic.wizards.com/en/formats/commander) | 100 total, commander/color identity and singleton exceptions; multiplayer intent and declared power expectation |
| standard-brawl | [Standard Brawl](https://wpn.wizards.com/en/news/brawl-quick-guide-magics-new-format) | 60 total and Standard pool; select paper/Arena profile explicitly, not the 100-card Arena Brawl entry |
| oathbreaker | [Oathbreaker](https://oathbreakermtg.org/rules/) | 60 total including planeswalker and signature spell; singleton/color identity and its own list, 20 starting life |
| freeform | [Freeform / kitchen-table](https://magic.wizards.com/en/formats/freeform) | Declared pool, deck rules and multiplayer agreement; never label it as sanctioned deck validation |

### Commander-specific expectations

The [Commander page](https://magic.wizards.com/en/formats/commander) describes optional brackets and Game Changers. Store the published bracket-system revision and selected bracket when used, plus a short power/style note and proxy/playtest agreement. cEDH search resolves to Commander plus competitive intent and the appropriate published bracket; it must not silently resolve to Duel Commander or MTGO Commander 1v1. A deck strength number alone is not an objective compatibility guarantee.

For Commander/Duel/PDH/Highlander families, keep separate fields for commander eligibility/count, singleton rule, color-identity policy, point budget, starting-life guidance and banned-list reference where applicable. A signature spell belongs to Oathbreaker, not ordinary Commander. Limited Commander can relax singleton/deck-size rules under its product profile; never inherit every constructed Commander constraint automatically.

## Limited and prepared-pool catalogue

| Code | Entry | Required planning information |
| --- | --- | --- |
| booster-draft | [Booster Draft](https://magic.wizards.com/en/formats/booster-draft) | Set/product, packs per person, draft pod size, land provider, build time and ownership of drafted cards; usually 40+ decks |
| pick-two-draft | [Pick-Two Draft](https://magic.wizards.com/en/formats/pick-two-draft) | Explicit two-card picks and pod size; recommended four-person recipe, product-specific packs and build time |
| sealed-deck | [Sealed Deck](https://magic.wizards.com/en/formats/sealed-deck) | Product and pool per player, normally six boosters, basic lands and build time; usually 40+ decks |
| commander-draft | [Commander Limited: draft](https://magic.wizards.com/en/formats/commander-limited) | Product-specific draft/pick rules and commander selection; typical 60+ deck and repeat-copy allowance |
| commander-box-league | [Commander Box League](https://magic.wizards.com/en/formats/commander-limited) | Box/pool reference and modified 60-card/color rules; plan one meeting without automatic league standings |
| commander-sealed | Commander sealed / product-specific limited | Require the exact product/host rules profile; do not assume normal sealed means a legal 100-card Commander pool |
| conspiracy-draft | [Conspiracy](https://magic.wizards.com/en/formats/conspiracy) | Draft-affecting cards, conspiracy use and multiplayer pods; declare the product and special setup |
| team-booster-draft | [Team Booster Draft](https://magic.wizards.com/en/formats/team-booster-draft) | Two three-person teams per draft pod, alternating seating and individual matches; total human seats and communication rule disclosure |
| team-sealed | [Team Sealed Deck](https://magic.wizards.com/en/formats/team-sealed-deck) | Three-player team pools; ordinary reference uses twelve boosters per team for three decks, with product overrides explicit |
| cube-draft | Cube draft | Owner-supplied Cube identity/revision, pack emulation, pick method, power/pool summary and return-card policy |
| cube-sealed | Cube sealed | Owner-supplied Cube identity/revision, pool size per player, land supply and return-card policy |
| winston-draft | [Winston Draft](https://magic.wizards.com/en/news/feature/winston-draft-2005-03-25) | Small-group pool and hidden-pile method; explicit host setup rather than eight-person booster assumptions |
| winchester-draft | Winchester draft | Face-up pile method; host rules reference and supplied pool are required |
| grid-draft | Grid draft | Grid/row selection method, player count and pool are host-declared |
| rochester-draft | Rochester draft | Face-up pick/seating order and pool; no automatic draft execution |
| solomon-draft | Solomon draft | Two-player split/choose preparation; require pool and build-time details |
| rotisserie-draft | Rotisserie draft | Public complete pool, pick order and allocated time; no online draft-room service |
| chaos-draft | Chaos draft | Mixed product manifest and pick rules rather than a single set assumption |
| jumpstart | [Jumpstart](https://magic.wizards.com/en/product-guide) | Compatible half-deck product/pool, sealed versus reusable packs and who supplies them |
| pack-wars | Mini-Master / Pack Wars | Booster/land/mulligan house recipe and ownership agreement, not an inferred 40-card sealed default |

Cube is a supplied-card environment, not a universal power level or singleton promise. Sealed/draft packs differ by product: require an explicit manifest and do not hardcode every pack to fifteen cards. Store a bounded materials manifest with productLabel, itemUnit (booster/box/half-deck/cards), quantity and allocation (per-player/per-team/per-session), not one ambiguous packs field. A box must not be counted as a booster. The older [Wizards casual-format descriptions](https://magic.wizards.com/en/news/feature/casual-formats-2008-08-11) document Cube, Solomon, Mini-Master and other variants; distinguish historical examples from a current product rule.

## Multiplayer structures and modifiers

These entries can be discovered by their familiar name and resolve to a complete compatible recipe rather than losing their underlying pool/preparation.

| Code | Entry | Structure/materials to declare |
| --- | --- | --- |
| two-headed-giant | [Two-Headed Giant / 2HG](https://magic.wizards.com/en/formats/two-headed-giant) | Two-person teams and shared life; choose Constructed versus Limited pool and single game versus several rotating teams |
| planechase | [Planechase](https://magic.wizards.com/en/formats/planechase) | Base format plus planar deck arrangement and planar die provider |
| archenemy | [Archenemy](https://magic.wizards.com/en/formats/archenemy) | One archenemy versus a coalition; choose who supplies the scheme deck and assign the unique role |
| archenemy-commander | [Archenemy Commander](https://magic.wizards.com/en/formats/archenemy-commander) | Commander decks, archenemy/coalition roles and its own scheme/shared-life profile; not just ordinary Archenemy with renamed cards |
| supervillain-rumble | [Supervillain Rumble](https://magic.wizards.com/en/formats/archenemy-commander) | Multiplayer Commander recipe with each player supplying a scheme deck |
| vanguard | [Vanguard](https://magic.wizards.com/en/rules) | Extra Vanguard card/avatar and its hand/life modifiers; base format remains explicit |
| emperor | [Emperor](https://magic.wizards.com/en/news/feature/casual-formats-2008-08-11) | Usually two teams of three, emperor/general roles and seating/influence arrangement |
| grand-melee | [Grand Melee](https://magic.wizards.com/en/rules) | Large multiplayer structure; range/turn-marker rules declared; pilot still caps the human session at sixteen |
| star | Star / pentagram | Five-role/color/seating house profile, with explicit victory conditions |
| cooperative-custom | Cooperative / Horde / product adventure | Human party size, supplied adversary/product, pilot/facilitator role and host-defined win/setup rules; automated adversaries are not user seats |
| shared-library-custom | Battle Box / shared library / Type 4 / Forgetful Fish | Supplied pool owner/revision and draw/land/mana rules; different names do not imply identical rules |

The [Comprehensive Rules](https://magic.wizards.com/en/rules) are the authority for their named multiplayer options; community recipes require their own references. This app records configuration and seating; it does not execute turn order, compute influence or keep life totals. Unknown modifier combinations require explicit house-rule disclosure rather than being advertised as an official combination.

## Community and historical catalogue

| Code | Entry | Required distinction and reference |
| --- | --- | --- |
| duel-commander | [Duel Commander](https://www.duelcommander.org/) | Community 1v1 Commander, 100-card singleton and 20 life; its own committee/list, distinct from MTGO 1v1 |
| pauper-commander | [Pauper Commander / PDH](https://pdhhomebase.com/rules/) | Common-card deck and uncommon eligible commander profile, 30 life; commander need not be legendary |
| canadian-highlander | [Canadian Highlander](https://canadianhighlander.ca/about/) | 100+ singleton, no commander/sideboard, ten-point budget and authority-specific list |
| premodern | [Premodern](https://www.premodernmagic.com/) | Fourth Edition through Scourge pool and its own bans; no blanket all-old-frame assumption |
| middle-school | [Middle School](https://www.eternalcentral.com/middleschoolrules/) | Named historical community pool and rule exceptions; do not alias to Premodern |
| old-school | Old School / 93-94 | Region/organizer rules authority, legal sets, reprint policy, restrictions and date required; no universal Swedish/EC/Atlantic rule assumed |
| australian-highlander | Australian / pointed Highlander | Require authority/rules version, deck/sideboard guidance and exact point budget; never inherit Canadian 100-card/ten-point rules |
| european-highlander | European / regional Highlander | Require governing reference and singleton/pool rules; a commander is not implied by Highlander |
| peasant | Peasant | Authority-specific common/uncommon allowance including sideboard policy; not an alias of Pauper |
| artisan | Artisan | Explicit platform/pool and common/uncommon restriction; distinguish an Arena event from a house paper pool |
| singleton | Singleton variants | Required deck size/pool/duplicate exceptions, rather than treating every singleton deck as Commander |
| historical-constructed | Historical Standard, Extended, Block Constructed, former queues | Original format name, legal-pool cutoff, rule/list snapshot and status; archived rules do not claim current sanction/client availability |
| custom | Other named or invented formats | Moderated label plus complete host declaration; includes unlisted regional, Tiny Leaders, budget, Judge's Tower and future community variants |

Reviewed summary references above were checked on 27 November 2026. Regional Highlander/Old School/Peasant templates intentionally require a host-declared authority instead of invented universal rules; their maintainer details must be verified before a preset is labelled reviewed. Preserve source provenance and distinguish community authority from Wizards recognition. The Duel Commander reference currently used by its committee is duelcommander.org; do not select a rules authority solely because another site has a similar name.

## Digital formats at a physical gathering

| Code | Entry | Planning constraints |
| --- | --- | --- |
| alchemy | [Alchemy](https://magic.wizards.com/en/mtgarena/alchemy) | Arena client/account and declared current rules reference; digital cards are not silently presented as paper-compatible |
| historic | [Historic](https://magic.wizards.com/en/formats/historic) | Arena-specific pool and current list/client behavior |
| timeless | [Timeless](https://magic.wizards.com/en/formats/timeless) | Arena pool with its restricted-list policy, not Legacy or Vintage |
| arena-brawl | [Brawl / formerly Historic Brawl](https://magic.wizards.com/en/formats/brawl) | Arena's 100-card, 25-life profile; distinct from Standard Brawl |
| arena-competitive-brawl | [Competitive Brawl](https://magic.wizards.com/en/news/mtg-arena/state-of-the-formats-2026) | Separate digital rules/list profile; unlike cEDH intent, it must not be collapsed into the other Brawl list |
| mtgo-commander-1v1 | [Commander 1v1](https://magic.wizards.com/en/formats/commander-1v1) | MTGO-specific 1v1 entry and current list; never alias to the Duel Commander committee's rules |
| momir-basic | [Momir Basic](https://magic.wizards.com/en/formats/momir-basic) | MTGO/avatar/client requirements; no physical client emulation |
| digital-custom | Other client/community/event formats | Exact client, current event/direct-challenge availability and supplied rules reference; covers Penny Dreadful and temporary Arena variants without claiming every queue is permanently available |

Standard, Pioneer and suitable Limited recipes can also select a supported device-play profile; a matching name does not prove identical card availability or BO1/BO3 lists. Use the source/client profile rather than copying an entire paper card pool into an Arena promise.

The [September 2026 Arena update](https://magic.wizards.com/en/news/mtg-arena/state-of-the-formats-2026) identifies Explorer as the former Pioneer name, documents Competitive Brawl, and announces Alchemy retirement for 2 February 2027. Record these dated facts as catalogue lifecycle/alias metadata, not timeless rules. Review the announced change before making it effective; do not promise an unavailable queue. A historic reconstruction remains possible under an explicitly declared custom profile.

## Official hub coverage

Every label currently shown by Wizards has a defined organizing path. This is a coverage checklist, not a claim that all labels describe independent deck formats.

| Hub label | Canonical organizing entry |
| --- | --- |
| Commander | commander |
| Booster Draft | booster-draft |
| Pick-Two Draft | pick-two-draft |
| Standard | standard |
| Pioneer | pioneer with selected paper/Arena profile |
| Modern | modern |
| Sealed Deck | sealed-deck |
| Alchemy | alchemy with lifecycle date |
| Historic | historic |
| Timeless | timeless |
| Commander Limited | commander-draft / commander-box-league / explicitly declared commander-sealed |
| Pauper | pauper |
| Brawl | arena-brawl or explicitly selected standard-brawl |
| Conspiracy | conspiracy-draft |
| Two-Headed Giant | two-headed-giant recipe with base pool/preparation |
| Legacy | legacy |
| Vintage | vintage |
| Planechase | planechase with base format |
| Archenemy | archenemy with base pool |
| Archenemy Commander | archenemy-commander |
| Oathbreaker | oathbreaker |
| Team Booster Draft | team-booster-draft |
| Team Sealed Deck | team-sealed |
| Freeform | freeform |
| Momir Basic | momir-basic |
| Commander 1v1 | mtgo-commander-1v1 |

## Session configuration schema

Each selectable entry has an immutable FormatDefinitionRevision and one or more compatible RulesProfiles. A recipe can reference a base format plus preparation/modifiers instead of duplicating its legal-pool definition. The host chooses a recipe, and the server validates its typed configuration; the client cannot invent catalogue IDs, rule-schema code or compatibility flags.

| Stored concept | Required content |
| --- | --- |
| Definition identity | Stable code/UUID, en/it labels, aliases and family tags; definition kind is base, preparation recipe, modifier recipe, community, historical or custom |
| Rules profile | Stable profile code/revision, authority label/reference, reviewedAt, effective dates and review status; guidance for deck/sideboard/commander/points/life and compatible media/preparation/layouts |
| Field schema | schemaVersion, allowed fields/types/enums, conditional requirements and bounds. Declarative validation only; no executable scripts or remotely loaded schema code |
| Session selection | formatCode, formatRevision, profileCode, schemaVersion, medium, playIntent, gamePlan, bounded settings/logistics and an immutable sanitized rules/configuration snapshot |
| Recipe dependencies | Pinned base-format/profile and modifier revisions. Maximum four modifiers; reject duplicate/cyclic dependencies and silently conflicting requirements |
| Host declaration | Explicit agreement fields for proxies, borrowed decks, current/frozen rules and deviations; required custom label/pool/deck/layout notes for a host-declared profile |
| Provenance | Source URL/authority/version or a host reference label; checked date and review status are separate from claims of sanction or client availability |

Profile guidance fields may be unknown until an authority is verified; store null and require host declaration rather than inventing a deck size, starting life or points budget. The regional/community templates remain usable as Host-declared rules with the required disclosures. A profile is labelled Reviewed summary only after its source has been checked. This is a data-quality distinction, not another open product choice.

`medium` is `paper` or `device-play`; device play additionally requires a client (`arena`, `mtgo` or a named host-declared client) and device/network provision. `gamePlan.mode` is `single-game`, `multiple-pods`, `duel-rotation` or `team-match`; a team-match profile declares the number/structure of individual games rather than treating three 1v1s as one six-player game; `structure` describes duel, free-for-all, teams, asymmetric or cooperative/solo. Restrict compatible fields by profile, never assume every base format accepts every option. A new or incompatible house combination uses an explicitly labelled house profile with required notes and valid structural bounds.

`playIntent` is learning/casual/tuned/competitive. Commander bracket metadata is conditional on a compatible Commander profile and bracket revision. BO1/BO3, proxies and power expectations are independent of deck format. Physical equipment belongs in logistics, not the legal-pool definition. Do not collect digital passwords, payment credentials, online handles or Wi-Fi credentials; describe network availability and arrange any password in person.

All custom labels, house-rule notes, product descriptions and equipment notes pass the existing plain-text moderation rules. Curated rules links are operator-reviewed catalogue data. User-authored reference text is not an active hyperlink; no arbitrary URL fetch, deck import or SSRF-capable lookup is introduced.

## Required host fields and player experience

| Chosen capability | Required fields | Experience for players |
| --- | --- | --- |
| Any format | Recipe/profile revision, medium, intent, layout, human capacity, play language, deck/material provision and deviations | Searchable name/family and a readable rules/preparation summary before requesting a seat |
| Bring a deck | Current/frozen card-pool reference, proxy/borrowed-deck agreement, any power expectation | Know which deck to bring; no inference from a generic casual tag |
| Commander / signature / pointed profile | Compatible commander/signature/points rule reference and any selected bracket | Distinguish EDH, cEDH intent, Duel, PDH, Oathbreaker and regional Highlander |
| Booster/limited | Product manifest, quantity and unit, supplier, basic-land supplier, preparation/build time and pool ownership | Know whether to bring boosters and whether cards are kept, returned or redistributed by prior agreement |
| Cube/shared library | Owner/reference label/revision, pool description, chosen pick/preparation method and return policy | Know the environment without uploading a card list or assuming a powered Cube |
| Teams | Team size, planned team slots/game groupings and assignment policy | One account still requests one player seat; team preference never reserves another player's seat |
| Asymmetric roles | Planned roles and scheme/adversary/equipment provider | Role readiness is visible; an adversary deck is not an extra booked account |
| Historical/custom | Name, authority or house rules, explicit pool cutoff/reference and relevant deck/layout constraints | Clearly labelled Historical or House rules; not presented as current official legality |
| Device play | Exact client, event/direct-challenge availability, own/provided devices, network/power availability | Know this is a physical gathering using clients, not paper-compatible digital cards or a remote invite |

Use a progressive form: searchable recipe first, then show only relevant conditional controls. Remember neither an old private address nor a prior rules revision as an undisclosed default when duplicating a session. Provide en/it labels, explanations and validation; preserve entered product/Cube/rules names as user content. Unknown required setup blocks publication with a field error, not a generic Invalid format message.

The public discovery card shows a compact format/variant/medium/intent label, total available human seats and city only. Detail shows the complete non-location rules/material summary before joining; precise location still uses the protected endpoint after approval. Never put a venue name/address into a format label or rules summary. Profile/rules changes are material edits: preserve memberships, notify recipients, clear obsolete role/format projections and let players withdraw until end under the normal policy.

## Seats, teams, pods and safety

- `capacity` counts 1-16 human players for the whole gathering. The host's default playing membership occupies exactly one seat regardless of team, commander count or multiple decks. A host who only supplies space is not assigned a player role.
- Validate the advertised layout against planned capacity, not current confirmed attendance. A new session can be published before enough players are approved. A single duel plans two humans; eight-person duel rotation may contain several duels. Odd attendance can rotate, without tournament byes/standings generated by the app.
- A single 2HG game plans four humans in two two-person teams. Several rotating 2HG teams can use other compatible even capacities. A Team Draft pod plans six humans; two such pods plan twelve. A standard Emperor table plans six. Several pods validate the sum of planned slots, not a global Commander equals four assumption.
- Grand Melee surfaces its source's recommended scale and declared layout within the sixteen-seat pilot limit; do not turn a recommended player count into an invented universal minimum. Solo/one-human capacity is allowed only for a declared solo/cooperative/custom layout; ordinary duel/multiplayer recipes reject it. Products/variants larger than the pilot limit can be described but cannot publish an oversized session.
- Requests optionally indicate an existing planned team/role preference. Every approval still allocates only one seat under the session lock; no group booking or teammate auto-approval exists. Hosts assign approved participants to planned groups/teams/roles, either during approval or later. Only the host can change assignments, with normal version/conflict checks and recipient updates.
- A playing host defaults to the first compatible team/role slot and can change it. An assigned unique role such as archenemy has at most one approved owner per game. Non-playing hosts cannot be assigned. No assignment may exceed planned team/group capacity or place a user in overlapping simultaneous game slots.
- Readiness is informational: waiting for players, needs assignments or ready per game group. It does not auto-start a match or block withdrawal. An incomplete pod/team is not a new seat-reservation state, and full session capacity does not guarantee every role has been filled.
- Withdrawal, safety blocking, deletion or host opt-out removes the affected assignment and can make a group unready; these operations must succeed even when team sizes become uneven or a unique role becomes vacant. Joining still stops at session start; a missing teammate does not authorize late joins automatically.
- Host changes to recipe/layout/capacity run atomically with assignments and version changes. Preserve approved memberships, reject capacity below confirmed count, remove only incompatible assignments and mark needs assignments. Never silently evict a player to satisfy a new layout. Exclude pending/blocked profiles from public roster/team views under existing authorization rules.

## Custom and future formats

`custom` and host-declared templates are available at launch. Require a moderated 2-80 character label, base/pool description, deck/duplicate/commander/points rules where relevant, human layout, source/authority label or explicit house agreement, logistics and a bounded summary of deviations. Unknown metadata never becomes a guessed Standard/Commander profile. Do not allow free text to create global catalogue codes or override validation, location privacy, account scope or pilot limits.

Aliases normalize case/Unicode for search, but ambiguous names prompt a real choice: Brawl asks Standard/Arena/Competitive; Highlander asks the authority; Commander 1v1 asks Duel/MTGO/house rules. EDH resolves to Commander. cEDH suggests competitive Commander metadata. Historic Brawl resolves to Arena Brawl; Explorer is a dated former Arena Pioneer name, not a second current card pool. A user may deliberately record an older reconstruction under a historical/custom profile with its original date.

A host can submit a format-catalogue suggestion with name, plain-text source/authority reference and rationale. States are pending/reviewing/accepted/rejected/expired; unresolved suggestions expire after 90 days from submission and operators review them during weekly catalogue upkeep. Suggestions are private to the operator, moderated and bounded; submitting one is not required to publish a valid custom session. Operator review creates a versioned catalogue change, tests alias collisions and translates labels before promotion. Every invented community variant is therefore representable without waiting for a mobile release or pretending the seed list is exhaustive.

## Catalogue lifecycle and rules changes

Maintain small version-controlled data definitions and generated client snapshots, not a paid format service or a live card database. The API owns published revisions and declarative validation. Expose minimum supported schema/capabilities: older clients can read a safe label/history but must update before editing fields they cannot represent; never silently drop unknown settings. New named house variants remain representable through the existing Custom schema without inventing executable field types. Bundled/cached metadata supports picker browsing; explicit picker/detail use can refresh by ETag/revision. Never refetch a catalogue or database automatically for each push.

Each revision records authority, source references, checkedAt and review status (`reviewed-summary`, `host-declared`, `historical`, `needs-review`). Separately record lifecycle (`active`, `historical`, `retired`) and medium/client availability. Missing source access must be visible; it is not verification. Review current sources weekly during pilot and before a known rotation/announcement takes effect. Recheck unavailable community references before marking a preset reviewed. Record scheduled availability reviews, such as the announced Alchemy change, without asserting every future queue will remain open.

Publication pins the selected definition/profile/dependency revisions and a sanitized configuration snapshot. Current means the latest reviewed reference when the host confirms the selection, always displayed with its checked/as-of date; it is not a guarantee that a future event will use an unchanged or automatically updated list. Future catalogue changes do not alter an existing session. A host explicitly upgrades or changes it through a versioned material edit; original snapshots remain readable for existing history. Retired profiles cannot be silently selected for a new current-client event; a historical/house reconstruction must be labelled and supported by the declared medium. A source outage can leave a readable cached reference with its stale/review label; never substitute another format.

Keep non-personal catalogue revisions for historical interpretation; custom host text follows session/history retention and deletion. Store no location inside format snapshots. Budget, safety and disclosure constraints apply equally to obscure variants. All-format coverage is required at first release through reviewed presets plus host-declared/custom configuration; optional later features are gameplay/deck verification and tournament automation, not the ability to name an unsupported format.

## Example session fragments

These are valid illustrative JSON fragments, not implemented endpoints or claims that a catalogue revision is deployed. Revision labels are application catalogue versions, not Wizards rules dates. Other required session fields, including time and protected venue selection, are omitted here.

Commander pod:

```json
{
  "capacity": 4,
  "hostParticipates": true,
  "formatSelection": {
    "formatCode": "commander",
    "formatRevision": "2026-11-27.1",
    "profileCode": "commander-multiplayer",
    "schemaVersion": 1,
    "medium": "paper",
    "playIntent": "casual",
    "gamePlan": { "mode": "single-game", "structure": "free-for-all", "playersPerGame": 4 },
    "settings": { "commander": { "bracketRevision": "2026-11-27.1", "targetBracket": 2 } },
    "logistics": { "deckSupply": "bring-own", "proxyPolicy": "by-agreement" }
  }
}
```

Team Sealed gathering:

```json
{
  "capacity": 6,
  "hostParticipates": false,
  "formatSelection": {
    "formatCode": "team-sealed",
    "formatRevision": "2026-11-27.1",
    "profileCode": "three-player-team-sealed",
    "schemaVersion": 1,
    "medium": "paper",
    "playIntent": "casual",
    "gamePlan": { "mode": "team-match", "structure": "teams", "teamSize": 3, "teamCount": 2, "matchesPerTeamPairing": 3, "matchStructure": "duel" },
    "settings": {
      "limited": { "materials": [{ "productLabel": "Declared Play Booster product", "itemUnit": "booster", "quantity": 12, "allocation": "per-team" }], "buildMinutes": 45 }
    },
    "logistics": { "productSupply": "host-provided", "basicLands": "host-provided", "poolOwnership": "keep" }
  }
}
```

Device-play Brawl gathering:

```json
{
  "capacity": 4,
  "hostParticipates": true,
  "formatSelection": {
    "formatCode": "arena-brawl",
    "formatRevision": "2026-11-27.1",
    "profileCode": "arena-brawl-100",
    "schemaVersion": 1,
    "medium": "device-play",
    "playIntent": "casual",
    "gamePlan": { "mode": "duel-rotation", "structure": "duel", "playersPerGame": 2 },
    "settings": { "digital": { "client": "arena", "availability": "host-confirmed" } },
    "logistics": { "devices": "bring-own", "networkAvailable": true, "powerAvailable": true }
  }
}
```

## Format acceptance criteria

1. Every one of the 26 current official hub labels maps to a canonical organizing entry, with no duplicate false identity for cEDH/EDH or aliases.
2. Each catalogue entry has typed media/layout/setup requirements, en/it labels, provenance/review state and an immutable revision; unknown authority details require host declaration, not guessed values.
3. Constructed, limited, team/asymmetric, shared-pool, historical, community and device-play examples can publish/discover/request/approve with valid setup and the same privacy/safety guarantees.
4. Any unlisted format can be declared under Custom, found through the Custom/family/name filters and suggested privately for the catalogue, without enabling arbitrary executable schemas or user-created global IDs.
5. Team/role assignment never bypasses host approval, consumes an extra hidden seat or prevents an immediate safety withdrawal. No format allows late joining by changing only a role.
6. Source/list updates, retired clients, current/frozen rules and custom overrides are visible and cannot silently rewrite existing sessions or advertise unsupported current availability.
7. UI/metadata refresh, query indexes and compact push projections stay within the zero-budget/no-polling design. No deck legality, price/ban-list completeness or sanctioned tournament claim is inferred from hosting support.
8. The [test matrix](./testing-strategy.md#format-coverage-and-configuration-tests) covers the entire catalogue and each conditional field/layout branch, including the JSON examples above.
