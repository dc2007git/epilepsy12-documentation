---
title: Epilepsy12 database
reviewers: Dr Simon Chapman
---

## Frameworks

The platform has been written in Django 4.0 with a [Postgresql Database backend](manual-setup.md).

## Structure

Database structure largely follows the elements of the order. All the models largely have a one to one relationship, with some exceptions. The models are as follows:

### Base Tables

- **Case**: There is one record for each child in the audit
- **Registration**: The is one registration for each case
- **FirstPaediatricAssessment**: This a key milestone in the audit - the date of the first paediatric assessment triggers the initiation of the audit for that child. There can only be one assessment per registration.
- **EpilepsyContext**: The fields in this model describe the risk factors for epilepsy for each child in the audit. There can be only one record in this model per registration.
- **MultiaxialDiagnosis**: This is the formulation of the child or young person's epilepsy and describes their epilepsy in a multiaxial way using the DESSCRIBE approach. There can be only one multiaxial diagnosis record per registration.
- **Episode**: The Episode model records information about each seizure-type. A child's epilepsy may compromise multiple different seizure types, some of which are epileptic, some of which are not. One record in the MultiaxialDiagnosis model can therefore have multiple Episode records. For the MultiaxialDiagnosis record to be complete, there must be at least one associated Episode record which is epileptic.
- **Syndrome**: A child's epilepsy can be part of a broader syndrome or syndromes. The Syndrome model stores information about date of diagnosis and syndrome name. It is possible for a child to have more than one Syndrome associated with their epilepsy, therefore there is a one to many relationship between MultiaxialDiagnosis and Syndrome models.
- **Investigations**: This stores information on whether key investigations have been performed in a timely way and covers ECG, EEG and neuroimaging such as CT or MRI. One record in the Investigations model exists per registration.
- **Assessment**: This is termed Milestones in the audit and in due course the model and its related views will be refactored. It relates particularly to dates and location of key caregivers: in particular the consultant paediatrician with expertise in epilepsy, the paediatric neurologist, the children's epilepsy surgery centre if eligible and the paediatric epilepsy nurse specialist. There is one record in the Assessment model per registration.
- **Management**: This stores information about medications and individualized care plans for each child in the audit. There is only one record in the Management model per registration.
- **AntiEpilepsyMedicine**: This model has a many to one relationship with the Management model. One child may be on more than one medicine, and the medicines maybe used either for the epilepsy or as rescue for a seizure. Information is stored here about start date, whether the medicine is for rescue, and whether aspects relating to side-effects and so on have been discussed. It has a one to many relationship with MedicineEntity.

### Link Tables

There are some many to many relationships. Django normally handles this for you, but the development team chose to implement the link tables in these cases separately to be able to store information about the relationship between the tables.

- **Site**: The relationships here are complicated since one child may have their epilepsy care for different things in different Hospital Trusts. Each Case therefore can have a many to many relationship with the Organisation trust model (since one Organisation can have multiple Cases and one Case can have multiple Organisations). The Site model therefore is a link model between the two. It is used in this way, rather than relying on the Django built-in many-to-many solution, because additional information relating to the organisation can be stored per Case, for example whether the site is actively involved in epilepsy care and what service it provides (acute paediatric, tertiary neurology or epilepsy surgery care).
- **Comorbidity**: The Comorbidity model captures information principly on development, educational and behavioural comorbid diagnoses a child may have. Since it is possible to have more than one, one record in MultiaxialDiagnosis can have several records (or none) in the Comorbidity model. The look up table for Comorbidity is ComorbidityEntiy, which is seeded from SNOMED. This allows a many to many relationship between MultiaxialDiagnosis and CormorbidityEntity
- **AntiEpilepsyMedicine**: is a link table between Management and MedicineEntity

### Lookup Tables (entity)

These classes are used as look up tables throughout the Epilepsy12 application. They are seeded in the first migrations, either pulling content from the the ```constants``` folder, or from SNOMED CT.

- **Organisation**: This model stores information about each Organisation in England, Scotland and Wales. It is used as a lookup for clinicians as well as children in Epilepsy12. It has a many to many relationship with Case and a many to one relationship with Epilepsy12User. It is seeded from the ```constants``` folder with a ```JSON`` list of hospital trusts.
- **Epilepsy12User**: The User base model in Django is too basic for the requirements of Epilepsy12 and therefore a custom class has been created to describe the different users who either administer or deliver the audit, either on behalf of RCPCH, or the hospital trusts.
- **Keyword**: This model stores the keywords that are used to describe the semiology of each seizure event. The original list is taken from the International League against Epilepsy 2017, but is actually badly in need of enrichening. Even the word 'shaking' is missing. Part of the Epilepsy12 project is to validate the description of a seizure using keywords stored in this model. The original list of words is seeded on first run from the ```constants``` folder.
- **Group**: Not strictly an Epilepsy12 model, but a Django model tied to the User class. There are 6 custom groups (3 RCPCH, 3 hospital trust) with differing levels of access depending on status. The permissions, which are granular and relate to the individual model fields, can then be allocated to groups, allowing admin staff to ensure that permissions are granted in a systematic way.
- **EpilepsyCause**: Seeded from ```constants```, provides a look up for MultiaxialDiagnosis.
- **MedicineEntity**: Seeded from SNOMED, provides lookup for the AntiepilepsyMedicine model.
- **SyndromeEntity**: Seeded from SNOMED, provides lookup for the MultiaxialDiagnosis model.
- **IntegratedCareBoardEntity**: Seeded from ```constants``` provides a list of Integrated Care Boards and identifiers
- **OpenUKNetworkEntity**: Seeded from ```constants``` provides a list of OPENUK Networks and identifiers
- **LocalHealthBoardEntity**: Seeded from ```constants``` provides a list of Local Health Boards in Wales and identifiers
- **NHSRegionEntity**: Seeded from ```constants``` provides a list of NHS England regions and identifiers
- **ONSRegionEntity**: Seeded from ```constants``` provides a list of ONS regions and identifiers
- **ONSCountryEntity**: Seeded from ```constants``` provides a list of ONS countries in the UK and identifiers

#### Boundary files and geography extension pack

We have included the Django GIS extension allowing geographic data to be stored. This allowed for `.shp` files for the different regions to be stored and mapping therefore to be possible. The `.shp` files are stored in the following models:

- **IntegratedCareBoardBoundaries**
- **LocalHealthBoardBoundaries**
- **NHSEnglandRegionBoundaries**
- **CountryBoundaries**

In future it is planned to use anonymised, aggregated patient postcodes to calculate distance to epilepsy treatment centres and correlate this against Key Performance Indicators.

## Migrations

Any changes to the database structure are captured in the migrations, and this is run at each deploy, with any fresh migrations being applied if present at that point. They are stored in the ```migrations``` folder and these files should not be altered and should be checked into version control. The initial migrations contain seed functions to seed the database on creation with data required for the lookup tables listed above.
