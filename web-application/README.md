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

## Create the Installer

### Launch Visual Studio and create a new IsWiX Solution:

![Installer New Project](/Images/web-application/Installer-NewProject.png)

* Use File | New | Project and select the IsWiX Solution for WiX v3 template.
* The .NET Framework version isn't important here.  Perhaps one day someone will submit a pull request to help automate a step down the road based on the selection here.
* Call the project WebApp.  (This will result in your installer being called WebApp.msi.)
* Point the Location to the root of your GitHub repos or to a location in your version control system.
   * For tools like Team Foundation Version Control you may want to point it to something like `C:\workspace\TeamProject\ApplicationName\main\` as this will give you the folder `main` that can the be branched to say `dev` and used by build definitions.   This isn't needed in GitHub since the branching model is implemented differently.
* Make sure the "Create directory for solution" option is selected.
* Make sure the "Add Files to Source Control" option is selected if using TFVC.

### Project Structure Overview:

![Mode](/Images/web-application/Installer-ProjectStructure.png)

### The IsWiX solution template created a solution with two projects based on the name:

* Setup Project 
   * Named WebApp
   * Creates an .MSI (Microsoft Installer) database that is installable by a user.
   * Expresses the Manufacturer, Product Name, Upgrade story, Feature Tree,  User Interface,  Installation Requirements and Installation path of the product.

* Merge Module Project
   * WebAppMM  (Merge Module)
   * Creates an .MSM (Micrsoft Merge Module) database that is not installable by a user.
   * Similar to a .LIB in C/C++.  Database contents get merged into the .MSI at build time.
   * Expresses the files, subfolders, registry entries, shortcuts, services, environment variables and so on that will make up your product.
   * Somewhat serves as an atomic abstract encapsulation of a portion of your installer logic.  You may have more then one merge module consumed by one or more features across one consumed by one or more setup projects.  This is an advanced topic that may be covered by another tutorial some day.  For now just keep it simple with a 1 product with 1 feature consuming 1 merge module.
   
### Author the merge module:

![Merge Module Source before](/Images/web-application/Installer-MergeModuleBefore.png)

Select the WebAppMM.wxs (.wix source) in the WebAppMM project.

Observe line 5: `<?define SourceDir="..\Deploy"?>` This will guide IsWiX in knowing where to find your applications files relative to the location of this wix project (.wixproj).  Our project structure choices cause this to automatically align with the output of our previous publish command.   

### Launch IsWiX:

![Launch IsWiX](/Images/web-application/Installer-LaunchIsWiX.png)

Select Tools | Launch IsWiX to send the current .wxs document to IsWiX.

![IsWiX Launched](/Images/web-application/Installer-IsWiX.png)

* Our WebAppMM.wxs document is now loaded in IsWiX.  
* We are given a single document interface (SDI) with multiple designers along the left side. 
* Each designer expresses a particular view of our .wxs file.  
* The default designer is the General Information designer.  
   * This designer tells us about the module signature and dependencies of this module.  
   * Every merge module has a unique identity similar to a strong name in .NET.  
   * Merge modules can also have dependencies on other merge modules.  This may be covered in another tutorial one day.
   
### Click on the Files and Folders designer:

![IsWiX Files and Folder designer - before](/Images/web-application/Installer-IsWiXFFBefore.png)

* The top  two quadrants of this designer represent source files and folders that are on your developer machine  that can be included in the installation.  
* The bottom two quadrants represent what will be installed on the destination machine.  
* `[MergeRedirectFolder]` is an abstract folder which represents the root of this merge module.  Later it will be associated with your products installation directory: `[ProgramFilesFolder]\Company\Product`.
* The merge module has no knowledge of what product will consume it. Therefore, folder name abstractions like above are used.
* Notice that `<?define SourceDir="..\Deploy"?>` has been resolved to a fully qualified path and your files are in the upper right quadrant.
   * The use of relative path filenames will be critical when we get to build automation as all of your file path dependencies must be found on the build server for your solution to successfully build.
   * Our choice of Application.sln and Installer.sln is important also as we avoid tricky project dependency / build order race conditions.
   
### Author your files:

![IsWiX Files and Folder designer - After](/Images/web-application/Installer-IsWiXFFAfter.png)

* Click on the `[MergeRedirectFolder]` in the destination view.
* Drag the file WebSite folder from the source files to the destination files quadrant.

![Visual Studio Reload Document](/Images/web-application/Installer-VSReload.png)

* Press Control-S to save the changes back to disk and close the IsWiX application.
* This should cause Visual Studio to detect that the .wxs file was modified out of process.  Click Yes to cause it to reload in the IDE.

### Observe the XML changes

![IsWiX Merge Module - After](/Images/web-application/Installer-XMLChanges.png)

* If you compare the XML before/after you should notice that IsWiX authored many elements for you. 
   * `Directory`, `Component` and `File` elements describing what your deployed website's directory tree looks like.
   * IsWiX manages this document using a hashing and sorting alogorythm.  You should not modify these elements.
      * You may however add additional child elements and/or author elements in the `-custom.wxs` fragment.  This is an advanced topic that may be covered by another tutorial one day.
   * WiX will only include these files in the MSI.  If you later add content to your website you must use IsWiX to update the WebAppMM.wxs.
     * New .CS files will be included in your WebApp.dll so no updates are needed.
     * Due to the way IsWiX hashes and sorts this document it is typically safe to simply remove the WebSite directory from the destination and redrag it to "rebase" the directory.  Compare the WebSiteMM.wxs against the previous version in source control and you should clearly see what the differences are. (Note: "rebasing" would lose any custom child elements you authored.)

### Explore IIS Support

![Installer -IISMeta.wxs](/Images/web-application/Installer-IISMeta.png)

* The IsWiX project template embedded a dormant (not referenced) IISMeta.wxs fragment in your WebApp project to help get you started in case you were creating a web application.. Take a look at it and observe how it defines a basic website.
* You may need to make changes to this file to support different scenarios. It is beyond the scope of this tutorial to explain those details.

### Enable IIS Support

* The IsWiX project template embedded a dormant IISMeta.wxs fragment in your WebApp project. Take a look at it and observe how it defines a basic website.

![Installer - Built MSI](/Images/web-application/Installer-Features.png)

* The IsWiX project template embedded a Features.wxs fragment in your WebApp project.  This file describes your feature tree and the feature / merge modules relationship.
* Uncomment the `<ComponentGroupRef Id="webSite"/>` element to cause the dormant IISMeta.wxs fragement to become referenced.

### Build the MSI

![Installer - Built MSI](/Images/web-application/Installer-BuiltMSI.png)

## Build considerations

Now that you have a source tree of application and installer code you will likely want to build it with a continuous integration system. WiX suports msbuild and as such is easily compiled.  While IsWiX merely authors the WiX projects and isn't technically part of the build process (and not needed on the build server)  it does include some XML tags in the DesktopApplication.wixproj that can be useful for versioning.

`<MSIProductVersion Condition="'$(MSIProductVersion)' == ''">$([System.Text.RegularExpressions.Regex]::Match($(TF_BUILD_BUILDNUMBER), "\d+.\d+.\d+.\d+"))</MSIProductVersion>`

This allows you to pass an MSIProductVersion property into the build or directly infer it in the case of a TFS XAML build definition. For TFS v.Next builds change `TF_BUILD_BUILDNUMBER` to `BUILD_BUILDNUMBER`.   As Windows Installer ignores the fourth field, you should always change one of the first three fields in order to ensure Major Upgrades will work correctly.  

`1.0.0.0, 1.0.1.0, 1.0.2.0, 1.1.0.0, 1.1.1.0, 2.0.0.0, 2.0.1.0` and so on...

In this tutorial we manually built and published the Web Project using Visual Studio.  During an automated CI build, MSBuild will not automatically publish the profile during a build unless told to do so.  You may do accomplish this by doing one or the below:

  * Add the following properties to your projects .CSPROJ file: `<PropertyGroup><DeployOnBuild>true</DeployOnBuild>  <PublishProfile>Deploy</PublishProfile></PropertyGroup>`

  * Pass the following properties to MSBuild during the build: `/p:DeployOnBuild=true /p:PublishProfile=Deploy`

## Conclusion

Hopefully this tutorial gives you a complete picture of how to quickly come up to speed on creating an MSI installer using WiX / IsWiX. This is only the beginning though as eventually you'll need to learn more about the underlying technologies as you attempt more complicated authoring.
