trigger:
  branches:
    include:
      - main
  paths:
    include:
      - result/*

variables:
  dockerRegistryServiceConnection: 'acr-docker-connection'
  imageRepository: 'resultapp'
  dockerfilePath: '$(Build.SourcesDirectory)/result/Dockerfile'
  tag: '$(Build.BuildId)'

pool:
  vmImage: 'ubuntu-latest'

stages:
  - stage: BuildAndPush
    jobs:
      - job: BuildAndPush
        steps:
          - task: Docker@2
            inputs:
              containerRegistry: '$(dockerRegistryServiceConnection)'
              repository: '$(imageRepository)'
              command: 'buildAndPush'
              Dockerfile: '$(dockerfilePath)'
              tags: |
                $(tag)
