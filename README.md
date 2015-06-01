#Guide for .Net4.5 EF6 OData WebAPI SqlServerCompact

###Versions

-	Visual Studio Community 2013
-	.Net 4.5
-	EntityFramework 6
-	Web API 2.2
-	OData V5
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


### App_Start/WebApiConfig.cs

Change the code to the following:

```
namespace Catalog
{
    public static class WebApiConfig
    {
        public static void Register(HttpConfiguration config)
        {
            config.Routes.MapHttpRoute(
                name: "DefaultApi",
                routeTemplate: "api/{controller}/{id}",
                defaults: new { id = RouteParameter.Optional }
            );
           
        }
    }
}
```

### App_Start/ODataConfig.cs

Create the following class and replace it with this code:

```
using System.Web.Http;
using System.Web.OData.Extensions;
using System.Web.OData.Builder;
using Catalog.Models;

namespace Catalog
{
    public class ODataConfig
    {
        public static void Register(HttpConfiguration config)
        {
            config.MapHttpAttributeRoutes(); //This has to be called before the following OData mapping, so also before WebApi mapping

            ODataConventionModelBuilder builder = new ODataConventionModelBuilder();

            builder.EntitySet<Product>("Products");
            builder.EntitySet<Group>("Groups");

            config.MapODataServiceRoute("ODataRoute", "api", builder.GetEdmModel());
        }
    }
}
```

### Modify Web.config

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
namespace Catalog.Migrations
{
    using System.Data.Entity.Migrations;
    using Catalog.Models;

    internal sealed class Configuration : DbMigrationsConfiguration<Catalog.Models.CatalogContext>
    {
        public Configuration()
        {
            AutomaticMigrationsEnabled = false;
        }

        protected override void Seed(Catalog.Models.CatalogContext context)
        {
			context.Groups.AddOrUpdate(x => x.Id,
			    new Group() { Id = 1, Name = "Software Eng", Active=true },
			    new Group() { Id = 2, Name = "Software Eng - Infrastructure", Active = true }
			);
			context.Products.AddOrUpdate(x => x.Id,
			    new Product() { Id = 1, Name = "WebStorm 9", GroupId = 1, Active = true },
			    new Product() { Id = 2, Name = "WebStorm 10", GroupId = 1, Active = true }
			);
 
        }
    }
}
```

```
PM> Add-Migration Initial
PM> Update-Database
```

You should now see the database `Catalog.sdf` under the `App_Data` directory.  *(**Note**: You may need to click the `Show all files` icon in the Solution Explorer first.)*

> **Note:** as you are testing your models, you can always delete all the migration steps under the `Migrations` directory and delete the database and rerun the `Add-Migration Initial` and `Update-Database` steps.


### Explorer the Database

Install the Visual Studio [SQL Server Compact & SQLite Toolbox extension](https://sqlcetoolbox.codeplex.com/documentation), connect to the database, and explorer the tables and data.


### Add Controller Classes and the Data Context Class

Add controllers of type `Web API 2 OData Controller with actions, using Entity Framework` for each model class.

- Click the `+` button to add a Data Context Class called `CatalogContext.cs`
- Check `Use async controller actions`

> **Note:** With **Visual Studio 2013** you may get an error when clicking `Add` to create the controller, try rebuilding the project before creating each controller.

In Visual Studio 2013, the controller references old OData 3.5 libraries.  This will give you `406` errors.  You must change all the references starting with  `System.Web.Http.OData` references to `System.Web.OData`.


###Explore the API

Press F5 to run the application in debug mode. Go to `/api/Groups` 


###Expose the API to other devices

IIS Express defaults to `localhost` and does not expose API.  Do expose it, do the following:

- edit the IISExpress/config/applicationhost.config file.  Set the port to 8080.  Add the second `<binding>` element below with your hosting computers IP address.  If you have any other issues, take a look at this blog post:  http://bendetat.com/access-iis-express-from-another-machine.html

```
<site name="Catalog" id="3">
    <application path="/" applicationPool="Clr4IntegratedAppPool">
        <virtualDirectory path="/" physicalPath="C:\Users\bxe004\Documents\Visual Studio 2013\Projects\Catalog\Catalog" />
    </application>
    <bindings>
      <binding protocol="http" bindingInformation="*:8080:localhost" />
      <binding protocol="http" bindingInformation="*:8080:10.89.141.159" />
    </bindings>
</site>
```
