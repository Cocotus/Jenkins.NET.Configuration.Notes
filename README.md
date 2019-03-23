# JenkinsCIConcept
Jenkins Visual Studio project build, NUnit unit testing, and OpenCover code coverage.



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
