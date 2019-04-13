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

### 2. Required Jenkins Plugins
- Git Plugin: allows use of Git as a build Source Code Management
- MSBuild Plugin: allows use of MSBuild.exe to build .NET projects
- Warnings Plugin: collects compiler warnings of the project modules and visualizes the results
- NUnit Plugin: allows you to publish NUnit test results
- Cobertura Plugin: allows you to capture and visualize code coverage data 
- HTML Publisher Plugin: allows you to publish HTML reports for Job builds

### 3. Windows service configuration
To avoid problems with the NugetPackage restore step you should set the user of Jenkins service to the local admin account instead of default systemaccount. For that go to Windows service menu, right click on Jenkins entry and set account details to admin account. 

### 4. Manage Roles & Assign Roles
I suggest that anonym users can see projects without the need to login (so they can see project statistics). Of course they should not be able to configure any settings. For that do the follwing:

Install that plugin in Jenkins:
https://wiki.jenkins.io/display/JENKINS/Role+Strategy+Plugin plugin

After that there's a new menu entry in Jenkins main menu and you can configure like that:
Screenshots: 
![Roles.png](https://github.com/Cocotus/JenkinsCIConcept/blob/master/Roles.png)
![Roles2.png](https://github.com/Cocotus/JenkinsCIConcept/blob/master/Roles2.png)

### 5. Fix Javascript rights to display HTML Report
We will use the ReportGenerator.exe in our build scripts later on and it won't display the result correct without the following fix.
(The following solution is for Windows.)

In [Jenkins directory]\jenkins.xml search following lines:
<executable>java.exe</executable>
<arguments>[arguments are here]</arguments>

Add the following argument to the whitespace-separated list of arguments:

**-Dhudson.model.DirectoryBrowserSupport.CSP=**

Then restart the Jenkins service to pick up the change

### 6. For projects hosted on GITHUB: Github Token configuration
1. Jenkins > Credentials > System > Global Credentials
    Add Credential --> Secret Text --> Github Personal Access Token (GPAT):
    (admin:repo_hook, repo) -> Save
2. Go to Jenkins > Global Configuration
    Add Git Server, Automanage Hooks -->  Test Connection

### 7. MSBuild Plugin configuration
1. Jenkins Configuration > Helper Tools configuration:
2. MSBuild Path is MSBuild.exe path to VisualStudio installation:
C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\MSBuild\15.0\Bin\

### 8. For projects hosted on GITHUB: Installation of Git
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
   
###  1.Change assembly:
Assembly Version: $SVN_REVISION
FileName: **/MyProject/Directory.Build.props
RegexPattern: <Version>(\d+).(\d+).(\d+).(\d+)<\/Version>
ReplacementPattern: <Version>$1.$2.%s<\/Version>
 
 
### Source Code Management: 
Set up Github or other type like SVN by youself. Should not be that hard!

### Build Triggers
Nothing special here, set up as you like

### Build Environment: 
Check: Delete workspace before build starts
Check: Inject environment variables
Properties Content:
JOBPROJECTNAME=MyProject
JOBTESTPROJECTNAME=MyProject.Tests

### 1. Build action: Execute Windows batch command 
Configure NuGet Package Restoration. For that paste a command like the following in the command text box: 
```shell
::%%%%% Restore NUGET Dependecies before MSBuild %%%%%
C:\JENKINSTOOLS\nuget.exe restore "%WORKSPACE%\%JOBPROJECTNAME%\%JOBPROJECTNAME%.sln"
```

### 2. Build action: Configure MSBuild Process using Visual Studio
MSBuild Version: Select the correct entry to msbuild.exe here
MSBuild build file: ${WORKSPACE}\${JOBPROJECTNAME}\${JOBPROJECTNAME}.sln
Command Line Arguments: /p:Configuration=Release

Add following Parameter command in textbox:
```shell
/p:Configuration=Release
```

### 3. Build action: Execute Windows batch command 

```shell
::%%%%% STATISTIKEN generieren: OpenCover, Cobertura, HTMLReport %%%%%

rmdir /q /s target
mkdir target

::%%%%% PFADE SETZEN %%%%%
set "opencover=C:\JENKINSTOOLS\opencover\4.7.922\tools\OpenCover.Console.exe"
set "reportgenerator=C:\JENKINSTOOLS\reportgenerator\4.0.15\tools\net47\ReportGenerator.exe"
set "dotnet=C:\Program Files\dotnet\dotnet.exe"
set "targetdir=target\Coverage"
mkdir %targetdir%
 
::%%%%% UNIT TEST REPORT GENERIEREN, TRX %%%%%
"%dotnet%" test "%WORKSPACE%\%JOBPROJECTNAME%\%JOBTESTPROJECTNAME%\%JOBTESTPROJECTNAME%.csproj" --logger "trx;LogFileName=UnitTests.trx"

::%%%%% OPENCOVER.EXE FÜR COVERAGERESULT.XML %%%%%
"%opencover%" -register:user "-target:%dotnet%" -targetargs:"test %JOBPROJECTNAME%\%JOBTESTPROJECTNAME%\%JOBTESTPROJECTNAME%.csproj" -output:%targetdir%\OpenCover.coverageresults.xml -filter:"+[*]* -[Serilog*]* -[GalaSoft*]* -[*Tests]*" -returntargetcode -coverbytest:* -mergebyhash -oldstyle

::%%%%% COVERAGERESULT.XML NUTZEN UM REPORT UND COBERTURA DIAGRAMM ZU BAUEN %%%%%
"%reportGenerator%" -reports:%targetdir%\OpenCover.coverageresults.xml -reporttypes:HtmlInline;Cobertura -targetdir:%targetdir% -assemblyfilters:-xunit* -classfilters:-AutoGeneratedProgram
```

### 4. Build action: Process xUnit test result report
Type: MS Test Version NA
```shell
${JOBPROJECTNAME}\${JOBTESTPROJECTNAME}\TestResults\UnitTests.trx
```

### 4. Build action: Execute Powershell command 

This will create nuget package and submit to Nuget server

```shell

####### NUGET-ABLAGE (für BaGet Nugetserver) - KEINE Anpassungen hier nötig, Pfade belassen!######
####### Übergibt erstelltes nupkg-Paket an Baget und sichert package-Ordner von BaGet im Netzlaufwerk######

# Package Verzeichnis auf Nugetserver Baget
$NUGETSERVERREPOSITORY = "C:\inetpub\wwwroot\BaGet\packages"

# Nuget in Ordner im Workspace kopieren, damit im Post-Build Schritt der Ordnerinhalt zum Netzlaufwerk kopiert wird
$NUGETBACKUPSERVER = "$ENV:WORKSPACE\NUGETS\"
mkdir $NUGETBACKUPSERVER

# Dateiname des generierten Nugetpakets
$NUGETFILENAME = Get-ChildItem -Path "$ENV:WORKSPACE\$ENV:JOBPROJECTNAME\$ENV:JOBPROJECTNAME\bin\Release\" -Filter "*.nupkg" |Select -First 1

# Übergabe an Nugetserver (erst in Workspace ablegen, da ansonsten Pfad zu lang für Push)
$NUGETTEMP = "$ENV:WORKSPACE\NUGETTEMP\"
mkdir $NUGETTEMP
Copy-Item $ENV:WORKSPACE\$ENV:JOBPROJECTNAME\$ENV:JOBPROJECTNAME\bin\Release\$NUGETFILENAME $NUGETTEMP -Recurse -Force

dotnet nuget push -s http://10.3.57.141:5000/v3/index.json $NUGETTEMP$NUGETFILENAME

# Backups: lokal und auf Netzwerklaufwerk vorbereiten
Copy-Item $NUGETSERVERREPOSITORY $NUGETBACKUPSERVER -Recurse -Force


```

### 1.Post-build Actions: 1. Publish HTML reports
HTML directory to archive: target\Coverage
Index page[s]: index.htm

### 2.Post-build Actions: 2. Record compiler warning and statitics results
Tool: MsBuild

### 2.Post-build Actions: 3. Publish Cobertura Coverage report
Cobertura xml report pattern: **/target/Coverage/*Cobertura.xml

### 3.Send build artifacts to windows share (optional)
  SOURCE: NUGETS/
  Remove prefix: NUGETS/
  Remote directory:NUGETS/

# Additional tips for Jenkins setup
## Replace JENKINS label with your own text
1. Install _Simple Theme_ Plugin in Jenkins
2. _Manage Jenkins -> Configure System -> Theme -> Add -> URL of theme CSS:_
``` Shell
http://JENKINSIP/userContent/theme.css
```
2. Create a theme.css file with following content and place it in userContent folder of Jenkins:
``` CSS
@charset “utf-8”;
 
/* Custom style for CloudBees Jenkins Platform 
.logo img {
content:url("cksplus.ico");
height: 32px;
vertical-align:middle;
}
*/

.logo #jenkins-name-icon {
display: none;
}

.logo:after {
content: 'JENKINS COCOTUS';
font-weight: bold;
font-size: 20px;
margin-left: 50px;
margin-right: 12px;
color: white;
line-height: 40px;
}
```
