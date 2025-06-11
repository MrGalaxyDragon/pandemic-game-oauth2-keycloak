<div align="center">
    <h1>Pandemic Game OAuth2 Keycloak</h1>
    <p>
        <b>OAuth 2.0 identity providers for the game <a href="https://github.com/PandemicGame">Pandemic</a></b>
    </p>
</div>

[![GitHub contributors](https://img.shields.io/github/contributors/PandemicGame/pandemic-game-oauth2-keycloak?style=for-the-badge)](https://github.com/PandemicGame/pandemic-game-oauth2-keycloak/graphs/contributors)
[![GitHub forks](https://img.shields.io/github/forks/PandemicGame/pandemic-game-oauth2-keycloak?style=for-the-badge)](https://github.com/PandemicGame/pandemic-game-oauth2-keycloak/forks)
[![GitHub Repo stars](https://img.shields.io/github/stars/PandemicGame/pandemic-game-oauth2-keycloak?style=for-the-badge)](https://github.com/PandemicGame/pandemic-game-oauth2-keycloak/stargazers)
[![License: GPL-3.0](https://img.shields.io/github/license/PandemicGame/pandemic-game-oauth2-keycloak?style=for-the-badge)](https://opensource.org/license/gpl-3-0)

> [!TIP]
> see also [Backend](https://github.com/PandemicGame/pandemic-game-backend-java)
> <br>
> see also [Frontend](https://github.com/PandemicGame/pandemic-game-frontend-svelte)

> [!NOTE]
> based on the [Keycloak Dockerized - Multiple Instances](https://github.com/MrGalaxyDragon/keycloak-dockerized-multiple-instances) project

## About

This project provides two (or more) identically configured Keycloak instances as [OAuth 2.0](https://oauth.net/2/) identity providers for the game [Pandemic](https://github.com/PandemicGame).
It uses Bash scripts and [Docker Compose](https://docs.docker.com/compose/) to create Keycloak instances with the same configuration, including the same admin username and password as well as the same realms.
All Keycloak instances will share the common prefix `pandemic-game-oauth2-keycloak` as [Docker project name](https://docs.docker.com/compose/how-tos/project-name/), thus making it easier to distinguish them from other distinct instances running on your machine.

This project can obviously also be used to create only a single instance.
This requires no additional changes to the setup, except for the required configuration explained [here](#creating-keycloak-instances).

Utilizing this project can make developing and playing the game [Pandemic](https://github.com/PandemicGame) easier.
A default realm configuration for the game is included [here](./import-dev/pandemic-game-oauth2-keycloak-0-keycloak-1/).

To make exporting realms easier, this project also includes a script for exporting realms from already existing Keycloak Docker containers.
To export a realm from a non-dockerized Keycloak instance, look [here](#exporting-a-realm).

> [!WARNING]
> **This project should only be used for development purposes.**
> **Do not use it in a production environment for reasons listed [below](#disclaimer)!**

### Disclaimer

When you export a realm configuration using the [export utils](./export-utils/export-realm.sh) provided in this project, they will also export all client secrets, user credentials and other sensitive information.
This is intentional, as one wants to have as little effort as possible when setting up multiple instances.
That is what this project is all about.
However, having all of the secrets exposed may be problematic when it comes to security.
That is why all realms not in the [`import-dev` directory](./import-dev/) are ignored by Git by default (`/import` and `/export-utils/keycloak-export` directories in [.gitignore](./.gitignore)).
The [`import-dev` directory](./import-dev/) contains a configured development realm for the game [Pandemic](https://github.com/PandemicGame).

For development purposes and to easily share the Keycloak configuration between developers, it may be smart to include the realm configuration in the repository.
If you want to update the development configuration, please make sure to only commit changes to it in the [`import-dev` directory](./import-dev/).
But be aware that you do not use a realm configuration that includes production secrets.
Instead, manually re-create a realm with the same configuration and create example users and clients for your application.

### Technology Overview

- [![.env](https://img.shields.io/badge/.env-gray?logo=dotenv)](https://www.dotenv.org/)
- [![Bash](https://img.shields.io/badge/Bash-gray?logo=gnubash)](https://www.gnu.org/software/bash/)
- [![Docker](https://img.shields.io/badge/Docker-gray?logo=docker)](https://www.docker.com/)
- [![Keycloak](https://img.shields.io/badge/Keycloak-gray?logo=keycloak)](https://www.keycloak.org/)
- [![PostgreSQL](https://img.shields.io/badge/PostgreSQL-gray?logo=postgresql)](https://www.postgresql.org/)

## Getting Started

To use this project for your own purposes, you can follow the steps listed below.

### Prerequisites

You need to have the following programs installed on your machine:
- [![Docker](https://img.shields.io/badge/Docker-gray?logo=docker)](https://www.docker.com/)
- [![Git](https://img.shields.io/badge/Git-gray?logo=git)](https://git-scm.com/)

### Installation

1. Clone [this repository](https://github.com/PandemicGame/pandemic-game-oauth2-keycloak):
    ```sh
    git clone https://github.com/PandemicGame/pandemic-game-oauth2-keycloak.git
    ```
2. Navigate to its directory:
    ```sh
    cd pandemic-game-oauth2-keycloak
    ```

### Usage

Before using either the realm export or the Keycloak instance creation feature, you have to configure environment variables.
For guidance on how to do that and then using those features, read the sections below.

#### Environment Variables

Environment variables for both the main [start](./start.sh) and [stop](./stop-and-remove.sh) scripts as well as the [export](./export-utils/export-realm.sh) script are stored in `.env` files.
The corresponding `.env.example` files show what variables are needed for configuration.
Before starting any script, you need to make a copy of the corresponding `.env.example` file ([`.env.example`](./.env.example) for [start](./start.sh) and [stop](./stop-and-remove.sh), [`.env.example`](./export-utils/.env.example) for [export](./export-utils/export-realm.sh)) and rename it to `.env`.
Thereafter, you can change the required environment variables.
By default, all environment variables are configured with valid values.
If you do not want to adjust the configuration, you can start [here](#creating-keycloak-instances).
Customizing the environment variables and the constraints applying to them are described [here](#configuration).

#### Suggested Workflows

The following workflow was used to configure the development realm for [Pandemic](https://github.com/PandemicGame):
1. Create one Keycloak instance:
    1. Configure [.env](./.env) so that there is only one variable called `PORT_0`.
    2. Create a Keycloak instance as specified [here](#creating-keycloak-instances).
    3. Navigate to the [Keycloak UI](http://localhost:9000/) in your browser and login with admin credentials as specified by [.env](./.env).
    4. Create and configure a new realm as per [this guide](#pandemic-development-realm).
    5. Export the realm as specified [here](#exporting-a-realm).
    6. Delete the instance as specified [here](#deleting-keycloak-instances).
2. Comment out the port variable called `PORT_0` and add two new port variables called `PORT_1` and `PORT_2` to [.env](./.env).
3. Create the Keycloak instances as described [here](#creating-keycloak-instances).

##### Exporting a Realm

If you have a pre-existing dockerized Keycloak instance, use the [export script](./export-utils/export-realm.sh) to export its configuration:
1. Update `KEYCLOAK_CONTAINER_NAME` and `REALM_NAME` in [.env](./export-utils/.env) if necessary (optional).
2. Switch to directory of [export-utils script](./export-utils/export-realm.sh):
    ```sh
    cd export-utils
    ```
3. Run the [export script](./export-utils/export-realm.sh), for example in [Git Bash](https://git-scm.com/book/en/v2/Appendix-A:-Git-in-Other-Environments-Git-in-Bash):
    ```sh
    ./export-realm.sh
    ```
4. Files will be exported to a directory named in accordance to `EXPORT_DIR_NAME` in [.env](./export-utils/.env) ([`import-dev`](./import-dev/) by default).

If you have a pre-existing non-dockerized Keycloak instance, use the following script while in the directory you have installed Keycloak in:
```sh
./bin/kc.sh export --dir='$EXPORT_PATH' --realm='$REALM_NAME' --users='$USERS_EXPORT_STRATEGY'
```
> Of course, replace `$EXPORT_PATH`, `$REALM_NAME` and `$USERS_EXPORT_STRATEGY` with the correct values.

> [!NOTE]
> If that does not work, consult the [Keycloak documentation](https://www.keycloak.org/server/importExport#_exporting_a_realm_to_a_directory).

##### Creating Keycloak Instances

1. Update `PROJECT_ROOT_NAME` and port variables in [.env](./.env) if necessary ([more info](#configuration)) (optional).
2. Run the [start script](./start.sh), for example in [Git Bash](https://git-scm.com/book/en/v2/Appendix-A:-Git-in-Other-Environments-Git-in-Bash):
    ```sh
    ./start.sh
    ```

##### Deleting Keycloak Instances

1. Make sure you have the correct `PROJECT_ROOT_NAME` and port variables in [.env](./.env).
    > You can also only remove some of your Keycloak instances by removing some port variables you used when [creating instances](#creating-keycloak-instances).
    > The remove script will then only remove the instances of which the port number **is still in** [.env](./.env).
2. Run the [stop script](./stop-and-remove.sh), for example in [Git Bash](https://git-scm.com/book/en/v2/Appendix-A:-Git-in-Other-Environments-Git-in-Bash):
    ```sh
    ./stop-and-remove.sh
    ```

#### Configuration

##### Main

For [main .env](./.env):

<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Default Value</th>
            <th>Possible Values</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <code>IMPORT_DIR_NAME</code>
            </td>
            <td>
                <code>./import-dev/pandemic-game-oauth2-keycloak-0-keycloak-1</code>
            </td>
            <td>
                any path to an existing directory
            </td>
            <td>
                This is the directory from which the realm or realms will be imported during Keycloak instance creation.
            </td>
        </tr>
        <tr>
            <td>
                <code>PROJECT_ROOT_NAME</code>
            </td>
            <td>
                <code>pandemic-game-oauth2-keycloak</code>
            </td>
            <td>
                any string
                <br>
                <small>(<code>kebab-case</code> is recommended)</small>
            </td>
            <td>
                This is the beginning of the project name of every created Keycloak instance.
                The number of the port variable of the concrete instance will be added to the root name to create its full project name, e.g.:
                <code>PROJECT_ROOT_NAME</code>: <code>example-name</code>,
                instance port variable name: <code>PORT_1</code>,
                full project name: <code>example-name-1</code>.
            </td>
        </tr>
        <tr>
            <td>
                <code>COMPOSE_FILE</code>
            </td>
            <td>
                <code>./docker-compose.yml</code>
            </td>
            <td>
                path to a valid <a href="https://docs.docker.com/compose/">Docker Compose file</a>
            </td>
            <td>
                If you want to specify a different Docker Compose file than <a href="./docker-compose.yml">the included one</a>, you can specify that here.
                It will be called by the <a href="./start.sh">start</a> and <a href="./stop-and-remove.sh">stop</a> scripts for every port variable in the <a href="./.env">.env</a>.
            </td>
        </tr>
        <tr>
            <td>
                <code>KEYCLOAK_ADMIN_USERNAME</code>
            </td>
            <td>
                <code>admin</code>
            </td>
            <td>
                any string
            </td>
            <td>
                This will be the username of the admin of every created Keycloak instance.
            </td>
        </tr>
        <tr>
            <td>
                <code>KEYCLOAK_ADMIN_PASSWORD</code>
            </td>
            <td>
                <code>admin</code>
            </td>
            <td>
                any string
            </td>
            <td>
                This will be the password of the admin of every created Keycloak instance.
            </td>
        </tr>
        <tr>
            <td>
                <code>KEYCLOAK_DATABASE_NAME</code>
            </td>
            <td>
                <code>keycloak</code>
            </td>
            <td>
                any <a href="https://www.postgresql.org/docs/current/sql-syntax-lexical.html#SQL-SYNTAX-IDENTIFIERS">allowed PostgreSQL database name</a>
            </td>
            <td>
                This will be the database name of every created database the corresponding Keycloak instance uses.
            </td>
        </tr>
        <tr>
            <td>
                <code>KEYCLOAK_DATABASE_USERNAME</code>
            </td>
            <td>
                <code>keycloak</code>
            </td>
            <td>
                any string
            </td>
            <td>
                This will be the database user's username of every created database the corresponding Keycloak instance uses.
            </td>
        </tr>
        <tr>
            <td>
                <code>KEYCLOAK_DATABASE_PASSWORD</code>
            </td>
            <td>
                <code>keycloak</code>
            </td>
            <td>
                any string
            </td>
            <td>
                This will be the database user's password of every created database the corresponding Keycloak instance uses.
            </td>
        </tr>
        <tr>
            <td>
                <code>PORT_0</code>
                <br>
                <code>PORT_1</code>
                <br>
                <code>PORT_2</code>
            </td>
            <td>
                <code>9000</code>
                <br>
                <code>9001</code>
                <br>
                <code>9002</code>
            </td>
            <td>
                any not-used <a href="https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers#Registered_ports">registered port</a>
            </td>
            <td>
                This is a list of ports the Keycloak instances will run on.
                To create more instances, add variables in form <code>PORT_&lt;number&gt;</code>.
                It is encouraged to provide port numbers as a sequence and then give the port variables the last digit of the port numbers as name, e.g.:
                <code>PORT_1</code>: <code>9001</code>,
                <code>PORT_2</code>: <code>9002</code>,
                <code>PORT_3</code>: <code>9003</code>...
                <br>
                <b>Note:</b>
                The variable <code>PORT_0</code> in <a href="./.env.example">.env.example</a> is a comment.
                It was only used for the instance involved in initially creating the development realm.
            </td>
        </tr>
    </tbody>
</table>

##### Export utils

For [export utils .env](./export-utils/.env):

<table>
    <thead>
        <tr>
            <th>Name</th>
            <th>Default Value</th>
            <th>Possible Values</th>
            <th>Description</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>
                <code>EXPORT_DIR_NAME</code>
            </td>
            <td>
                <code>../import-dev</code>
            </td>
            <td>
                a valid directory path
                <br>
                <small>(A directory will be created if it does not exist.)</small>
            </td>
            <td>
                This is the directory to which the realm will be exported.
            </td>
        </tr>
        <tr>
            <td>
                <code>KEYCLOAK_CONTAINER_NAME</code>
            </td>
            <td>
                <code>pandemic-game-oauth2-keycloak-0-keycloak-1</code>
            </td>
            <td>
                valid Docker container name on your system
            </td>
            <td>
                This is the container name of the Keycloak instance a realm is to be exported out of.
                <br>
                <b>Note:</b>
                Run <code>docker ps -a</code> if you want to see all containers on your system. 
                The container name is in the <code>NAMES</code> column.
            </td>
        </tr>
        <tr>
            <td>
                <code>REALM_NAME</code>
            </td>
            <td>
                <code>pandemic</code>
            </td>
            <td>
                valid realm name in the targetted Keycloak instance
            </td>
            <td>
                This is the name of the realm that is to be exported.
            </td>
        </tr>
        <tr>
            <td>
                <code>USERS_EXPORT_STRATEGY</code>
            </td>
            <td>
                <code>same_file</code>
            </td>
            <td>
                <code>different_files</code>
                <br>
                <code>skip</code>
                <br>
                <code>realm_file</code>
                <br>
                <code>same_file</code>
            </td>
            <td>
                A list of all possible strategies including explanations can be found <a href="https://www.keycloak.org/server/importExport#_configuring_how_users_are_exported">here</a>.
            </td>
        </tr>
    </tbody>
</table>

##### Pandemic development realm

The Keycloak instance and realm for the development of [Pandemic](https://github.com/PandemicGame) were configured as per [this tutorial](https://phasetwo.io/blog/instant-user-management-and-sso-for-sveltekit/).

###### Realm Configuration

1. Create realm: `pandemic`
2. Create client:
    1. General settings:
        - Client type: `OpenID Connect`
        - Client ID: `pandemic-game-frontend-svelte`
        - Name: `Pandemic Game Frontend Svelte`
    2. Capability config:
        - Client authentication: `On`
        - Authorization: `Off`
        - Standard flow: `On`
        - Direct access grants `On`
        - All other options are turned off.
    3. Login settings:
        - Valid redirect URIs: `http://localhost:5173/*`
        - Web origins: `http://localhost:5173`

###### OIDC Config

Relevant fields:
- `realm`: `pandemic`
- `auth-server-url`: `http://localhost:9000/` (during realm creation)
- `resource`: `pandemic-game-frontend-svelte`
- `client secret`: `rIimJQsaDR0XrvjLiNO5OAOYFeaEAsyI` (only used during development)

###### User Config

1. Create new user:
    - Username: `user`
2. Navigate to <i>Credentials</i> tab and click on <i>Set Password</i>:
    - Password: `user`
    - Password confirmation: `user`
    - Temporary: `Off`

## Contributors

<a href="https://github.com/PandemicGame/pandemic-game-oauth2-keycloak/graphs/contributors">
    <img
        src="https://contrib.rocks/image?repo=PandemicGame/pandemic-game-oauth2-keycloak"
        alt="GitHub contributors" />
</a>