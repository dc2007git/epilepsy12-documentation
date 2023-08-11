---
title: Architecture Overview
authors: Dr Marcus Baw
---

## Epilepsy12 Application Architectural Overview

### Introduction

This overview was originally prepared in order to assist the Privacy Impact Assessment of the E12 platform, but it is of general utility and interest, so has been made part of the documentation site for the project.

### Cloud services

All of the Epilepsy12 application

### Epilepsy12 Django Application

The main application is a web application, written in **Python** using the **Django** framework. It is currently served using Azure App Services which is a cloud-based application deployment platform, but in the future we are transitioning to use a more 'traditional' approach, deploying the application and all dependencies to a virtual machine in the cloud.

A secure connection (HTTPS, also known as SSL) is used to access the application from the web.

### Database

The application uses a **PostgreSQL** database, which is a common and reliable open-source relational database management system used by millions of other applications. The database is hosted on Azure, and is accessed by the application using a secure connection.

All the data in the live instance of the application is stored in the database. The database is backed up daily, and the backups are stored in a secure location on Azure.

A system called **Redis** is used to cache data from the database, which improves the performance of the application. This is managed by **Celery**, which is a system for running background tasks.

### Authentication

Users authenticate with the application using an email address, and a password. The password is stored in the database, but is encrypted using a secure hashing algorithm, which performs a kind of one-way mathematical function on the password data. This means that even if the database is compromised, the passwords cannot be discovered. However the application can still check that someone has entered the right password by comparing the hash of the password they entered with the hash stored in the database.

Accounts are created manually by super users who first verify that the proposed new user has the right to access the application. The application does not allow users to create their own accounts, since our user base is very specifically defined as the contributors to the Epilepsy12 audit.

### Authorisation

User accounts are limited to a single role, which determines what they can do within the application. The exact roles and capabilities are detailed in the [admin user guide](/admin-users/admin-user-guide.md).

#### Access constraints

* Users at a specific clinical site can only access data for children at that Trust.
* Users with a national role can access data for all children.



