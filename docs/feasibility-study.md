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
### 7.2 Frontend
### 7.3 Route Planning
### 7.4 Data Storage
### 7.5 Deployment

## 8. Operational Feasibility

## 9. Legal and Data Usage Considerations

## 10. Risks and Mitigations

## 11. Required Resources

## 12. Estimated Development Phases

## 13. Success Criteria

## 14. Conclusion
