# Module Startup Template

This template can be used to create a **reusable [application module](../../modules)** based on the [module development best practices & conventions](../../framework/architecture/best-practices). It is also suitable for creating **microservices** (with or without UI).

## How to Start With?

You can use the [ABP CLI](../../cli) to create a new project using this startup template. Alternatively, you can generate a CLI command from the [Get Started](https://abp.io/get-started) page. CLI approach is used here.

First, install the ABP CLI if you haven't installed before:

```bash
dotnet tool install -g Volo.Abp.Studio.Cli
```

Then use the `abp new` command in an empty folder to create a new solution:

```bash
abp new-module Acme.BookStore
```

- `Acme.IssueManagement` is the solution name, like *YourCompany.YourProduct*. You can use single level, two-levels or three-levels naming.

### Specifying the User Interface

The template comes without a user interface by default. You can use the `mvc`, `blazor`, `blazor-server`, or `angular` options to include any of these UI layers. You can also combine them. For example, you can use `mvc,angular` to include both MVC and Angular UI. To create a module without a user interface, don't specify any value.

````bash
abp new-module Acme.IssueManagement -u mvc,angular
````

#### Specifying the Database Provider

The template comes with the *EntityFrameworkCore* database provider by default. You can use the `ef` or `mongodb` options to include either of these providers. You can also combine them. For example, you can use `ef,mongodb` to include both EntityFrameworkCore and MongoDB.

````bash
abp new-module Acme.IssueManagement -d ef,mongodb
````

## Solution Structure

Based on the options you've specified, you will get a slightly different solution structure. If you don't specify any option, you will have a solution like shown below:

![issuemanagement-module-solution](../../images/issuemanagement-module-solution.png)

Projects are organized as `src` and`test` folders:

* `src` folder contains the actual module which is layered based on [DDD](../../framework/architecture/domain-driven-design) principles.
* `test` folder contains unit & integration tests.

The diagram below shows the layers & project dependencies of the module:

![layered-project-dependencies-module](../../images/layered-project-dependencies-module.png)

Each section below will explain the related project & its dependencies.

### .Domain.Shared Project

This project contains constants, enums and other objects these are actually a part of the domain layer, but needed to be used by all layers/projects in the solution.

An `IssueType` enum and an `IssueConsts` class (which may have some constant fields for the `Issue` entity, like `MaxTitleLength`) are good candidates for this project.

- This project has no dependency to other projects in the solution. All other projects depend on this directly or indirectly.

### .Domain Project

This is the domain layer of the solution. It mainly contains [entities, aggregate roots](../../framework/architecture/domain-driven-design/entities.md), [domain services](../../framework/architecture/domain-driven-design/domain-services.md), value types, [repository interfaces](../../framework/architecture/domain-driven-design/repositories.md) and other domain objects.

An `Issue` entity, an `IssueManager` domain service and an `IIssueRepository` interface are good candidates for this project.

- Depends on the `.Domain.Shared` because it uses constants, enums and other objects defined in that project.

### .Application.Contracts Project

This project mainly contains [application service](../../framework/architecture/domain-driven-design/application-services.md) **interfaces** and [Data Transfer Objects](../../framework/architecture/domain-driven-design/data-transfer-objects.md) (DTO) of the application layer. It does exists to separate interface & implementation of the application layer. In this way, the interface project can be shared to the clients as a contract package.

An `IIssueAppService` interface and an `IssueCreationDto` class are good candidates for this project.

- Depends on the `.Domain.Shared` because it may use constants, enums and other shared objects of this project in the application service interfaces and DTOs.

### .Application Project

This project contains the [application service](../../framework/architecture/domain-driven-design/application-services.md) **implementations** of the interfaces defined in the `.Application.Contracts` project.

An `IssueAppService` class is a good candidate for this project.

- Depends on the `.Application.Contracts` project to be able to implement the interfaces and use the DTOs.
- Depends on the `.Domain` project to be able to use domain objects (entities, repository interfaces... etc.) to perform the application logic.

### .EntityFrameworkCore Project

This is the integration project for EF Core. It defines the `DbContext` and implements repository interfaces defined in the `.Domain` project.

- Depends on the `.Domain` project to be able to reference to entities and repository interfaces.

> You can delete this project if you don't want to support EF Core for your module.

### .MongoDB Project

This is the integration project for MongoDB.

- Depends on the `.Domain` project to be able to reference to entities and repository interfaces.

> You can delete this project if you don't want to support MongoDB for your module.

### Test Projects

The solution has multiple test projects, one for each layer:

- `.Domain.Tests` is used to test the domain layer.
- `.Application.Tests` is used to test the application layer.
- `.EntityFrameworkCore.Tests` is used to test EF Core configuration and custom repositories.
- `.MongoDB.Tests` is used to test MongoDB configuration and custom repositories.
- `.TestBase` is a base (shared) project for all tests.

Test projects are prepared for integration testing;

- It is fully integrated to ABP and all services in your application.
- It uses SQLite in-memory database for EF Core. For MongoDB, it uses the [EphemeralMongo](https://github.com/asimmon/ephemeral-mongo) library.
- Authorization is disabled, so any application service can be easily used in tests.

You can still create unit tests for your classes which will be harder to write (because you will need to prepare mock/fake objects), but faster to run (because it only tests a single class and skips all initialization process).

> Domain & Application tests are using EF Core. If you remove EF Core integration or you want to use MongoDB for testing these layers, you should manually change project references & module dependencies.

### Host Project

The solution doesn't have a host application to run your module. However, you can create a [single-layer](../../get-started/single-layer-web-application.md) or [layered](../../get-started/layered-web-application.md) application and [import](../../studio/solution-explorer.md#imports) the created module into the host application.

## UI

### Angular UI

The solution will have a folder called `angular` in it. This is where the Angular client-side code is located. When you open that folder in an IDE, the folder structure will look like below:

![Folder structure of ABP Angular module project](../../images/angular-module-folder-structure.png)

* _angular/projects/issue-management_ folder contains the Angular module project.
* _angular/projects/dev-app_ folder contains a development application that runs your module.

The server-side is similar to the solution described above. After you create a *Host* application, the API and the `Angular` demo application consume it.

#### How to Run the Angular Development App

For module development, you will need the `dev-app` project up and running. So, here is how we can start the development server.

First, we need to install dependencies:

1. Open your terminal at the root folder, i.e. `angular`.
2. Run `yarn` or `npm install`.

The dependencies will be installed and some of them are ABP modules published as NPM packages. To see all ABP packages, you can run the following command in the `angular` folder:

```bash
yarn list --pattern abp
```

> There is no equivalent of this command in npm.

The module you will develop depends on two of these ABP packages: _@abp/ng.core_ and _@abp/ng.theme.shared_. Rest of the ABP modules are included in _package.json_ because of the `dev-app` project.

Once all dependencies are installed, follow the steps below to serve your development app:

1. Make sure *Host* application project is up and running.
2. Change the `environment.ts` file in the `angular/projects/dev-app/src/environments` folder to match your *Host* application URL.
3. Open your terminal at the root folder, i.e. `angular`.
4. Run `yarn start` or `npm start`.

![ABP Angular module dev-app project](../../images/angular-module-dev-app-project.png)

The issue management page is empty in the beginning. You may change the content in `IssueManagementComponent` at the _angular/projects/issue-management/src/lib/issue-management.component.ts_ path and observe that the view changes accordingly.

Now, let's have a closer look at some key elements of your project.

#### Main Module

`IssueManagementModule` at the _angular/projects/issue-management/src/lib/issue-management.module.ts_ path is the main module of your module project. There are a few things worth mentioning in it:

- Essential ABP modules, i.e. `CoreModule` and `ThemeSharedModule`, are imported.
- `IssueManagementRoutingModule` is imported.
- `IssueManagementComponent` is declared.
- It is prepared for configurability. The `forLazy` static method enables [a configuration to be passed to the module when it is loaded by the router](https://volosoft.com/blog/how-to-configure-angular-modules-loaded-by-the-router).


#### Main Routing Module

`IssueManagementRoutingModule` at the _angular/projects/issue-management/src/lib/issue-management-routing.module.ts_ path is the main routing module of your module project. It currently does two things:

- Loads `DynamicLayoutComponent` at base path it is given.
- Loads `IssueManagementComponent` as child to the layout, again at the given base path.

You can rearrange this module to load more than one component at different routes, but you need to update the route provider at _angular/projects/issue-management/config/src/providers/route.provider.ts_ to match the new routing structure with the routes in the menu. Please check [Modifying the Menu](../../framework/ui/angular/modifying-the-menu.md) to see how route providers work.

#### Config Module

There is a config module at the _angular/projects/issue-management/config/src/issue-management-config.module.ts_ path. The static `forRoot` method of this module is supposed to be called at the route level. So, you may assume the following will take place:

```js
@NgModule({
  imports: [
    /* other imports */

    IssueManagementConfigModule.forRoot(),
  ],

  /* rest of the module meta data */
})
export class AppModule {}
```

You can use this static method to configure an application that uses your module project. An example of such configuration is already implemented and the `ISSUE_MANAGEMENT_ROUTE_PROVIDERS` token is provided here. The method can take options which enables further configuration possibilities.

The difference between the `forRoot` method of the config module and the `forLazy` method of the main module is that, for smallest bundle size, the former should only be used when you have to configure an app before your module is even loaded.

#### Testing Angular UI

Please see the [testing document](../../framework/ui/angular/testing.md).
