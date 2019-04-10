# Installation of Jenkins (for Microsoft Builds (.NETCore/NETFramework/Xamarin/NETStandard..)
Following instructions describe setup of a Jenkins configuration which is used too automate builds of Microsoft Visual Studio projects.
It's recommended to create Nuget packages instead of simple dll files as build output of your visual studio class projects and host those nuget packages in your own nuget server and uses that server in your projects. 

Furthermore this tutorial will tell you how to achieve following for each job in Jenkins:

- Display of warnings graph, test results graph, codecoverage graph, detailed HTML report of codecodeverage
- Trigger release build of project, create nuget package and push that to your own nuget server
![alt text](https://github.com/Cocotus/JenkinsCIConcept/blob/master/Jenkins_Overview.png "Logo Title Text 1")

## Preparations
### 1. Tools required
- Jenkins Installation on Windows machine
- Visual Studio 2017+ Installation on same machine where Jenkins is running. Use installer to choose necessary features like Xamarin, NETCore....
- Nuget.exe https://www.nuget.org/downloads
- OpenCover.exe https://www.nuget.org/packages/OpenCover/
- ReportGenerator.exe https://www.nuget.org/packages/ReportGenerator/
- NugetServer like Baget (its free and open source and works) https://github.com/loic-sharma/BaGet/

Copy Nuget.exe together with Installation folders of OpenCover and Reporter Nuget Package in a new folder called JENKINSTOOLS and move that folder to C:\ root. So result structure should look something like that:

![alt text](https://github.com/Cocotus/JenkinsCIConcept/blob/master/Jenkins_Tools.png "Logo Title Text 1")

Tip:
You can simply create a new Visual Studio project and get the Opencover, Reporter and Cobertura NugetPackages using Nuget packet managment menu. After installation go to .package folder and copy the opencover and reporter folders from there to JENKINSTOOLS. Right click on folder after moving it to C:\ and set folder rights to read/write for users.

### 2. Windows service configuration
To avoid problems with the NugetPackage restore step you should set the user of Jenkins service to the local admin account instead of default systemaccount. For that go to Windows service menu, right click on Jenkins entry and set account details to admin account. 

### 3. Manage Roles & Assign Roles
I suggest that anonym users can see projects without the need to login (so they can see project statistics). Of course they should not be able to configure any settings. For that do the follwing:

Install that plugin in Jenkins:
https://wiki.jenkins.io/display/JENKINS/Role+Strategy+Plugin plugin

After that there's a new menu entry in Jenkins main menu and you can configure like that:
Screenshots: 
![Roles.png](https://github.com/Cocotus/JenkinsCIConcept/blob/master/Roles.png)
![Roles2.png](https://github.com/Cocotus/JenkinsCIConcept/blob/master/Roles2.png)

### 4. Fix Javascript rights to display HTML Report
We will use the ReportGenerator.exe in our build scripts later on and it won't display the result correct without the following fix.
(The following solution is for Windows.)

In [Jenkins directory]\jenkins.xml search following lines:
<executable>java.exe</executable>
<arguments>[arguments are here]</arguments>

Add the following argument to the whitespace-separated list of arguments:

**-Dhudson.model.DirectoryBrowserSupport.CSP=**

Then restart the Jenkins service to pick up the change

### 5. For projects hosted on GITHUB: Github Token configuration
1. Jenkins > Credentials > System > Global Credentials
    Add Credential --> Secret Text --> Github Personal Access Token (GPAT):
    (admin:repo_hook, repo) -> Save
2. Go to Jenkins > Global Configuration
    Add Git Server, Automanage Hooks -->  Test Connection

### 6. MSBuild Plugin configuration
1. Jenkins Configuration > Helper Tools configuration:
2. MSBuild Path is MSBuild.exe path to VisualStudio installation:
C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\MSBuild\15.0\Bin\

### 7. For projects hosted on GITHUB: Installation of Git
1. Download Git: https://git-scm.com/downloads
2. Jenkins Configuration > Helper Tools configuration:
3. Change Path to Git:  C:\Program Files\Git\cmd\git.exe

## Job configuration
Below I will list all configuration steps needed to configure a job in Jenkins to achieve following results:

- Testresults in graph 
- CodeCoverage in graph 
- Warnings in graph
- Coverage report in HTML

### 1. Manage SourceCode Repository: GitProject
1. Create new project
2. Source Code Management -> Git
   --> set URL Repository
   --> set Credentials from list
   
### 2. Build action: Configure NuGet Package Restoration
You need to paste a command like the following in the Command text box: 
```shell
C:\JENKINSTOOLS\nuget.exe restore  "%WORKSPACE%\CKS.PlausiValidation\CKS.PlausiValidation.sln"
```

### 3. Build action: Configure MSBuild Process
MSBuildFile:
${WORKSPACE}\CKS.PlausiValidation\CKS.PlausiValidation.sln

Add following Parameter command in textbox:
```shell
/p:Configuration=Release
```
### 4. Generate Statistics/Testresults
The following lines will do the following:
1. Run dotnet test to generate testresult .trx file for text project
2. Run opencover.exe to run dotnet test again and produce coverage.xml
3. Run reportgenerator to generate html report from coverage.xml

Add following in Windows Shell Command: 


# Jenkins Continuous Integration Flow



Source Code Management:

- Specify Git repository URL and credentials to fetch code

- Specify branches to build



Build Triggers:

- Specify GitHub hook trigger for GITScm polling



Build Environment:

- Delete workspace before build starts



Build Actions:

- Use NuGet to restore packages before build operation

- Build Visual Studio project solution using MSBuild

- Start to run NUnit unit tests and execute code coverage with OpenCover

- Convert OpenCover XML report into Cobertura XML report using OpenCoverToCoberturaConverter



Post-Build Actions:

- Convert NUnit XML report into a readable HTML page

- Convert OpenCover XML report into readable HTML pages using ReportGenerator

- Scan MSBuild compiler log for warnings and publish report to Jenkins

- Publish NUnit test result report to Jenkins job build

- Publish converted Cobertura code coverage report to Jenkins job build

- Publish generated HTML report pages  to Jenkins job build

- Send build notifications to Slack channel



# Required Jenkins Plugins

- Git Plugin: allows use of Git as a build Source Code Management

- MSBuild Plugin: allows use of MSBuild.exe to build .NET projects

- Post Build Task Plugin: allows post build execution of a shell or batch script

- Warnings Plugin: collects compiler warnings of the project modules and visualizes the results

- NUnit Plugin: allows you to publish NUnit test results

- Cobertura Plugin: allows you to capture and visualize code coverage data 

- HTML Publisher Plugin: allows you to publish HTML reports for Job builds

- Slack Notification Plugin: allows build notifications to a Slack channel



# Required System Tools

- Visual Studio 2017: allows you to build Visual Studio projects

- .NET Framework: software framework for building web applications and services

- NuGet Command Line Interface: package manager for .NET projects

- OpenCover: code coverage tool for .NET projects

- ReportGenerator: converts OpenCover XML reports into readable HTML pages

- OpenCoverToCoberturaConverter: converts OpenCover XML reports to Cobertura XML reports

- NUnit: unit-testing framework for .NET projects

- NUnit HTML Report Generator: converts NUnit XML reports into a self-contained Bootstrap 3 based HTML page
