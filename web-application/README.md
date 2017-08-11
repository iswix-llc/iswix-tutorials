# IsWiX Web Application Tutorial

This tutorial will walk you through how to build a [Windows Installer](https://msdn.microsoft.com/en-us/library/windows/desktop/cc185688(v=vs.85).aspx) (aka .MSI) setup package for a typical Windows IIS/ASP.NET web application.  It is assumed you have read the [Overview](https://github.com/iswix-llc/iswix-tutorials) instructions first.

## Create the Web Application

### Launch Visual Studio and create a new Web application:

![Application New Project](/Images/web-application/Web-NewProject.png)

* Use File | New | Project and select the C# ASP.NET Web (.NET Framework) template.
  * The available project templates vary from one version of Visual Studio to another.  Select a similar ASP.NET project template if needed.
* Target .NET Framework 4.5. You can select a different version but you will have an additional step later in the process.
* Call the project WebApp
* Point the Location to the root of your GitHub repos or to a location in your version control system.
   * For tools like Team Foundation Version Control you may want to point it to something like `C:\workspace\TeamProject\ApplicationName\main\` as this will give you the folder `main` that can the be branched to say `dev` and used by build definitions.   This isn't needed in GitHub since the branching model is implemented differently.
* Make sure the "Create directory for solution" option is selected.
* Make sure the "Add Files to Source Control" option is selected if using TFVC.

### Author a Publish Profile:
![test](/Images/web-application/Web-Publish.png)

Using the Solution Explorer, right click your Web project and select the publish option.

![test](/Images/web-application/Web-Publish2.png)

* Click the Custom option.

![test](/Images/web-application/Web-Publish3.png)

* Name it `Deploy`.

![test](/Images/web-application/Web-Publish4.png)

* Change the publish method to `File System`
* Set the target location to `..\..\Installer\Deploy\WebSite`
* Click Publish

This will help us create a directory structure that models what your deployed application should look like.  Go ahead and take a look at that directory in your workspace.

![test](/Images/web-application/Web-Publish5.png)

### Build Consideration

* We just manually built and published the Web Project using Visual Studio.  During an automated CI build, MSBuild will not automatically publish the profile during a build unless told to do so.  You may do accomplish this by doing one or the below:

  * Add the following properties to your projects .CSPROJ file: `<PropertyGroup><DeployOnBuild>true</DeployOnBuild>  <PublishProfile>Deploy</PublishProfile></PropertyGroup>`

  * Pass the following properties to MSBuild during the build: `/p:DeployOnBuild=true /p:PublishProfile=Deploy`


## Create the Installer

### Launch Visual Studio and create a new IsWiX Solution:

![Installer New Project](/Images/web-application/Installer-NewProject.png)

* Use File | New | Project and select the IsWiX Solution for WiX v3 template.
* The .NET Framework version isn't important here.  Perhaps one day someone will submit a pull request to help automate a step down the road based on the selection here.
* Call the project DesktopApplication.  (This will result in your installer being called DesktopApplication.msi.)
* Point the Location to the root of your GitHub repos or to a location in your version control system.
   * For tools like Team Foundation Version Control you may want to point it to something like `C:\workspace\TeamProject\ApplicationName\main\` as this will give you the folder `main` that can the be branched to say `dev` and used by build definitions.   This isn't needed in GitHub since the branching model is implemented differently.
* Make sure the "Create directory for solution" option is selected.
* Make sure the "Add Files to Source Control" option is selected if using TFVC.
