# Springboot-Application
Azure pipeline With Spring Boot Application 



# Docker
# Build and push an image to Azure Container Registry
# https://docs.microsoft.com/azure/devops/pipelines/languages/docker

trigger:
- develop

resources:
- repo: self

variables:
  # Container registry service connection established during pipeline creation
  dockerRegistryServiceConnection: '4ce41710-faa1-41cf-9af5-d7968b582b34'
  imageRepository: 'shubhamimage'
  containerRegistry: 'ttmsacr.azurecr.io'
  dockerfilePath: '$(Build.SourcesDirectory)/Dockerfile'
  tag: '$(Build.BuildId)'

  # Agent VM image name
  vmImageName: 'ubuntu-latest'

stages:
- stage: Build
  displayName: Build stage
  jobs:
  - job: Build
    displayName: Build
    pool:
      vmImage: $(vmImageName)
    steps:
    - task: Maven@3
      inputs:
       mavenPomFile: 'pom.xml'
       mavenOptions: '-Xmx3072m'
       javaHomeOption: 'JDKVersion'
       jdkVersionOption: '1.17'
       jdkArchitectureOption: 'x64'
       publishJUnitResults: false
       testResultsFiles: '**/surefire-reports/TEST-*.xml'
    - task: CopyFiles@2
      inputs:
           SourceFolder: '$(system.defaultworkingdirectory)'
           Contents: '**/*.jar'
           TargetFolder: '$(build.artifactstagingdirectory)'

    - task: Maven@3
      displayName: Build Docker image
      inputs:
        mavenPomFile: 'pom.xml'
        goals: 'spring-boot:build-image'
        publishJUnitResults: true
        jdkVersionOption: '1.17'
        
    - task: PublishBuildArtifacts@1
      inputs:
        PathtoPublish: '$(Build.ArtifactStagingDirectory)'
        ArtifactName: 'drop'
        publishLocation: 'Container'
    - task: Docker@2
      displayName: Build and push an image to container registry
      inputs:
        command: buildAndPush
        repository: $(imageRepository)
        dockerfile: $(dockerfilePath)
        containerRegistry: $(dockerRegistryServiceConnection)
        tags: |
          $(tag)
   
