# IsWiX ASP.NET Core Web API Tutorial

This tutorial will walk you through how to build a [Windows Installer](https://msdn.microsoft.com/en-us/library/windows/desktop/cc185688(v=vs.85).aspx) (aka .MSI) setup package for a typical ASP.NET Core Web API. It is assumed you have read the [Overview](https://github.com/iswix-llc/iswix-tutorials) instructions first.

## Create the ASP.NET Core Web API

### Launch Visual Studio and create a new ASP.NET Core Web API

Create a GitHub repos using the tool of your choice. Create or use a .gitignore file designed for Visual Studio file exclusions.

![Application New Project](/Images/web-api/appproject1.png)
 * With no solution open, create a new project using the C# ASP.NET Core Web API template. The available project templates vary from one version of Visual Studio to another. Select a similar project template if needed.

![Application New Project Settings](/Images/web-api/appproject2.png)
* Enter Api for the project name.
* For location, enter the path to the root of the repos that you created. (C:\GitHub\Api)
* Application for the solution name and make sure "Place solution and project in the same directory" is not selected.

![Application New Project Additional Settings](/Images/web-api/appproject3.png)
* Target .NET 7 . You can select a different version but you will have an additional step later in the process.
* Set the authentication type to none. You can implement a custom middleware later.
* Unselect configure for HTTPS. This will be handled by IIS.
* Unselect enable Docker.
* The remaining options are your personal preference. I prefer Open API support, controllers and top-level statements.
* Click Create.

![Publish](/Images/web-api/apppublish1.png)
* Using the Solution Explorer, right click your Api project and select the publish option.

![Publish Type](/Images/web-api/apppublish2.png)
* Choose the Folder target publish type and click Next.

![Publish Location](/Images/web-api/apppublish3.png)
* Set the folder location to `..\..\Installer\Deploy\WebSite` and click Finish again.

![Publish Created](/Images/web-api/apppublish4.png)
* Visual Studio will inform you that a publish profile was created. You will use this with `dotnet publish` in your CI/CD pipeline. For now just click Close.
* Pro Tip: The publish profile is probably excluded from Git by your .gitignore.  Go ahead and manually add it to your commit or you may have issues in your CI/CD pipeline.

![Publish Settings](/Images/web-api/apppublish5.png)
* Set the target framework to .NET 7.0.
* Set the deployment model to Self-Contained. This will include a private copy of ASP.NET core with your application.  You will still need to install the IIS Hosting Bundle though.
* Set the target runtime to win-x64.
* Click publish your application.  This will help us create a directory structure that models what your deployed application should look like. 

![Published Files](/Images/web-api/apppublish6.png)

* Go ahead and take a look at that directory in your workspace.  These are the files that will be available to your installer.  
* Pro Tip:  You probably don't want to commit these files to Git so go ahead and add `/Installer/Deploy` to your .gitignore.

## Create the Installer
### Launch Visual Studio and create a new IsWiX Solution:

![Installer New Project](/Images/web-api/installer1.png)

* Use File | New | Project and select the IsWiX MSI Package (WiX v4) template.

![Installer New Project](/Images/web-api/installer2.png)

* Call the project WebApi.  (This will result in your installer being called WebApi.msi.)
* Point the Location to the root of your GitHub repos.
* Installer for the solution name and make sure "Place solution and project in the same directory" is not selected.
* Click Create

### Project Structure Overview:

![Mode](/Images/web-api/installer3.png)

### The IsWiX solution template created a solution with two projects based on the name:

* Setup Project 
   * Named WebApi
   * Creates an .MSI (Microsoft Installer) database that is installable by a user.
   * Contains a Product.wxs that expresses the Manufacturer, Product Name, Upgrade story, Feature Tree,  User Interface,  Installation Requirements and Installation path of the product.
   * Contains a WebApiFiles.wxs that expresses the files, subfolders, registry entries, shortcuts, services, environment variables and so on that will make up your product.  This was formally implemented as a merge module.
   
### Author the WiXApiFiles fragment:

![WiXApiFiles fragment before](/Images/web-api/installer3a.png)

Select the WebApiFiles.wxs (.wix source) in the WebApi project. It should already be opened by default.

Observe line 5: `<?define SourceDir="..\Deploy"?>` This will guide IsWiX in knowing where to find your applications files relative to the location of this wix project (.wixproj).  Our project structure choices cause this to automatically align with the output of our previous publish command.   

### Launch IsWiX:

![Launch IsWiX](/Images/web-api/installer3b.png)

Select Tools | Launch IsWiX to send the current active .wxs document to IsWiX.

![IsWiX Launched](/Images/web-api/installer4a.png)

* Our WebApiFiles.wxs document is now loaded in IsWiX.  
* We are given a single document interface (SDI) with multiple designers along the left side. 
* Each designer expresses a particular view of our .wxs file.  
   
### Click on the Files and Folders designer:

![IsWiX Files and Folder designer - before](/Images/web-api/installer4b.png)

* The top  two quadrants of this designer represent source files and folders that are on your developer machine  that can be included in the installation.  
* The bottom two quadrants represent what will be installed on the destination machine.  
* Right Click Destination Computer and create the `INSTALLLOCATION` folder.
* Notice that `<?define SourceDir="..\Deploy"?>` has been resolved to a fully qualified path and your files are in the upper right quadrant.
   * The use of relative path filenames will be critical when we get to build automation as all of your file path dependencies must be found on the build server for your solution to successfully build.
   * Our choice of Application.sln and Installer.sln is important also as we avoid tricky project dependency / build order race conditions.
   
### Author your files:

![IsWiX Files and Folder designer - After](/Images/web-api/installer5.png)

* Click on the `[INSTALLLOCATION]` folder in the destination view.
* Drag the file WebSite folder from the source files to the destination files quadrant.
* Press Control-S to save the changes back to disk and close the IsWiX application.
* This should cause Visual Studio to detect that the .wxs file was modified out of process.  Click Yes to cause it to reload in the IDE.

### Observe the XML changes

![IsWiX Merge Module - After](/Images/web-api/installer6.png)

* If you compare the XML before/after you should notice that IsWiX authored many elements for you. 
   * `Directory`, `Component` and `File` elements describing what your deployed website's directory tree looks like.
   * You may hadd additional child elements. This is an advanced topic that may be covered by another tutorial one day.
   * WiX will only include these files in the MSI.  If you later add content to your website you must use IsWiX to update the WebApiFiles.wxs.
     * New .CS files will be included in your Api.dll so no updates are needed.
     * Due to the way IsWiX sorts this document it is typically safe to simply remove the WebSite directory from the destination and redrag it to "rebase" the directory.  Compare the WebApiFiles.wxs against the previous version in source control and you should clearly see what the differences are. (Note: "rebasing" would lose any custom child elements you authored.)

### Explore IIS Support

![Installer -IISMeta.wxs](/Images/web-api/installer7.png)

* The IsWiX project template embedded a dormant (not referenced) IISMeta.wxs fragment in your WebApp project to help get you started in case you were creating a web application.. Take a look at it and observe how it defines a basic website.
* You may need to make changes to this file to support different scenarios. It is beyond the scope of this tutorial to explain those details.

### Enable IIS Support

![Installer - Enable IISMeta.wxs](/Images/web-api/installer8.png)

* The IsWiX project template embedded a Product.wxs fragment in your WebApp project.  This file describes your feature tree and the components in your WebApiFiles.wxs ComponentGroup.
* Uncomment the `<ComponentGroupRef Id="webSite"/>` element to cause the dormant IISMeta.wxs fragement to become visible to the compiler.

### Build the MSI

![Installer - Built MSI](/Images/web-api/installer9.png)

Right click the WebAPI project and select rebuild. You will now have a built MSI.

## Build considerations

Now that you have a source tree of application and installer code you will likely want to build it with a continuous integration system. WiX suports msbuild and as such is easily compiled.  While IsWiX merely authors the WiX projects and isn't technically part of the build process (and not needed on the build server)  it does include some XML tags in the DesktopApplication.wixproj that can be useful for versioning.

`<MSIProductVersion Condition="'$(MSIProductVersion)' == ''">$([System.Text.RegularExpressions.Regex]::Match($(TF_BUILD_BUILDNUMBER), "\d+.\d+.\d+.\d+"))</MSIProductVersion>`

This allows you to pass an MSIProductVersion property into the build or directly infer it in the case of a TFS XAML build definition. For TFS v.Next builds change `TF_BUILD_BUILDNUMBER` to `BUILD_BUILDNUMBER`.   As Windows Installer ignores the fourth field, you should always change one of the first three fields in order to ensure Major Upgrades will work correctly.  

`1.0.0.0, 1.0.1.0, 1.0.2.0, 1.1.0.0, 1.1.1.0, 2.0.0.0, 2.0.1.0` and so on...

In this tutorial we manually built and published the Web Project using Visual Studio.  During an automated CI build, you can build the Api project with the command `dotnet publish -p:PublishProfile=FolderProfile /p:AssemblyVersion=$(Build.BuildNumber) -c Release` You can use MSBuild to build the Installer solution. 

  
## Conclusion

Hopefully this tutorial gives you a complete picture of how to quickly come up to speed on creating an MSI installer using WiX / IsWiX. This is only the beginning though as eventually you'll need to learn more about the underlying technologies as you attempt more complicated authoring.

