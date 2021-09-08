# US001 Create standard category

# 1. Requirements

_As a system manager I want to create a standard category._

To create a root standard category we first need to know if the name already exists. If it does not
exist, then it can be added to the category list.

To create a child standard category, the name should not exist within the child categories of the
same parent, and the parent category has to exist.

## 1.1. System Sequence Diagram

The System Sequence Diagram below represents the interaction between a System Manager and the
Application.

<!--
@startuml US001_SSD
header SSD
title Create a standard category
autonumber
actor "System Manager" as SM
participant ": Application" as App

SM -> App : Create a new category
activate SM
activate App
return ask category name and parentId
deactivate App

SM -> App : input category name and parentId
activate App

return inform result
deactivate App

deactivate SM
@enduml
-->

![US001 SSD](diagrams/US001_SSD.png)

## 1.2. Dependency of other user stories

This US has no dependencies of any other User Stories, nevertheless [US002] needs to create
categories to show the categories tree and [US110] needs categories associated with the Family
[US010].

# 2. Analysis

## 2.1 Category entry

According to what was presented in the US, a standard category is created upon request from the
System Manager.

These categories can be used by any user of the Application.

A category should be created with an alphanumeric string as its description/title, and repeated
categories (i.e. with the same name and parent) should not exist among the standard categories
names.

Using this strategy, it was decided that it would be better to create the category, verify if its
name already exists and, if not, then add it to the list of categories.

The identification of the categories across the application is obtained by the combination  
of its name and its parent.

Regarding [US002], categories should also have a tree structure (i.e. levels), such as:

```
CATEGORIES
|-- Shopping (1)
|   |-- Groceries (1.1)
|   |   |-- Vegetables (1.1.1)
|   |   |-- Protein (1.1.2)
|   |   |-- Carbs (1.1.3)
|   |-- Clothes (1.2)
|-- Bills (2)
|   |-- Energy (2.1)
(...)
```

With that said, a category should have the following attributes:

| Attributes | Rules                                                                  |
| ---------- | --------------------------------------------------------------         |
| name       | required, alphanumeric, String                                         |
| id         | numeric. The identification of the category                            |
| parentId   | numeric. The identification of the parent category                     |
| isStandard | a boolean to identify what categories are shipped with the Application |

The default isStandard value is false.

## 2.2 Domain Model Excerpt

For quick reference, there's a relevant extract of the domain model.

```puml
<!--
@startuml US001_DM
hide methods
title Domain model US001
header DM
skinparam linetype ortho

object StandardCategory {
    
}

object BaseCategory {
    - designation 
}

StandardCategory "0..*" -- "1"StandardCategory : childOf 
BaseCategory ^-- StandardCategory


@enduml
-->
```

<!--![US001 Domain Model](diagrams/US001_DM.png)-->

# 3. Design

## 3.1. Functionality Development

Regarding the creation of a new standard category, the requirements specified
in [Analysis](#2-analysis) should be accommodated.

The System Diagram is the following:

```puml
<!---
@startuml US001_SD
header SD
title Create a standard category
autonumber
actor "System Manager" as SM
participant ": UI" as UI
participant ": CategoryController" as CCC
participant "categoryService :\n ICategoryService" as CS <<interface>>
participant ": CategoryAssembler" as CA
participant "categoryName\n :CategoryName" as catName
participant "categoryInputDTO\n :categoryInputDTO" as CDTO
participant "parentCategoryId\n :CategoryId" as catParent


SM -> UI : Create a new standard category
activate SM
activate UI
return ask category name and parentId
deactivate UI

SM -> UI : input category name
activate UI
UI -> CCC : createStandardCategory(categoryInputDTO)
activate CCC

CCC -> CS : createStandardCategory(categoryInputDTO)
activate CS

CS -> CA : fromDTOToCategoryName(categoryInputDTO)
activate CA

CA --> catName **: create(categoryInputDTO)
deactivate CA

CS -> CDTO : getParentId()
activate CDTO
CDTO -> CS : parentId
deactivate CDTO

alt parentId is null

    ref over CS
        createStandardCategory(categoryName)
    end ref    

else parentId not null

CS -> CA : fromDTOToCategoryParent(categoryInputDTO)
activate CA
CA --> catParent **: create(categoryInputDTO)
deactivate CA

    ref over CS
        createStandardCategory(categoryName, parentCategoryId)
    end ref

end

CCC <-- CS: categoryOutputDTO
deactivate CS

UI <-- CCC: categoryOutputDTO
deactivate CCC
return inform result
deactivate UI

deactivate SM

@enduml
-->
```

<!-- ![US001 Sequence Diagram Detailed 2: Create standard category method](diagrams/US001_SD.png)-->

- To create a root category, the sequence action createStandardCategory(name) is the following:

```puml
<!--
@startuml US001_SD_Ref
header ref
title createStandardCategory()
autonumber
participant ": CategoryService" as CS
participant ": CategoryMapper" as CM
participant ": CategoryRepository" as CR
participant "aStandardCategory\n: StandardCategory" as newCat
participant "categoryAssembler\n: CategoryDomainDataAssembler" as CDDA
participant "categoryRepositoryJPA\n: ICategoryRepositoryJPA" as CRJPA <<interface>>

[-> CS : createStandardCategory(categoryName)
activate CS

CS -> newCat ** : create(categoryName) 

activate CR
CS -> CR : saveCategory(aStandardCategory) 
CR -> newCat : getId()
activate newCat
newCat --> CR : categoryId
deactivate newCat

CR -> CR : existsRepeatedCategory(categoryId)



CR -> CDDA : toData(aStandardCategory)
activate CDDA

ref over CDDA
toData()
end

CDDA --> CR : standardCategoryJPA
deactivate CDDA

CR -> CRJPA : save(standardCategoryJPA)
activate CRJPA
deactivate CRJPA

ref over CR
getCategory(categoryId)
end

CR --> CS : standardCategoryFromJPA

CS -> CM : standardCategoryToDTO(standardCategoryFromJPA)
activate CM
CM --> CS : categoryOutputDTO
deactivate CM

[<-- CS: categoryOutputDTO
deactivate CR
deactivate CS
@enduml
-->
```

<!-- ![US001 Sequence Diagram Detailed: Create standard category method](diagrams/US001_SD_Ref.png) -->

- To create a child category, the sequence action createStandardCategory(categoryName, parentId) is
  the following:

```puml


<!--
@startuml US001_SD_Ref2
header ref
title createStandardCategory()
autonumber
participant ": CategoryService" as CS
participant ": CategoryMapper" as CM
participant ": CategoryRepository" as CR
participant "categoryParentIdJPA\n: CategoryIdJPA" as newCatIdJPA
participant "categoryRepositoryJPA\n: ICategoryRepositoryJPA" as CRJPA <<interface>>
participant "aStandardCategory\n: StandardCategory" as newCat
participant "categoryAssembler\n: CategoryDomainDataAssembler" as CDDA


[-> CS : createStandardCategory(categoryName, parentId)
activate CS

CS -> CR : validateCategoryParent(parentId)
activate CR

CR -> newCatIdJPA ** : create(parentId.getId()) 
CR -> CRJPA : existsById(categoryParentIdJPA)
activate CRJPA
deactivate CRJPA

deactivate CR

CS -> newCat ** : create(categoryName, parentId) 

activate CR
CS -> CR : saveCategory(aStandardCategory) 
CR -> newCat : getId()
activate newCat
newCat --> CR : categoryId
deactivate newCat

CR -> CR : existsRepeatedCategory(categoryId)

CR -> CDDA : toData(aStandardCategory)
activate CDDA

ref over CDDA
toData()
end

CDDA --> CR : standardCategoryJPA
deactivate CDDA

CR -> CRJPA : save(standardCategoryJPA)
activate CRJPA
deactivate CRJPA

ref over CR
getCategory(categoryId)
end

CR --> CS : standardCategoryFromJPA

CS -> CM : standardCategoryToDTO(standardCategoryFromJPA)
activate CM
CM --> CS : categoryOutputDTO
deactivate CM

[<-- CS: categoryOutputDTO
deactivate CR
deactivate CS
@enduml
-->
```

<!-- ![US001 Sequence Diagram Detailed 2: Create standard category method](diagrams/US001_SD_Ref2.png) -->

```puml
<!--
@startuml US001_SD_Ref3
header ref
title toData()
autonumber
participant "categoryAssembler\n: CategoryDomainDataAssembler" as CDDA
participant "aStandardCategory\n: StandardCategory" as SC
participant "categoryIdJPA\n :CategoryIdJPA" as CIdJPA
participant "categoryNameJPA\n :CategoryNameJPA" as CNJPA
participant "categoryParentJPA\n :CategoryIdJPA" as CPIdJPA
participant "standardCategoryJPA\n :StandardCategoryJPA" as SCJPA

[-> CDDA: toData(aStandardCategory)

activate CDDA
CDDA -> SC: getId()
activate SC
SC--> CDDA : categoryId
deactivate SC
CDDA --> CIdJPA ** : create(categoryId)

CDDA -> SC: getName()
activate SC
SC--> CDDA : categoryName
deactivate SC
CDDA --> CNJPA ** : create(categoryName)

CDDA -> SC: getParentId()
activate SC
SC--> CDDA : parentId
deactivate SC

alt parentId not null

    CDDA --> CPIdJPA ** : create(parentId)
    CDDA --> SCJPA ** : create(categoryIdJPA, categoryNameJPA, categoryParentJPA)


else parentId is null

    CDDA --> SCJPA ** : create(categoryIdJPA, categoryNameJPA)

end

[<-- CDDA: standardCategoryJPA
deactivate CDDA

@enduml
-->
```

<!--  ![US001 Sequence Diagram Detailed: toData method](diagrams/US001_SD_Ref3.png) -->

```puml
<!--
@startuml US001_SD_Ref4
header ref
title getCategory()
autonumber

participant "categoryRepository\n :CategoryRepository" as CR
participant "categoryIdJPA:\nCategoryIdJPA" as CIdJPA
participant "categoryRepositoryJPA\n: ICategoryRepositoryJPA" as CRJPA <<interface>>
participant "categoryAssembler\n: CategoryDomain\nDataAssembler" as CDDA
participant "categoryJPA\n: StandardCategoryJPA" as SCJPA
participant "standardCategory\n: StandardCategory" as SC

[-> CR: getCategory(categoryId)
activate CR
CR --> CIdJPA ** : create(categoryId.getId())
CR -> CRJPA: findById(categoryIdJPA)
activate CRJPA
CRJPA --> CR : categoryJPA
deactivate CRJPA


CR -> CDDA : fromDataToDomainCategoryName(categoryJPA)
activate CDDA
CDDA --> CR : categoryName
deactivate CDDA

CR -> SCJPA : getParentId()
activate SCJPA
SCJPA --> CR: parentId
deactivate SCJPA

alt parentId is null

    ref over CR
        getCategory(categoryJPA, categoryName)
    end


else parentId is not null
CR -> CDDA : fromDataToDomainParentCategoryId(categoryJPA)
activate CDDA
CDDA --> CR : categoryParentId
deactivate CDDA

    ref over CR
        getCategory(categoryJPA, categoryName, categoryParentId)
    end

end

CR -> SCJPA : getId()
activate SCJPA
SCJPA --> CR: id
deactivate SCJPA

CR -> SC : setIdDatabase(id)
activate SC
deactivate SC

[<-- CR: standardCategory
deactivate CR

@enduml
-->
```

<!--![US001 System Diagram Ref: Get Category Ref5](diagrams/US001_SD_Ref4.png)-->


```puml
<!--
@startuml US001_SD_Ref5
header ref
title getCategory(categoryJPA, categoryName)
autonumber

participant "categoryRepository\n :CategoryRepository" as CR
participant "categoryAssembler\n: CategoryDomain\nDataAssembler" as CDDA
participant "standardCategory\n: StandardCategory" as SC
participant "familyCategory\n: FamilyCategory" as FC

[-> CR: getCategory(categoryJPA, categoryName)

alt categoryJPA is not standard
activate CR
CR -> CDDA : fromDataToDomainFamilyId(categoryJPA)
activate CDDA
CDDA --> CR : familyId
deactivate CDDA

    CR --> FC ** : create(categoryName, familyId)

else categoryJPA is standard

    CR --> SC ** : create(categoryName)

end


[<-- CR: aCategory
deactivate CR

@enduml
-->
```

<!--![US001 System Diagram Ref: Get Category Ref6](diagrams/US001_SD_Ref5.png)-->

```puml
<!--
@startuml US001_SD_Ref6
header ref
title getCategory(categoryJPA, categoryName, categoryParentId)
autonumber

participant "categoryRepository\n :CategoryRepository" as CR
participant "categoryAssembler\n: CategoryDomain\nDataAssembler" as CDDA
participant "standardCategory\n: StandardCategory" as SC
participant "familyCategory\n: FamilyCategory" as FC

[-> CR: getCategory(categoryJPA, categoryName, categoryParentId)

alt categoryJPA is not standard
activate CR
CR -> CDDA : fromDataToDomainFamilyId(categoryJPA)
activate CDDA
CDDA --> CR : familyId
deactivate CDDA

    CR --> FC ** : create(categoryName, categoryParentId, familyId)

else categoryJPA is standard

    CR --> SC ** : create(categoryName, categoryParentId)

end


[<-- CR: aCategory
deactivate CR

@enduml
-->
```

<!--![US001 System Diagram Ref: Get Category Ref 3](diagrams/US001_SD_Ref6.png)-->



When the System Manager inputs the required data for a standard category to be created, the
Application should then operate the required methods creating a new and valid standard category.

## 3.2. Class Diagram

The Class Diagram is the following:

```puml
<!---
@startuml US001_CD
skinparam guillemet false
skinparam linetype ortho
header CD
title Class Diagram US001


class CategoryInputDTO{
 - name : String
 - parentId : Integer
 + getName()
 + setName()
 + getParentId()
 + setParentId()

}

class CategoryAssembler{
 + fromDTOToCategoryName()
 + fromDTOToCategoryParent()
}


class CategoryOutputDTO{
 - categoryId : int
 - categoryName : String
 - parentCategoryId : int
 + getCategoryId()
 + getCategoryName()
 + getParentCategoryId()

}

class CategoryMapper{
 + standardCategoryToDTO()

}

interface ICreateStandardCategoryController {
 + createStandardCategory()
}

class CategoryController{
 + createStandardCategory()
 - addLinksToDTO()
}

class CategoryService{

 + createStandardCategory()
 - createStandardCategory()
}

interface ICategoryService {
 + createStandardCategory()
}

interface ICategoryRepositoryJPA {
 + save()
}

interface CrudRepository {
}
    
package "CategoryJPAAggregate" {
    
    class StandardCategoryJPA <<Entity>> {
     - isStandard : boolean
    }
}

class CategoryNameJPA << Value-Object >> {
 - name: String
}

class CategoryIdJPA << Value-Object >> << Id >> {
 - id: int 
}

class CategoryDomainDataAssembler {
 + toData()
 + fromDataToDomainCategoryName()
 + fromDataToDomainCategoryId()
 + fromDataToDomainParentCategoryId()
 
}  

class CategoryRepository{
 + saveCategory()
 + validateCategoryParent()
 + getCategoryListSize()
 - existsCategory()
 - getCategory()
 - existsCategoryJPA()
}

package "CategoryAggregate" {
    
    class StandardCategory <<Entity>> {
    }
    
    abstract class BaseCategory <<Entity>> <<Root>>{
        # isStandard : boolean
        + getId()
        + setId()
        + getName()
        + hasId()
        + getParentId()
    } 
}

class CategoryName << Value-Object >> {
 - name: String
 - validateStringName()
 - checkFormat()
 - checkStringName()
 - validate()
}

class CategoryId << Value-Object >> << Id >> {
 - id: int 
 + getId()
}

ICreateStandardCategoryController "1" <|.r. "1" CategoryController
CategoryController ..> ICategoryService

CategoryMapper "0..*" *-- "1" StandardCategory : aStandardCategory >
CategoryMapper "1" --> "1" CategoryOutputDTO

CategoryAssembler "0..*" *-- "1" CategoryId : categoryId >
CategoryAssembler "0..*" *-- "1" CategoryName : categoryName >
CategoryAssembler "1" --> "1" CategoryInputDTO

ICategoryService "1" <|.r. "1" CategoryService
CategoryService "1" ..> "1" CategoryRepository
CategoryService "1" ..> "1" CategoryAssembler
CategoryService "1" ..> "1" CategoryMapper

CategoryRepository "1" ..> "1" CategoryDomainDataAssembler
CategoryRepository "1" ..> "1" ICategoryRepositoryJPA
CrudRepository "1" <|.. "1" ICategoryRepositoryJPA

StandardCategory "0..*" - "1" StandardCategory : childOf v
StandardCategory -^ BaseCategory
BaseCategory "\n1" *-- "\n1" CategoryId
BaseCategory "\n0..* " *-- " parentId \n 0..1 " CategoryId
BaseCategory "1..*" *-- "1" CategoryName

StandardCategoryJPA "0..*" - "1" StandardCategoryJPA : childOf v
StandardCategoryJPA "    \n1" *-r- "  \n  1" CategoryIdJPA
StandardCategoryJPA "\n0..* " *-- " parentId \n 0..1 " CategoryIdJPA
StandardCategoryJPA "  1..*" *-- " 1" CategoryNameJPA

ICategoryRepositoryJPA "1" *-- "0..*" StandardCategoryJPA

CategoryDomainDataAssembler "1" *-- "1" StandardCategoryJPA
CategoryDomainDataAssembler "1" *-- "1" CategoryIdJPA
CategoryDomainDataAssembler "1" *-- "1" CategoryNameJPA

@enduml
-->
```

<!---![Class Diagram US001](diagrams/US001_CD.png)-->

## 3.3. Applied Patterns

In order to achieve best practices in software development, to implement this US the following were
used:

- *Single Responsibility Principle* - Classes should have one responsibility, which means, only one
  reason to change;
- *Information Expert* - Assign a responsibility to the class that has the information needed to
  fulfill it;
- *Pure Fabrication* - CategoryService was implemented to manage all things related to add a
  Category.
- *Creator* - To create a category we need to check if the category name doesn't exist. 
- *Controller* - CreateStandardCategoryController was created;
- *Low Coupling* - Classes were assigned responsibilities so that coupling remains as low as
  possible, reducing the impact of any changes made to the objects later on;
- *High Cohesion* - Classes were assigned responsibilities so that cohesion remains high(they are
  strongly related and highly focused). This helps to keep the objects understandable and
  manageable, and also goes hand in hand with the low coupling principle.

## 3.4. Tests

### 3.4.1 Unit Tests

Referring different aspects of the Categories attributes, it is necessary to establish a set of unit
tests in relation to the domain classes, namely the **StandardCategory** and the Value Objects that
make up the aggregate. The unit tests are defined below, organized by the corresponding classes:

- **Unit Test 1:** Assert the creation of a new root standard category.

```java
 @Test
    void createRootStandardCategory(){
        //arrange
            String name="Shopping";
            CategoryInputDTO categoryInputDTO=new CategoryInputDTO();
            categoryInputDTO.setName(name);
            CategoryId categoryId=new CategoryId(new Random().nextInt());

            CategoryOutputDTO categoryDTO=new CategoryOutputDTO(categoryId.getId(),name);
            when(categoryService.createStandardCategory(categoryInputDTO)).thenReturn(categoryDTO);

            ResponseEntity<Object> result;
            HttpStatus expected=HttpStatus.CREATED;

        //act
            result=categoryController.createStandardCategory(categoryInputDTO);

        //assert
            assertNotNull(result);
            assertEquals(expected,result.getStatusCode());
        }
```

- **Unit Test 2:** Assert the creation of a new child standard category.

```java
@Test
    void createChildStandardCategory(){
            //arrange
            String name="Shopping";
            CategoryInputDTO categoryInputDTO=new CategoryInputDTO();
            categoryInputDTO.setName(name);
            CategoryId categoryId=new CategoryId(new Random().nextInt());
            CategoryOutputDTO categoryDTO=new CategoryOutputDTO(categoryId.getId(),name);
            when(categoryService.createStandardCategory(categoryInputDTO)).thenReturn(categoryDTO);
            categoryController.createStandardCategory(categoryInputDTO);


            String childName="Clothing";
            CategoryInputDTO categoryChildInputDTO=new CategoryInputDTO();
            categoryChildInputDTO.setName(childName);
            categoryChildInputDTO.setParentId(categoryId.getId());
            CategoryId childId=new CategoryId(new Random().nextInt());
            CategoryOutputDTO categoryChildDTO=new CategoryOutputDTO(childId.getId(),childName,categoryId.getId());
            when(categoryService.createStandardCategory(categoryChildInputDTO)).thenReturn(categoryChildDTO);

            ResponseEntity<Object> result;
            HttpStatus expected=HttpStatus.CREATED;

        //act
            result=categoryController.createStandardCategory(categoryChildInputDTO);

        //assert
            assertNotNull(result);
            assertEquals(expected,result.getStatusCode());
        }
```

- **Unit Test 3:** Throw an error when creating a category with invalid name.

```java
@ParameterizedTest
@NullAndEmptySource
    void ensureRootStandardCategoryNotCreatedWhenInvalidName(String name){
        //arrange
            CategoryInputDTO categoryInputDTO=new CategoryInputDTO();
            categoryInputDTO.setName(name);
            when(categoryService.createStandardCategory(categoryInputDTO)).thenThrow(InvalidNameException.class);

            ResponseEntity<Object> result;
            HttpStatus expected=HttpStatus.BAD_REQUEST;

        //act
        result=categoryController.createStandardCategory(categoryInputDTO);

        //assert
        assertEquals(expected,result.getStatusCode());
        }
```

- **Unit Test 4:** Do not create child category when parent category does not exist.

```java
@Test
    void ensureChildStandardCategoryNotCreatedWithNonExistingParent(){
        //arrange
            String childName="Vegetables";
            int childParentId=new Random().nextInt();
            CategoryInputDTO categoryInputDTO=new CategoryInputDTO();
            categoryInputDTO.setName(childName);
            categoryInputDTO.setParentId(childParentId);
            when(categoryService.createStandardCategory(categoryInputDTO)).thenThrow(ObjectDoesNotExistException.class);

            ResponseEntity<Object> result;
            HttpStatus expected=HttpStatus.BAD_REQUEST;

        //act
            result=categoryController.createStandardCategory(categoryInputDTO);
    
        //assert
            assertEquals(expected,result.getStatusCode());
        }
```

- **Unit Test 5:** Do not create category already existing.

```java
@Test
    void ensureExistingStandardRootCategoryNotCreatedAgain(){
        //arrange
            String name="Shopping";
            CategoryId categoryId=new CategoryId(new Random().nextInt());
            CategoryInputDTO categoryInputDTO=new CategoryInputDTO();
            categoryInputDTO.setName(name);
            CategoryOutputDTO categoryDTO=new CategoryOutputDTO(categoryId.getId(),name);
            when(categoryService.createStandardCategory(categoryInputDTO)).thenReturn(categoryDTO);
            categoryController.createStandardCategory(categoryInputDTO);

            when(categoryService.createStandardCategory(categoryInputDTO)).thenThrow(DuplicateObjectException.class);

            ResponseEntity<Object> result;
            HttpStatus expected=HttpStatus.BAD_REQUEST;

        //act
            result=categoryController.createStandardCategory(categoryInputDTO);

        //assert
            assertEquals(expected,result.getStatusCode());
        }
```

### 3.4.2 Integration Tests

In order to ensure that of all parts of the system and functionalities are working correctly (e.g.
Controller, Service, Repository, Model), it is necessary to define a set of Integration Tests that
will simulate the system use cases, such as:

- **Integration Test 1:** Assert the creation of a new root standard category.

```java
 @Test
    void ensureRootStandardCategoryIsCreated(){
        //arrange
            String name="Shopping";
            ResponseEntity<Object> result;

            CategoryInputDTO categoryInputDTO=new CategoryInputDTO();
            categoryInputDTO.setName(name);

        //act
             result=categoryController.createStandardCategory(categoryInputDTO);

        //assert
             assertNotNull(result);
            assertEquals(HttpStatus.CREATED,result.getStatusCode());
        }
```

- **Integration Test 2:** Assert the creation of a new child standard category.

```java
@Test
    void ensureChildStandardCategoryIsCreated(){
        //arrange
            String name="Shopping";
            CategoryInputDTO categoryInputDTO=new CategoryInputDTO();
            categoryInputDTO.setName(name);

            CategoryOutputDTO categoryDTO=
            (CategoryOutputDTO)categoryController.createStandardCategory(categoryInputDTO).getBody();
            int parentId=categoryDTO.getCategoryId();

            String childName="Clothing";
            CategoryInputDTO childInputDTO=new CategoryInputDTO();
            childInputDTO.setName(childName);
            childInputDTO.setParentId(parentId);

            ResponseEntity<Object> result;
            HttpStatus expected=HttpStatus.CREATED;

        //act
            result=categoryController.createStandardCategory(childInputDTO);

        //assert
            assertNotNull(result);
            assertEquals(expected,result.getStatusCode());
        }

```

- **Integration Test 3:** Throw an error when creating a category with invalid name.

```java
@ParameterizedTest
@NullAndEmptySource
    void ensureRootStandardCategoryNotCreatedWhenInvalidName(String name){
        //arrange
            CategoryInputDTO categoryInputDTO=new CategoryInputDTO();
            categoryInputDTO.setName(name);

            ResponseEntity<Object> result;
            HttpStatus expected=HttpStatus.BAD_REQUEST;

        //act
            result=categoryController.createStandardCategory(categoryInputDTO);

        //assert
            assertEquals(expected,result.getStatusCode());
        }
```

- **Integration Test 4:** Do not create child category when parent category does not exist.

```java
@Test
    void ensureChildStandardCategoryNotCreatedWithNonExistingParent(){
        //arrange
            String childName="Vegetables";
            int childParentId=new Random().nextInt();
            CategoryInputDTO categoryInputDTO=new CategoryInputDTO();
            categoryInputDTO.setName(childName);
            categoryInputDTO.setParentId(childParentId);

            ResponseEntity<Object> result;
            HttpStatus expected=HttpStatus.BAD_REQUEST;

        //act
        result=categoryController.createStandardCategory(categoryInputDTO);

        //assert
        assertEquals(expected,result.getStatusCode());
        }

```

- **Integration Test 5:** Do not create child category already existing.

```java
@Test
    void ensureExistingStandardChildCategoryNotCreatedAgain(){
        //arrange
            String name="Shopping";
            CategoryInputDTO categoryInputDTO=new CategoryInputDTO();
            categoryInputDTO.setName(name);
            CategoryOutputDTO categoryDTO=
            (CategoryOutputDTO)categoryController.createStandardCategory(categoryInputDTO).getBody();

            int childParentId=categoryDTO.getCategoryId();
            String childName="Clothing";
            CategoryInputDTO childInputDTO=new CategoryInputDTO();
            childInputDTO.setName(childName);
            childInputDTO.setParentId(childParentId);
            categoryController.createStandardCategory(childInputDTO);

            ResponseEntity<Object> result;
            HttpStatus expected=HttpStatus.BAD_REQUEST;

        //act
            result=categoryController.createStandardCategory(childInputDTO);

        //assert
            assertNotNull(result);
            assertEquals(expected,result.getStatusCode());
        }
```

# 4. Implementation

The main challenges that were found while implementing this functionality were:

- The need to identify a parent;

To minimize these difficulties, a lot of research and study of reliable documentation was done.

So that we could present a reliable functionality, many tests were done, to identify as many
possible errors in the implementation as possible.

# 5. Integration/Demonstration

As mentioned before, this functionality will be indirectly necessary for [US002], the standard
categories will be shown in the category tree of the [US002] and for [US110], this User Story will
associate the categories with some instance of Family [US010].

At the moment, no other user stories are dependent on this one, so its integration with other
functionalities cannot be tested further.

# 6. Comments

[us002]: US002_Get_Standard_Categories.md

[us010]: US010_Create_Family_And_Set_Family_Administrator.md

[us110]: US110_Get_Family_Category_Tree.md
