# DEVOPS + ACR + TRIVY

Greetings my fellow Technology Advocates and Specialists.

In this Session, I will demonstrate how to __Scan Docker Images__ in __AZURE CONTAINER REGISTRY__ with __AQUASEC TRIVY__ using __AZURE DEVOPS PIPELINES__

I had the Privilege to talk on this topic in __TWO__ Azure Communities:-

| __NAME OF THE AZURE COMMUNITY__ | __TYPE OF SPEAKER SESSION__ |
| --------- | --------- |
| __Azure Zurich User Group__ | __In Person__ |
| __Microsoft Azure Pakistan Community__ | __Virtual__ |
 
| __IN-PERSON SESSION:-__ |
| --------- |
| I presented ACR (Architecture and Best Practices) + Scan Images in ACR Using TRIVY and DEVOPS in __AZURE ZURICH USER GROUP__ Forum/Platform |
| __Event Meetup Announcement:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jtq9o4y4z95uziaygvp0.png) |
| __Moment Captured with Founder of Azure Zurich User Group "MANUEL MEYER", Co-organizer "THOMAS HAFERMALZ" and Co-Speaker "MOHAMMAD NOFAL" :-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kcrq2xc69sw5qyr1u7jw.png) |

| __VIRTUAL SESSION:-__ |
| --------- |
| I presented ACR (Architecture and Best Practices) + Scan Images in ACR Using TRIVY and DEVOPS in __MICROSOFT AZURE PAKISTAN COMMUNITY__ Forum/Platform |
| __Event Meetup Announcement:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0qsjncev5uboixqnza50.png) |
| __LIVE RECORDED SESSION:-__ |
| __LIVE DEMO__ was Recorded as part of my Presentation. |
| Duration of My Demo = __52 Mins 45 Secs__ |
| {% youtube 6fGGT1uBImw %} |

| __REQUIREMENTS:-__ |
| --------- |

1. Azure Container Registry
2. Azure Storage Account 
3. Azure Resource Manager Service Connection
4. Docker Registry (Azure Container Registry) Service Connection
5. Dockerfile
6. Sample HTML File
7. Azure DevOps Pipeline (YAML)
8. Trivy Ignore file (.trivyignore)

| __WHAT DOES THE PIPELINE DO:-__ |
| --------- |

| # | PIPELINE TASKS | 
| --------- | --------- |
| 1. | BUILD AND PUSH THE IMAGE IN ACR |
| 2. | DOWNLOAD AND INSTALL AQUASEC TRIVY | 
| 3. | EXECUTE TRIVY SCAN AND COPY THE SCAN RESULTS IN ARTIFACTS STAGING DIRECTORY |
| 4. | PUBLISH THE ARTIFACTS |
| 5. | DOWNLOAD THE PUBLISHED ARTIFACTS |
| 6. | COPY THE AQUASEC TRIVY SCAN REPORTS TO BLOB STORAGE CONTAINER WITH DATE TIME STAMP DIRECTORY  |


| __WHY IS TRIVY IGNORE FILE (.trivyignore) REQUIRED ?__ |
| --------- |
| After Scanning of the Image, we identify LOW, MEDIUM, HIGH and CRITICAL Vulnerabilities. The CVE (Common Vulnerabilities and Exposures) gets listed in the Report. If for some reasons, Application team accepts the risk and wants to skip the LOW and MEDIUM Vulnerabilities from the Scan report, all we have to do is list the respective CVEs in the .trivyignore file and run the pipeline again to scan. The listed CVEs will no longer be in the Scan Report. |


| __Below follows the contents of the Docker File:-__ |
| --------- |
```
FROM nginx:1.15.9-alpine
COPY . /usr/share/nginx/html
```

| __Below follows the contents of the HTML File:-__ |
| --------- |

```
<html>
<head>
    <title>TEST - ARINDAM MITRA</title>
</head>

</style>
<body>
    <h1 style="font-size:50px; color:#000000; text-align: center">TEST - ARINDAM MITRA</h1>
</body>
</html>
```

| __Below follows the contents of the YAML File (Azure DevOps):-__ |
| --------- |

```
trigger:
  none

resources:
- repo: self

###############################################
# All Declared Variables are listed below:-
###############################################
variables:
  dockerRegistryServiceConnection: '52409f6c-7855-4be2-a142-7192521b3e3f'
  imageRepository: 'amimagescantrivy'
  containerRegistry: 'ampocapplacr.azurecr.io'
  storageaccount: 'am4prodvs4core4shell'
  resourcegroup: 'Demo-Blog-RG'
  sacontainername: 'trivy-scan-reports'
  serviceconn: 'amcloud-cicd-service-connection' 
  dockerfilePath: '$(Build.SourcesDirectory)/ACR+Trivy/Dockerfile'
  target: $(build.artifactstagingdirectory)
  artifact: AM
  tag: '$(Build.BuildId)'

  vmImageName: 'ubuntu-latest'

stages:
- stage: BUILD
  displayName: Build and Push Stage
  jobs:
  - job: BUILD_JOB
    displayName: BUILD IMAGE
    pool:
      vmImage: $(vmImageName)
    steps:

#####################################
# Build and Push the Image to ACR:-
#####################################    
    - task: Docker@2
      displayName: BUILD AND PUSH IMAGE TO ACR
      inputs:
        command: buildAndPush
        repository: $(imageRepository)
        dockerfile: $(dockerfilePath)
        containerRegistry: $(dockerRegistryServiceConnection)
        tags: |
          $(tag)

#######################################
# Download and Install Aquasec Trivy:-
#######################################
    - task: CmdLine@2
      displayName: DOWNLOAD AND INSTALL AQUASEC TRIVY
      inputs:
        script: |
         sudo apt-get install wget apt-transport-https gnupg lsb-release
         wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
         echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
         sudo apt-get update
         sudo apt-get install trivy
         trivy -v
         pwd

##################################################################################
# Execute Trivy Scan and Copy the Scan Results in Artifacts Staging Directory:-
##################################################################################
    - task: CmdLine@2
      displayName: RUN AQUASEC TRIVY SCAN AND COPY TO ARTIFACTS STAGING DIRECTORY
      inputs:
        script: |
          trivy image --exit-code 0 --severity LOW,MEDIUM $(containerRegistry)/$(imageRepository):$(tag) > low-med.txt
          trivy image --exit-code 1 --severity HIGH,CRITICAL $(containerRegistry)/$(imageRepository):$(tag) > high-critical.txt
          ls -l
          cp -rvf *.txt $(target)

##########################
# Publish the Artifacts:-
##########################
    - task: PublishBuildArtifacts@1
      displayName: PUBLISH ARTIFACTS
      inputs:
        targetPath: '$(target)'
        artifactName: '$(artifact)'

######################################
# Download the Published Artifacts:-
######################################
    - task: DownloadBuildArtifacts@1
      displayName: DOWNLOAD ARTIFACTS
      inputs:
        buildType: 'current'
        downloadType: 'single'
        downloadPath: '$(System.ArtifactsDirectory)'

#########################################################
# The Below Code Snippet did not work because it is 
# only supported by "Windows" Build Agent
#########################################################    
    # - task: AzureFileCopy@4
    #   displayName: COPY AQUASEC TRIVY SCAN REPORTS TO BLOB STORAGE
    #   inputs:
    #     SourcePath: '$(System.ArtifactsDirectory)'
    #     azureSubscription: 'amcloud-cicd-service-connection'
    #     Destination: 'AzureBlob'
    #     storage: '$(storageaccount)'
    #     ContainerName: '$(sacontainername)/$(Date)'

###################################################################################
# Cmd in Line 118 - It works on Linux Build Agent because of the Date Format
# Cmd in Line 119 - It works on Windows Build Agent because of the Date Format
###################################################################################
    - task: AzureCLI@1
      displayName: COPY AQUASEC TRIVY SCAN REPORTS TO BLOB STORAGE
      inputs:
        azureSubscription: '$(serviceconn)'
        scriptType: ps
        scriptLocation: inlineScript
        inlineScript: |
          az storage blob upload-batch -d $(sacontainername)/$(date "+%d-%m-%Y_%H-%M-%S") --account-name $(storageaccount) --account-key $(az storage account keys list -g $(resourcegroup) -n $(storageaccount) --query [0].value -o tsv) -s $(System.ArtifactsDirectory)/AM
#         az storage blob upload-batch -d trivy-scan-reports/$(Get-Date -Format dd-MM-yyyy_HH-mm) --account-name $(storageaccount) --account-key $(az storage account keys list -g $(resourcegroup) -n $(storageaccount) --query [0].value -o tsv) -s $(System.ArtifactsDirectory)/AM

```
| __Below follows the contents of Trivy Ignore File (.trivyignore):-__ |
| --------- |

```
# All LOW and MEDIUM Vulnerabilities has been ignored (Except 2)!!!
#CVE-2018-5711
#CVE-2018-14048
CVE-2018-14498
CVE-2019-1547
CVE-2019-1549
CVE-2019-1551
CVE-2019-1563
CVE-2019-7317
CVE-2019-11038
CVE-2019-12904
CVE-2019-13627
CVE-2020-1971
CVE-2020-14155
CVE-2020-15999
CVE-2020-24977
CVE-2020-28928
CVE-2021-3449
CVE-2021-23841
CVE-2021-23839
```

| __PIPELINE RESULTS:-__ |
| --------- |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4x8q2ujk8ney1vtp0b2k.png) |

| __ARTIFACTS PUBLISHED:-__ |
| --------- |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gkiwsquultdc8h7403t9.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/oibtfxw2k4eqyvstr02b.png) |

| __IMAGE SCAN REPORT STORED INSIDE STORAGE CONTAINER UNDER DATE TIME STAMP DIRECTORY:-__ |
| --------- |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/e2umu2vymusgdove9db7.png) |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mebah8oubi7pnw2ta0rn.png) |

| __HOW IMAGE SCAN REPORT LOOKS LIKE:-__ |
| --------- |

| HIGH AND CRITICAL VULNERABILITIES IMAGE SCAN REPORT:- |
| --------- |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/s1rr0rn6cgnx210asc39.png) |

| LOW AND MEDIUM VULNERABILITIES IMAGE SCAN REPORT:- |
| --------- |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y83jm776l1ux8xlqj1yc.png) |
| __Note-__ The Only Reason we see 2 CVEs with Severity MEDIUM is because they are commented out in __.trivyignore__ file |
