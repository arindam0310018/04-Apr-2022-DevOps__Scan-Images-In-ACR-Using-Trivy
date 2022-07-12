# DEVOPS + ACR + TRIVY

Greetings my fellow Technology Advocates and Specialists.

In this Session, I will demonstrate how to __Scan Docker Images__ in __AZURE CONTAINER REGISTRY__ with __AQUASEC TRIVY__ using __AZURE DEVOPS PIPELINES__

| __IN-PERSON SESSION:-__ |
| --------- |
| I presented ACR (Architecture and Best Practices) + Scan Images in ACR Using TRIVY and DEVOPS in __AZURE ZURICH USER GROUP__ Forum/Platform |
| __Event Meetup Announcement:-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jtq9o4y4z95uziaygvp0.png) |
| __Moment Captured with Founder of Azure Zurich User Group "MANUEL MEYER", Co-organizer "THOMAS HAFERMALZ" and Co-Speaker "MOHAMMAD NOFAL" :-__ |
| ![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kcrq2xc69sw5qyr1u7jw.png) |


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
