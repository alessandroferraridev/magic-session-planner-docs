# Open product decisions

The requested technical foundations are fixed: NestJS for the backend, React Native for the app, separate Git repositories, and geographic discovery of available Magic: The Gathering tables.

## Confirmed decisions

| Decision | Confirmed choice | Effect on development |
| --- | --- | --- |
| Joining policy | Host approval | Players request a seat; only approval confirms participation and occupies a seat |
| Host participation | Chosen when creating the session; participating is the default | A playing host occupies one seat; a host who only provides the table occupies none and still manages the session |
| Authentication methods | Email magic link, Google, and Apple | All three sign-in methods are in scope |
| Authentication ownership | Custom authentication in the NestJS backend | Own application users, identity associations, sessions, and tokens; integrate directly with Google and Apple for social sign-in |
| Platforms | iOS and Android through Expo | Both platforms are required for development, testing, and distribution |
| Geographic availability | Worldwide session hosting and discovery | Accept valid locations globally; do not restrict use to a launch country or region |
| Interface languages | English and Italian | Localize the app and user-facing authentication emails in en and it; language preference is independent of session location |
| Git workflow | Gitflow using main, dev, feature, fix, release, and hotfix branches | main holds releases; dev integrates changes and feeds staging; use pull requests and independent repository versions |
| Testing | Unit tests, integration tests, and E2E tests for API and app | Implement suites from scaffolding and add feature coverage continuously; required failures block integration or release |

## Working assumptions and open decisions

The remaining defaults support planning only. They have not been confirmed.

| Decision | Proposed default | Effect on development |
| --- | --- | --- |
| Who can host | Players and store representatives | Requires venue ownership and clear store affiliation wording |
| Private home visibility | Public area before approval; exact address after approval | Requires separate location fields and protected access |
| Initial formats | Commander first | Keeps filters and forms small; format storage remains extensible |
| Search experience | List first; map later | Delivers useful discovery before map integration |
| Discovery origin | Current foreground location or manually selected town | Requires permission handling and geocoding |
| Notifications | In-app refresh first; push later | Core participation works without notification delivery |
| Authentication implementation details | Libraries, email delivery, token lifetimes, and account linking rules to be selected | Custom backend ownership is confirmed; implementation choices remain open |
| Changes to host participation | To be decided | Define whether and how the host can change their playing choice after session creation |
| Infrastructure budget | To be set | Determines hosting, database, geocoder, and build service choices |
| Retention and audience | To be set before pilot | Determines deletion jobs, privacy wording, and distribution requirements |
| Test infrastructure details | Confirm library versions, Android and iOS runners, CI budget, and initial coverage thresholds during scaffolding | Test levels are confirmed; execution infrastructure and numeric coverage gates remain to be selected |

## Questions still open

1. Should players and stores both host, and should private addresses stay hidden until a seat is confirmed?
2. Should the first version focus on Commander or launch with multiple formats?
3. Which email delivery service and authentication libraries should be used, and how should accounts be linked across sign-in methods?
4. Can an existing session's host change between playing and only providing the table?

Before implementation, also confirm the available monthly budget and the remaining product defaults above. Global availability and English/Italian interface support are confirmed. Keep the affected scope, architecture, API, and roadmap sections aligned as answers arrive.
