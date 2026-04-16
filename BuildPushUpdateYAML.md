# Docker
# Build and push an image to Azure Container Registry
# https://docs.microsoft.com/azure/devops/pipelines/languages/docker

# Docker
# Build and push an image to Azure Container Registry
# Build, Push and Update Kubernetes manifests for Voting App

trigger:
  branches:
    include:
      - main
  paths:
    include:
      - vote/*

resources:
- repo: self

variables:
  dockerRegistryServiceConnection: 'azure1container1reg'   # your service connection
  imageRepository: 'votingapp'
  dockerfilePath: '$(Build.SourcesDirectory)/vote/Dockerfile'
  tag: '$(Build.BuildId)'

pool:
  name: 'azureagent'

stages:

# ---------------- BUILD ----------------
- stage: Build
  displayName: Build Image
  jobs:
  - job: Build
    displayName: Build
    steps:
    - task: Docker@2
      displayName: Build Docker Image
      inputs:
        containerRegistry: '$(dockerRegistryServiceConnection)'
        repository: '$(imageRepository)'
        command: 'build'
        Dockerfile: '$(dockerfilePath)'
        tags: |
          $(tag)
          latest

# ---------------- PUSH ----------------
- stage: Push
  displayName: Push Image
  dependsOn: Build
  jobs:
  - job: Push
    displayName: Push
    steps:
    - task: Docker@2
      displayName: Push Docker Image
      inputs:
        containerRegistry: '$(dockerRegistryServiceConnection)'
        repository: '$(imageRepository)'
        command: 'push'
        tags: |
          $(tag)
          latest

# ---------------- UPDATE ----------------
- stage: Update
  displayName: Update K8s Manifests
  dependsOn: Push
  jobs:
  - job: Update
    displayName: Update
    steps:
    - task: ShellScript@2
      displayName: Update Kubernetes Manifests
      inputs:
        scriptPath: 'scripts/updateK8sManifests.sh'
        args: 'vote $(imageRepository) $(tag)'
