## Purpose

Defines how a user chooses which pilot city (Montréal or Toronto) to browse shapes and routes in, so browsing is simple and never requires sharing the user's location.

## ADDED Requirements

### Requirement: Manual pilot-city selection
The app SHALL let the user choose the city to browse from a list of available pilot cities. For the MVP the list contains Montréal and Toronto. The app SHALL NOT pick the city automatically from the user's location.

#### Scenario: First launch asks for a city
- **WHEN** a signed-in user opens shape browsing for the first time on a device
- **THEN** the app asks the user to choose a city from the available pilot cities before showing shapes

#### Scenario: Shapes shown are for the chosen city
- **WHEN** the user has chosen Toronto
- **THEN** only Toronto's shapes and routes are listed

### Requirement: Browsing does not require location permission
Choosing a city and browsing its shapes and route previews SHALL work without the user granting location permission. Location permission SHALL only be requested when a feature needs the user's position (starting a walk).

#### Scenario: Browsing with location permission denied
- **WHEN** a user has denied location permission
- **THEN** they can still choose a city, browse its shapes, and open route previews

### Requirement: City choice is remembered and changeable
The app SHALL remember the chosen city on the device across app restarts, and the user SHALL be able to switch city at any time from the browsing screen.

#### Scenario: Choice survives restart
- **WHEN** a user chooses Montréal and later restarts the app
- **THEN** browsing opens on Montréal without asking again

#### Scenario: Switching city
- **WHEN** a user switches from Montréal to Toronto
- **THEN** the shape list updates to Toronto's shapes and the new choice is remembered

### Requirement: City list comes from the catalog
The list of selectable cities SHALL come from the shape and route catalog rather than being fixed in the app, and a city SHALL be offered only when it has at least one shipped route.

#### Scenario: City with no shipped routes
- **WHEN** a city in the catalog has no shipped routes
- **THEN** it is not offered in the city list

#### Scenario: Adding a city later
- **WHEN** a new city with shipped routes is added to the catalog in a later release
- **THEN** it appears in the city list without changing the city-selection screen
