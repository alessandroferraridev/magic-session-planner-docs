# Open product decisions

The requested technical foundations are fixed: NestJS for the backend, React Native for the app, separate Git repositories, and geographic discovery of available Magic: The Gathering tables.

## Confirmed decisions

| Decision | Confirmed choice | Effect on development |
| --- | --- | --- |
| Joining policy | Host approval | Players request a seat; only approval confirms participation and occupies a seat |
| Host participation | Chosen when creating the session; participating is the default | A playing host occupies one seat; a host who only provides the table occupies none and still manages the session |
| Authentication | Email magic link, Google, and Apple | All three sign-in methods are in scope; the authentication provider is still to be chosen |
| Platforms | iOS and Android through Expo | Both platforms are required for development, testing, and distribution |

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
| Authentication provider | Managed provider proposed; vendor to be chosen | Must support email magic links, Google, and Apple; callback and account linking policies remain open |
| Changes to host participation | To be decided | Define whether and how the host can change their playing choice after session creation |
| Launch geography and language | One pilot area; initial language to confirm | Determines geocoder coverage, content, and testing |
| Infrastructure budget | To be set | Determines hosting, database, geocoder, and build service choices |
| Retention and audience | To be set before pilot | Determines deletion jobs, privacy wording, and distribution requirements |

## Questions still open

1. Should players and stores both host, and should private addresses stay hidden until a seat is confirmed?
2. Should the first version focus on Commander or launch with multiple formats?
3. Which authentication provider should supply the three confirmed sign-in methods, and how should accounts be linked across methods?
4. Can an existing session's host change between playing and only providing the table?

Before implementation, also confirm the launch area, initial languages, available monthly budget, and the remaining product defaults above. Keep the affected scope, architecture, API, and roadmap sections aligned as answers arrive.
