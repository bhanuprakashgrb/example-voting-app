# Docker
# Build and push an image to Azure Container Registry
# https://docs.microsoft.com/azure/devops/pipelines/languages/docker

# Docker
# Build and push an image to Azure Container Registry

trigger:
  branches:
    include:
      - main
  paths:
    include:
      - result/*

variables:
  dockerRegistryServiceConnection: 'acrAzureDevops1992'
  imageRepository: 'resultapp'
  dockerfilePath: '$(Build.SourcesDirectory)/result/Dockerfile'
  tag: '$(Build.BuildId)'

pool:
  name: 'azure-devop-agent'   # self-hosted VM agent

stages:
  - stage: BuildAndPush
    displayName: Build and Push Image
    jobs:
      - job: BuildAndPush
        displayName: Build and Push
        steps:
          - task: Docker@2
            displayName: Build and Push Docker Image
            inputs:
              containerRegistry: '$(dockerRegistryServiceConnection)'
              repository: '$(imageRepository)'
              command: 'buildAndPush'
              Dockerfile: '$(dockerfilePath)'
              tags: |
                $(tag)
                latest
