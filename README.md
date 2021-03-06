# Snowflake Utils

This [dbt](https://github.com/fishtown-analytics/dbt) package contains Snowflake-specific macros that can be (re)used across dbt projects.

## Installation Instructions
Check [dbt Hub](https://hub.getdbt.com/fishtown-analytics/snowflake_utils/latest/) for the latest installation instructions, or [read the docs](https://docs.getdbt.com/docs/package-management) for more information on installing packages.

## Prerequisites
Snowflake Utils is compatible with dbt 0.15.0 and later.

----

## Macros

### snowflake_utils.warehouse_size() ([source](macros/warehouse_size.sql))
This macro returns an alternative warehouse if conditions are met. It will, in order, check the following conditions for incremental models:

- The relation doesn't exist (initial run) _and_ a warehouse has been configured
- Full refresh run _and_ a warehouse has been configured

Otherwise, it returns the target warehouse configured in the profile.

#### Usage

Call the macro from the `snowflake_warehouse` model configuration:
```
{{ 
    config(
      snowflake_warehouse=snowflake_utils.warehouse_size()
    )
}}
```

#### Macro Configuration

Out-of-the-box, the macro will return the `target.warehouse` for each condition, unless exceptions are configured using one or more of the following [variables](https://docs.getdbt.com/docs/using-variables):

| variable | information | required |
|----------|-------------|:--------:|
|snowflake_utils:initial_run_warehouse|Alternative warehouse when the relation doesn't exist|No|
|snowflake_utils:full_refresh_run_warehouse|Alternative warehouse when doing a `--full-refresh`|No|

An example `dbt_project.yml` configuration:

```yml
# dbt_project.yml

...

models:
    my_project:
        vars:
            'snowflake_utils:initial_run_warehouse': "transforming_xl_wh"
            'snowflake_utils:full_refresh_run_warehouse': "transforming_xl_wh"


```

#### Console Output

When a variable is configured for a conditon _and_ that condition is matched when executing a run, a log message will confirm which condition was matched and which warehouse is being used.

```
12:00:00 | Concurrency: 16 threads (target='dev')
12:00:00 | 
12:00:00 | 1 of 1 START incremental model DBT_MGUINDON.fct_orders... [RUN]
12:00:00 + Initial Run - Using warehouse TRANSFORMING_XL_WH
```

### snowflake_utils.clone_schema ([source](macros/clone_schema.sql))
This macro clones the source schema into the destination schema.

#### Arguments
* `source_schema` (required): The source schema name
* `destination_schema` (required): The destination schema name

#### Usage

Call the macro as an [operation](https://docs.getdbt.com/docs/using-operations):

```
# for multiple arguments, use the dict syntax
dbt run-operation clone_schema --args "{'source_schema': 'analytics', 'destination_schema': 'ci_schema'}"
```
----
## Models

### snowflake_utils.snowflake_query_history
This model permanently stores an audit trail from your Snowflake QUERY_HISTORY in your environment.

#### Usage

To use the `snowflake_query_history` model, you need to enable it in your `dbt_project.yml` file, along with some base variables.


An example `dbt_project.yml` configuration:

```yml
# dbt_project.yml

...

models:
  snowflake:
    enabled: true
    vars:
      'snowflake_utils:minutes_per_batch' : 30 
      ## Time period to request at once. If over 10,000 queries were run in the time period, Snowflake will return the last 10,000. #}
      'snowflake_utils:max_load_minutes': 4320
      ## the number of minutes in 3 days - the maximum time to run for.
      'snowflake_utils:first_run': false
      ## this must be set to `true` on first run to properly set up the incremental model.

```

## Contributions
Contributions to this package are very welcome! Please create issues for bugs or feature requests, or open PRs against `master`.
