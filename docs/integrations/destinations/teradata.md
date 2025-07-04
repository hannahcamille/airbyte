# Teradata

This page guides you through the process of setting up the Teradata destination connector.

## Prerequisites

To use the Teradata destination connector, you'll need:

- Access to a Teradata Vantage instance

  **Note:** If you need a new instance of Vantage, you can install a free version called Vantage Express in the cloud on [Google Cloud](https://quickstarts.teradata.com/vantage.express.gcp.html), [Azure](https://quickstarts.teradata.com/run-vantage-express-on-microsoft-azure.html), and [AWS](https://quickstarts.teradata.com/run-vantage-express-on-aws.html). You can also run Vantage Express on your local machine using [VMware](https://quickstarts.teradata.com/getting.started.vmware.html), [VirtualBox](https://quickstarts.teradata.com/getting.started.vbox.html), or [UTM](https://quickstarts.teradata.com/getting.started.utm.html).

You'll need the following information to configure the Teradata destination:

- **Host** - The host name of the Teradata Vantage instance.
- **Authorization Mechanism** - Specifies the Logon Mechanism, which determines the connection's authentication and encryption capabilities. The default value is `TD2`. This connector supports TD2, LDAP and BROWSER authentication mechanisms.
- **User** - The username to use to connect to the Teradata Vantage instance. The user must have the necessary permissions to create tables and write data. 
- **Password** - The password to use to connect to the Teradata Vantage instance.
- **SSL modes** - Specifies the mode for connections to the database. Refer to the [Teradata JDBC documentation](https://teradata-docs.s3.amazonaws.com/doc/connectivity/jdbc/reference/current/jdbcug_chapter_2.html#URL_SSLMODE) for more information. Enable SSL Connection option to use SSL mode.
- **Default Schema Name** - Specify the schema (or several schemas separated by commas) to be set in the search-path. These schemas will be used to resolve unqualified object names used in statements executed over this connection.
- **JDBC URL Params** (optional)
- **Query Band** (optional) - The [query band ](https://teradata-docs.s3.amazonaws.com/doc/connectivity/jdbc/reference/current/jdbcug_chapter_2.html#BGEGBBAA)is a set of name-value pairs that can be assigned to a Teradata database session. It helps identify the source of SQL requests originating from Airbyte. You can customize the query band to include relevant information such as application name, organization, and user identifiers.
  Each entry should be formatted as key=value, separated by semicolons (;). 
  Example: `appname=myApp;org=myOrganization;`

[Refer to this guide for more details](https://downloads.teradata.com/doc/connectivity/jdbc/reference/current/jdbcug_chapter_2.html#BGBHDDGB)

## Sync overview

### Output schema (Raw Tables)

Each stream will be mapped to a separate raw table in Teradata. The default schema in which the raw
tables are created is `airbyte_internal`. This can be overridden in the configuration. Each table
will contain below columns:

- `_airbyte_raw_id`: a unique uuid assigned by Airbyte to each event that is processed. This is the primary index column. The column type in Teradata is `VARCHAR(256)`.
- `_airbyte_extracted_at`: a timestamp representing when the event was pulled from the data source. The column type in Teradata is `TIMESTAMP WITH TIME ZONE`.
- `_airbyte_loaded_at`: a timestamp representing when the row was processed into final table. The column type in Teradata is `TIMESTAMP WITH TIME ZONE`.
- `_airbyte_data`: a json blob representing with the event data. The column type in Teradata is `JSON`.
- `_airbyte_meta`: a json blob representing per-row error/change handling. The column type in Teradata is `JSON`.
- `_airbyte_generation_id`: This is one of metadata field and incremented each time you execute a [refresh](https://docs.airbyte.com/operator-guides/refreshes). The column type in Teradata is `BIGINT`.

[Refer to this guide for more details](https://docs.airbyte.com/understanding-airbyte/airbyte-metadata-fields)

### Final Tables Data type mapping

| Airbyte Type               | Teradata Type            |
| :------------------------- |:-------------------------|
| string                     | VARCHAR                  |
| number                     | FLOAT                    |
| integer                    | BIGINT                   |
| boolean                    | SMALLINT                  |
| object                     | JSON                     |
| array                      | JSON                     |
| timestamp_with_timezone    | TIMESTAMP WITH TIME ZONE |
| timestamp_without_timezone | TIMESTAMP                |
| time_with_timezone         | TIME WITH TIME ZONE      |
| time_without_timezone      | TIME                     |
| date                       | DATE                     |

### Features

The Teradata destination connector supports the
following[ sync modes](https://docs.airbyte.com/cloud/core-concepts#connection-sync-modes):

| Feature                        | Supported?\(Yes/No\) | Notes |
| :----------------------------- |:---------------------| :---- |
| Full Refresh Sync              | Yes                  |       |
| Incremental - Append Sync      | Yes                  |       |
| Incremental - Append + Deduped | Yes                  |       |
| Namespaces                     | Yes                  |       |


## Schema map



### Performance considerations

## Getting started

### Requirements

You need a Teradata user with the following permissions:

- can create tables and write permission.
- can create schemas e.g:

You can create such a user by running:

```
CREATE USER airbyte_user  as perm=10e6, PASSWORD=<password>;
GRANT ALL on dbc to airbyte_user;

```

You can also use a pre-existing user but we highly recommend creating a dedicated user for Airbyte.

### Setup guide

#### Set up the Teradata Destination connector

1. Log into your Airbyte Open Source account.
2. Click **Destinations** and then click **+ New destination**.
3. On the Set up the destination page, select **Teradata** from the **Destination type** dropdown.
4. Enter the **Name** for the Teradata connector.
5. For **Host**, enter the host domain of the Teradata instance
6. For **Default Schema**, enter the Default Schema name. The default value is public.
7. For **User** and **Password**, enter the database username and password.
8. To customize the JDBC connection beyond common options, specify additional supported [JDBC URL parameters](https://downloads.teradata.com/doc/connectivity/jdbc/reference/current/jdbcug_chapter_2.html#BGBHDDGB) as key-value pairs separated by the symbol & in the **JDBC URL Params** field.
   Example: key1=value1&key2=value2&key3=value3

   These parameters will be added at the end of the JDBC URL that the AirByte will use to connect to your Teradata database.
9. To customize the [query band](https://teradata-docs.s3.amazonaws.com/doc/connectivity/jdbc/reference/current/jdbcug_chapter_2.html#BGEGBBAA), specify set of name-value pairs in the **Query Band** field that can be set to the current database session.

   

## Changelog

<details>
  <summary>Expand to review</summary>

| Version | Date       | Pull Request                                     | Subject                                                  |
|:--------|:-----------|:-------------------------------------------------|:---------------------------------------------------------|
| 1.0.0   | 2025-04-03 | https://github.com/airbytehq/airbyte/pull/56985  | Upgrade to DV2 output format                             |
| 0.2.1   | 2025-03-26 | https://github.com/airbytehq/airbyte/pull/56414  | Migrated unit and integration tests to Kotlin            |
| 0.2.0   | 2025-03-24 | https://github.com/airbytehq/airbyte/pull/56362  | Added LDAP and SSO authentication mechanism              |
| 0.1.9   | 2025-03-17 | https://github.com/airbytehq/airbyte/pull/55800  | Added Query Band Support                                 |
| 0.1.8   | 2025-03-14 | https://github.com/airbytehq/airbyte/pull/55771  | Migration to Kotlin                                      |
| 0.1.7   | 2025-03-07 | https://github.com/airbytehq/airbyte/pull/55183  | Upgrade teradata jdbc driver to 20.00.00.43              |
| 0.1.6   | 2024-06-24 | https://github.com/airbytehq/airbyte/pull/39455  | Fix for Parameter 2 length size                          |
| 0.1.5   | 2024-01-12 | https://github.com/airbytehq/airbyte/pull/33872  | Added Primary Index on \_airbyte_ab_id to fix NoPI issue |
| 0.1.4   | 2023-12-04 | https://github.com/airbytehq/airbyte/pull/28667  | Make connector available on Airbyte Cloud                |
| 0.1.3   | 2023-08-17 | https://github.com/airbytehq/airbyte/pull/30740  | Enable custom DBT transformation                         |
| 0.1.2   | 2023-08-09 | https://github.com/airbytehq/airbyte/pull/29174  | Small internal refactor                                  |
| 0.1.1   | 2023-03-03 | https://github.com/airbytehq/airbyte/pull/21760  | Added SSL support                                        |
| 0.1.0   | 2022-12-13 | https://github.com/airbytehq/airbyte/pull/20428  | New Destination Teradata Vantage                         |

</details>
