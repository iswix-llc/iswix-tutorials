# IsWiX Tutorials Overview

These tutorials will walk you through how to build [Windows Installer](https://msdn.microsoft.com/en-us/library/windows/desktop/cc185688(v=vs.85).aspx) (aka .MSI) setup packages using the [WiX Toolset](http://wixtoolset.org/) (aka Windows Installer XML aka WiX) and [Industrial Strength Windows Installer XML](http://iswix.com/projects/) (aka IsWiX) for a variety of application types.

## Prerequisites

### These tutorials require you to download and install:

* Microsoft Visual Studio
   * 2022 (Sorry, we've dropped support for previous versions of Visual Studio as it's just a testing time sink. VS2022 is awesome! )
   * [Community Edition](https://www.visualstudio.com/vs/community/) or higher. 
* Windows Installer XML [3.11.2 - 4.0.0](https://wixtoolset.org/). 
* WiX Toolset Visual Studio Extension for v3 support and or HeatWave Visual Studio Extension for v4 support.
   * Install [WiX ToolSet Votive](https://marketplace.visualstudio.com/items?itemName=WixToolset.WixToolsetVisualStudio2022Extension) extension for Visual Studio 2022 to use with WiX v3 projects.
   * Install this [FireGiant HeatWave](https://marketplace.visualstudio.com/items?itemName=FireGiant.FireGiantHeatWaveDev17) extensionfor Visual Studio 2022 to use with WiX v4 projects.
    
* IsWiX [5.0.9](https://github.com/iswix-llc/iswix/releases) or higher.

## Backstory

Windows Installer is a platform service created by Microsoft in 1999. Originally created with the needs of Microsoft Office in mind, it created a standardized way for vendors to package applications for Windows.  It was designed to be declarative (in the 90s that meant database driven), transactional, transformable with standardized command line driven installation arguments and logging.  Many tools have been created to author MSIs including InstallShield, Advanced Installer, InstallAware, Wise, Visual Studio deployment projects and others.

WiX is an open source project started by former Microsoft employee Rob Mensching while he was still at Microsoft. The first release was back in [2004](http://robmensching.com/blog/posts/2004/4/5/windows-installer-xml-wix-toolset-has-released-as-open-source-on-sourceforge.net/). WiX is essentially an XSD based Domain Specific Langauge and related compiler chain tools that allows a developer to generate Windows Installer Databases from XML documents.  WiX is a very powerful toolset that serves as a thin abstraction of the underlying Windows Installer and emphasis a 'code-first' strategy.  This has the unfortunate drawback of a steep learning curve and hand authoring of many, many lines of XML.

IsWiX is an open source project started by Windows Installer / InstallShield / WiX expert [Christopher Painter](http://iswix.com/) while he was working at Textron - Overwatch Systems division in [2010](http://blog.deploymentengineering.com/2010/05/introducing-iswix.html).  The project was initially funded by Textron OWS as part of a business process transformation effort. Although Textron OWS always wanted to publish the project as open source, the sensitive nature of our business made that difficult. After a very successful pilot, the IP rights were reassigned to Christopher Painter so that he could share it with the world.  InstallShield was overall a good product but Textron OWS had strained it way beyond what InstallShield was ever designed to do.  InstallShield's binary or DTD base XML project files were not branch/merge friendly and the licensing model did not encourage agile/lean workflows. WiX could solve these problems except that Textron OWS's team of 12 build and release engineers would have had to train 250+ developers how to develop in it.  IsWiX was born to give developers an InstallShield like experience for the core installer changes they would need to make.  Basically think 'minimum viable product'.  All other changes would be done by MSI experts in WiX by hand.  Later the product was expanded to include Visual Studio template to simplify the development of  desktop applications, windows services and web application.  The tutorial you are reading today is how to handle the desktop application scenario.

## Tutorials

[Desktop Application](https://github.com/iswix-llc/iswix-tutorials/tree/master/desktop-application) A simple WPF application with a shortcut on the desktop.

[Web Application](https://github.com/iswix-llc/iswix-tutorials/tree/master/web-application) A basic ASP.NET website installed on IIS. Implemented in .NET Framework 4.5 and WiX v3.11. (Votive)

[Windows Service](https://github.com/iswix-llc/iswix-tutorials/tree/master/windows-service) A basic Windows Service that installs and starts without using InstallUtil. Implemented in .NET Framework 4.0 and WiX v3.11. (Votive)

[Worker Service](https://github.com/iswix-llc/iswix-tutorials/tree/master/worker-service) A basic Worker Service that installs and starts without using InstallUtil.  Implemented in .NET 7.0 and WiX v4.0. (Heatwave)
