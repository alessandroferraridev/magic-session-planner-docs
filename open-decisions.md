# Open product decisions

The requested technical foundations are fixed: NestJS for the backend, React Native for the app, separate Git repositories, and geographic discovery of available Magic: The Gathering tables.

The following defaults make planning concrete. They remain assumptions until confirmed and can be revised before implementation.

| Decision | Proposed default | Effect on development |
| --- | --- | --- |
| Who can host | Players and store representatives | Requires venue ownership and clear store affiliation wording |
| Private home visibility | Public area before approval; exact address after approval | Requires separate location fields and protected access |
| Initial formats | Commander first | Keeps filters and forms small; format storage remains extensible |
| Joining policy | Host approval | Requires pending requests and host management screens |
| Host participation | Host occupies one seat | Defines capacity counting and initial membership |
| Platforms | iOS and Android through Expo | Requires two device test and distribution paths |
| Search experience | List first; map later | Delivers useful discovery before map integration |
| Discovery origin | Current foreground location or manually selected town | Requires permission handling and geocoding |
| Notifications | In-app refresh first; push later | Core participation works without notification delivery |
| Authentication | Managed provider | Provider and sign-in methods must be chosen in step 0 |
| Launch geography and language | One pilot area; initial language to confirm | Determines geocoder coverage, content, and testing |
| Infrastructure budget | To be set | Determines hosting, database, geocoder, and build service choices |
| Retention and audience | To be set before pilot | Determines deletion jobs, privacy wording, and distribution requirements |

## Questions sent to the product owner

1. Should players and stores both host, and should private addresses stay hidden until a seat is confirmed?
2. Should the first version focus on Commander or launch with multiple formats?
3. Should players request approval or join immediately while seats remain?

Before implementation, also confirm the launch area, initial languages, authentication preferences, available monthly budget, and whether the host always plays. These choices do not prevent repository setup and planning.

## Recorded decisions

No additional product answers have been recorded yet. Update this section and the affected scope, architecture, API, and roadmap sections together when answers arrive.
