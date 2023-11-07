# IsWiX Worker Service Tutorial

This tutorial will walk you through how to build a [Windows Installer](https://msdn.microsoft.com/en-us/library/windows/desktop/cc185688(v=vs.85).aspx) (aka .MSI) setup package for a typical .NET Worker Service.  It is assumed you have read the [Overview](https://github.com/iswix-llc/iswix-tutorials) instructions first.

## Create the Worker Service

### Launch Visual Studio and create a new Worker Service

![Application New Project](/Images/worker-service/appproject1.png)

* Use File | New | Project and select the C# Worker SErvice template.
  * The available project templates vary from one version of Visual Studio to another.  Select a similar project template if needed.
* Target .NET 7   . You can select a different version but you will have an additional step later in the process.
* Call the project worker-service.  
* Point the Location to the root of your GitHub repos or to a location in your version control system.
* Call the solution Application and make sure the "Create directory for solution" option is selected.

### Author a Publish Profile:
![test](/Images/worker-service/apppublish1.png)

* Using the Solution Explorer, right click your worker service project and select the publish option then select folder target and click Finish.

![test](/Images/worker-service/apppublish2.png)

* Set the folder location to `..\..\Installer\Deploy` and click Finish again.

![test](/Images/worker-service/apppublish3.png)

* Click Close.

![test](/Images/worker-service/apppublish4.png)

* Set the target framework to .NET 7.0.
* Set the deployment model to Self-Contained.
* Set the target runtime to win-x64.
* Click Save and then click Publish to publish your application.

This will help us create a directory structure that models what your deployed application should look like.  Go ahead and take a look at that directory in your workspace.

![test](/Images/worker-service/apppublish5.png)

## Create the Installer

TODO: Work In Progress
