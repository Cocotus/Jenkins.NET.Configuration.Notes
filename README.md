# Installation of Jenkins (for Microsoft Builds (.NETCore/NETFramework/Xamarin/NETStandard..)
Following instructions describe setup of a Jenkins configuration which is used too automate builds of Microsoft Visual Studio projects.
It's recommended to create Nuget packages instead of simple dll files as build output of your visual studio class projects and host those nuget packages in your own nuget server and uses that server in your projects. 

Furthermore this tutorial will tell you how to achieve following for each job in Jenkins:

- Display of warnings graph, test results graph, codecoverage graph, detailed HTML report of codecodeverage
- Trigger release build of project, create nuget package and push that to your own nuget server

## Preparations
**1. Tools required:**
- Jenkins Installation on Windows machine
- Visual Studio 2017+ Installation on same machine where Jenkins is running
https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2017
Use installer to choose necessary features like Xamarin, NETCore....
- Nuget.exe (Download nuget.exe here: )
- OpenCover.exe (Nuget-Package)
- Reporter.exe (Nuget-Package)
- NugetServer like Baget (its free and open source and works)

Copy Nuget.exe together with Installation folders of OpenCover and Reporter Nuget Package in a new folder called JENKINSTOOLS and move that folder to C:\ root. So result structure should look something like that:

PIC

Tip:
You can simply create a new Visual Studio project and get the Opencover, Reporter and Cobertura NugetPackages using Nuget packet managment menu. After installation go to .package folder and copy the opencover and reporter folders from there to JENKINSTOOLS. Right click on folder after moving it to C:\ and set folder rights to read/write for users.

* * *

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
