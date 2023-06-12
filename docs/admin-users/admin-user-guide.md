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

### RCPCH Audit Lead

| Permission Target   | Permissions                            |
| :------------------ | :------------------------------------- |
| E12 User            | Create, View, Update, Delete           |
| E12 Patients        | Create, View, Update, Delete, Transfer |
| E12 Patient Records | Create, View, Update, Delete           |
| **Scope: National** |

### Audit Centre Lead Clinician

| Permission Target   | Permissions                            |
| :------------------ | :------------------------------------- |
| E12 User            | Create, View, Update, Delete           |
| E12 Patients        | Create, View, Update, Delete, Transfer |
| E12 Patient Records | Create, View, Update, Delete           |
| **Scope: Trust**    |

### Audit Centre Clinician

| Permission Target       | Permissions                  |
| :---------------------- | :--------------------------- |
| E12 User                | View                         |
| E12 Patients            | Create, View, Update, Delete |
| E12 Patient Records     | Create, View, Update, Delete |
| **Scope: Organisation** |

### Audit Centre Administrator

| Permission Target       | Permissions          |
| :---------------------- | :------------------- |
| E12 User                | View                 |
| E12 Patients            | Create, View, Update |
| E12 Patient Records     | View                 |
| **Scope: Organisation** |

### RCPCH Audit Children and Family

- Can **view** their own data.
- Can consent to participation. Can remove consent. Can opt out.
  - Opting out leads to all data related to them being **delete**d, except the Epilepsy12 Unique Identifier.
