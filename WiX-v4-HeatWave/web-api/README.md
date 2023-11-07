# IsWiX ASP.NET Core Web API Tutorial

This tutorial will walk you through how to build a [Windows Installer](https://msdn.microsoft.com/en-us/library/windows/desktop/cc185688(v=vs.85).aspx) (aka .MSI) setup package for a typical ASP.NET Core Web API. It is assumed you have read the [Overview](https://github.com/iswix-llc/iswix-tutorials) instructions first.

## Create the ASP.NET Core Web API

### Launch Visual Studio and create a new ASP.NET Core Web API

Create a GitHub repos using the tool of your choice. Create or use a .gitignore file designed for Visual Studio file exclusions.

![Application New Project](/Images/web-api/appproject1.png)
 With no solution open, create a new project using the C# ASP.NET Core Web API template. The available project templates vary from one version of Visual Studio to another. Select a similar project template if needed.

![Application New Project Settings](/Images/web-api/appproject2.png)
Enter Api for the project name.
For location, enter the path to the root of the repos that you created. (C:\GitHub\Api)
Application for the solution name and make sure "Place solution and project in the same directory" is not selected.

![Application New Project Additional Settings](/Images/web-api/appproject3.png)
Target .NET 7 . You can select a different version but you will have an additional step later in the process.
Set the authentication type to none. You can implement a custom middleware later.
Unselect configure for HTTPS. This will be handled by IIS.
Unselect enable Docker.
The remaining options are your personal preference. I prefer Open API support, controllers and top-level statements.
Click Create

![Publish](/Images/web-api/apppublish1.png)
Using the Solution Explorer, right click your Api project and select the publish option.

![Publish Type](/Images/web-api/apppublish2.png)
Choose the Folder target publish type and click Next.

![Publish Location](/Images/web-api/apppublish3.png)
Set the folder location to `..\..\Installer\Deploy\WebSite` and click Finish again.

![Publish Created](/Images/web-api/apppublish4.png)
Visual Studio will inform you that a publish profile was created. You will use this with `dotnet publish` in your CI/CD pipeline. For now just click Close.

![Publish Settings](/Images/web-api/apppublish5.png)
Set the target framework to .NET 7.0.
Set the deployment model to Self-Contained. This will include a private copy of ASP.NET core with your application.  You will still need to install the IIS Hosting Bundle though.
Set the target runtime to win-x64.
Click publish your application.  This will help us create a directory structure that models what your deployed application should look like. Go ahead and take a look at that directory in your workspace.

## Create the Installer

TODO: Work In Progress
