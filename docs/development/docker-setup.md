---
title: Docker development
reviewers: Dr Marcus Baw, Dr Simon Chapman, Dr Anchit Chandran
---

To simplify the development environment setup and provide greater consistency between development and production environments, we have packaged the application as a Docker image and we used `docker compose` to set up the app and database containers in development. This means you don't need to worry about conflicts of Python versions, Python library versions, or Python virtual environments. Everything is specified and isolated inside the Docker container.

## Setup for development using Docker Compose

1. Install Docker on your development machine. Instructions for all supported platforms are here <https://docs.docker.com/get-docker/>

1. Clone this repository to your code folder:

    ```console
    git clone https://github.com/rcpch/rcpch-audit-engine.git
    ```

1. Navigate into the folder:

    ```console
    cd rcpch-audit-engine
    ```
    !!! warning "Environment Variables"
        Developers working on the audit engine require access to the environment variables. Please contact the Incubator Team in order to obtain these. Once obtained, ensure your working directory is `rcpch-audit-engine`, and execute the following command:
        ```console
        mkdir envs
        ```
        Now load the `.env` file, containing environment variables, into this directory. Do not share this file, and ensure all files with the `.env` extension are contained in the `.gitignore` file.
    !!! warning "Windows Setup"
        **If you are on Windows**, after installing Docker and cloning the repository, please now skip to the [(Windows) Setup for development using Docker Compose](./docker-setup.md#windows-setup-for-development-using-docker-compose) section.


1. In your IDE, ensure to enter the `development` branch of the audit engine. In Visual Studio code, find the branch you are on in the bottom left of the screen, on the blue bar, and click next to the Version Control icon. From here, select `development`.

1. Start the development environment for the first time using our startup script
`docker compose -f docker-compose.local-dev.yml up --build`  

    This script automates all the setup steps including upgrading `pip`, installing all development dependencies with `pip install`, migrating the database, seeding the database, and creating a superuser.

    > Development superuser credentials are: username: `e12-dev-user@rcpch.tech` and password `pw`.
    > **Note that these insecure default credentials are ONLY ever used in development for simplicity and ease of use and NEVER used in testing/staging or live.**

!!! tip "`docker-compose` is now `docker compose`!" 
    Note that the command changed from `docker-compose` to `docker <space> compose` with more recent Docker versions.

The `docker compose -f docker-compose.local-dev.yml up --build` script will remove any old containers you have, and start a brand new set from scratch.

Output of `docker compose -f docker-compose.local-dev.yml up --build` should look something like this
```bash
╰─$ docker compose -f docker-compose.local-dev.yml up --build 
[+] Running 3/3
 ⠿ Container rcpch-audit-engine-web-1  Removed                                                          0.0s
 ⠿ Container rcpch-audit-engine-db-1   Removed                                                          0.0s
 ⠿ Network rcpch-audit-engine_default  Removed                                                          0.2s
[+] Running 3/0
 ⠿ Network rcpch-audit-engine_default  Created                                                          0.0s
 ⠿ Container rcpch-audit-engine-db-1   Created                                                          0.0s
 ⠿ Container rcpch-audit-engine-web-1  Created                                                          0.0s
```

This should create a `db` container for the database and another `web` container for the Django app, as well as a network to connect them together.

The `web` container is built with the correct Python version, all development dependencies are automatically installed, the database connection is created, migrations applied and seed data added to the database. The entire process takes less than 30 seconds.

!!! failure "Terminal not fully executing?"
    If your Terminal seems to have frozen, you can simply quit the Terminal and re-run the `docker compose -f docker-compose.local-dev.yml up --build` command.

View the application in a browser at <http://0.0.0.0:8000/> and login using the credentials above.

Changes you make in your development folder are **automatically synced to inside the Docker container**, and will show up in the application right away.

This Docker setup is quite new so please do open an issue if there is anything that doesn't seem to work properly. Suggestions and feature requests welcome.

## (Windows) Setup for development using Docker Compose

You should have already [downloaded Docker](https://docs.docker.com/get-docker/) and cloned the repository.

### Enabling the WSL Terminal within VS Code

There are scripts in the `s/` which streamline the setup for the development process. Unfortunately, Windows does not natively support running these through the Command Prompt. Instead, we must first install the **Windows Subsystem for Linux (WSL)** to run the scripts.

VS Code has a helpful extension to do just this!

Follow this guide ([Windows Subsystem for Linux VSCode Extension](https://code.visualstudio.com/docs/remote/wsl-tutorial)) to enable usage of a *Ubuntu (WSL)* terminal within VS Code.

### Running Scripts

Open a new WSL terminal by selecting it:

![Screenshot of WSL Terminal in VS Code](../_assets/_images/windev_wsl_terminal.png)

If you haven't already, `cd` into the root folder

```console
cd rcpch-audit-engine/
```

Finally, you should be able to run the setup script by typing:

```console
sh s/docker-init
```

!!! info "Setup errors"
    Sometimes, the easiest fix for many headaches, relating to installation and setup, is to simply restart your computer and try again!

### What does `s/docker-init` do?

This script automates all the setup steps including:

- upgrading `pip`
- installing all development dependencies with `pip install`
- migrating the database
- seeding the database
- creating a superuser

> Development superuser credentials are: username: `e12-dev-user@rcpch.tech` and password `pw`.
> <br> **Note these insecure default credentials are ONLY ever used in development for simplicity and ease of use and NEVER used in testing/staging or live.**

!!! tip "`docker-compose` is now `docker compose`!"
    Note that the command changed from `docker-compose` to `docker <space> compose` with more recent Docker versions.

The `s/docker-init` script will remove any old containers, and start a brand new set from scratch.

It should create a db container for the database and another web container for the Django app, as well as a network to connect them together.

The web container is built with the correct Python version, all development dependencies are automatically installed, the database connection is created, migrations applied and seed data added to the database. The entire process takes less than 30 seconds.

View the application in a browser at <http://localhost:8000/> and login using the credentials above.

Changes you make in your development folder are automatically synced to inside the Docker container, and will show up in the application right away.

This Docker setup is quite new so please do open an issue if there is anything that doesn't seem to work properly. Suggestions and feature requests welcome.

## Executing commands in the context of the `web` container

You can run commands in the context of any of the containers using `docker compose`.

The below command will execute `<command>` inside the `web` container.

```console
docker compose exec web <command>
```

For example, to create a superuser

```console
sudo docker compose exec web python manage.py createsuperuser
```

## Running the test suite

```console
sudo docker compose exec web coverage run manage.py test
```

## Shutting down the Docker Compose environment

++ctrl+c++ will shut down the `web` and `db` containers but will leave them built. You can restart them rapidly with `docker compose up`.

To shut down and destroy the containers so that you can start again from scratch (for example if you want to rebuild and re-seed the database) then use

```console
docker compose down
```

## Tips and Tricks

* Although the `docker compose` setup is very convenient, and it installs all the runtime development dependencies _inside_ the `web` container, one thing it can't do is install any _local_ Python packages which are required for text editing, linting, and similar utilities _outside_ the container. Examples are `pylint`, `pylint_django`, etc. You will still need to install these locally, ideally in a virtual environment.