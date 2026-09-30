# Feasibility Study — Budapest Public Transport Journey Planner

**Status:** Draft  
**Last reviewed:** 30 September 2026

## 1. Purpose

This document evaluates whether a Java-based public transport journey planner for Budapest can be developed successfully by one developer. It examines the proposed scope, available data, technical and operational feasibility, legal considerations, risks, required resources, development phases, and measurable success criteria.

The first releasable version is treated as the final product of the initial development cycle. Earlier prototypes are milestones used to validate the data model, routing logic, and user interface. Features deliberately postponed beyond this release are listed as possible system evolution.

## 2. Problem Statement

Passengers need route suggestions that reflect a chosen travel date and time, permitted transfers, and personal route preferences. Existing journey planners may provide suitable routes, but users may also want to exclude particular public transport routes before planning—for example, because they prefer not to use a specific tram or bus service.

The project therefore needs to determine whether a small but realistic planner can process Budapest's published timetable data and return valid, understandable, and useful alternatives without requiring real-time information in its first release.

## 3. Proposed Solution

The proposed system is a web application with a Java backend. The user selects an origin stop, destination stop, date and time, and may exclude one or more routes. The system searches the static timetable, applies the restrictions before route selection, and returns several distinct alternatives when similarly suitable journeys exist.

Each result should show the services used, boarding and alighting stops, departure and arrival times, transfers, and total journey duration. If no valid journey remains after filtering, the application must explain this clearly.

## 4. Project Scope

### 4.1 In Scope

The first release will include:

- stop-to-stop journey planning within the BKK static GTFS network;
- user-selected travel date and departure time;
- processing of service calendars, timetable times, stop order, and boarding or alighting restrictions;
- transfers at the same stop, or through explicitly published `pathways.txt` connections using their traversal times and a defined transfer margin;
- exclusion of selected routes before the journey search;
- multiple distinct, reasonably competitive alternatives when available;
- a simple web interface and REST-based Java backend;
- understandable itinerary details and clear validation or no-route messages;
- tests on a manually created network, then a small GTFS subgraph, and finally the complete static feed.

### 4.2 Out of Scope

The first release will not include:

- address-to-address planning or geocoding;
- street-level walking, including walking between nearby but unlinked stops;
- inferred transfers based only on geographic distance;
- real-time delays, disruption data, or automatic replanning;
- fares, ticket purchasing, or fare optimisation;
- accessibility-specific routing;
- user accounts, saved preferences, or journey history;
- a dedicated mobile application.

These exclusions keep the first release achievable and avoid dependencies whose cost, licence, availability, or technical complexity is not yet sufficiently certain.

## 5. Alternatives Considered

| Decision | Alternative | Reason for the selected approach |
|---|---|---|
| Journey endpoints | Addresses with street walking | Stop-to-stop planning avoids an early dependency on geocoding and pedestrian-routing services. |
| Initial data | Complete GTFS feed from the start | A manual network and a small subgraph make routing behaviour easier to verify before scaling up. |
| Timetable type | Real-time data in the first release | Static GTFS is sufficient to validate the core planner; real-time support can be added later. |
| Storage | PostgreSQL as a mandatory first step | An in-memory or file-backed prototype reduces early complexity; PostgreSQL remains an optional later optimisation. |
| Client | Native mobile application | A responsive web interface is less costly and supports the intended demonstration and testing needs. |

## 6. Target Users

The primary users are Budapest passengers who want to plan a public transport journey between two stops for a specified date and time. They are expected to understand common transport concepts such as stops, routes, directions, and transfers, but should not need technical knowledge of GTFS.

The route-exclusion option particularly supports passengers who wish to avoid a specific service for comfort, perceived safety, crowding, or another personal reason. The application does not assess whether such concerns are objectively justified; it applies the user's stated preference.

## 7. Data Sources

### 7.1 Development Data

A small manually created network will be used first to test individual cases with known expected results. It will include direct journeys, valid and invalid transfers, route exclusions, timetable boundaries, and cases in which no journey exists. This isolates algorithmic errors before the full dataset is introduced.

### 7.2 BKK Static GTFS Data

The first release will use the static GTFS feed published by BKK. Relevant files include `stops.txt`, `routes.txt`, `trips.txt`, `stop_times.txt`, `calendar_dates.txt`, `pathways.txt`, `agency.txt`, and `feed_info.txt`. The examined feed contains detailed stop areas and pathway traversal times but no separate `transfers.txt`; consequently, only same-stop transfers and explicit pathways will be accepted.

The feed has a limited validity period indicated by `feed_info.txt`. Imported data must therefore be validated, and the dataset must be replaceable when BKK publishes a newer version. The exact feed version and validity dates are operational data, not permanent requirements of the application.

### 7.3 Volume and Coverage

The timetable contains millions of stop-time records, making `stop_times.txt` the main performance concern. Development should progress from controlled data to a subgraph and only then to the complete feed. Efficient parsing, indexed lookups, and memory measurements are required before the final storage design is fixed.

Future versions may use real-time BKK data or external geocoding and pedestrian-routing services, but their availability, cost, licences, and integration requirements must be evaluated separately before they enter the committed scope.

## 8. Technical Feasibility

### 8.1 Backend

Java is suitable for GTFS parsing, timetable processing, graph-based search, testing, and REST APIs. Spring Boot can provide request handling, validation, configuration, and a clear separation between import, routing, and presentation logic. The main challenge is not the language but the correct modelling of time-dependent journeys and transfers.

### 8.2 Frontend

A small interface can be implemented with HTML, CSS, and limited JavaScript. It needs stop selection, date and time input, route-exclusion controls, a search action, and readable result cards. Advanced mapping and framework-heavy frontend development are not necessary for the first release.

### 8.3 Route Planning

The transport network can be represented as timetable connections between stop events. A valid search must respect service dates, departure and arrival times—including GTFS times beyond `24:00:00`—stop sequence, pickup and drop-off restrictions, pathway traversal times, and a transfer margin. Route exclusions must be applied before candidate journeys are generated.

A time-dependent shortest-path method, connection-scan approach, or another timetable-routing algorithm is feasible in Java. Alternative journeys should be meaningfully distinct and remain within predefined quality limits compared with the best result. The exact algorithm should be selected after experiments on the manual network and a real-data subgraph.

### 8.4 Data Storage

The first prototypes may keep processed data in memory or use compact local files. PostgreSQL may be introduced if it materially improves import management, indexing, repeatable queries, or application startup. It is useful for demonstration and future growth, but it is not required before measurements show a clear benefit.

### 8.5 Deployment

The application can initially run locally and later be packaged with Docker. A small hosting service should be sufficient for demonstration use, subject to measured memory and response-time requirements. Production-scale availability, horizontal scaling, and complex orchestration are outside the first release.

## 9. Operational Feasibility

The intended workflow is simple: select two stops, enter a date and time, optionally exclude routes, and compare the returned journeys. Clear labels, validation, loading feedback, and no-result explanations are necessary because routing constraints can legitimately eliminate every option.

Operation also requires a repeatable process for downloading, validating, importing, and replacing the static GTFS feed. Import failures must not silently corrupt the active timetable. Basic logging and documented startup, update, testing, and deployment procedures are sufficient for the first release.

## 10. Legal and Data Usage Considerations

The examined BKK static GTFS dataset is published under the Creative Commons Attribution 4.0 International licence. The application must preserve the required attribution, identify significant modifications where relevant, and include a link to the licence. A suitable notice is: **Data source: BKK Zrt., CC BY 4.0.**

Attribution should appear in a user-visible location such as an About or Data Sources section, and it should also be included in the repository README. The licence and BKK's current data-access conditions must be checked again before public release because publication terms may change.

The first release does not require personal accounts or persistent storage of user journeys, which limits privacy concerns. If analytics, accounts, saved searches, or location data are introduced later, their privacy and data-retention implications will require a separate review.

## 11. Risks and Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Incorrect handling of dates, times, or boarding rules | Invalid journey results | Create focused unit tests, including exceptions and times beyond 24:00. |
| Missing or invalid transfer links | Impossible or unrealistic changes | Permit only same-stop and explicit pathway transfers; validate referenced stops and traversal times. |
| Incomplete pathway coverage | Some realistic journeys cannot be found | Document this limitation and test representative interchange stations. |
| Full-feed size causes slow import or search | Poor usability or memory failure | Test progressively, profile the importer and search, and add indexes or persistence only where measured. |
| Alternatives are duplicates or much worse than the best route | Low user value | Define distinctness and quality thresholds, then test them on reference journeys. |
| Route exclusions remove every valid journey | Empty result set | Treat this as expected behaviour and provide a clear explanation with removable filters. |
| Feed format or version changes | Import or timetable failure | Validate each import, keep the previous working dataset, and document feed replacement. |
| Limited experience and single-developer capacity | Schedule or quality risk | Use small milestones, prioritise core routing, and postpone nonessential technology. |
| Licence or access conditions change | Public release may be restricted | Recheck official terms before deployment and record dataset provenance. |

## 12. Required Resources

The software will be implemented by one developer, with non-programmer volunteers assisting in usability testing. Existing knowledge includes Java, object-oriented programming, basic graph theory and search algorithms, HTML, CSS, Git, and SQL. Learning is required for Spring Boot, GTFS import and validation, time-dependent routing, Java database integration, JavaScript, Docker, and deployment.

Development tools may include IntelliJ IDEA, Git, GitHub, Spring Boot, JUnit, and optionally PostgreSQL and Docker. These have free options suitable for this project. A normal desktop computer with several gigabytes of available memory should be adequate, although the complete-feed tests must confirm this.

The only required external dataset is BKK's static GTFS feed. No paid service is necessary for the defined first release. Possible later expenses are hosting and an optional domain name.

## 13. Estimated Development Phases

1. **Requirements and design:** finalise this study, functional and non-functional requirements, use cases, user stories, architecture, data model, interface wireframes, and only those diagrams that clarify the solution.
2. **Prototype 1:** implement approximately 30% of the use cases on a manual network; establish version control and validate basic route search.
3. **Prototype 2:** implement approximately 90% of the use cases; import a real-data subgraph, add continuous integration, and introduce partial unit testing.
4. **Final product:** support the full static feed and complete agreed functionality; improve code quality, documentation, automated tests, usability, and deployment readiness.
5. **System evolution:** consider real-time data, address search, street walking, accessibility-aware routing, saved preferences, and other features only after separate feasibility checks.

## 14. Success Criteria

The initial product will be considered successful when:

- all agreed functional acceptance tests pass;
- a current complete BKK static GTFS feed can be validated and imported without manual data editing;
- at least 20 predefined reference journeys produce timetable-valid results;
- every result respects the selected date and time, service availability, stop order, pickup and drop-off rules, transfer rules, and excluded routes;
- multiple alternatives are shown when the defined distinctness and quality conditions are met, while duplicate or unreasonably poor options are omitted;
- at least 95% of the reference searches finish within two seconds on the documented development computer after data loading;
- at least four of five usability-test participants can complete a basic search and exclude a route without assistance;
- an invalid or inconsistent feed is rejected with a clear error instead of replacing the last valid dataset.

## 15. Conclusion

The project is provisionally feasible for one developer if the first release remains limited to stop-to-stop planning with static GTFS data and explicitly supported transfers. Java and Spring Boot are suitable, and the required core tools and dataset can be used without mandatory licence fees.

The highest uncertainties are timetable-routing correctness, pathway coverage, and full-feed performance. The staged approach—manual network, real-data subgraph, then complete feed—allows these risks to be tested before architecture or storage choices become expensive to change. Address-based planning, street walking, real-time information, and accessibility-specific routing should remain evolution candidates until their dependencies and effort are evaluated separately.

## References

- BKK Centre for Budapest Transport, static GTFS dataset and data-access information.
- General Transit Feed Specification (GTFS) Schedule Reference.
- Creative Commons Attribution 4.0 International licence.
