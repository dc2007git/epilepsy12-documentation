---
title: Overview
reviewers: Dr Marcus Baw
---

## Admin Users

This section is to be developed alongside application functionality

## User groups

#### Audit Centre Lead Clinician

#### Audit Centre Clinician

#### Audit Centre Administrator

#### RCPCH Audit Team

#### RCPCH Audit Children and Family

## User permissions

These are the permissions held by each User Group.

|         User Group          |    Scope     | Permission Target   | Permissions                            |
| :-------------------------: | :----------: | :------------------ | :------------------------------------- |
|      RCPCH Audit Team       |   National   | E12 User            | Create, View, Update, Delete           |
|                             |   National   | E12 Patients        | Create, View, Update, Delete, Transfer |
|                             |   National   | E12 Patient Records | Create, View, Update, Delete           |
| Audit Centre Lead Clinician |    Trust     | E12 User            | Create, View, Update, Delete           |
|                             |    Trust     | E12 Patients        | Create, View, Update, Delete, Transfer |
|                             |    Trust     | E12 Patient Records | Create, View, Update, Delete           |
|   Audit Centre Clinician    | Trust | E12 User            | View                                   |
|                             | Trust | E12 Patients        | Create, View, Update, Delete           |
|                             | Trust | E12 Patient Records | Create, View, Update, Delete           |
| Audit Centre Administrator  | Trust | E12 User            | View                                   |
|                             | Trust | E12 Patients        | Create, View, Update                   |
|                             | Trust | E12 Patient Records | View                                   |

### RCPCH Audit Children and Family

- Can **view** their own data.
- Can consent to participation. Can remove consent. Can opt out.
  - Opting out leads to all data related to them being **delete**d, except the Epilepsy12 Unique Identifier.
