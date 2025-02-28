# dbt-coves

## Sponsor

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="images/datacoves-dark.png">
  <img alt="Datacoves" src="images/datacoves-light.png" width="150">
</picture>

Hosted VS Code, dbt-core, SqlFluff, and Airflow, find out more at [Datacoves.com](https://datacoves.com/product).

## What is dbt-coves?

dbt-coves is a CLI tool that automates certain tasks for [dbt](https://www.getdbt.com), making life simpler for the dbt user.

dbt-coves generates dbt sources, staging models and property(yml) files by analyzing information from the data warehouse and creating the necessary files (sql and yml).

Finally, dbt-coves includes functionality to bootstrap a dbt project and to extract and load configurations from Airbyte.

## Supported dbt versions

| Version | Status           |
| ------- | ---------------- |
| \< 1.0  | ❌ Not supported |
| >= 1.0  | ✅ Tested        |

From `dbt-coves` 1.4.0 onwards, our major and minor versions match those of [dbt-core](https://github.com/dbt-labs/dbt-core).
This means we release a new major/minor version once it's dbt-core equivalent is tested.
Patch suffix (1.4.X) is exclusive to our continuous development and does not reflect a version match with dbt

## Supported adapters

| Feature                           | Snowflake | Redshift  | BigQuery  |
| --------------------------------- | --------- | --------- | --------- |
| dbt project setup                 | ✅ Tested | ✅ Tested | ✅ Tested |
| source model (sql) generation     | ✅ Tested | ✅ Tested | ✅ Tested |
| model properties (yml) generation | ✅ Tested | ✅ Tested | ✅ Tested |

NOTE: Other database adapters may work, although we have not tested them. Feel free to try them and let us know so we can update the table above.

### Here\'s the tool in action

[![image](https://cdn.loom.com/sessions/thumbnails/74062cf71cbe4898805ca508ea2d9455-1624905546029-with-play.gif)](https://www.loom.com/share/74062cf71cbe4898805ca508ea2d9455)

# Installation

```console
pip install dbt-coves
```

We recommend using [python
virtualenvs](https://docs.python.org/3/tutorial/venv.html) and create
one separate environment per project.

# Command Reference

For a complete list of options, please run:

```console
dbt-coves -h
dbt-coves <command> -h
```

## Environment setup

You can configure different components:

Set up `git` repository of dbt-coves project

```console
dbt-coves setup git
```

Set up `dbt` within the project (delegates to dbt init)

```console
dbt-coves setup dbt
```

Set up SSH Keys for dbt project. Supports the argument `--open_ssl_public_key` which generates an extra Public Key in Open SSL format, useful for configuring certain providers (i.e. Snowflake authentication)

```console
dbt-coves setup ssh
```

Set up pre-commit for your dbt project. In this, you can configure different tools that we consider essential for proper dbt usage: `sqlfluff`, `yaml-lint`, and `dbt-checkpoint`

```console
dbt-coves setup precommit
```

## Models generation

```console
dbt-coves generate <resource>
```

Where _\<resource\>_ could be _sources_, _properties_ or _metadata_.

```console
dbt-coves generate sources
```

This command will generate the dbt source configuration as well as the initial dbt staging model(s). It will look in the database defined in your `profiles.yml` file or you can pass the `--database` argument or set up default configuration options (see below)

```console
dbt-coves generate sources --database raw
```

Supports Jinja templates to adjust how the resources are generated. See below for examples.

Every `dbt-coves generate <resource>` supports `--no-prompt` flag, which will silently generate all sources/models/properties/metadata without asking anything to the user.

### Source Generation Arguments

dbt-coves can be used to create the initial staging models. It will do the following:

1. Create / Update the source yml file
2. Create the initial staging model(sql) file and offer to flatten VARIANT(JSON) fields
3. Create the staging model's property(yml) file.

`dbt-coves generate sources` supports the following args:

See full list in help

```console
dbt-coves generate sources -h
```

```console
--database
# Database to inspect
```

```console
--schemas
# Schema(s) to inspect. Accepts wildcards (must be enclosed in quotes if used)
```

```console
--select-relations
# List of relations where raw data resides. The parameter must be enclosed in quotes. Accepts wildcards.
```

```console
--exclude-relations
# Filter relation(s) to exclude from source file(s) generation. The parameter must be enclosed in quotes. Accepts wildcards.
```

```console
--sources-destination
# Where sources yml files will be generated, default: 'models/staging/{{schema}}/sources.yml'
```

```console
--models-destination
# Where models sql files will be generated, default: 'models/staging/{{schema}}/{{relation}}.sql'
```

```console
--model-props-destination
# Where models yml files will be generated, default: 'models/staging/{{schema}}/{{relation}}.yml'
```

```console
--update-strategy
# Action to perform when a property file already exists: 'update', 'recreate', 'fail', 'ask' (per file)
```

```console
--templates-folder
# Folder with jinja templates that override default sources generation templates, i.e. 'templates'
```

```console
--metadata
# Path to csv file containing metadata, i.e. 'metadata.csv'
```

```console
--flatten-json-fields
# Flag: flatten model JSON fields
```

```console
--overwrite-staging-models
# Flag: overwrite existent staging (SQL) files
```

```console
--skip-model-props
# Flag: don't create model's property (yml) files
```

```console
--no-prompt
# Silently generate source dbt models
```

### Properties Generation Arguments

You can use dbt-coves to generate and update the properties(yml) file for a given dbt model(sql) file.

`dbt-coves generate properties` supports the following args:

```console
--destination
# Where models yml files will be generated, default: '{{model_folder_path}}/{{model_file_name}}.yml'
```

```console
--update-strategy
# Action to perform when a property file already exists: 'update', 'recreate', 'fail', 'ask' (per file)
```

```console
-s --select
# Filter model(s) to generate property file(s)
```

```console
--exclude
# Filter model(s) to exclude from property file(s) generation
```

```console
--selector
# Specify dbt selector for more complex model filtering
```

```console
--templates-folder
# Folder with jinja templates that override default properties generation templates, i.e. 'templates'
```

```console
--metadata
# Path to csv file containing metadata, i.e. 'metadata.csv'
```

```console
--no-prompt
# Silently generate dbt models property files
```

Note: `--select (or -s)`, `--exclude` and `--selector` work exactly as `dbt ls` selectors do. For usage details, visit [dbt list docs](https://docs.getdbt.com/reference/commands/list)

### Metadata Generation Arguments

You can use dbt-coves to generate the metadata file(s) containing the basic structure of the csv that can be used in the above `dbt-coves generate sources/properties` commands.
Usage of these metadata files can be found in [metadata](https://github.com/datacoves/dbt-coves#metadata) below.

`dbt-coves generate metadata` supports the following args:

```console
--database
# Database to inspect
```

```console
--schemas
# Schema(s) to inspect. Accepts wildcards (must be enclosed in quotes if used)
```

```console
--select-relations
# List of relations where raw data resides. The parameter must be enclosed in quotes. Accepts wildcards.
```

```console
--exclude-relations
# Filter relation(s) to exclude from source file(s) generation. The parameter must be enclosed in quotes. Accepts wildcards.
```

```console
--destination
# Where csv file(s) will be generated, default: 'metadata.csv'
# Supports using the Jinja tags `{{relation}}` and `{{schema}}`
# if creating one csv per relation/table in schema, i.e: "metadata/{{relation}}.csv"
```

```console
--no-prompt
# Silently generate metadata
```

### Metadata

dbt-coves supports the argument `--metadata` which allows users to specify a csv file containing field types and descriptions to be used when creating the staging models and property files.

```console
dbt-coves generate sources --metadata metadata.csv
```

Metadata format:
You can download a [sample csv file](sample_metadata.csv) as reference

| database | schema | relation                          | column          | key  | type    | description                                     |
| -------- | ------ | --------------------------------- | --------------- | ---- | ------- | ----------------------------------------------- |
| raw      | raw    | \_airbyte_raw_country_populations | \_airbyte_data  | Year | integer | Year of country population measurement          |
| raw      | raw    | \_airbyte_raw_country_populations | \_airbyte_data  |      | variant | Airbyte data columns (VARIANT) in Snowflake     |
| raw      | raw    | \_airbyte_raw_country_populations | \_airbyte_ab_id |      | varchar | Airbyte unique identifier used during data load |

## Extract configuration from Airbyte

```console
dbt-coves extract airbyte
```

Extracts the configuration from your Airbyte sources, connections and destinations (excluding credentials) and stores it in the specified folder. The main goal of this feature is to keep track of the configuration changes in your git repo, and rollback to a specific version when needed.

Full usage example:

```console
dbt-coves extract airbyte --host http://airbyte-server --port 8001 --path /config/workspace/load/airbyte
```

## Load configuration to Airbyte

```console
dbt-coves load airbyte
```

Loads the Airbyte configuration generated with `dbt-coves extract airbyte` on an Airbyte server. Secrets folder needs to be specified separately. You can use [git-secret](https://git-secret.io/) to encrypt secrets and make them part of your git repo.

### Loading secrets

Secret credentials can be approached in two different ways: locally or remotely (through a provider/manager).

In order to load encrypted fields locally:

```console
dbt-coves load airbyte --secrets-path /path/to/secret/directory

# This directory must have 'sources', 'destinations' and 'connections' folders nested inside, and inside them the respective JSON files with unencrypted fields.
# Naming convention: JSON unencrypted secret files must be named exactly as the extracted ones.
```

To load encrypted fields through a manager (in this case we are connecting to Datacoves' Service Credentials):

```console
--secrets-manager datacoves
```

```console
--secrets-url https://api.datacoves.localhost/service-credentials/airbyte
```

```console
--secrets-token <secret token>
```

Full usage example:

```console
dbt-coves load airbyte --host http://airbyte-server --port 8001 --path /config/workspace/load/airbyte --secrets-path /config/workspace/secrets
```

## Extract configuration from Fivetran

```console
dbt-coves extract fivetran
```

Extracts the configuration from your Fivetran destinations and connectors (excluding credentials) and stores it in the specified folder. The main goal of this feature is to keep track of the configuration changes in your git repo, and rollback to a specific version when needed.

Full usage example:

```console
dbt-coves extract fivetran --credentials /config/workspace/secrets/fivetran/credentials.yml --path /config/workspace/load/fivetran
```

## Load configuration to Fivetran

```console
dbt-coves load fivetran
```

Loads the Fivetran configuration generated with `dbt-coves extract fivetran` on a Fivetran instance. Secrets folder needs to be specified separately. You can use [git-secret](https://git-secret.io/) to encrypt secrets and make them part of your git repo.

### Credentials

In order for extract/load fivetran to work properly, you need to provide an api key-secret pair (you can generate them [here](https://fivetran.com/account/settings/account)).

These credentials can be used with `--api-key [key] --api-secret [secret]`, or specyfing a YML file with `--credentials /path/to/credentials.yml`. The required structure of this file is the following:

```yaml
account_name: # Any name, used by dbt-coves to ask which to use when more than one is found
  api_key: [key]
  api_secret: [secret]
account_name_2:
  api_key: [key]
  api_secret: [secret]
```

This YML file approach allows you to both work with multiple Fivetran accounts, and treat this credentials file as a secret.

> :warning: **Warning**: --api-key/secret and --credentials flags are mutually exclusive, don't use them together.

### Loading secrets

Secret credentials can be approached via `--secrets-path` flag

```console
dbt-coves load fivetran --secrets-path /path/to/secret/directory
```

#### Field naming convention

Although secret files can have any name, unencrypted JSON files must follow a simple structure:

- Keys should match their corresponding Fivetran destination ID: two words automatically generated by Fivetran, which can be found in previously extracted data.
- Inside those keys, a nested dictionary of which fields should be overwritten

For example:

```json
{
  "extract_muscle": {
    // Internal ID that Fivetran gave to a Snowflake warehouse Destination
    "password": "[PASSWORD]" // Field:Value pair
  },
  "centre_straighten": {
    "password": "[PASSWORD]"
  }
}
```

## Run dbt commands

```shell
dbt-coves dbt <arguments> -- <command>
```

Run dbt commands on special environments such as Airflow, or CI workers, with the possibility of changing dbt project location and activating a specific virtual environment in which running commands.

### Arguments

`dbt-coves dbt` supports the following arguments

```shell
--project-dir
# Path of the dbt project where command will be executed, i.e.: /opt/user/dbt_project
```

```shell
--virtualenv
# Virtual environment path. i.e.: /opt/user/virtualenvs/airflow
```

### Sample usage

```shell
dbt-coves dbt --project-dir /opt/user/dbt_project --virtualenv /opt/user/virtualenvs/airflow -- run -s model --vars \"{key: value}\"
# Make sure to escape special characters such as quotation marks
# Double dash (--) between <arguments> and <command> are mandatory
```

# Settings

dbt-coves will read settings from `.dbt_coves/config.yml`. A standard settings files could look like this:

```yaml
generate:
  sources:
    database: "RAW" # Database where to look for source tables
    schemas: # List of schema names where to look for source tables
      - RAW
    select_relations: # list of relations where raw data resides
      - TABLE_1
      - TABLE_2
    exclude_relations: # Filter relation(s) to exclude from source file(s) generation
      - TABLE_1
      - TABLE_2
    sources_destination: "models/staging/{{schema}}/{{schema}}.yml" # Where sources yml files will be generated
    models_destination: "models/staging/{{schema}}/{{relation}}.sql" # Where models sql files will be generated
    model_props_destination: "models/staging/{{schema}}/{{relation}}.yml" # Where models yml files will be generated
    update_strategy: ask # Action to perform when a property file already exists. Options: update, recreate, fail, ask (per file)
    templates_folder: ".dbt_coves/templates" # Folder where source generation jinja templates are located. Override default templates creating  source_props.yml, source_model_props.yml, and source_model.sql under this folder
    metadata: "metadata.csv" # Path to csv file containing metadata

  properties:
    destination: "{{model_folder_path}}/{{model_file_name}}.yml" # Where models yml files will be generated
    # You can specify a different path by declaring it explicitly, i.e.: "models/staging/{{model_file_name}}.yml"
    update-strategy: ask # Action to perform when a property file already exists. Options: update, recreate, fail, ask (per file)
    select: "models/staging/bays" # Filter model(s) to generate property file(s)
    exclude: "models/staging/bays/test_bay" # Filter model(s) to generate property file(s)
    selector: "selectors/bay_selector.yml" # Specify dbt selector for more complex model filtering
    templates_folder: ".dbt_coves/templates" # Folder where source generation jinja templates are located. Override default template creating model_props.yml under this folder
    metadata: "metadata.csv" # Path to csv file containing metadata

  metadata:
    database: RAW # Database where to look for source tables
    schemas: # List of schema names where to look for source tables
      - RAW
    select_relations: # list of relations where raw data resides
      - TABLE_1
      - TABLE_2
    exclude_relations: # Filter relation(s) to exclude from source file(s) generation
      - TABLE_1
      - TABLE_2
    destination: # Where metadata file will be generated, default: 'metadata.csv'

extract:
  airbyte:
    path: /config/workspace/load/airbyte # Where json files will be generated
    host: http://airbyte-server # Airbyte's API hostname
    port: 8001 # Airbyte's API port
  fivetran:
    path: /config/workspace/load/fivetran # Where Fivetran export will be generated
    api_key: [KEY] # Fivetran API Key
    api_secret: [SECRET] # Fivetran API Secret
    credentials: /opt/fivetran_credentials.yml # Fivetran set of key:secret pairs
    # 'api_key' + 'api_secret' are mutually exclusive with 'credentials', use one or the other

load:
  airbyte:
    path: /config/workspace/load
    host: http://airbyte-server
    port: 8001
    secrets_manager: datacoves # (optional) Secret credentials provider (secrets_path OR secrets_manager should be used, can't load secrets locally and remotely at the same time)
    secrets_path: /config/workspace/secrets # (optional) Secret files location if secrets_manager was not specified
    secrets_url: https://api.datacoves.localhost/service-credentials/airbyte # Secrets url if secrets_manager is datacoves
    secrets_token: <TOKEN> # Secrets auth token if secrets_manager is datacoves
  fivetran:
    path: /config/workspace/load/fivetran # Where previous Fivetran export resides, subject of import
    api_key: [KEY] # Fivetran API Key
    api_secret: [SECRET] # Fivetran API Secret
    secrets_path: /config/workspace/secrets/fivetran # Fivetran secret fields
    credentials: /opt/fivetran_credentials.yml # Fivetran set of key:secret pairs
    # 'api_key' + 'api_secret' are mutually exclusive with 'credentials', use one or the other
```

## Telemetry

dbt-coves has telemetry built in to help the maintainers from Datacoves understand which commands are being used and which are not to prioritize future development of dbt-coves. We do not track credentials nor details of your dbt execution such as model names. The one detail we do use related to dbt is the anonymous user_id to help us identify distinct users.

By default this is turned on – you can opt out of event tracking at any time by adding the following to your dbt-coves `config.yaml` file:

```yaml
disable-tracking: true
```

## Override generation templates

Customizing generated models and model properties requires placing
template files under the `.dbt-coves/templates` folder.

There are different variables available in the templates:

- `adapter_name` refers to the Adapter's class name being used by the target, e.g. `SnowflakeAdapter` when using [Snowflake](https://github.com/dbt-labs/dbt-snowflake/blob/21b52127e7d221db8b92114aae066fb8a7151bba/dbt/adapters/snowflake/impl.py#L33).
- `columns` contains the list of relation columns that don't contain nested (JSON) data, it's type is `List[Item]`.
- `nested` contains a dict of nested columns, grouped by column name, it's type is `Dict[column_name, Dict[nested_key, Item]]`.

`Item` is a `dict` with the keys `id`, `name`, `type`, and `description`, where `id` contains an slugified id generated from `name`.

### dbt-coves generate sources

#### Source property file (.yml) template

This file is used to create the sources yml file

[source_props.yml](dbt_coves/templates/source_props.yml)

#### Staging model file (.sql) template

This file is used to create the staging model (sql) files.

[staging_model.sql](dbt_coves/templates/staging_model.sql)

#### Staging model property file (.yml) template

This file is used to create the model properties (yml) file

[staging_model_props.yml](dbt_coves/templates/staging_model_props.yml)

### dbt-coves generate properties

This file is used to create the properties (yml) files for models

[model_props.yml](dbt_coves/templates/model_props.yml)

# Thanks

The project main structure was inspired by [dbt-sugar](https://github.com/bitpicky/dbt-sugar). Special thanks to [Bastien Boutonnet](https://github.com/bastienboutonnet) for the great work done.

# Authors

- Sebastian Sassi [\@sebasuy](https://twitter.com/sebasuy) -- [Datacoves](https://datacoves.com/)
- Noel Gomez [\@noel_g](https://twitter.com/noel_g) -- [Datacoves](https://datacoves.com/)
- Bruno Antonellini -- [Datacoves](https://datacoves.com/)

# About

Learn more about [Datacoves](https://datacoves.com).

⚠️ **dbt-coves is still in development, make sure to test it for your dbt project version and DW before using in production and please submit any issues you find. We also welcome any contributions from the community**

# Metrics

[![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)](https://github.com/datacoves/dbt-coves/graphs/commit-activity)
[![PyPI version
fury.io](https://badge.fury.io/py/dbt-coves.svg)](https://pypi.python.org/pypi/dbt-coves/)
[![Code
Style](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/ambv/black)
[![Imports:
isort](https://img.shields.io/badge/%20imports-isort-%231674b1?style=flat&labelColor=ef8336)](https://pycqa.github.io/isort/)
[![Imports:
python](https://img.shields.io/badge/python-3.8%20%7C%203.9-blue)](https://img.shields.io/badge/python-3.8%20%7C%203.9-blue)
[![Build](https://github.com/datacoves/dbt-coves/actions/workflows/main_ci.yml/badge.svg)](https://github.com/datacoves/dbt-coves/actions/workflows/main_ci.yml/badge.svg)

<!-- [![codecov](https://codecov.io/gh/datacoves/dbt-coves/branch/main/graph/badge.svg?token=JB0E0LZDW1)](https://codecov.io/gh/datacoves/dbt-coves) -->

[![Maintainability](https://api.codeclimate.com/v1/badges/1e6a887de605ef8e0eca/maintainability)](https://codeclimate.com/github/datacoves/dbt-coves/maintainability)
[![Downloads](https://pepy.tech/badge/dbt-coves)](https://pepy.tech/project/dbt-coves)
