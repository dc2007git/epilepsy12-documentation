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

#### RCPCH Audit Lead

#### RCPCH Audit Analyst

#### RCPCH Audit Administrator

#### RCPCH Audit Children and Family

## User permissions

These are the permissions held by each User Group.

|         User Group          |    Scope     | Permission Target   | Permissions                            |
| :-------------------------: | :----------: | :------------------ | :------------------------------------- |
|      RCPCH Audit Lead       |   National   | E12 User            | Create, View, Update, Delete           |
|                             |              | E12 Patients        | Create, View, Update, Delete, Transfer |
|                             |              | E12 Patient Records | Create, View, Update, Delete           |
| Audit Centre Lead Clinician |    Trust     | E12 User            | Create, View, Update, Delete           |
|                             |              | E12 Patients        | Create, View, Update, Delete, Transfer |
|                             |              | E12 Patient Records | Create, View, Update, Delete           |
|   Audit Centre Clinician    | Organisation | E12 User            | View                                   |
|                             |              | E12 Patients        | Create, View, Update, Delete           |
|                             |              | E12 Patient Records | Create, View, Update, Delete           |
| Audit Centre Administrator  | Organisation | E12 User            | View                                   |
|                             |              | E12 Patients        | Create, View, Update                   |
|                             |              | E12 Patient Records | View                                   |

#### RCPCH Audit Children and Family

- Can **view** their own data.
- Can consent to participation. Can remove consent. Can opt out.
  - Opting out leads to all data related to them being **delete**d, except the Epilepsy12 Unique Identifier.
