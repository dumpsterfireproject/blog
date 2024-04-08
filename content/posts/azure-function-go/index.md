---
title: "Go Function Deployment in an Azure Pipeline"
date: 2024-04-04
slug: "/azure-function-go"
description: Configuration of the YAML file for deployment of a Go Function in Azure.
tags:
  - Azure
  - Go
  - CI/CD
---
In this article, we'll walk you through the process of taking an Azure Function created using Terraform and deploying it effortlessly using Azure DevOps. We'll go step-by-step through creating the Azure function handler in Go and deploying it via an Azure DevOps pipeline. In the starting point for this function, our cloud engineer has already created the function using Terraform with the app_settings having a FUNCTIONS_WORKER_RUNTIME of "custom". A valuable reference for creating an Azure Function in Go or Rust is available at https://learn.microsoft.com/en-us/azure/azure-functions/create-first-function-vs-code-other. Let's dive in and unlock the full potential of your Azure environment!

Starting from scratch, I first created a new Go project using `go mod init`. Next, I added to the project a boilerplate host.json file in the project root.
```json
{
  "version": "2.0",
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "excludedTypes": "Request"
      }
    }
  },
  "customHandler": {
    "description": {
      "defaultExecutablePath": "handler",
      "workingDirectory": "",
      "arguments": []
    },
    "enableForwardingHttpRequest": true
  }
}
```
The import part here to note is the `defaultExecutablePath` setting. This is the name of the executable that will be deployed as the function handler. I also added a boilerplate `local.settings.json` file.
```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "FUNCTIONS_WORKER_RUNTIME": "custom"
  }
}
```
Also, a boilerplate function.json was created in a myapp folder.
```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "authLevel": "anonymous",
      "direction": "in",
      "name": "req",
      "methods": ["get", "post"]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```
The type of `httpTrigger` means this function will be called over a web URL. 
Note that by default, the path for the function will be `api/<folder name containing the function.json file>`. 
If you wanted to specify the route prefix explicitly, in your host.json file, you'd add the following entry. 
So in our case, with a `myapp` folder, the path will be `api/myapp`.
```json
  "extensions": {
    "http": {
      "routePrefix": "<customPrefix>"
    }
  }
```
Next, we implement the http handler. The standard http library is perfectly fine for implementing the http handler. We use Gin for our Azure App Services, so we
opted to be consistent and stay with the Gin http server in our functions.
```go
  port, exists := os.LookupEnv("FUNCTIONS_CUSTOMHANDLER_PORT")
  if !exists {
    port = "8080"
  }
  router := gin.Default()

  router.GET("/api/myapp", func(c *gin.Context) {
    // overwrite with your implementation
    c.JSON(http.StatusOK, gin.H{"message": "ok"})
  })

  err := router.Run(fmt.Sprintf(":%s", port))
  // use a logger in production
  fmt.Println(err)
```
Finally, we created the yaml file for the Azure DevOps pipeline. For your production environement, you'll also want to add tasks for running tests
and publishing code coverage reports. Note that for deployment as a linux app function, the `GOOS=linux GOARCH=amd64` variables will need to be set
when creating the exectuable.
```yaml
trigger:
- main

variables:
  azureSubscription: '<your subscription here>'
  appName: '<your app name here>'

pool:
  vmImage: ubuntu-latest

steps:
  - task: GoTool@0
    inputs:
      version: '1.22.0'

  # See https://stackoverflow.com/questions/75171491/injection-of-golang-environment-variables-into-azure-pipeline
  # We hit an issue where the GOOS=linux GOARCH=amd64 weren't being picked up by the go build command and we had a bad binary
  - task: Bash@3
    env:
      GOOS: linux
      GOARCH: amd64
    inputs:
      workingDirectory: '$(System.DefaultWorkingDirectory)'
      targetType: 'inline'
      script: |
       go build -o handler ./cmd/main.go

  - task: ArchiveFiles@2
    displayName: 'Create artifact'
    inputs:
      rootFolderOrFile: '$(System.DefaultWorkingDirectory)'
      includeRootFolder: false
      archiveType: 'zip'
      archiveFile: '$(System.ArtifactsDirectory)/$(Build.BuildId).zip'
      replaceExistingArchive: true

  - task: AzureFunctionApp@2
    inputs:
      connectedServiceNameARM: $(azureSubscription)
      appType: functionAppLinux
      appName: $(appName)
      package: $(System.ArtifactsDirectory)/$(Build.BuildId).zip
```
And that was it. A quick word on the zip file - for an Azure function with a Go or Rust handler, essentially,
Azure stores your app with the executable in a storage blob until it's needed when the function is called from
a cold start. In the pipeline, we create the executable for the proper architecture, zip up the directory with
the executable, the host.json, and the directory function.json in it, and deploy it to Azure via an AzureFunctionApp@2 task.
In the first function we deployed to Azure several years ago, it was created and deployed using the Azure extension
for VS Code. When deploying an Azure function through the VS Code extension, it's basically creating the same zip
file and uploading it to Azure (you had to create the executable locally with the proper GOOS and GOARCH variables
set before deploying).
