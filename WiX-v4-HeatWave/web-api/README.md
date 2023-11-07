# IsWiX ASP.NET Core Web API Tutorial

This tutorial will walk you through how to build a [Windows Installer](https://msdn.microsoft.com/en-us/library/windows/desktop/cc185688(v=vs.85).aspx) (aka .MSI) setup package for a typical ASP.NET Core Web API.  It is assumed you have read the [Overview](https://github.com/iswix-llc/iswix-tutorials) instructions first.

## Create the ASP.NET Core Web API

### Launch Visual Studio and create a new ASP.NET Core Web API

![Application New Project](/Images/web-api/appproject1.png)

* Use File | New | Project and select the C# ASP.NET Core Web API template.
  * The available project templates vary from one version of Visual Studio to another.  Select a similar project template if needed.
* Call the project Api.  
* Point the Location to the root of your GitHub repos.
* Call the solution Application and make sure the "Create directory for solution" option is selected.
* Target .NET 7   . You can select a different version but you will have an additional step later in the process.

### Author a Publish Profile:
![test](/Images/web-api/apppublish1.png)

* Using the Solution Explorer, right click your Web API project and select the publish option then select folder target and click Finish.

![test](/Images/web-api/apppublish2.png)

* Set the folder location to `..\..\Installer\Deploy\WebSite` and click Finish again.

![test](/Images/web-api/apppublish3.png)

* Click Close.

![test](/Images/web-api/apppublish4.png)

* Set the target framework to .NET 7.0.
* Set the deployment model to Self-Contained.
* Set the target runtime to win-x64.
* Under File Publish Options select Delete all existing files prior to publish.
* Click Save and then click Publish to publish your application.

This will help us create a directory structure that models what your deployed application should look like.  Go ahead and take a look at that directory in your workspace.

![test](/Images/web-api/apppublish5.png)

## Create the Installer

TODO: Work In Progress
