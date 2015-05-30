#Guide for .Net4 EF6 oData SqlServerCompact WebAPI

###Versions

-	.Net 4
-	EntityFramework 6
-	Web API 2.2
-	oData V5
-	SqlServerCompact 4

###Sources

- [WebAPI with EF6](http://www.asp.net/web-api/overview/data/using-web-api-with-entity-framework)
- [oData v4 with Web Api](http://www.asp.net/web-api/overview/odata-support-in-aspnet-web-api/odata-v4/create-an-odata-v4-endpoint)
- [EF with SqlServer Compact: ](http://www.codeproject.com/Articles/680116/Code-First-with-SQL-CE)
- http://chsakell.com/2015/04/04/asp-net-web-api-feat-odata/

###Requirements

- [Visual Studio Community 2013](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx)
- [SQL Server Compact Toolbox](http://sqlcetoolbox.codeplex.com/)

###Create the solution

Create a new solution following File -> New -> Project -> Web, then choose `ASP.NET Web Application`. Name it with `Catalog`. Then in the upcoming dialogue box, choose Empty and check `Web API`, click OK.

###Install NuGet packages

Run the following command in the Package Manager Console.
```
PM> Install-Package Microsoft.AspNet.OData
PM> Install-Package EntityFramework
PM> Install-Package EntityFramework.SqlServerCompact 
```
###Create Models

Create a `Model` folder and class `CatalogModels.cs`

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using System.ComponentModel.DataAnnotations;
using System.Data.Entity;

namespace Catalog.Models
{
    public class Product
    {
        [Key]
        public int Id { get; set; }
        [Required]
        [StringLength(50)]
        public string Name { get; set; }
        [StringLength(4000)]
        public string Descr { get; set; }
        [Required]
        public bool Active { get; set; }

        public int GroupId { get; set; }
        public virtual Group Group { get; set; }

    }

    public class Group
    {
        [Key]
        public int Id { get; set; }
        [Required]
        [StringLength(50)]
        public string Name { get; set; }
        [StringLength(1000)]
        public string Descr { get; set; }
        [Required]
        public bool Active { get; set; }

        public virtual List<Product> Products { get; set; }

    }
}
```

### Add Controller Classes and the Data Context Class

Add controllers of type `Web API 2 OData Controller with actions, using Entity Framework` for each model class.

- Click the `+` button to add a Data Context Class called `CatalogContext.cs`
- Check `Use async controller actions`  (NOT SURE WE NEED WITH THIS W/ODATA)

> **Note:** With **Visual Studio 2013** you may get an error when clicking `Add` to create the controller, try rebuilding the project before creating each controller.

### App_Start/WebApiConfig

Add the following code to the end of the `Register` method:

```
  ODataModelBuilder builder = new ODataConventionModelBuilder();
  builder.EntitySet<Group>("Groups");
  builder.EntitySet<Product>("Products");
  config.MapODataServiceRoute(
      routeName: "ODataRoute",
      routePrefix: null,
      model: builder.GetEdmModel());
```
### Setup Web.config

```
<connectionStrings>
  <add name="CatalogContext" connectionString="Data Source=|DataDirectory|\Catalog.sdf" providerName="System.Data.SqlServerCe.4.0" />
</connectionStrings>
...
<entityFramework>
  <defaultConnectionFactory type="System.Data.Entity.Infrastructure.SqlCeConnectionFactory, EntityFramework">
    <parameters>
      <parameter value="System.Data.SqlServerCe.4.0" />
    </parameters>
  </defaultConnectionFactory>
  <providers>
    <provider invariantName="System.Data.SqlServerCe.4.0" type="System.Data.Entity.SqlServerCompact.SqlCeProviderServices, EntityFramework.SqlServerCompact" />
  </providers>
</entityFramework>
  <system.data>
  <DbProviderFactories>
    <remove invariant="System.Data.SqlServerCe.4.0" />
    <add name="Microsoft SQL Server Compact Data Provider 4.0" invariant="System.Data.SqlServerCe.4.0" description=".NET Framework Data Provider for Microsoft SQL Server Compact" type="System.Data.SqlServerCe.SqlCeProviderFactory, System.Data.SqlServerCe, Version=4.0.0.0, Culture=neutral, PublicKeyToken=89845dcd8080cc91" />
  </DbProviderFactories>
</system.data>
```

###Setup Migrations and Seed Database

`PM> Enable-Migrations`

Open Migrations/Configuration.cs.  Add `using Catalog.Models` and the following to the `Seed` method:

```
context.Groups.AddOrUpdate(x => x.Id,
    new Group() { Id = 1, Name = "Software Eng", Active=true },
    new Group() { Id = 2, Name = "Software Eng - Infrastructure", Active = true }
);

context.Products.AddOrUpdate(x => x.Id,
    new Product() { Id = 1, Name = "WebStorm 9", GroupId = 1, Active = true },
    new Product() { Id = 2, Name = "WebStorm 10", GroupId = 1, Active = true }
);
```

```
PM> Add-Migration Initial
PM> Update-Database
```

You should now see your data under the `App_Data` directory.  *(**Note**: You may need to click the `Show all files` icon in the Solution Explorer first.)*

###Explore the API (Optional)

Press F5 to run the application in debug mode. Go to `/api/products` 


