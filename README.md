# Piranha Core CMS with MySQL as Database

This is a simple project demonstrating how to use a MySQL database instead of SQLLite with the (Piranha Core CMS)[https://github.com/piranhacms/piranha.core] in its version `2.2.7.2`.

The motivaton behind is not that SQLite is unpreferrable - it is simply that a hosting provider's default database type could be MySQL which might be provided with the luxory of automated backups and default encryption which saves quite some maintenace and securing efforts.

## Dev Stack

The tutorial was created on a Mac running macOS Sierra. MySQL was installed including MySQL Workbench. The MySQL server was used on the machine itself using the default settings of `localhost` at `port 3306`. The database user was `root` where you should pick your own password - the tutorial password is *password*.

As IDE Visual Studio Code was used with `.NET Command Line Tools (2.1.104)` installed.

## How to Make it Work

### Prepare the Database

You either have an existing database at your local machine or remote server or you might need to create a new one. A few cases are considered below. All have in common that you need the following data for your database connection string:

- Server
- Port
- Name of database / schema
- User name / uid
- Password

which are combined to a connection string like

```csharp
"server=<THE-HOST-URL>;port=<THE-PORT>;database=<THE-DATABASE-NAME>;uid=<THE-DATABASE-USER>;password=<THE-PASSWORD>"
```

#### Existing Database on a Remote Server

Get the database server data as decribed above.

#### Mac

Install MySQL and MySQL Workbench and create a new database schema on your local machine. Enter a name for the schema, e.g. `piranha-mysql`. You do not neccessarily need to choose a Default Collation but if so use `Latin-1 Default Collation` or `UTF-8 Default Collation`.

Now that the schema is set up get the database connection properties as described above resulting is a connection string, such as

```csharp
"server=localhost;port=3306;database=piranha-mysql;uid=root;password=password"
```

Keep that string in mind for later and make sure that the MySQL server is running (see System Preferences > MySQL).

### Project Setup

> **Note:** **Do not** call `dontnet run` yet - call it after the setup of EF Core with MySQL was finished.

Set up the project as you would normally do, either by your own or use the project template. In this tutorial, we will use the template that comes with Piranha Core.

If you want to use the template make sure it is installed

```bash
dotnet new -i Piranha.BasicWeb.CSharp
```

Create a new folder with your desired project name, e.g. `piranha-mysql`. Now `cd` with the terminal to that folder and run

```bash
dotnet new piranha
```

You can also open your newly create folder in Visual Studio Code and user the build in terminal which is already in the correct location.

Finally, restore all packages using

```bash
dotnet restore
```

Now that the project is setup we can continue to use MySQL. 

### EF Core with MySQL

MySQL is not supported by default by EF Core but there are mainly two Nuget packages out there which extend EF Core to be able to talk to a MySQL database:

1. Pomelo.EntityFrameworkCore.MySql
2. MySql.Data.EntityFrameworkCore

Currently, only the variant using `Pomelo.EntityFrameworkCore.MySql` works as the package provided by MySQL itself throws an error while mapping the GUID values. The issue was disscussed at [this Stackoverflow post](https://stackoverflow.com/questions/45120152/guid-property-on-mysql-entity-framework).

The problem with the Pomelo package is, that it was not updated in a while and that they are actively [looking for contributers](https://github.com/PomeloFoundation/Pomelo.EntityFrameworkCore.MySql/issues/522). Nevertheless, the package works like a charm in its current version `2.0.1`.

So, let's add the package to the project

```bash
dotnet add package Pomelo.EntityFrameworkCore.MySql
```

and perform a `dotnet restore`.

Now go to the `Starup.cs` class in your project folder and replace in the `public IServiceProvider ConfigureServices(IServiceCollection services)` method the following line which was added by the template

```csharp
services.AddDbContext<Db>(options => 
    options.UseSqlite("Filename=./piranha.db"));
```

with the following line using the connection string to your MySQL database connection string from the preparations (see above).

```csharp
services.AddDbContext<Db>(options =>
    options.UseMySql("server=localhost;port=3306;database=piranha-mysql;uid=root;password=password"));
```

That's it if no error was thrown. Piranha will create the database tables in the  MySQL database instead of the default SQLite database.

In order to check if it worked, call

```bash
dotnet run
```

and visit the site using the default url `localhost:5000` or the one you might have set.

## Helpful and Related Sources:

- https://medium.com/@balramchavan/setup-entity-framework-core-for-mysql-in-asp-net-core-2-5b40a5a3af94
- https://damienbod.com/2016/08/26/asp-net-core-1-0-with-mysql-and-entity-framework-core/
