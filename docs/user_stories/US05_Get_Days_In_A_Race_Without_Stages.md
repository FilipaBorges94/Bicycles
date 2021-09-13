# US05 Get days in a race without stages

# 1. Requirements

_As an administrator, I want to know, which are the days off, i.e, the days when there are no stages._

The race has a name and has several stages, these stages have an order.

To create a race we first need to know the name of the race, and the initial and end date of the race must also be
known.

## 1.1. System Sequence Diagram

The System Sequence Diagram below represents the interaction between an Administrator and the Application.

```puml
@startuml US05_RSD
header RSD
title Get days in a race without stages
autonumber
actor "Administrator" as Admin
participant ": Application" as App

Admin -> App : Get days without stages
activate Admin
activate App
return Show days without stages
deactivate App

deactivate Admin
@enduml
```

## 1.2. Dependency of other user stories

This US has dependencies on the [US01] and [US02], since it needs an existing Race and existing Stages within the Race.

# 2. Analysis

## 2.1 Race entry

According to what was presented in the US, a race is created upon request from the Administrator.

A race should be created with an alphanumeric string as its name, initial and end date. In addition, a Race will have its
own classification.

The identification of the race across the application is obtained by the combination  
of its name and its parent.

With that said, a race should have the following attributes:

| Value Objects          | Business Rules                                                         |
| -------------------    | --------------------------------------------------------------         |
| Name                   | required, alphanumeric, String                                         |
| Race Id                | numeric. The identification of the race                                |
| Initial date           | alphanumeric (String), with format "31/12/2021", required              |
| End date               | alphanumeric (String), with format "31/12/2021", required              |
| Stages                 | a list of the stages in the race                                       |
| Teams                  | a list of all the teams competing in the race                          |

## 2.2 Domain Model Excerpt

For quick reference, there's a relevant extract of the domain model.

```puml
<!--
@startuml US001_DM
hide methods
title Domain model US05
header DM
skinparam linetype ortho

object Race {
    - race name 
}
object RaceId {
    - id
}
object Date {
    
}
object Stage {
    - initial hour
    - length 
}
object StageId {
    - id
}


Race "1" --> "1" RaceId  
Race "1..*" --> "1" Date : initial date 
Race "1..*" --> "1" Date : end date 
Race "1" -l-> "1..*" StageId : has
Stage "1" -> "1" StageId 


@enduml
-->
```

# 3. Design

## 3.1. Functionality Development

The System Diagram is the following:

```puml
@startuml
autonumber
header SD
title Get days in a race without stages

actor "Administrator" as ADMIN
participant ": UI" as UI
participant ": RaceController" as RC
participant "raceService\n: RaceService" as RS
participant "aRaceId\n: RaceId" as RID
participant "raceRepository\n: IRaceRepository" as RR
participant "aRace\n: Race" as R
participant "stageRepository\n: IStageRepository" as SR
participant "stageDomainService\n: StageDomainService" as SDS
participant ": DateMapper" as mapper

ADMIN -> UI : Get days without stages
activate ADMIN
activate UI

UI -> RC : getDaysWithoutStages(raceId)
activate RC

RC -> RS : getDaysWithoutStages(raceId)
activate RS

RS --> RID ** : create(raceId)

RS -> RR : getRaceByRaceId(aRaceId)
activate RR


ref over RR
getRaceByRaceId()
end ref

RR --> RS : aRace
deactivate RR

RS -> R : getStages()
activate R

R --> RS : stages
deactivate R

RS -> SR : getStagesDates(stages)
activate SR

ref over SR
getStagesDates()
end ref

SR --> RS : stagesDates
deactivate SR


RS -> SDS : getDaysWithoutStages(aRace, stagesDates)
activate SDS

ref over SDS
getDaysWithoutStages()
end ref

SDS --> RS : daysWithoutStages
deactivate SDS

|||

RS --> mapper** : create()


RS -> mapper : getDaysWithoutStagesDTO(daysWithoutStages)
activate mapper

ref over mapper
getDaysWithoutStagesDTO()
end

mapper --> RS : daysWithoutStagesDTO
deactivate mapper

RS --> RC  : daysWithoutStagesDTO
deactivate RS

RC --> UI : daysWithoutStagesDTO
deactivate RC

UI --> ADMIN : Show days without stages
deactivate UI

deactivate ADMIN
@enduml
```

```puml
@startuml 
header ref
title getRaceByRaceId()
autonumber
participant "raceRepository\n: RaceRepository" as RR
participant "raceIdJPA\n: RaceIdJPA" as RIdJPA
participant "arace\n: Race" as R
participant "raceRepositoryJPA\n: IRaceRepositoryJPA" as RRJPA
participant "raceDomainDataAssembler\n: raceDomainDataAssembler" as RDDA


[-> RR: getRaceByRaceId(aRaceId)
activate RR

RR -> RIdJPA** : create(aRaceId)

RR -> RRJPA : findById(raceIdJPA)
activate RRJPA
return raceJPAOptional

RR -> RDDA : fromDataToDomainRaceName(raceJPAOptional)
activate RDDA
return raceName

RR -> RDDA : fromDataToDomainRaceDate(raceJPAOptional)
activate RDDA
return initialRaceDate

RR -> RDDA : fromDataToDomainRaceDate(raceJPAOptional)
activate RDDA
return endRaceDate

RR -> RDDA : fromDataToDomainStages(raceJPAOptional)
activate RDDA
return stages

RR -> R ** : create(raceName, initialRaceDate, endRaceDate, stages)

[<- RR: aRace
deactivate RR
@enduml
```

```puml
@startuml 
header ref
title getStagesDates()
autonumber
participant "stageRepository\n: StageRepository" as SR
participant "raceIdJPA\n: RaceIdJPA" as RIdJPA
participant "arace\n: Race" as R
participant "raceRepositoryJPA\n: IRaceRepositoryJPA" as RRJPA
participant "raceDomainDataAssembler\n: raceDomainDataAssembler" as RDDA


[-> SR: getStagesDates(stages)
activate SR


SR -> RRJPA : findById(raceIdJPA)
activate RRJPA
return raceJPAOptional

SR -> RDDA : fromDataToDomainRaceName(raceJPAOptional)
activate RDDA
return raceName

SR -> RDDA : fromDataToDomainRaceDate(raceJPAOptional)
activate RDDA
return initialRaceDate

SR -> RDDA : fromDataToDomainRaceDate(raceJPAOptional)
activate RDDA
return endRaceDate

SR -> RDDA : fromDataToDomainStages(raceJPAOptional)
activate RDDA
return stages

SR -> R ** : create(raceName, initialRaceDate, endRaceDate, stages)

[<- SR: aRace
deactivate SR
@enduml
```

## 3.2. Class Diagram

The Class Diagram is the following:

## 3.3. Applied Patterns

In order to achieve best practices in software development, to implement this US the following were used:

- *Single Responsibility Principle* - Classes should have one responsibility, which means, only one reason to change;
- *Information Expert* - Assign a responsibility to the class that has the information needed to fulfill it;
- *Pure Fabrication* - CategoryService was implemented to manage all things related to add a Category.
- *Creator* - To create a category we need to check if the category name doesn't exist.
- *Controller* - CreateStandardCategoryController was created;
- *Low Coupling* - Classes were assigned responsibilities so that coupling remains as low as possible, reducing the
  impact of any changes made to the objects later on;
- *High Cohesion* - Classes were assigned responsibilities so that cohesion remains high(they are strongly related and
  highly focused). This helps to keep the objects understandable and manageable, and also goes hand in hand with the low
  coupling principle.

## 3.4. Tests

### 3.4.1 Unit Tests

Referring different aspects of the User Story, it is necessary to establish a set of unit tests in relation to the
domain classes and Value Objects that make up the aggregate. The unit tests are defined below :

- **Unit Test 1:** Assert the

```java
```

- **Unit Test 2:** Assert the

```java

```

- **Unit Test 3:** Throw an error

```java

```

- **Unit Test 4:**

```java

```

- **Unit Test 5:**

```java

```

### 3.4.2 Integration Tests

In order to ensure that of all parts of the system and functionalities are working correctly (e.g. Controller, Service,
Repository, Domain Model), it is necessary to define a set of Integration Tests that will simulate the system use cases,
such as:

- **Integration Test 1:** Assert the

```java
 
```

- **Integration Test 2:** Assert the

```java

```

- **Integration Test 3:** Throw an error

```java

```

- **Integration Test 4:**

```java

```

- **Integration Test 5:** Do not create child category already existing.

```java

```

# 4. Implementation

The main challenges that were found while implementing this functionality were:

# 5. Integration/Demonstration

# 6. Comments

[us02]: US02.md

[us06]: US06.md

