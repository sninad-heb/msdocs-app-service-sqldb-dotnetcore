---
languages:
- csharp
- aspx-csharp
- bicep
page_type: sample
products:
- azure
- aspnet-core
- azure-app-service
- azure-database-postgresql
- azure-virtual-network
urlFragment: msdocs-app-service-sqldb-dotnetcore
name: Deploy an ASP.NET Core web app with PostgreSQL in Azure
description: "A sample ASP.NET Core application configured to use PostgreSQL with Entity Framework Core."
---

# ASP.NET Core web app with PostgreSQL

This ASP.NET Core application uses Entity Framework Core with PostgreSQL. The dev container includes a local PostgreSQL database; Azure infrastructure is not included in this repository.
## Run in Azure
> [!IMPORTANT]
Provision an Azure App Service and Azure Database for PostgreSQL Flexible Server separately. Configure network access between them, then set the App Service application setting `ConnectionStrings__MyDbConnection` to the PostgreSQL connection string. Keep the password in an application setting or secret store, and require TLS for the Azure database connection.

Apply the schema from a trusted environment that can reach the database:

```shell
dotnet ef database update
```

The migrations in [Migrations](Migrations) are PostgreSQL-specific. Do not use the former SQL Server migrations to create or update a PostgreSQL database.
    azd up
    ```

    It will prompt you to create a deployment environment name, pick a subscription, and provide a location (like `westeurope`). Then it will provision the resources in your account and deploy the latest code. If you get an error with deployment, changing the location (like to "centralus") can help, as there may be availability constraints for some of the resources.

1. When `azd` has finished deploying, you'll see an endpoint URI in the command output. Visit that URI, and you should see the Todo app! 🎉 If you see an error, open the Azure Portal from the URL in the command output, navigate to the App Service, select Logstream, and check the logs for any errors.

1. When you've made any changes to the app code, you can just run:

    ```shell
    azd deploy
    ```

## How is database migrations automated?

The [AZD template](infra/resources.bicep) in this repo secures the database in a virtual network through a private endpoint. The web app can access the database through the private endpoint because it's integrated with the virtual network. In this architecture, the simplest way to do database migrations is directly from within the web app itself.

Because the Linux .NET container in App Service doesn't come with the .NET SDK, you cannot run the migrations command `dotnet ef database update` easily. However, you can upload a [self-contained migrations bundle](https://learn.microsoft.com/ef/core/managing-schemas/migrations/applying?tabs=dotnet-core-cli#bundles). This repo automates the deployment of the migrations bundle as follows:

- In [azure.yaml](azure.yaml), use the `prepackage` hook to generate a *migrationsbundle* file with `dotnet ef migrations bundle`.
- In the [.csproj](DotNetCoreSqlDb.csproj) file, include the generated *migrationsbundle* file. During the `azd package` stage, *migrationsbundle* will be added to the deploy package.
- In [infra/resources.bicep](infra/resources.bicep), add the `appCommandLine` property to the web app to run the uploaded *migrationsbundle*.

## Getting help

If you're working with this project and running into issues, please post in [Issues](/issues).
