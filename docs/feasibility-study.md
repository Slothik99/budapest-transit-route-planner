# Feasibility Study

## 1. Purpose

This feasibility study assesses whether a passenger-facing Budapest public transport journey planner can be developed and maintained using static timetable data. The first releasable version would plan journeys between stops for a selected departure date and time, support verifiable transfers, and show multiple suitable alternatives when available. The study identifies the required data, development and operational work, principal risks, and checks needed before release.

## 2. Problem Statement

Passengers travelling across Budapest may need to combine several public transport services to reach their destination. Choosing a journey requires more than finding a connection between stops: the services must operate on the selected date, the departure and arrival times must fit, and transfers must be achievable. When several journeys are possible, passengers also need enough information to compare them and choose one that suits their trip. Passengers may also have personal preferences that rule out particular services. A useful journey plan should respect these preferences when presenting and comparing alternatives.

## 3. Proposed Solution

The proposed application is a web-based Budapest public transport journey planner. Passengers select a departure stop, a destination stop, and a departure date and time. They may also exclude services they prefer not to use. Using static timetable data, the application searches the remaining services for journeys with valid connections and presents several comparably good alternatives when available. Each result shows the services to take, departure and arrival times, and any transfers. If no journey meets the selected conditions, the application explains that no result was found.

## 4. Project Scope

### 4.1 In Scope

The first releasable version will:

- plan journeys between two user-selected public transport stops;
- use real static BKK GTFS timetable data;
- allow the user to specify a departure date and time;
- consider only services operating on the selected date;
- respect scheduled arrival and departure times and boarding and alighting restrictions;
- support transfers when the next service departs from the same stop_id, or when the arrival and departure points are connected through pathways.txt;
- include pathway traversal time and an additional transfer margin;
- allow the user to exclude selected routes before journey calculation;
- return multiple meaningfully different and comparably suitable journeys when available;
- present the ordered journey legs, services, stops, departure and arrival times, transfers, and total journey time;
- clearly report when no journey satisfies the selected conditions;
- provide a usable web interface.

### 4.2 Out of Scope

The first releasable version will not include:

- address-to-address journey planning;
- address search or geocoding;
- walking between an address and a stop;
- walking between separate street stops without a GTFS pathway;
- transfers inferred only from geographic proximity, matching stop names or a shared parent_station;
- real-time delays, disruptions or vehicle positions;
- automatic replanning after a delay or disruption;
- fare and ticket-price calculation;
- user accounts, saved journeys or preference profiles;
- a dedicated mobile application.

## 5. Target Users

The primary target users are passengers who plan journeys within the area covered by the imported BKK static timetable data. This includes regular commuters, occasional passengers and visitors who can identify their departure and destination stops.

Users are expected to access the application through a web browser and understand basic public transport concepts such as stops, routes, departure times and transfers. They should not need technical knowledge of GTFS data or the underlying route-planning process.

The first releasable version is particularly relevant to passengers who want to compare several suitable journeys or avoid selected routes according to their personal preferences. Since it does not support addresses or unrestricted walking between stops, it is less suitable for users who do not know which stops to select.

## 6. Data Sources

### 6.1 Manually Created Test Data

The first prototype will use a small, manually created transport network. It will contain enough stops, routes, trips, departure times and transfer possibilities to test direct journeys, journeys with transfers, alternative routes, route exclusions and cases where no valid journey exists.

The dataset will be designed to produce known and verifiable results. It will be clearly identified as fictional and will not be presented as real passenger information.

### 6.2 BKK Static GTFS Data

Later prototypes and the first releasable version will use static timetable data published through the BKK OpenData portal in GTFS format. At the time of this feasibility assessment, the static feed can be downloaded without a paid subscription or API key. Its current licence and attribution requirements must be checked again before publication.

The inspected feed contains the data required to associate transport routes with individual scheduled trips, stopping points, arrival and departure times, and applicable service dates. It also provides station hierarchy and pathway data for some locations.

The inspected feed does not contain a separate transfers.txt file. Transfers will therefore be supported only when consecutive journey legs use the same stop or when the relevant arrival and departure points are connected by the available station pathway data. The application will not infer walking connections between separate stops from their names or geographical proximity.

Static timetable feeds cover a limited period and can change over time. The application will therefore require a repeatable process for downloading, validating and replacing its timetable data. Searches outside the period covered by the loaded feed must be rejected with a clear explanation.

### 6.3 Data Volume and Coverage

The stop-time dataset is substantially larger than the other feed files. Import duration, storage requirements, memory usage and journey-planning performance must therefore be measured with a complete feed before the first release. The feasibility assessment does not assume that all source records can be stored and processed as an unstructured in-memory collection.

Station pathway data includes traversal times and directions, making it potentially suitable for calculating supported station-internal transfers. However, the inspected data does not establish complete pathway coverage for every stop and station. Coverage must be measured during the full-feed feasibility phase, and transfers without a verifiable connection must not be offered.

### 6.4 Future Data Sources

Real-time delay, disruption and vehicle-position data may be investigated in a later evolution phase. Its availability, access conditions, usage limits, reliability and integration effort must be assessed separately before it is added to the planned scope.

Address search and pedestrian-network data would also require separate data sources and technical evaluation. Since address-to-address planning and unrestricted walking are outside the first releasable version, these sources are not dependencies of the current project scope.

## 7. Technical Feasibility

### 7.1 Backend

Java is technically suitable for implementing the timetable importer, route-planning logic and web backend. Its standard libraries support file processing, collections and date-time handling, while established frameworks can provide the web interface and persistence integration. The principal technical uncertainty is not language capability but the performance and correctness of full-feed processing and time-dependent journey calculation. These must be evaluated through incremental prototypes and measurements.

### 7.2 Frontend

A browser-based user interface is technically feasible using standard web technologies. The first releasable version requires forms for selecting the origin and destination stops, departure date and time, and any routes to exclude. Search results must present the ordered journey legs, departure and arrival times, transfers and total journey duration in a form that can be compared easily.

A server-rendered interface integrated with the Java backend is the preferred initial approach because it reduces the number of separate technologies and deployment components. The application structure should nevertheless keep presentation concerns separate from journey-planning logic so that a REST API or a more interactive frontend can be introduced later if justified.

HTML and CSS are sufficient for the basic page structure and presentation. A limited amount of JavaScript may be required for usability features such as searchable stop selection and dynamic route-exclusion controls. Mobile-browser usability and understandable validation and error messages must be considered before release.

### 7.3 Route Planning

The route-planning problem is technically feasible but represents the main algorithmic risk of the project. The transport network is time-dependent: a geographical connection is usable only when the relevant service operates on the selected date and its scheduled times allow the passenger to complete the journey and any required transfers.

Before journey calculation, services belonging to routes excluded by the user will be removed from consideration. The planner must then respect service dates, stop order, scheduled arrival and departure times, permitted boarding and alighting, pathway direction and traversal time, and a defined connection margin.

A transfer will be considered only when the next journey leg departs from the same stop_id or when a traversable chain in pathways.txt connects the arrival and departure points. Connections based solely on similar stop names, geographical proximity or a shared parent station will not be inferred in the first releasable version.

The planner should return several alternatives only when they are both sufficiently suitable and meaningfully different. Returning the first several paths found would not satisfy this requirement. The comparison criteria may include arrival time, total duration, number of transfers and waiting time, while the exact ranking and diversity rules must be established in the requirements specification.

The implementation will first be validated on a manually created network with known results, followed by a coherent subset of real timetable data and finally the complete static feed. Candidate algorithms must be compared using correctness, query time and memory usage before a final selection is made.

### 7.4 Data Storage

The manually created prototype network will be stored in files and loaded into memory. PostgreSQL is planned for the real-data stages to store imported GTFS records, their relationships and feed metadata in a structured and queryable form.

Route calculation will not necessarily operate directly on repeated database queries. The application may load and transform the required timetable data into route-planning structures after import. This separates persistent timetable storage from the representation optimised for journey calculation.

Introducing PostgreSQL will be postponed until the basic route-planning workflow works on the controlled dataset. Its use with the complete feed remains subject to import-time, storage and performance measurements. This staged approach retains the technical and maintainability benefits of a relational database without making it a dependency of the earliest prototype.

### 7.5 Deployment

The early prototypes will run locally and will not require a public hosting environment. This is sufficient for validating the timetable import, route-planning logic and initial user interface.

A later demonstration version can be deployed as a single Java web application with its required database. Packaging the application and database with Docker and Docker Compose may provide a reproducible local and deployment environment, but containerisation is not required for the first prototype.

Public deployment of the first releasable version is technically feasible, but the hosting platform should be selected only after measuring memory usage, storage requirements, timetable import duration and expected request load. A publicly available passenger-facing service would also require monitoring, error handling and a reliable timetable-update process.

Kubernetes and a distributed deployment architecture are not justified for the current project size.

## 8. Operational Feasibility

The early prototypes can be operated locally by the developer and do not require continuous availability or automated timetable updates. Their operational purpose is to validate the route-planning workflow and provide reproducible demonstrations.

A publicly available version will require a repeatable process for obtaining, validating, importing and activating new static GTFS feeds. A newly downloaded feed must not replace the currently active data until its structure, required references and service period have been checked. If an update fails, the application should retain the last known valid feed and report the problem to the maintainer.

The user interface must communicate whether the requested date is covered by the loaded timetable and distinguish between no available journey, invalid search conditions and a system or data error. The source and freshness of the timetable data should also be visible to users.

The application can initially be maintained by one developer. However, a passenger-facing public service would require regular attention to failed imports, application errors, data changes and reports of incorrect journeys. Continuous availability and formal support commitments are therefore not assumed for the prototype or demonstration stages.

Operational feasibility is considered conditional on demonstrating that feed updates can be performed reliably and that the application can retain a valid dataset when an update fails.

## 9. Legal and Data Usage Considerations

The application will use timetable data published through the BKK OpenData portal. At the time of this feasibility assessment, the published datasets are available under the Creative Commons Attribution 4.0 International licence. The licence permits copying, redistribution and adaptation, including commercial use, provided that its attribution requirements are satisfied.

BKK requests the following attribution:

**Data source: BKK Zrt., CC BY 4.0.**

The project README will contain this attribution together with links to the BKK data source and the CC BY 4.0 licence. The deployed web application will also display the attribution in a reasonably visible location, such as its footer or an accessible information page. The feasibility study will identify and reference BKK as the source of the examined data, but the attribution does not need to be repeated in every section.

The applicable licence, requested attribution and access conditions must be checked again before public release because the publisher may update them. If the application modifies or derives data from the original feed, this must be indicated where required by the licence.

The application must not imply that it is an official BKK product or that BKK endorses the application or its calculated journeys. The user interface should identify the timetable source and explain that results depend on the currently loaded static data. Short-notice service changes, disruptions and real-time delays may not be represented.

The first releasable version will not require user accounts, saved locations or persistent personal preferences. Journey-search parameters will not deliberately be retained as personal profiles. If accounts, saved places, analytics or precise user-location data are introduced later, the applicable privacy and data-protection requirements must be reassessed before implementation.

Licences and terms for software dependencies, map content and any later external services must be reviewed separately. Permission to use the BKK timetable data does not automatically grant permission to use third-party map tiles, geocoding services or real-time APIs under the same conditions.

The licence and attribution conclusions in this study are preliminary project-planning findings rather than legal advice.

## 10. Risks and Mitigations

| Risk | Potential impact | Mitigation |
| --- | --- | --- |
| Incorrect handling of dates, times or service calendars | The application may offer journeys that do not operate at the selected time. | Test known journeys on different dates, including weekends, exceptional service dates and journeys after midnight. |
| Invalid or unrealistically short transfers | Passengers may be shown connections they cannot complete. | Allow transfers only at the same stop or through a valid pathway, include pathway traversal time and define an additional connection margin. |
| Incomplete pathway coverage | Valid journeys may be omitted because some transfers cannot be verified. | Measure pathway coverage using the complete feed, document the limitation and reconsider the transfer rules if important connections are excluded. |
| Incorrect boarding or alighting assumptions | The planner may allow passengers to enter or leave a service where this is prohibited. | Respect the GTFS pickup and drop-off restrictions and include them in automated tests. |
| Large timetable data causes excessive import time or memory usage | Full-feed processing may be too slow or may exceed available resources. | Process the data incrementally, create suitable indexes and measure performance first on a subset and then on the complete feed. |
| Alternative journeys are nearly identical or poorly ranked | Multiple results may provide little practical value to passengers. | Define measurable suitability and diversity rules and validate the results using representative journey searches. |
| Route exclusions remove every possible journey | The user may receive no result without understanding why. | Clearly report that no journey satisfies the selected exclusions and allow the search to be repeated with fewer restrictions. |
| A timetable update is missing, invalid or incompatible | Journey results may become outdated or the application may stop working. | Validate every new feed before activation and retain the last known valid dataset if an update fails. |
| The scope becomes too large for one developer | Important functionality may remain unfinished or insufficiently tested. | Preserve the staged milestones and move address search, unrestricted walking and real-time information to separately assessed evolution phases. |
| Limited frontend experience increases development time | The web interface may delay or weaken the release. | Begin with a server-rendered interface and introduce additional JavaScript only where it provides a clear usability benefit. |
| Licence or attribution requirements are overlooked | Public release may need to be delayed or changed. | Record the data source and licence, display the requested attribution and recheck the applicable terms before release. |
| Route results are technically correct but difficult to understand | Users may select the wrong journey or distrust the application. | Review wireframes early and test complete search tasks with people who did not design the interface. |



## 11. Required Resources

## 12. Estimated Development Phases

## 13. Success Criteria

## 14. Conclusion
