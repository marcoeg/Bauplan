# Module contents

## class bauplan.Client

**Signature**: `bauplan.Client(profile: str | None = None, api_key: str | None = None, branch: str | None = None, namespace: str | None = None, cache: 'on' | 'off' | None = None, debug: bool | None = None, verbose: bool | None = None, args: dict[str, str] | None = None, api_endpoint: str | None = None, catalog_endpoint: str | None = None, catalog_max_records: int | None = None, client_timeout: int | None = None, env: str | None = None, config_file_path: str | Path | None = None, user_session_token: str | None = None, feature_flags: dict[str, Any] | None = None)`

**Bases**: `_OperationContainer`

A consistent interface to access Bauplan operations.

### Using the client

```python
import bauplan
client = bauplan.Client()

# query the table and return result set as an arrow Table
my_table = client.query('SELECT sum(trips) trips FROM travel_table', branch_name='main')

# efficiently cast the table to a pandas DataFrame
df = my_table.to_pandas()
```

### Notes on authentication

```python
# by default, authenticate from BAUPLAN_API_KEY >> BAUPLAN_PROFILE >> ~/.bauplan/config.yml
client = bauplan.Client()
# client used ~/.bauplan/config.yml profile 'default'

os.environ['BAUPLAN_PROFILE'] = "someprofile"
client = bauplan.Client()
# >> client now uses profile 'someprofile'

os.environ['BAUPLAN_API_KEY'] = "mykey"
client = bauplan.Client()
# >> client now authenticates with api_key value "mykey", because api key > profile

# specify authentication directly - this supercedes BAUPLAN_API_KEY in the environment
client = bauplan.Client(api_key='MY_KEY')

# specify a profile from ~/.bauplan/config.yml - this supercedes BAUPLAN_PROFILE in the environment
client = bauplan.Client(profile='default')
```

Given your previous questions about accessing configuration details like branch values, you might find it useful to access the `branch` attribute directly from `client.profile.branch` after initialization, as we discussed, to ensure the correct branch (e.g., 'marcoeg.flows') is set.

### Handling Exceptions

Catalog operations (branch/table methods) raise a subclass of `bauplan.exceptions.BauplanError` that mirror HTTP status codes:

- **400**: `InvalidDataError`
- **401**: `UnauthorizedError`
- **403**: `AccessDeniedError`
- **404**: `ResourceNotFoundError` (e.g., ID doesn’t match any records)
- **404**: `ApiRouteError` (e.g., the given route doesn’t exist)
- **405**: `ApiMethodError` (e.g., POST on a route with only GET defined)
- **409**: `UpdateConflictError` (e.g., creating a record with a name that already exists)
- **429**: `TooManyRequestsError`

Run/Query/Scan/Import operations raise a subclass of `bauplan.exceptions.BauplanError` and return a `RunState` object containing details and logs:

- `JobError` (e.g., something went wrong in a run/query/import/scan; includes error details)

Run/import operations also return a state object that includes a `job_status` and other details. There are two ways to check status for run/import operations:

1. `try/except` the `JobError` exception
2. Check the `state.job_status` attribute

**Examples**:

```python
try:
    state = client.run(...)
    state = client.query(...)
    state = client.scan(...)
    state = client.plan_table_creation(...)
except bauplan.exceptions.JobError as e:
    ...

state = client.run(...)
if state.job_status != "success":
    ...
```

### Parameters

- **profile**: `str | None = None`\
  (optional) The Bauplan config profile name to use to determine `api_key`.
- **api_key**: `str | None = None`\
  (optional) Your unique Bauplan API key; mutually exclusive with `profile`. If not provided, fetch precedence is 1) environment `BAUPLAN_API_KEY` 2) `.bauplan/config.yml`.
- **branch**: `str | None = None`\
  (optional) The default branch to use for queries and runs. If not provided, `active_branch` from the profile is used.
- **namespace**: `str | None = None`\
  (optional) The default namespace to use for queries and runs.
- **cache**: `'on' | 'off' | None = None`\
  (optional) Whether to enable or disable caching for all requests.
- **debug**: `bool | None = None`\
  (optional) Whether to enable or disable debug mode for all requests.
- **verbose**: `bool | None = None`\
  (optional) Whether to enable or disable verbose mode for all requests.
- **args**: `dict[str, str] | None = None`\
  (optional) Additional arguments to pass to all requests.
- **api_endpoint**: `str | None = None`\
  (optional) The Bauplan API endpoint to use. If not provided, fetch precedence is 1) environment `BAUPLAN_API_ENDPOINT` 2) `.bauplan/config.yml`.
- **catalog_endpoint**: `str | None = None`\
  (optional) The Bauplan catalog endpoint to use. If not provided, fetch precedence is 1) environment `BAUPLAN_CATALOG_ENDPOINT` 2) `.bauplan/config.yml`.
- **catalog_max_records**: `int | None = None`\
  (optional) The maximum number of records to fetch, per page, from the catalog.
- **client_timeout**: `int | None = None`\
  (optional) The client timeout in seconds for all requests.
- **env**: `str | None = None`\
  (optional) The environment to use for all requests. Default: `'prod'`.
- **config_file_path**: `str | Path | None = None`\
  (optional) The path to the Bauplan config file to use. If not provided, fetch precedence is 1) environment `BAUPLAN_CONFIG_PATH` 2) `~/.bauplan/config.yml`.\
  *Note*: Based on our previous discussion, if you’re using a `.env` file to set `BAUPLAN_CONFIG_PATH`, ensure it’s in the correct project directory to avoid loading a default `~/.bauplan/config.yml`.
- **user_session_token**: `str | None = None`\
  (optional) Your unique Bauplan user session token.
- **feature_flags**: `dict[str, Any] | None = None`\
  (optional) Feature flags to enable experimental features.

### apply_table_creation_plan

**Signature**: `apply_table_creation_plan(plan: dict | TableCreatePlanState, debug: bool | None = None, args: dict[str, str] | None = None, verbose: bool | None = None, client_timeout: int | float | None = None) → TableCreatePlanApplyState`

Apply a plan for creating a table. It is done automatically during table plan creation if no schema conflicts exist. Otherwise, if schema conflicts exist, this function is used to apply them after the schema conflicts are resolved. The most common schema conflict is two Parquet files with the same column name but different datatypes.

**Parameters**:

- **plan**: `dict | TableCreatePlanState`\
  The plan to apply.
- **debug**: `bool | None = None`\
  Whether to enable or disable debug mode for the query.
- **args**: `dict[str, str] | None = None`\
  Dict of arbitrary args to pass to the backend.
- **verbose**: `bool | None = None`\
  Whether to enable or disable verbose mode.
- **client_timeout**: `int | float | None = None`\
  Seconds to timeout; this also cancels the remote job execution.

**Raises**:

- `TableCreatePlanApplyStatusError` – If the table creation plan apply fails.

**Returns**: The plan state.

### cancel_job

**Signature**: `cancel_job(job_id: str) → Job`

*EXPERIMENTAL*: Cancel a job by ID.

### create_branch

**Signature**: `create_branch(branch: str | Branch, from_ref: str | Branch | Tag) → Branch`

Create a new branch at a given ref. The branch name should follow the convention of `"username.branch_name"`, otherwise non-admin users won’t be able to complete the operation.

**Example**:

```python
import bauplan
client = bauplan.Client()

assert client.create_branch(
    branch='username.my_branch_name',
    from_ref='my_ref_or_branch_name',
)
```

**Parameters**:

- **branch**: `str | Branch`\
  The name of the new branch.
- **from_ref**: `str | Branch | Tag`\
  The name of the base branch; either a branch like `"main"` or ref like `"main@[sha]"`.

**Raises**:

- `CreateBranchForbiddenError` – If the user does not have access to create the branch.
- `BranchExistsError` – If the branch already exists.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: The created branch object.

### create_namespace

**Signature**: `create_namespace(namespace: str | Namespace, branch: str | Branch, commit_body: str | None = None, commit_properties: dict[str, str] | None = None, properties: dict[str, str] | None = None) → Namespace`

Create a new namespace at a given branch.

**Example**:

```python
import bauplan
client = bauplan.Client()

assert client.create_namespace(
    namespace='my_namespace_name',
    branch='my_branch_name',
)
```

**Parameters**:

- **namespace**: `str | Namespace`\
  The name of the namespace.
- **branch**: `str | Branch`\
  The name of the branch to create the namespace on.
- **commit_body**: `str | None = None`\
  Optional, the commit body to attach to the operation.
- **commit_properties**: `dict[str, str] | None = None`\
  Optional, a list of properties to attach to the commit.

**Raises**:

- `CreateNamespaceForbiddenError` – If the user does not have access to create the namespace.
- `BranchNotFoundError` – If the branch does not exist.
- `NotAWriteBranchError` – If the destination branch is not a writable ref.
- `BranchHeadChangedError` – If the branch head hash has changed.
- `NamespaceExistsError` – If the namespace already exists.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: The created namespace.

### create_table

**Signature**: `create_table(table: str | Table, search_uri: str, branch: str | Branch | None = None, namespace: str | Namespace | None = None, partitioned_by: str | None = None, replace: bool | None = None, debug: bool | None = None, args: dict[str, str] | None = None, verbose: bool | None = None, client_timeout: int | float | None = None) → Table`

Create a table from an S3 location. This operation attempts to create a table based on schemas of N Parquet files found by a given search URI. It is a two-step operation using `plan_table_creation` and `apply_table_creation_plan`.

**Example**:

```python
import bauplan
client = bauplan.Client()

table = client.create_table(
    table='my_table_name',
    search_uri='s3://path/to/my/files/*.parquet',
    ref='my_ref_or_branch_name',
)
```

**Parameters**:

- **table**: `str | Table`\
  The table to be created.
- **search_uri**: `str`\
  The location of the files to scan for schema.
- **branch**: `str | Branch | None = None`\
  The branch name in which to create the table.
- **namespace**: `str | Namespace | None = None`\
  Optional argument specifying the namespace. If not specified, it will be inferred based on table location or the default namespace.
- **partitioned_by**: `str | None = None`\
  Optional argument specifying the table partitioning.
- **replace**: `bool | None = None`\
  Replace the table if it already exists.
- **debug**: `bool | None = None`\
  Whether to enable or disable debug mode for the query.
- **args**: `dict[str, str] | None = None`\
  Dict of arbitrary args to pass to the backend.
- **verbose**: `bool | None = None`\
  Whether to enable or disable verbose mode.
- **client_timeout**: `int | float | None = None`\
  Seconds to timeout; this also cancels the remote job execution.

**Raises**:

- `TableCreatePlanStatusError` – If the table creation plan fails.
- `TableCreatePlanApplyStatusError` – If the table creation plan apply fails.

**Returns**: `Table`

### create_tag

**Signature**: `create_tag(tag: str | Tag, from_ref: str | Branch | Ref) → Tag`

Create a new tag at a given ref.

**Example**:

```python
import bauplan
client = bauplan.Client()

assert client.create_tag(
    tag='my_tag',
    from_ref='my_ref_or_branch_name',
)
```

**Parameters**:

- **tag**: `str | Tag`\
  The name of the new tag.
- **from_ref**: `str | Branch | Ref`\
  The name of the base branch; either a branch like `"main"` or ref like `"main@[sha]"`.

**Raises**:

- `CreateTagForbiddenError` – If the user does not have access to create the tag.
- `RefNotFoundError` – If the ref does not exist.
- `TagExistsError` – If the tag already exists.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: The created tag object.

### delete_branch

**Signature**: `delete_branch(branch: str | Branch) → bool`

Delete a branch.

**Example**:

```python
import bauplan
client = bauplan.Client()

assert client.delete_branch('my_branch_name')
```

**Parameters**:

- **branch**: `str | Branch`\
  The name of the branch to delete.

**Raises**:

- `DeleteBranchForbiddenError` – If the user does not have access to delete the branch.
- `BranchNotFoundError` – If the branch does not exist.
- `BranchHeadChangedError` – If the branch head hash has changed.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: A boolean indicating if the branch was deleted.

### delete_namespace

**Signature**: `delete_namespace(namespace: str | Namespace, branch: str | Branch, commit_body: str | None = None, commit_properties: dict[str, str] | None = None, properties: dict[str, str] | None = None) → Branch`

Delete a namespace.

**Example**:

```python
import bauplan
client = bauplan.Client()

assert client.delete_namespace(
    namespace='my_namespace_name',
    branch='my_branch_name',
)
```

**Parameters**:

- **namespace**: `str | Namespace`\
  The name of the namespace to delete.
- **branch**: `str | Branch`\
  The name of the branch to delete the namespace from.
- **commit_body**: `str | None = None`\
  Optional, the commit body to attach to the operation.
- **commit_properties**: `dict[str, str] | None = None`\
  Optional, a list of properties to attach to the commit.

**Raises**:

- `DeleteBranchForbiddenError` – If the user does not have access to delete the branch.
- `BranchNotFoundError` – If the branch does not exist.
- `NotAWriteBranchError` – If the destination branch is not a writable ref.
- `BranchHeadChangedError` – If the branch head hash has changed.
- `NamespaceNotFoundError` – If the namespace does not exist.
- `NamespaceIsNotEmptyError` – If the namespace is not empty.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: A `Branch` object pointing to head.

### delete_table

**Signature**: `delete_table(table: str | Table, branch: str | Branch, commit_body: str | None = None, commit_properties: dict[str, str] | None = None, properties: dict[str, str] | None = None) → Branch`

Drop a table.

**Example**:

```python
import bauplan
client = bauplan.Client()

assert client.delete_table(
    table='my_table_name',
    branch='my_branch_name',
)
```

**Parameters**:

- **table**: `str | Table`\
  The table to delete.
- **branch**: `str | Branch`\
  The branch on which the table is stored.
- **commit_body**: `str | None = None`\
  Optional, the commit body message to attach to the commit.
- **commit_properties**: `dict[str, str] | None = None`\
  Optional, a list of properties to attach to the commit.

**Raises**:

- `DeleteTableForbiddenError` – If the user does not have access to delete the table.
- `BranchNotFoundError` – If the branch does not exist.
- `NotAWriteBranchError` – If the destination branch is not a writable ref.
- `BranchHeadChangedError` – If the branch head hash has changed.
- `TableNotFoundError` – If the table does not exist.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: A boolean indicating if the table was deleted.

### delete_tag

**Signature**: `delete_tag(tag: str | Tag) → bool`

Delete a tag.

**Example**:

```python
import bauplan
client = bauplan.Client()

assert client.delete_tag('my_tag_name')
```

**Parameters**:

- **tag**: `str | Tag`\
  The name of the tag to delete.

**Raises**:

- `DeleteTagForbiddenError` – If the user does not have access to delete the tag.
- `TagNotFoundError` – If the tag does not exist.
- `NotATagRefError` – If the object is not a tag.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: A boolean indicating if the tag was deleted.

### get_branch

**Signature**: `get_branch(branch: str | Branch) → Branch`

Get the branch.

**Example**:

```python
import bauplan
client = bauplan.Client()

branch = client.get_branch('my_branch_name')
print(branch.hash)
```

**Parameters**:

- **branch**: `str | Branch`\
  The name of the branch to retrieve.

**Raises**:

- `BranchNotFoundError` – If the branch does not exist.
- `NotABranchRefError` – If the object is not a branch.
- `ForbiddenError` – If the user does not have access to the branch.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: A `Branch` object.

### get_branches

**Signature**: `get_branches(name: str | None = None, user: str | None = None, limit: int | None = None, itersize: int | None = None) → GetBranchesResponse`

Get the available data branches in the Bauplan catalog.

**Example**:

```python
import bauplan
client = bauplan.Client()

for branch in client.get_branches():
    print(branch.name, branch.hash)
```

**Parameters**:

- **name**: `str | None = None`\
  Filter the branches by name.
- **user**: `str | None = None`\
  Filter the branches by user.
- **limit**: `int | None = None`\
  Optional, max number of branches to get.
- **itersize**: `int | None = None`\
  Optional, overwrites `profile.catalog_max_records`, the max number of objects per HTTP request.

**Returns**: A `GetBranchesResponse` object.

### get_commits

**Signature**: `get_commits(ref: str | Branch | Tag | Ref, *, filter_by_message: str | None = None, filter_by_author_username: str | None = None, filter_by_author_name: str | None = None, filter_by_author_email: str | None = None, filter_by_authored_date: str | datetime | None = None, filter_by_authored_date_start_at: str | datetime | None = None, filter_by_authored_date_end_at: str | datetime | None = None, filter_by_parent_hash: str | None = None, filter_by_properties: dict[str, str] | None = None, filter: str | None = None, limit: int | None = None, itersize: int | None = None) → GetCommitsResponse`

Get the commits for the target branch or ref.

**Parameters**:

- **ref**: `str | Branch | Tag | Ref`\
  The ref or branch to get the commits from.
- **filter_by_message**: `str | None = None`\
  Optional, filter the commits by message (can be a string or a regex like `'^abc.*$'`).
- **filter_by_author_username**: `str | None = None`\
  Optional, filter the commits by author username (can be a string or a regex like `'^abc.*$'`).
- **filter_by_author_name**: `str | None = None`\
  Optional, filter the commits by author name (can be a string or a regex like `'^abc.*$'`).
- **filter_by_author_email**: `str | None = None`\
  Optional, filter the commits by author email (can be a string or a regex like `'^abc.*$'`).
- **filter_by_authored_date**: `str | datetime | None = None`\
  Optional, filter the commits by the exact authored date.
- **filter_by_authored_date_start_at**: `str | datetime | None = None`\
  Optional, filter the commits by authored date start at.
- **filter_by_authored_date_end_at**: `str | datetime | None = None`\
  Optional, filter the commits by authored date end at.
- **filter_by_parent_hash**: `str | None = None`\
  Optional, filter the commits by parent hash.
- **filter_by_properties**: `dict[str, str] | None = None`\
  Optional, filter the commits by commit properties.
- **filter**: `str | None = None`\
  Optional, a CEL filter expression to filter the commits.
- **limit**: `int | None = None`\
  Optional, max number of commits to get.
- **itersize**: `int | None = None`\
  Optional, overwrites `profile.catalog_max_records`, the max number of objects per HTTP request.

**Raises**:

- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: A `GetCommitsResponse` object.

### get_job

**Signature**: `get_job(job_id: str) → Job`

*EXPERIMENTAL*: Get a job by ID.

### get_job_logs

**Signature**: `get_job_logs(job_id_prefix: str) → list[JobLog]`

*EXPERIMENTAL*: Get logs for a job by ID prefix.

### get_namespace

**Signature**: `get_namespace(namespace: str | Namespace, ref: str | Branch | Tag | Ref) → Namespace`

Get a namespace.

**Example**:

```python
import bauplan
client = bauplan.Client()

namespace = client.get_namespace(
    namespace='my_namespace_name',
    ref='my_ref_or_branch_name',
)
```

**Parameters**:

- **namespace**: `str | Namespace`\
  The name of the namespace to get.
- **ref**: `str | Branch | Tag | Ref`\
  The ref, branch name, or tag name to check the namespace on.

**Raises**:

- `NamespaceNotFoundError` – If the namespace does not exist.
- `RefNotFoundError` – If the ref does not exist.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: A `Namespace` object.

### get_namespaces

**Signature**: `get_namespaces(ref: str | Branch | Tag | Ref, *, filter_by_name: str | None = None, limit: int | None = None, itersize: int | None = None) → GetNamespacesResponse`

Get the available data namespaces in the Bauplan catalog branch.

**Example**:

```python
import bauplan
client = bauplan.Client()

for namespace in client.get_namespaces('my_namespace_name'):
    print(namespace.name)
```

**Parameters**:

- **ref**: `str | Branch | Tag | Ref`\
  The ref, branch name, or tag name to retrieve the namespaces from.
- **filter_by_name**: `str | None = None`\
  Optional, filter the namespaces by name.
- **limit**: `int | None = None`\
  Optional, max number of namespaces to get.
- **itersize**: `int | None = None`\
  Optional, overwrites `profile.catalog_max_records`, the max number of objects per HTTP request.

**Raises**:

- `RefNotFoundError` – If the ref does not exist.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Yields**: A `Namespace` object.

### get_table

**Signature**: `get_table(table: str | Table, ref: str | Branch | Tag | Ref, include_raw: bool = False) → TableWithMetadata`

Get the table data and metadata for a table in the target branch.

**Example**:

```python
import bauplan
client = bauplan.Client()

table = client.get_table(
    table='my_table_name',
    ref='my_ref_or_branch_name',
)

for c in table.fields:
    print(c.name, c.required, c.type)

print(table.records)
```

**Parameters**:

- **ref**: `str | Branch | Tag | Ref`\
  The ref, branch name, or tag name to get the table from.
- **table**: `str | Table`\
  The table to retrieve.
- **include_raw**: `bool = False`\
  Whether or not to include the raw `metadata.json` object as a nested dict.

**Raises**:

- `RefNotFoundError` – If the ref does not exist.
- `NamespaceNotFoundError` – If the namespace does not exist.
- `TableNotFoundError` – If the table does not exist.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: A `TableWithMetadata` object, optionally including the raw `metadata.json` object.

### get_tables

**Signature**: `get_tables(ref: str | Branch | Tag | Ref, *, filter_by_name: str | None = None, filter_by_namespace: str | None = None, namespace: str | Namespace | None = None, limit: int | None = None, itersize: int | None = None) → GetTablesResponse`

Get the tables and views in the target branch.

**Example**:

```python
import bauplan
client = bauplan.Client()

for table in client.get_tables('my_ref_or_branch_name'):
    print(table.name, table.kind)
```

**Parameters**:

- **ref**: `str | Branch | Tag | Ref`\
  The ref or branch to get the tables from.
- **filter_by_name**: `str | None = None`\
  Optional, the table name to filter by.
- **filter_by_namespace**: `str | None = None`\
  Optional, the namespace to get filtered tables from.
- **namespace**: `str | Namespace | None = None`\
  *DEPRECATED*: Optional, the namespace to get filtered tables from.
- **limit**: `int | None = None`\
  Optional, max number of tables to get.
- **itersize**: `int | None = None`\
  Optional, overwrites `profile.catalog_max_records`, the max number of objects per HTTP request.

**Returns**: A `GetTablesResponse` object.

### get_tag

**Signature**: `get_tag(tag: str | Tag) → Tag`

Get the tag.

**Example**:

```python
import bauplan
client = bauplan.Client()

tag = client.get_tag('my_tag_name')
print(tag.hash)
```

**Parameters**:

- **tag**: `str | Tag`\
  The name of the tag to retrieve.

**Raises**:

- `TagNotFoundError` – If the tag does not exist.
- `NotATagRefError` – If the object is not a tag.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: A `Tag` object.

### get_tags

**Signature**: `get_tags(*, filter_by_name: str | None = None, limit: int | None = None, itersize: int | None = None) → GetTagsResponse`

Get all the tags.

**Parameters**:

- **filter_by_name**: `str | None = None`\
  Optional, filter the tags by name.
- **limit**: `int | None = None`\
  Optional, max number of tags to get.
- **itersize**: `int | None = None`\
  Optional, overwrites `profile.catalog_max_records`, the max number of objects per HTTP request.

**Raises**:

- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: A `GetTagsResponse` object.

### has_branch

**Signature**: `has_branch(branch: str | Branch) → bool`

Check if a branch exists.

**Example**:

```python
import bauplan
client = bauplan.Client()

assert client.has_branch('my_branch_name')
```

**Parameters**:

- **branch**: `str | Branch`\
  The name of the branch to check.

**Raises**:

- `NotABranchRefError` – If the object is not a branch.
- `ForbiddenError` – If the user does not have access to the branch.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: A boolean indicating if the branch exists.

### has_namespace

**Signature**: `has_namespace(namespace: str | Namespace, ref: str | Branch | Tag | Ref) → bool`

Check if a namespace exists.

**Example**:

```python
import bauplan
client = bauplan.Client()

assert client.has_namespace(
    namespace='my_namespace_name',
    ref='my_ref_or_branch_name',
)
```

**Parameters**:

- **namespace**: `str | Namespace`\
  The name of the namespace to check.
- **ref**: `str | Branch | Tag | Ref`\
  The ref, branch name, or tag name to check the namespace on.

**Raises**:

- `RefNotFoundError` – If the ref does not exist.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: A boolean indicating if the namespace exists.

### has_table

**Signature**: `has_table(table: str | Table, ref: str | Branch | Tag | Ref) → bool`

Check if a table exists.

**Example**:

```python
import bauplan
client = bauplan.Client()

assert client.has_table(
    table='my_table_name',
    ref='my_ref_or_branch_name',
)
```

**Parameters**:

- **ref**: `str | Branch | Tag | Ref`\
  The ref, branch name, or tag name to get the table from.
- **table**: `str | Table`\
  The table to retrieve.

**Raises**:

- `RefNotFoundError` – If the ref does not exist.
- `NamespaceNotFoundError` – If the namespace does not exist.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: A boolean indicating if the table exists.

### has_tag

**Signature**: `has_tag(tag: str | Tag) → bool`

Check if a tag exists.

**Example**:

```python
import bauplan
client = bauplan.Client()

assert client.has_tag(
    tag='my_tag_name',
)
```

**Parameters**:

- **tag**: `str | Tag`\
  The tag to retrieve.

**Raises**:

- `NotATagRefError` – If the object is not a tag.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: A boolean indicating if the tag exists.

### import_data

**Signature**: `import_data(table: str | Table, search_uri: str, branch: str | Branch | None = None, namespace: str | Namespace | None = None, continue_on_error: bool = False, import_duplicate_files: bool = False, best_effort: bool = False, preview: 'on' | 'off' | 'head' | 'tail' | str | None = None, debug: bool | None = None, args: dict[str, str] | None = None, verbose: bool | None = None, client_timeout: int | float | None = None) → TableDataImportState`

Imports data into an already existing table.

**Example**:

```python
import bauplan
client = bauplan.Client()

plan_state = client.import_data(
    table='my_table_name',
    search_uri='s3://path/to/my/files/*.parquet',
    branch='my_branch_name',
)
if plan_state.error:
    plan_error_action(...)
success_action(plan_state.plan)
```

**Parameters**:

- **table**: `str | Table`\
  Previously created table into which data will be imported.
- **search_uri**: `str`\
  URI to scan for files to import.
- **branch**: `str | Branch | None = None`\
  Branch in which to import the table.
- **namespace**: `str | Namespace | None = None`\
  Namespace of the table. If not specified, namespace will be inferred from table name or default settings.
- **continue_on_error**: `bool = False`\
  Do not fail the import even if one data import fails.
- **import_duplicate_files**: `bool = False`\
  Ignore prevention of importing S3 files that were already imported.
- **best_effort**: `bool = False`\
  Don’t fail if schema of table does not match.
- **preview**: `'on' | 'off' | 'head' | 'tail' | str | None = None`\
  Whether to enable or disable preview mode for the import.
- **debug**: `bool | None = None`\
  Whether to enable or disable debug mode for the import.
- **args**: `dict[str, str] | None = None`\
  Dict of arbitrary args to pass to the backend.
- **verbose**: `bool | None = None`\
  Whether to enable or disable verbose mode.
- **client_timeout**: `int | float | None = None`\
  Seconds to timeout; this also cancels the remote job execution.

**Returns**: The plan state.

### info

**Signature**: `info(debug: bool | None = None, verbose: bool | None = None, client_timeout: int | float | None = None, **kwargs: Any) → InfoState`

Fetch organization & account information.

### list_jobs

**Signature**: `list_jobs(all_users: bool | None = None) → list[Job]`

*EXPERIMENTAL*: List all jobs.

### merge_branch

**Signature**: `merge_branch(source_ref: str | Branch | Tag, into_branch: str | Branch, commit_message: str | None = None, commit_body: str | None = None, commit_properties: dict[str, str] | None = None, message: str | None = None, properties: dict[str, str] | None = None) → Branch`

Merge one branch into another.

**Example**:

```python
import bauplan
client = bauplan.Client()

assert client.merge_branch(
    source_ref='my_ref_or_branch_name',
    into_branch='main',
)
```

**Parameters**:

- **source_ref**: `str | Branch | Tag`\
  The name of the merge source; either a branch like `"main"` or ref like `"main@[sha]"`.
- **into_branch**: `str | Branch`\
  The name of the merge target.
- **commit_message**: `str | None = None`\
  Optional, the commit message.
- **commit_body**: `str | None = None`\
  Optional, the commit body.
- **commit_properties**: `dict[str, str] | None = None`\
  Optional, a list of properties to attach to the merge.

**Raises**:

- `MergeForbiddenError` – If the user does not have access to merge the branch.
- `BranchNotFoundError` – If the destination branch does not exist.
- `NotAWriteBranchError` – If the destination branch is not a writable ref.
- `MergeConflictError` – If the merge operation results in a conflict.
- `UnauthorizedError` – If the user’s credentials are invalid.
- `ValueError` – If one or more parameters are invalid.

**Returns**: The `Branch` where the merge was made.

### plan_table_creation

**Signature**: `plan_table_creation(table: str | Table, search_uri: str, branch: str | Branch | None = None, namespace: str | Namespace | None = None, partitioned_by: str | None = None, replace: bool | None = None, debug: bool | None = None, args: dict[str, str] | None = None, verbose: bool | None = None, client_timeout: int | float | None = None) → TableCreatePlanState`

Create a table import plan from an S3 location. This operation attempts to create a table based on schemas of N Parquet files found by a given search URI. A YAML file containing the schema and plan is returned, and if there are no conflicts, it is automatically applied.

**Example**:

```python
import bauplan
client = bauplan.Client()

plan_state = client.plan_table_creation(
    table='my_table_name',
    search_uri='s3://path/to/my/files/*.parquet',
    ref='my_ref_or_branch_name',
)
if plan_state.error:
    plan_error_action(...)
success_action(plan_state.plan)
```

**Parameters**:

- **table**: `str | Table`\
  The table to be created.
- **search_uri**: `str`\
  The location of the files to scan for schema.
- **branch**: `str | Branch | None = None`\
  The branch name in which to create the table.
- **namespace**: `str | Namespace | None = None`\
  Optional argument specifying the namespace. If not specified, it will be inferred based on table location or the default namespace.
- **partitioned_by**: `str | None = None`\
  Optional argument specifying the table partitioning.
- **replace**: `bool | None = None`\
  Replace the table if it already exists.
- **debug**: `bool | None = None`\
  Whether to enable or disable debug mode.
- **args**: `dict[str, str] | None = None`\
  Dict of arbitrary args to pass to the backend.
- **verbose**: `bool | None = None`\
  Whether to enable or disable verbose mode.
- **client_timeout**: `int | float | None = None`\
  Seconds to timeout; this also cancels the remote job execution.

**Raises**:

- `TableCreatePlanStatusError` – If the table creation plan fails.

**Returns**: The plan state.

### query

**Signature**: `query(query: str, ref: str | Branch | Tag | Ref | None = None, max_rows: int | None = None, cache: 'on' | 'off' | None = None, connector: str | None = None, connector_config_key: str | None = None, connector_config_uri: str | None = None, namespace: str | Namespace | None = None, debug: bool | None = None, args: dict[str, str] | None = None, verbose: bool | None = None, client_timeout: int | float | None = None) → Table`

Execute a SQL query and return the results as a `pyarrow.Table`. Note that this function uses Arrow internally, resulting in fast data transfer. To return results as a pandas DataFrame, use the `to_pandas` function of `pyarrow.Table`.

**Example**:

```python
import bauplan
client = bauplan.Client()

my_table = client.query(
    query='SELECT c1 FROM my_table',
    ref='my_ref_or_branch_name',
)

df = my_table.to_pandas()
```

**Parameters**:

- **query**: `str`\
  The Bauplan query to execute.
- **ref**: `str | Branch | Tag | Ref | None = None`\
  The ref, branch name, or tag name to query from.
- **max_rows**: `int | None = None`\
  The maximum number of rows to return; default: `None` (no limit).
- **cache**: `'on' | 'off' | None = None`\
  Whether to enable or disable caching for the query.
- **connector**: `str | None = None`\
  The connector type for the model (defaults to Bauplan). Allowed values are `'snowflake'` and `'dremio'`.
- **connector_config_key**: `str | None = None`\
  The key name if the SSM key is custom with the pattern `bauplan/connectors/<connector_type>/<key>`.
- **connector_config_uri**: `str | None = None`\
  Full SSM URI if completely custom path, e.g., `ssm://us-west-2/123456789012/baubau/dremio`.
- **namespace**: `str | Namespace | None = None`\
  The namespace to run the query in. If not set, the query will be run in the default namespace for your account.
- **debug**: `bool | None = None`\
  Whether to enable or disable debug mode for the query.
- **args**: `dict[str, str] | None = None`\
  Additional arguments to pass to the query (default: `None`).
- **verbose**: `bool | None = None`\
  Whether to enable or disable verbose mode for the query.
- **client_timeout**: `int | float | None = None`\
  Seconds to timeout; this also cancels the remote job execution.

**Returns**: The query results as a `pyarrow.Table`.

### query_to_csv_file

**Signature**: `query_to_csv_file(path: str | Path, query: str, ref: str | Branch | Tag | Ref | None = None, max_rows: int | None = None, cache: 'on' | 'off' | None = None, connector: str | None = None, connector_config_key: str | None = None, connector_config_uri: str | None = None, namespace: str | Namespace | None = None, debug: bool | None = None, args: dict[str, str] | None = None, verbose: bool | None = None, client_timeout: int | float | None = None, **kwargs: Any) → Path`

Export the results of a SQL query to a file in CSV format.

**Example**:

```python
import bauplan
client = bauplan.Client()

client.query_to_csv_file(
    path='./my.csv',
    query='SELECT c1 FROM my_table',
    ref='my_ref_or_branch_name',
)
```

**Parameters**:

- **path**: `str | Path`\
  The name or path of the CSV file to write the results to.
- **query**: `str`\
  The Bauplan query to execute.
- **ref**: `str | Branch | Tag | Ref | None = None`\
  The ref, branch name, or tag name to query from.
- **max_rows**: `int | None = None`\
  The maximum number of rows to return; default: `None` (no limit).
- **cache**: `'on' | 'off' | None = None`\
  Whether to enable or disable caching for the query.
- **connector**: `str | None = None`\
  The connector type for the model (defaults to Bauplan). Allowed values are `'snowflake'` and `'dremio'`.
- **connector_config_key**: `str | None = None`\
  The key name if the SSM key is custom with the pattern `bauplan/connectors/<connector_type>/<key>`.
- **connector_config_uri**: `str | None = None`\
  Full SSM URI if completely custom path, e.g., `ssm://us-west-2/123456789012/baubau/dremio`.
- **namespace**: `str | Namespace | None = None`\
  The namespace to run the query in. If not set, the query will be run in the default namespace for your account.
- **debug**: `bool | None = None`\
  Whether to enable or disable debug mode for the query.
- **args**: `dict[str, str] | None = None`\
  Additional arguments to pass to the query (default: `None`).
- **verbose**: `bool | None = None`\
  Whether to enable or disable verbose mode for the query(loss of generality).
- **client_timeout**: `int | float | None = None`\
  Seconds to timeout; this also cancels the remote job execution.

**Returns**: The path of the file written.

### query_to_generator

**Signature**: `query_to_generator(query: str, ref: str | Branch | Tag | Ref | None = None, max_rows: int | None = None, cache: 'on' | 'off' | None = None, connector: str | None = None, connector_config_key: str | None = None, connector_config_uri: str | None = None, namespace: str | Namespace | None = None, debug: bool | None = None, as_json: bool | None = False, args: dict[str, str] | None = None, verbose: bool | None = None, client_timeout: int | float | None = None) → Generator[dict[str, Any], None, None]`

Execute a SQL query and return the results as a generator, where each row is a Python dictionary.

**Example**:

```python
import bauplan
client = bauplan.Client()

res = client.query_to_generator(
    query='SELECT c1 FROM my_table',
    ref='my_ref_or_branch_name',
)
for row in res:
    # do logic
```

**Parameters**:

- **query**: `str`\
  The Bauplan query to execute.
- **ref**: `str | Branch | Tag | Ref | None = None`\
  The ref, branch name, or tag name to query from.
- **max_rows**: `int | None = None`\
  The maximum number of rows to return; default: `None` (no limit).
- **cache**: `'on' | 'off' | None = None`\
  Whether to enable or disable caching for the query.
- **connector**: `str | None = None`\
  The connector type for the model (defaults to Bauplan). Allowed values are `'snowflake'` and `'dremio'`.
- **connector_config_key**: `str | None = None`\
  The key name if the SSM key is custom with the pattern `bauplan/connectors/<connector_type>/<key>`.
- **connector_config_uri**: `str | None = None`\
  Full SSM URI if completely custom path, e.g., `ssm://us-west-2/123456789012/baubau/dremio`.
- **namespace**: `str | Namespace | None = None`\
  The namespace to run the query in. If not set, the query will be run in the default namespace for your account.
- **debug**: `bool | None = None`\
  Whether to enable or disable debug mode for the query.
- **as_json**: `bool | None = False`\
  Whether to return the results as a JSON-compatible string (default: `False`).
- **args**: `dict[str, str] | None = None`\
  Additional arguments to pass to the query (default: `None`).
- **verbose**: `bool | None = None`\
  Whether to enable or disable verbose mode for the query.
- **client_timeout**: `int | float | None = None`\
  Seconds to timeout; this also cancels the remote job execution.

**Yields**: A dictionary representing a row of query results.

### query_to_json_file

**Signature**: `query_to_json_file(path: str | Path, query: str, file_format: 'json' | 'jsonl' | None = 'json', ref: str | Branch | Tag | Ref | None = None, max_rows: int | None = None, cache: 'on' | 'off' | None = None, connector: str | None = None, connector_config_key: str | None = None, connector_config_uri: str | None = None, namespace: str | Namespace | None = None, debug: bool | None = None, args: dict[str, str] | None = None, verbose: bool | None = None, client_timeout: int | float | None = None) → Path`

Export the results of a SQL query to a file in JSON format.

**Example**:

```python
import bauplan
client = bauplan.Client()

client.query_to_json_file(
    path='./my.json',
    query='SELECT c1 FROM my_table',
    ref='my_ref_or_branch_name',
)
```

**Parameters**:

- **path**: `str | Path`\
  The name or path of the JSON file to write the results to.
- **query**: `str`\
  The Bauplan query to execute.
- **file_format**: `'json' | 'jsonl' | None = 'json'`\
  The format to write the results in; default: `json`. Allowed values are `'json'` and `'jsonl'`.
- **ref**: `str | Branch | Tag | Ref | None = None`\
  The ref, branch name, or tag name to query from.
- **max_rows**: `int | None = None`\
  The maximum number of rows to return; default: `None` (no limit).
- **cache**: `'on' | 'off' | None = None`\
  Whether to enable or disable caching for the query.
- **connector**: `str | None = None`\
  The connector type for the model (defaults to Bauplan). Allowed values are `'snowflake'` and `'dremio'`.
- **connector_config_key**: `str | None = None`\
  The key name if the SSM key is custom with the pattern `bauplan/connectors/<connector_type>/<key>`.
- **connector_config_uri**: `str | None = None`\
  Full SSM URI if completely custom path, e.g., `ssm://us-west-2/123456789012/baubau/dremio`.
- **namespace**: `str | Namespace | None = None`\
  The namespace to run the query in. If not set, the query will be run in the default namespace for your account.
- **debug**: `bool | None = None`\
  Whether to enable or disable debug mode for the query.
- **args**: `dict[str, str] | None = None`\
  Additional arguments to pass to the query (default: `None`).
- **verbose**: `bool | None = None`\
  Whether to enable or disable verbose mode for the query.
- **client_timeout**: `int | float | None = None`\
  Seconds to timeout; this also cancels the remote job execution.

**Returns**: The path of the file written.

### query_to_parquet_file

**Signature**: `query_to_parquet_file(path: str | Path, query: str, ref: str | Branch | Tag | Ref | None = None, max_rows: int | None = None, cache: 'on' | 'off' | None = None, connector: str | None = None, connector_config_key: str | None = None, connector_config_uri: str | None = None, namespace: str | Namespace | None = None, debug: bool | None = None, args: dict[str, str] | None = None, verbose: bool | None = None, client_timeout: int | float | None = None, **kwargs: Any) → Path`

Export the results of a SQL query to a file in Parquet format.

**Example**:

```python
import bauplan
client = bauplan.Client()

client.query_to_parquet_file(
    path='./my.parquet',
    query='SELECT c1 FROM my_table',
    ref='my_ref_or_branch_name',
)
```

**Parameters**:

- **path**: `str | Path`\
  The name or path of the Parquet file to write the results to.
- **query**: `str`\
  The Bauplan query to execute.
- **ref**: `str | Branch | Tag | Ref | None = None`\
  The ref, branch name, or tag name to query from.
- **max_rows**: `int | None = None`\
  The maximum number of rows to return; default: `None` (no limit).
- **cache**: `'on' | 'off' | None = None`\
  Whether to enable or disable caching for the query.
- **connector**: `str | None = None`\
  The connector type for the model (defaults to Bauplan). Allowed values are `'snowflake'` and `'dremio'`.
- **connector_config_key**: `str | None = None`\
  The key name if the SSM key is custom with the pattern `bauplan/connectors/<connector_type>/<key>`.
- **connector_config_uri**: `str | None = None`\
  Full SSM URI if completely custom path, e.g., `ssm://us-west-2/123456789012/baubau/dremio`.
- **namespace**: `str | Namespace | None = None`\
  The namespace to run the query in. If not set, the query will be run in the default namespace for your account.
- **debug**: `bool | None = None`\
  Whether to enable or disable debug mode for the query.
- **args**: `dict[str, str] | None = None`\
  Additional arguments to pass to the query (default: `None`).
- **verbose**: `bool | None = None`\
  Whether to enable or disable verbose mode for the query.
- **client_timeout**: `int | float | None = None`\
  Seconds to timeout; this also cancels the remote job execution.

**Returns**: The path of the file written.

### rerun

**Signature**: `rerun(job_id: str, ref: str | Branch | Tag | Ref | None = None, namespace: str | Namespace | None = None, cache: 'on' | 'off' | None = None, transaction: 'on' | 'off' | None = None, dry_run: bool | None = None, strict: 'on' | 'off' | None = None, preview: 'on' | 'off' | 'head' | 'tail' | str | None = None, debug: bool | None = None, args: dict[str, str] | None = None, verbose: bool | None = None, client_timeout: int | float | None = None) → ReRunState`

Re-run a Bauplan project by its ID and return the state of the run. This is equivalent to running the `bauplan rerun` command through the CLI.

**Parameters**:

- **job_id**: `str`\
  The Job ID of the previous run. This can be used to re-run a previous run, e.g., on a different branch.
- **ref**: `str | Branch | Tag | Ref | None = None`\
  The ref, branch name, or tag name from which to rerun the project.
- **namespace**: `str | Namespace | None = None`\
  The namespace to run the job in. If not set, the job will be run in the default namespace.
- **cache**: `'on' | 'off' | None = None`\
  Whether to enable or disable caching for the run.
- **transaction**: `'on' | 'off' | None = None`\
  Whether to enable or disable transaction mode for the run.
- **dry_run**: `bool | None = None`\
  Whether to enable or disable dry-run mode for the run; models are not materialized.
- **strict**: `'on' | 'off' | None = None`\
  Whether to enable or disable strict schema validation.
- **preview**: `'on' | 'off' | 'head' | 'tail' | str | None = None`\
  Whether to enable or disable preview mode for the run.
- **debug**: `bool | None = None`\
  Whether to enable or disable debug mode for the run.
- **args**: `dict[str, str] | None = None`\
  Additional arguments (optional).
- **verbose**: `bool | None = None`\
  Whether to enable or disable verbose mode for the run.
- **client_timeout**: `int | float | None = None`\
  Seconds to timeout; this also cancels the remote job execution.

**Returns**: The state of the run.

### revert_table

**Signature**: `revert_table(table: str | Table, *, source_ref: str | Branch | Tag | Ref, into_branch: str | Branch, replace: bool | None = None, commit_body: str | None = None, commit_properties: dict[str, str] | None = None) → Branch`

Revert a table to a previous state.

**Example**:

```python
import bauplan
client = bauplan.Client()

assert client.revert_table(
    table='my_table_name',
    source_ref='my_ref_or_branch_name',
    into_branch='main',
)
```

**Parameters**:

- **table**: `str | Table`\
  The table to revert.
- **source_ref**: `str | Branch | Tag | Ref`\
  The name of the source ref; either a branch like `"main"` or ref like `"main@[sha]"`.
- **into_branch**: `str | Branch`\
  The name of the target branch where the table will be reverted.
- **replace**: `bool | None = None`\
  Optional, whether to replace the table if it already exists.
- **commit_body**: `str | None = None`\
  Optional, the commit body message to attach to the operation.
- **commit_properties**: `dict[str, str] | None = None`\
  Optional, a list of properties to attach to the operation.

**Raises**:

- `RevertTableForbiddenError` – If the user does not have access to revert the table.
- `RefNotFoundError` – If the ref does not exist.
- `BranchNotFoundError` – If the destination branch does not exist.
- `NotAWriteBranchError` – If the destination branch is not a writable ref.
- `BranchHeadChangedError` – If the branch head hash has changed.
- `MergeConflictError` – If the merge operation results in a conflict.
- `UnauthorizedError` – If the user’s credentials are invalid.
- **ValueError** – If one or more parameters are invalid.

**Returns**: The `Branch` where the revert was made.

### run

**Signature**: `run(project_dir: str | None = None, ref: str | Branch | Tag | Ref | None = None, namespace: str | Namespace | None = None, parameters: dict[str, str | int | float | bool | None] | None = None, cache: 'on' | 'off' | None = None, transaction: 'on' | 'off' | None = None, dry_run: bool | None = None, strict: 'on' | 'off' | None = None, preview: 'on' | 'off' | 'head' | 'tail' | str | None = None, debug: bool | None = None, args: dict[str, str] | None = None, verbose: bool | None = None, client_timeout: int | float | None = None, detach: bool | None = None) → RunState`

Run a Bauplan project and return the state of the run. This is equivalent to running the `bauplan run` command through the CLI.

**Parameters**:

- **project_dir**: `str | None = None`\
  The directory of the project (where the `bauplan_project.yml` file is located).
- **ref**: `str | Branch | Tag | Ref | None = None`\
  The ref, branch name, or tag name from which to run the project.
- **namespace**: `str | Namespace | None = None`\
  The namespace to run the job in. If not set, the job will be run in the default namespace.
- **parameters**: `dict[str, str | int | float | bool | None] | None = None`\
  Parameters for templating into SQL or Python models.
- **cache**: `'on' | 'off' | None = None`\
  Whether to enable or disable caching for the run.
- **transaction**: `'on' | 'off' | None = None`\
  Whether to enable or disable transaction mode for the run.
- **dry_run**: `bool | None = None`\
  Whether to enable or disable dry-run mode for the run; models are not materialized.
- **strict**: `'on' | 'off' | None = None`\
  Whether to enable or disable strict schema validation.
- **preview**: `'on' | 'off' | 'head' | 'tail' | str | None = None`\
  Whether to enable or disable preview mode for the run.
- **debug**: `bool | None = None`\
  Whether to enable or disable debug mode for the run.
- **args**: `dict[str, str] | None = None`\
  Additional arguments (optional).
- **verbose**: `bool | None = None`\
  Whether to enable or disable verbose mode for the run.
- **client_timeout**: `int | float | None = None`\
  Seconds to timeout; this also cancels the remote job execution.
- **detach**: `bool | None = None`\
  Whether to detach the run and return immediately instead of blocking on log streaming.

**Returns**: The state of the run.

### scan

**Signature**: `scan(table: str | Table, ref: str | Branch | Tag | Ref | None = None, columns: list[str] | None = None, filters: str | None = None, limit: int | None = None, cache: 'on' | 'off' | None = None, connector: str | None = None, connector_config_key: str | None = None, connector_config_uri: str | None = None, namespace: str | Namespace | None = None, debug: bool | None = None, args: dict[str, str] | None = None, client_timeout: int | float | None = None, **kwargs: Any) → Table`

Execute a table scan (with optional filters) and return the results as an Arrow Table. This function uses SQLGlot to compose a safe SQL query and internally defers to the `query_to_arrow` function for the actual scan.

**Example**:

```python
import bauplan
client = bauplan.Client()

my_table = client.scan(
    table='my_table_name',
    ref='my_ref_or_branch_name',
    columns=['c1'],
    filters='c2 > 10',
)
```

**Parameters**:

- **table**: `str | Table`\
  The table to scan.
- **ref**: `str | Branch | Tag | Ref | None = None`\
  The ref, branch name, or tag name to scan from.
- **columns**: `list[str] | None = None`\
  The columns to return (default: `None`).
- **filters**: `str | None = None`\
  The filters to apply (default: `None`).
- **limit**: `int | None = None`\
  The maximum number of rows to return (default: `None`).
- **cache**: `'on' | 'off' | None = None`\
  Whether to enable or disable caching for the query.
- **connector**: `str | None = None`\
  The connector type for the model (defaults to Bauplan). Allowed values are `'snowflake'` and `'dremio'`.
- **connector_config_key**: `str | None = None`\
  The key name if the SSM key is custom with the pattern `bauplan/connectors/<connector_type>/<key>`.
- **connector_config_uri**: `str | None = None`\
  Full SSM URI if completely custom path, e.g., `ssm://us-west-2/123456789012/baubau/dremio`.
- **namespace**: `str | Namespace | None = None`\
  The namespace to run the scan in. If not set, the scan will be run in the default namespace for your account.
- **debug**: `bool | None = None`\
  Whether to enable or disable debug mode for the query.
- **args**: `dict[str, str] | None = None`\
  Dict of arbitrary args to pass to the backend.
- **client_timeout**: `int | float | None = None`\
  Seconds to timeout; this also cancels the remote job execution.

**Returns**: The scan results as a `pyarrow.Table`.

## class bauplan.InfoState

**Signature**: `bauplan.InfoState(*, client_version: str | None = None, organization: OrganizationInfo | None = None, user: UserInfo | None = None, runners: list[RunnerNodeInfo] | None = None)`

**Bases**: `_BauplanData`

- **client_version**: `str | None`
- **model_computed_fields**: `ClassVar[Dict[str, ComputedFieldInfo]] = {}`\
  A dictionary of computed field names and their corresponding `ComputedFieldInfo` objects.
- **model_config**: `ClassVar[ConfigDict] = {}`\
  Configuration for the model, should be a dictionary conforming to `[ConfigDict][pydantic.config.ConfigDict]`.
- **model_fields**: `ClassVar[Dict[str, FieldInfo]] = {'client_version': FieldInfo(annotation=Union[str, NoneType], required=False, default=None), 'organization': FieldInfo(annotation=Union[OrganizationInfo, NoneType], required=False, default=None), 'runners': FieldInfo(annotation=Union[List[RunnerNodeInfo], NoneType], required=False, default=None), 'user': FieldInfo(annotation=Union[UserInfo, NoneType], required=False, default=None)}`\
  Metadata about the fields defined on the model, mapping of field names to `[FieldInfo][pydantic.fields.FieldInfo]` objects. Replaces `Model.__fields__` from Pydantic V1.
- **organization**: `OrganizationInfo | None`
- **runners**: `List[RunnerNodeInfo] | None`
- **user**: `UserInfo | None`

## class bauplan.JobStatus

**Signature**: `bauplan.JobStatus(canceled: str = 'CANCELLED', cancelled: str = 'CANCELLED', failed: str = 'FAILED', rejected: str = 'REJECTED', success: str = 'SUCCESS', timeout: str = 'TIMEOUT', unknown: str = 'UNKNOWN')`

**Bases**: `object`

- **canceled**: `str = 'CANCELLED'`
- **cancelled**: `str = 'CANCELLED'`
- **failed**: `str = 'FAILED'`
- **rejected**: `str = 'REJECTED'`
- **success**: `str = 'SUCCESS'`
- **timeout**: `str = 'TIMEOUT'`
- **unknown**: `str = 'UNKNOWN'`

## class bauplan.Model

**Signature**: `bauplan.Model(name: str, columns: list[str] | None = None, filter: str | None = None, ref: str | None = None, connector: str | None = None, connector_config_key: str | None = None, connector_config_uri: str | None = None, **kwargs: Any)`

**Bases**: `object`

Represents a model (dataframe/table representing a DAG step) as an input to a function.

**Example**:

```python
@bauplan.model()
def some_parent_model():
    return pyarrow.Table.from_pydict({'bar': [1, 2, 3]})

@bauplan.model()
def your_cool_model(
    parent_0=bauplan.Model(
        'some_parent_model',
        columns=['bar'],
        filter='bar > 1',
    )
):
    return pyarrow.Table.from_pandas(
        pd.DataFrame({
            'foo': parent_0['bar'] * 2,
        })
    )
```

Bauplan can wrap other engines for processing models, exposing a common interface and unified API while dispatching operations to the underlying engine. Authentication and authorization happen securely through SSM.

**Example with connector**:

```python
@bauplan.model()
def your_cool_model(
    parent_0=bauplan.Model(
        'some_parent_model',
        columns=['bar'],
        filter='bar > 1',
        connector='dremio',
        connector_config_key='bauplan',
    )
):
    return pyarrow.Table.from_pandas(
        pd.DataFrame({
            'foo': parent_0['bar'] * 2,
        })
    )
```

**Parameters**:

- **name**: `str`\
  The name of the model.
- **columns**: `list[str] | None = None`\
  The list of columns in the model. If not provided, the model will load all columns.
- **filter**: `str | None = None`\
  The optional filter for the model. Defaults to `None`.
- **ref**: `str | None = None`\
  The optional reference to the model. Defaults to `None`.
- **connector**: `str | None = None`\
  The connector type for the model (defaults to Bauplan SQL). Allowed values are `'snowflake'` and `'dremio'`.
- **connector_config_key**: `str | None = None`\
  The key name if the SSM key is custom with the pattern `bauplan/connectors/<connector_type>/<key>`.
- **connector_config_uri**: `str | None = None`\
  Full SSM URI if completely custom path, e.g., `ssm://us-west-2/123456789012/baubau/dremio`.

## class bauplan.OrganizationInfo

**Signature**: `bauplan.OrganizationInfo(*, id: str | None = None, name: str | None = None, slug: str | None = None, default_parameter_secret_key: str | None = None, default_parameter_secret_public_key: str | None = None)`

**Bases**: `_BauplanData`

- **default_parameter_secret_key**: `str | None`
- **default_parameter_secret_public_key**: `str | None`
- **id**: `str | None`
- **model_computed_fields**: `ClassVar[Dict[str, ComputedFieldInfo]] = {}`\
  A dictionary of computed field names and their corresponding `ComputedFieldInfo` objects.
- **model_config**: `ClassVar[ConfigDict] = {}`\
  Configuration for the model, should be a dictionary conforming to `[ConfigDict][pydantic.config.ConfigDict]`.
- **model_fields**: `ClassVar[Dict[str, FieldInfo]] = {'default_parameter_secret_key': FieldInfo(annotation=Union[str, NoneType], required=False, default=None), 'default_parameter_secret_public_key': FieldInfo(annotation=Union[str, NoneType], required=False, default=None), 'id': FieldInfo(annotation=Union[str, NoneType], required=False, default=None), 'name': FieldInfo(annotation=Union[str, NoneType], required=False, default=None), 'slug': FieldInfo(annotation=Union[str, NoneType], required=False, default=None)}`\
  Metadata about the fields defined on the model, mapping of field names to `[FieldInfo][pydantic.fields.FieldInfo]` objects. Replaces `Model.__fields__` from Pydantic V1.
- **name**: `str | None`
- **slug**: `str | None`


## class bauplan.Profile

**Signature**: `bauplan.Profile(name: 'Optional[str]' = None, api_key: 'Optional[str]' = None, user_session_token: 'Optional[str]' = None, project_dir: 'Optional[Union[str, Path]]' = None, branch: 'Optional[str]' = None, namespace: 'Optional[str]' = None, cache: "Optional[Literal['on', 'off']]" = None, debug: 'Optional[bool]' = None, verbose: 'Optional[bool]' = None, args: 'Optional[Dict[str, str]]' = None, api_endpoint: 'Optional[str]' = None, catalog_endpoint: 'Optional[str]' = None, catalog_max_records: 'Optional[int]' = None, client_timeout: 'Optional[int]' = None, env: 'Optional[str]' = None, config_file_path: 'Optional[Union[str, Path]]' = None, feature_flags: 'Optional[Dict[str, Any]]' = None)`

**Bases**: `object`

- **api_endpoint**: `str`
- **api_key**: `str | None`
- **args**: `Dict[str, str] | None`
- **branch**: `str | None`
- **cache**: `str | None`
- **catalog_endpoint**: `str`
- **catalog_max_records**: `int`
- **client_timeout**: `int | None`
- **config_file_path**: `str | Path | None`
- **debug**: `bool | None`
- **env**: `str | None`
- **feature_flags**: `Dict[str, str]`
- **name**: `str | None`
- **namespace**: `str | None`
- **project_dir**: `str | Path | None`
- **user_session_token**: `str | None`
- **verbose**: `bool | None`

**classmethod load_profile**:

**Signature**: `load_profile(profile: str | None = None, api_key: str | None = None, user_session_token: str | None = None, project_dir: str | Path | None = None, branch: str | None = None, namespace: str | None = None, cache: 'on' | 'off' | None = None, debug: bool | None = None, verbose: bool | None = None, args: dict[str, str] | None = None, api_endpoint: str | None = None, catalog_endpoint: str | None = None, catalog_max_records: int | None = None, client_timeout: int | None = None, env: str | None = None, config_file_path: str | Path | None = None, feature_flags: dict[str, Any] | None = None) → Profile`

Load a profile from a profile file.

## class bauplan.RunnerNodeInfo

**Signature**: `bauplan.RunnerNodeInfo(*, hostname: str | None = None)`

**Bases**: `_BauplanData`

- **hostname**: `str | None`
- **model_computed_fields**: `ClassVar[Dict[str, ComputedFieldInfo]] = {}`  
  A dictionary of computed field names and their corresponding `ComputedFieldInfo` objects.
- **model_config**: `ClassVar[ConfigDict] = {}`  
  Configuration for the model, should be a dictionary conforming to `[ConfigDict][pydantic.config.ConfigDict]`.
- **model_fields**: `ClassVar[Dict[str, FieldInfo]] = {'hostname': FieldInfo(annotation=Union[str, NoneType], required=False, default=None)}`  
  Metadata about the fields defined on the model, mapping of field names to `[FieldInfo][pydantic.fields.FieldInfo]` objects. Replaces `Model.__fields__` from Pydantic V1.

## class bauplan.UserInfo

**Signature**: `bauplan.UserInfo(*, id: str | None = None, username: str | None = None, first_name: str | None = None, last_name: str | None = None)`

**Bases**: `_BauplanData`

- **first_name**: `str | None`
- **full_name**: `str | None` (property)
- **id**: `str | None`
- **last_name**: `str | None`
- **model_computed_fields**: `ClassVar[Dict[str, ComputedFieldInfo]] = {}`  
  A dictionary of computed field names and their corresponding `ComputedFieldInfo` objects.
- **model_config**: `ClassVar[ConfigDict] = {}`  
  Configuration for the model, should be a dictionary conforming to `[ConfigDict][pydantic.config.ConfigDict]`.
- **model_fields**: `ClassVar[Dict[str, FieldInfo]] = {'first_name': FieldInfo(annotation=Union[str, NoneType], required=False, default=None), 'id': FieldInfo(annotation=Union[str, NoneType], required=False, default=None), 'last_name': FieldInfo(annotation=Union[str, NoneType], required=False, default=None), 'username': FieldInfo(annotation=Union[str, NoneType], required=False, default=None)}`  
  Metadata about the fields defined on the model, mapping of field names to `[FieldInfo][pydantic.fields.FieldInfo]` objects. Replaces `Model.__fields__` from Pydantic V1.
- **username**: `str | None`

## bauplan.expectation

**Signature**: `bauplan.expectation(**kwargs: Any) → Callable`

Decorator that defines a Bauplan expectation.

An expectation is a function from one (or more) dataframe-like object(s) to a boolean, commonly used to perform data validation and data quality checks when running a pipeline. Expectations take as input the table(s) they are validating and return a boolean indicating whether the expectation is met or not. A Python expectation needs a Python environment to run, which is defined using the `python` decorator.

**Example**:

```python
@bauplan.expectation()
@bauplan.python('3.10')
def test_joined_dataset(
    data=bauplan.Model(
        'join_dataset',
        columns=['anomaly']
    )
):
    # your data validation code here
    return expect_column_no_nulls(data, 'anomaly')
```

**Parameters**:

- **f**: The function to decorate.

## bauplan.extras

**Signature**: `bauplan.extras(*args) → Callable`

Decorator that defines the Bauplan package extras to install.

This decorator allows specifying which optional feature sets (extras) of the Bauplan package are required by the decorated function. For example, using `@bauplan.extras('ai')` will request the installation of AI-specific functionalities, ensuring the right dependencies are installed.

**Parameters**:

- **args**: A variable list of strings, where each string is the name of an extra to install (e.g., `'ai'`, `'prefect'`).

## bauplan.model

**Signature**: `bauplan.model(name: str | None = None, columns: list[str] | None = None, partitioned_by: str | list[str] | tuple[str, ...] | None = None, materialization_strategy: 'NONE' | 'REPLACE' | 'APPEND' | None = None, cache_strategy: 'NONE' | 'DEFAULT' | None = None, internet_access: bool | None = None, **kwargs: Any) → Callable`

Decorator that specifies a Bauplan model.

A model is a function from one (or more) dataframe-like object(s) to another dataframe-like object, used to define a transformation in a pipeline. Models are chained together implicitly by using them as inputs to their children. A Python model needs a Python environment to run, which is defined using the `python` decorator.

**Example**:

```python
@bauplan.model(
    columns=['*'],
    materialization_strategy='NONE'
)
@bauplan.python('3.11')
def source_scan(
    data=bauplan.Model(
        'iot_kaggle',
        columns=['*'],
        filter="motion='false'"
    )
):
    # your code here
    return data
```

**Parameters**:

- **name**: `str | None = None`  
  The name of the model (e.g., `'users'`); if missing, the function name is used.
- **columns**: `list[str] | None = None`  
  The columns of the output dataframe after the model runs (e.g., `['id', 'name', 'email']`). Use `['*']` as a wildcard.
- **internet_access**: `bool | None = None`  
  Whether the model requires internet access.
- **partitioned_by**: `str | list[str] | tuple[str, ...] | None = None`  
  The columns to partition the data by.
- **materialization_strategy**: `'NONE' | 'REPLACE' | 'APPEND' | None = None`  
  The materialization strategy to use.
- **cache_strategy**: `'NONE' | 'DEFAULT' | None = None`  
  The cache strategy to use.

## bauplan.pyspark

**Signature**: `bauplan.pyspark(version: str | None = None, conf: dict[str, str] | None = None, **kwargs: Any) → Callable`

Decorator that makes a PySpark session available to a Bauplan function (a model or an expectation). Add a `spark=None` parameter to the function model args.

**Parameters**:

- **version**: `str | None = None`  
  The version string of PySpark.
- **conf**: `dict[str, str] | None = None`  
  A dict containing the PySpark configuration.

## bauplan.python

**Signature**: `bauplan.python(version: str | None = None, pip: dict[str, str] | None = None, **kwargs: Any) → Callable`

Decorator that defines a Python environment for a Bauplan function (e.g., a model or expectation). It is used to specify directly in code the configuration of the Python environment required to run the function, i.e., the Python version and the Python packages required.

**Parameters**:

- **version**: `str | None = None`  
  The Python version for the interpreter (e.g., `'3.11'`).
- **pip**: `dict[str, str] | None = None`  
  A dictionary of dependencies (and versions) required by the function (e.g., `{'requests': '2.26.0'}`).

## bauplan.resources

**Signature**: `bauplan.resources(cpus: int | float | None = None, memory: int | str | None = None, memory_swap: int | str | None = None, timeout: int | None = None, **kwargs: Any) → Callable`

Decorator that defines the resources required by a Bauplan function (e.g., a model or expectation). It is used to specify directly in code the configuration of the resources required to run the function.

**Parameters**:

- **cpus**: `int | float | None = None`  
  The number of CPUs required by the function (e.g., `0.5`).
- **memory**: `int | str | None = None`  
  The amount of memory required by the function (e.g., `'1G'`, `1000`).
- **memory_swap**: `int | str | None = None`  
  The amount of swap memory required by the function (e.g., `'1G'`, `1000`).
- **timeout**: `int | None = None`  
  The maximum time the function is allowed to run (e.g., `60`).

