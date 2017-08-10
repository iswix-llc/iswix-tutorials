# IsWiX Desktop Application Tutorial

This tutorial will walk you through how to build a [Windows Installer](https://msdn.microsoft.com/en-us/library/windows/desktop/cc185688(v=vs.85).aspx) (aka .MSI) setup package  using the [WiX Toolset](http://wixtoolset.org/) (aka Windows Installer XML aka WiX) and [Industrial Strength Windows Installer XML](http://iswix.com/projects/) (aka IsWiX) 

# Prerequisites

## This tutorial requires you to download and install:

* Microsoft Visual Studio
   * 2010 or newer. (2015 or 2017 highly reccomended.)
   * [Community Edition](https://www.visualstudio.com/vs/community/) or higher. 
   * Express Edition will not work.
* Windows Installer XML [3.11](http://wixtoolset.org/releases/)  or higher. 
   * Although IsWiX has project templates for WiX v4, WiX v4 is an alpha status experimental rewrite of WiX and will not work.
* WiX Toolset Visual Studio Extension
   * Install one [extension](http://wixtoolset.org/releases/) for each version of Visual Studio you wish to use. 
* IsWiX [4.11](https://github.com/iswix-llc/iswix/releases) or higher.
   * Will automatically target all of your Visual Studio installations without the need forseperate extension downloads.

## Backstory

Windows Installer is a platform service created by Microsoft in 1999. Originally created with the needs of Microsoft Office in mind, it created a standardized way for vendors to package applications for Windows.  It was designed to be declarative (in the 90s that meant database driven), transactional, transformable with standardized command line driven installation arguments and logging.  Many tools have been created to author MSIs including InstallShield, Advanced Installer, InstallAware, Wise, Visual Studio deployment projects and others.

WiX is an open source project started by former Microsoft employee Rob Mensching while he was still at Microsoft. The first release was back in [2004](http://robmensching.com/blog/posts/2004/4/5/windows-installer-xml-wix-toolset-has-released-as-open-source-on-sourceforge.net/). WiX is essentially an XSD based Domain Specific Langauge and related compiler chain tools that allows a developer to generate Windows Installer Databases from XML documents.  WiX is a very powerful toolset that serves as a thin abstraction of the underlying Windows Installer and emphasis a 'code-first' strategy.  This has the unfortunate drawback of a steep learning curve and hand authoring of many, many lines of XML.

IsWiX is an open source project started by Windows Installer / InstallShield / WiX expert [Christopher Painter](http://iswix.com/) while he was working at Textron - Overwatch Systems division in [2010](http://blog.deploymentengineering.com/2010/05/introducing-iswix.html).  The project was initially funded by Textron OWS as part of a business process transformation effort. Although Textron OWS always wanted to publish the project as open source, the sensitive nature of our business made that difficult. After a very successful pilot, the IP rights were reassigned to Christopher Painter so that he could share it with the world.  InstallShield was overall a good product but Textron OWS had strained it way beyond what InstallShield was ever designed to do.  InstallShield's binary or DTD base XML project files were not branch/merge friendly and the licensing model did not encourage agile/lean workflows. WiX could solve these problems except that Textron OWS's team of 12 build and release engineers would have had to train 250+ developers how to develop in it.  IsWiX was born to give developers an InstallShield like experience for the core installer changes they would need to make.  Basically think 'minimum viable product'.  All other changes would be done by MSI experts in WiX by hand.  Later the product was expanded to include Visual Studio template to simplify the development of  desktop applications, windows services and web application.  The tutorial you are reading today is how to handle the desktop application scenario.

# Create a source repository

While it assumed that you will want to check this code into some sort of source code repository, it is out of the scope of this tutorial to advise you how to do that since we don't know which tool you'll want to use.  We will give a few pointers for GitHub and Team Foundation Server since they are commonly used.  When creating a repos in GitHub, pick the Visual Studio .gitignore file.  This will prevent build artifacts from getting included in your commits.  Add one additional rule to this .gitignore to account for IsWiX specific behavior:

`Installer\Deploy`

Nothing special is needed to be done when using Microsoft Team Foundation Version Control (TFVC) as it already has good integration with Visual Studio and is deeply aware of what your project files are.

# Create the Desktop Application

## Launch Visual Studio and create a new WPF application:

![Application New Project](/Images/WPF-NewProject.png)

* Use File | New | Project and select the WPF C# template.
* Target .NET Framework 4. You can select a different version but you will have an additional step later in the process.
* Call the project DesktopApplication
* Point the Location to the root of your GitHub repos or to a location in your version control system.
   * For tools like Team Foundation Version Control you may want to point it to something like `C:\workspace\TeamProject\ApplicationName\main\` as this will give you the folder `main` that can the be branched to say `dev` and used by build definitions.   This isn't needed in GitHub since the branching model is implemented differently.
* Make sure the "Create directory for solution" option is selected.
* Make sure the "Add Files to Source Control" option is selected if using TFVC.

## Author a postbuild copy:

![Mode](/Images/WPF-PostbuildCopy.png)

Using the Solution Explorer, bring up the project properties for your main EXE project.  Author the following postbuild event:

`xcopy /iery "$(TargetDir)*.*" "$(SolutionDir)..\Installer\Deploy"`

This will help us create a directory structure that models what your deployed application should look like.  Go ahead and build the project and take a look at that directory in your workspace.

![Mode](/Images/WPF-InstallerModel.png)

We are done creating the application.   Close the Application.sln as we will be creating a new solution for the installer.

# Create the Installer

## Launch Visual Studio and create a new IsWiX Solution:

![Installer New Project](/Images/Installer-NewProject.png)

* Use File | New | Project and select the IsWiX Solution for WiX v3 template.
* The .NET Framework version isn't important here.  Perhaps one day someone will submit a pull request to help automate a step down the road based on the selection here.
* Call the project DesktopApplication.  (This will result in your installer being called DesktopApplication.msi.)
* Point the Location to the root of your GitHub repos or to a location in your version control system.
   * For tools like Team Foundation Version Control you may want to point it to something like `C:\workspace\TeamProject\ApplicationName\main\` as this will give you the folder `main` that can the be branched to say `dev` and used by build definitions.   This isn't needed in GitHub since the branching model is implemented differently.
* Make sure the "Create directory for solution" option is selected.
* Make sure the "Add Files to Source Control" option is selected if using TFVC.

## Project Structure Overview:

![Mode](/Images/Installer-ProjectStructure.png)

## The IsWiX solution template created a solution with two projects based on the name:

* Setup Project 
   * Named DesktopApplication
   * Creates an .MSI (Microsoft Installer) database that is installable by a user.
   * Expresses the Manufacturer, Product Name, Upgrade story, Feature Tree,  User Interface,  Installation Requirements and Installation path of the product.

* Merge Module Project
   * NamedDesktopApplicationMM  (Merge Module)
   * Creates an .MSM (Micrsoft Merge Module) database that is not installable by a user.
   * Similar to a .LIB in C/C++.  Database contents get merged into the .MSI at build time.
   * Expresses the files, subfolders, registry entries, shortcuts, services, environment variables and so on that will make up your product.
   * Somewhat serves as an atomic abstract encapsulation of a portion of your installer logic.  You may have more then one merge module consumed by one or more features across one consumed by one or more setup projects.  This is an advanced topic that may be covered by another tutorial some day.  For now just keep it simple with a 1 product with 1 feature consuming 1 merge module.
   
## Author the merge module:

![Merge Module Source before](/Images/Installer-MergeModuleBefore.png)

Select the DesktopApplicationMM.wxs (.wix source) in the DesktopApplication MM project.

Observe line 5: `<?define SourceDir="..\Deploy"?>` This will guide IsWiX in knowing where to find your applications files relative to the location of this wix project (.wixproj).  Our project structure choices cause this to automatically align with the output of our previous xcopy command.

## Launch IsWiX:

![Launch IsWiX](/Images/Installer-LaunchIsWiX.png)

Select Tools | Launch IsWiX to send the current .wxs document to IsWiX.

![IsWiX Launched](/Images/Installer-IsWiX.png)

* Our DesktopApplicationMM.wxs document is now loaded in IsWiX.  
* We are given a single document interface (SDI) with multiple designers along the left side. 
* Each designer expresses a particular view of our .wxs file.  
* The default designer is the General Information designer.  
   * This designer tells us about the module signature and dependencies of this module.  
   * Every merge module has a unique identity similar to a strong name in .NET.  
   * Merge modules can also have dependencies on other merge modules.  This may be covered in another tutorial one day.

## Click on the Files and Folders designer:

![IsWiX Files and Folder designer - before](/Images/Installer-IsWiXFFBefore.png)

* The top  two quadrants of this designer represent source files and folders that are on your developer machine  that can be included in the installation.  
* The bottom two quadrants represent what will be installed on the destination machine.  
* `[MergeRedirectFolder]` is an abstract folder which represents the root of this merge module.  Later it will be associated with your products installation directory: `[ProgramFilesFolder]\Company\Product`.
* The merge module has no knowledge of what product will consume it. Therefore, folder name abstractions like above are used.
* Notice that `<?define SourceDir="..\Deploy"?>` has been resolved to a fully qualified path and your files are in the upper right quadrant.
   * The use of relative path filenames will be critical when we get to build automation as all of your file path dependencies must be found on the build server for your solution to successfully build.
   * Our choice of Application.sln and Installer.sln is important also as we avoid tricky project dependency / build order race conditions.
   
## Author your files:

![IsWiX Files and Folder designer - after](/Images/Installer-IsWiXFFAfter.png)

* Click on the `[MergeRedirectFolder]` in the destination view.
* Drag the file DesktopApplication.exe from the source files to the destination files quadrant.
   * We don't need to ship the other files.
   
## Author your Shortcut:

![IsWiX ShortCuts designer](/Images/Installer-IsWiXShortCuts.png)

* Click on the ShortCuts designer. A view of your merge modules shortcuts will appear.

![IsWiX ShortCuts designer - create desktop folder](/Images/Installer-IsWiXShortCutsDesktopFolder.png)

* Right click `Destination Computer` and select the `DesktopFolder` option.
* Expand `Desitnation Computer` and select the `DesktopFolder` node.  (Maybe a Pull Request will fix this some day.)

![IsWiX ShortCuts designer - create shortcut](/Images/Installer-IsWiXShortCutsCreateShortCut.png)

* Right click `DesktopFolder` and select the `Create ShortCut option`.

![IsWiX ShortCuts designer - component picker](/Images/Installer-IsWiXShortCutsComponentPicker.png)

* Select the file that will serve as the target of this shortcut. `[MergeRedirectFolder\DesktopApplication.exe`  and click Select.

![IsWiX ShortCuts designer - shortcut properties](/Images/Installer-IsWiXShortCutsProperties.png)

* Use the property grid to give the shortcut a friendly name such as `Desktop Application`.

![Visual Studio Reload Document](/Images/Installer-VSReload.png)

* Press Control-S to save the changes back to disk and close the IsWiX application.
* This should cause Visual Studio to detect that the .wxs file was modified out of process.  Click Yes to cause it to reload in the IDE.

## Observe the XML changes

![IsWiX Merge Module - After](/Images/Installer-MergeModuleAfter.png)

* If you compare the XML before/after you should notice that IsWiX authored multiple elements for you. 
   * A `Directory` element referencing the DesktopFolder location.
   * `Component`, `File`, and `Shortcut` elements defining your new file and a shortcut pointing to it. 
   * IsWiX manages this document using a hashing and sorting alogorythm.  You should not modify these elements.
   * You may however add additional child elements and/or author elements in the `-custom.wxs` fragment.  This is an advanced topic that may be covered by another tutorial one day.
   
## Build the MSI

![Installer - Built MSI](/Images/Installer-BuiltMSI.png)

## Different .NET Version Scenario

![Installer - Built MSI](/Images/Installer-DotNetVersion.png)

* The IsWiX project template includes a .NET 4.0 prerequisite check by default.  This ensures that .NET 4.0 is installed  before a user can install your MSI.
* You may have choosen a different .NET version when building your EXE.  In this case find the Product.wxs file in the DesktopApplication wix project.
   * Visit the URL mentioned on line 19 to get the name of the property of the version of .NET you wish to require.
   * Update the `ProperfyRef` on line 20 and update the condition on line 21.
   * The `Installed or` portion of the condition is to ensure tha we don't require .NET during the uninstall as this would be a poor user experience.
   
## Build considerations

Now that you have a source tree of application and installer code you will likely want to build it with a continuous integration system. WiX suports msbuild and as such is easily compiled.  While IsWiX merely authors the WiX projects and isn't technically part of the build process (and not needed on the build server)  it does include some XML tags in the DesktopApplication.wixproj that can be useful for versioning.

`<MSIProductVersion Condition="'$(MSIProductVersion)' == ''">$([System.Text.RegularExpressions.Regex]::Match($(TF_BUILD_BUILDNUMBER), "\d+.\d+.\d+.\d+"))</MSIProductVersion>`

This allows you to pass an MSIProductVersion property into the build or directly infer it in the case of a TFS XAML build definition. For TFS v.Next builds change `TF_BUILD_BUILDNUMBER` to `BUILD_BUILDNUMBER`.   As Windows Installer ignores the fourth field, you should always change one of the first three fields in order to ensure Major Upgrades will work correctly.  

`1.0.0.0, 1.0.1.0, 1.0.2.0, 1.1.0.0, 1.1.1.0, 2.0.0.0, 2.0.1.0` and so on...

## Conclusion

Hopefully this tutorial gives you a complete picture of how to quickly come up to speed on creating an MSI installer using WiX / IsWiX. This is only the beginning though as eventually you'll need to learn more about the underlying technologies as you attempt more complicated authoring.
