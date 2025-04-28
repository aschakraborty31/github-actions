# CUS Orchestrator API

This project is a .Net API that manages the state of operations perfroms by the CUS Orchestrator.

# Orchestrator API Commands

This project writes data received from broker into database

# Orchestrator API Query

This project reads data from database and passed it to broker

# Orchestrator SQS Commands
 This project writes data received from tenant inventory into database

# Orchestrator SQS Publishers
This project publishes state changes through a SNS to tenant inventory

# Orchestrator SQS Subscription
This project subscribes to Tenant Inventory SNS 

## Local Machine Setup

> [!IMPORTANT]
> In order to pull dependencies for this project, you will need to enable pulling NuGet and Docker images from GitHub. Follow steps in the following document for configuration: 
* [Authentication for GitHub Packages](https://hyland.atlassian.net/wiki/x/mhj3TQ)
* [Authenctication for Hyland Experience Packages](https://hyland.atlassian.net/wiki/spaces/SEUTG/pages/2459009192/Development+Setup+Guide)

Once that is completed, you should be able to run the following commands to build and test the project.

```txt
dotnet tool restore
dotnet build
docker compose up -d
dotnet test
```

### Additional Tools to Install

* [AWS Toolkit](https://aws.amazon.com/visualstudio/) for Testing Lambda
* [psql](https://www.postgresql.org/docs/current/app-psql.html) for running PostgresSQL terminal commands.
* Postgres visual tool like [DBeaver](https://dbeaver.io/) or [PGAdmin](https://www.pgadmin.org/)

## Integration Tests

When running the integration tests locally, make sure you run `docker compose up` (pass the `-d` param for background.) to start the dependent services for the integration tests.

Services that run: Postgres, Redis, [Mock IAM API](https://hyland.atlassian.net/wiki/spaces/SAASD/pages/1308033548/Use+the+IAM+API+Mock+Docker+image) (see [Authentication for GitHub packages](https://hyland.atlassian.net/wiki/spaces/HXP/pages/1308039322/Authentication+for+GitHub+Packages) to pull GitHub Docker images).

## Database

These steps expect the Postgres to be running via Docker. Use `docker compose up` to run with expected settings. The commands use the Postgres Docker image with the `psql` command to keep consistent with how the same commands are run in CI pipelines.

The default local database name is `fccseu-orchestrator`. The `./.local/postgres/data` directory is expected to be present when using Docker.

### Create migrations

Any time the entities are changed, new migrations for the containing DbContext must be run and checked into source.

The migration script will create a class migrations and a SQL script which can be applied in CI pipelines. You will change relevant entities, update model creation logic and then run this command to make a new migration with the given name.

Note that any migration should be forwards compatible with the current version of the hosted service. This means destructive changes like renames or removals existing fields or tables need to follow a multi-phased approach to expand and contract the DB schema over multiple releases.

> [!NOTE]  
> You may need to change PowerShell to allow execution of the script.
> `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`

```sh
 ./scripts/create-migration.ps1 -name MigrationName
```

### Apply local migrations

This script will also ensure the database exists.

#### From Docker

No additional tools need to be installed for this command.

```sh
docker run -it --rm --volume "${PWD}:/var/app" postgres:latest /bin/bash -c "/var/app/scripts/migrate-database.sh"
```

#### From Bash

This will require `psql` is installed on the host machine.

```sh
./scripts/migrate-database.sh . "postgres://postgres:password@localhost:5432"
```

### Mock IAM API

The [Mock IAM API](https://hyland.atlassian.net/wiki/spaces/SAASD/pages/1308033548/Use+the+IAM+API+Mock+Docker+image) is configured to run with Docker Compose. It can be used for integration and local manual testing.

## Deployment

### Deployment to Dev and Staging

This project follows mainline development and will deploy to `sandbox` upon merge to `main`. Once sandbox is successful a deployment to  `dev` and `staging` will then be triggered.

### Feature Branch Deployment to Sandbox

Deployment to `sandbox` can be triggered by adding a `FEATUREDEPLOY` label to an open pull request. This will trigger the feature-branch-deploy workflow and push proposed changes for testing before merge to `main`.

## Connecting to API Host

Locally, you can connect to the `http://localhost:5000/index.html` to do a basic test.

In hosted environments you can connect to the address specified by the API Gateway address. Refer to the deployment directory for this configuration.

## DataDog Search

After you've consumed API endpoints on the service, you can find relevant details in DataDog. Filter traces and logs using `fccseu-api` service.

[Sandbox Traces](https://hxp.datadoghq.com/apm/traces?query=%40_top_level%3A1%20env%3Asandbox%20service%3Afccseu-api%20-%40http.route%3A%22%2Fkubernetes%2Fhealth%22)

[Dev Traces](https://hxp.datadoghq.com/apm/traces?query=%40_top_level%3A1%20env%3Adev%20service%3Afccseu-api%20-%40http.route%3A%22%2Fkubernetes%2Fhealth%22)

[Staging Traces](https://hxp.datadoghq.com/apm/traces?query=%40_top_level%3A1%20env%3Astaging%20service%3Afccseu-api%20-%40http.route%3A%22%2Fkubernetes%2Fhealth%22)

## Configuration

### Networking Configuration

The NetworkConfig class contains settings crucial for configuring the ForwardedHeadersOptions middleware, which handles headers forwarded by proxy servers.

The NetworkConfig class contains the following properties:

```cs
public class NetworkConfig
{
    public int TrustedHops { get; set; } = 2;
    public string KnownNetwork { get; set; } = "127.0.0.1";
    public int NetworkRange { get; set; } = 32;
}
```

#### Properties

| Property       | Type     | Description                                                                                       |
|----------------|----------|---------------------------------------------------------------------------------------------------|
| `TrustedHops`  | `int`    | Specifies the number of proxy hops to trust when forwarding headers. This setting is important in a network where requests pass through multiple layers before reaching your application. For example, an API Gateway counts as a proxy because it adds a forwarded header, the EKS Network Load Balancer (NLB) does not because it doesn't modify the headers, but NGINX does as it acts as another proxy. Therefore, if your application sits behind an API Gateway and an NGINX proxy, you would set TrustedHops to 2 to trust the headers from both proxies. |
| `KnownNetwork` | `string` | Defines the IP address of a trusted network. This address is used to identify and trust traffic coming from specific network ranges. |
| `NetworkRange` | `int`    | Defines the CIDR range of the known network. This range, in combination with `KnownNetwork`, specifies a subnet of trusted IP addresses. |

#### Example Configuration

To configure these settings, add a Networking section to your appsettings.json file:

```json
{
  "Networking": {
    "TrustedHops": 2,
    "KnownNetwork": "10.0.0.0",
    "NetworkRange": 16
  }
}
```# FCC SEU Contributing Guidelines

Refer to the FCC SEU Confluence for Team Git Standards

<https://hyland.atlassian.net/wiki/spaces/HFCH/pages/2467365893/Team+Git+Standards>
