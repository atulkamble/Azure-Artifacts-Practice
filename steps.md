```
// Azure Artifacts

1. note down

organisation - cloudnautic
project - project

https://dev.azure.com/cloudnautic/project

2. create feed - newfeed

3. create github repo - AzureArtifacts

git clone AzureArtifacts.git

mkdir package
cd package
touch config.json
touch app.py
touch desploy.sh

code .

// app.py

def hello() -> None:
    print("Hello from the Azure Artifacts package")


if __name__ == "__main__":
    hello()

// config.json

{
  "application": "cloudnautic-app",
  "environment": "training",
  "version": "1.0.0"
}

// deploy.sh

#!/bin/bash
echo "Starting application deployment"
echo "Package version: 1.0.0"
python app.py
echo "Deployment completed"


cd ..
git add .
git commit "code"
git push origin main


3. Create Token

Organsation >> Create Token >> packaging permissions

note down token

az extension add --name azure-devops

az extension update --name azure-devops

az devops configure --defaults organization=https://dev.azure.com/cloudnautic project=project

azure devops login

// paste token


4. Project >> Artifacts >> Feed >> Select Feed >> Settings >> Permissions >> Update Feed Owner to current root user and also update Project Build Service to Feed Owner

Orgnization Settings >> Users >> User >> Access Level Basic X , Visual Studio Enterprise

5.

ORGANIZATION="https://dev.azure.com/cloudnautic"
PROJECT="project"
FEED="newfeed"
PACKAGE_NAME="cloudnautic-tools"
PACKAGE_VERSION="1.0.9"
PACKAGE_PATH="./package"
DOWNLOAD_PATH="./downloaded-package"

6. // Publish

"$ARTIFACT_TOOL" \
universal publish \
--service "$ORGANIZATION" \
--patvar PATVAR \
--feed "$FEED" \
--package-name "$PACKAGE_NAME" \
--package-version "$PACKAGE_VERSION" \
--path "$PACKAGE_PATH" \
--project "$PROJECT" \
--description "Cloudnautic training package $PACKAGE_VERSION"

7. // Download

rm -rf "$DOWNLOAD_PATH"

mkdir -p "$DOWNLOAD_PATH"

"$ARTIFACT_TOOL" \
universal download \
--service "$ORGANIZATION" \
--patvar PATVAR \
--feed "$FEED" \
--package-name "$PACKAGE_NAME" \
--package-version "$PACKAGE_VERSION" \
--path "$DOWNLOAD_PATH" \
--project "$PROJECT"

8. create azure-pipelines.yml


trigger:
  - main

pool:
  vmImage: ubuntu-latest

variables:
  PROJECT_NAME: project
  FEED_NAME: newfeed
  PACKAGE_NAME: cloudnautic-tools
  PACKAGE_VERSION: 1.0.$(Build.BuildId)
  PACKAGE_SOURCE: $(Build.SourcesDirectory)/package
  DOWNLOAD_PATH: $(Build.SourcesDirectory)/downloaded-package

steps:
  - checkout: self

  - script: |
      echo "Testing Python application"
      python3 $(PACKAGE_SOURCE)/app.py

      echo "Testing deployment script"
      chmod +x $(PACKAGE_SOURCE)/deploy.sh
      cd $(PACKAGE_SOURCE)
      ./deploy.sh
    displayName: Test Package Files

  - task: CopyFiles@2
    displayName: Prepare Universal Package
    inputs:
      SourceFolder: $(PACKAGE_SOURCE)
      Contents: '**'
      TargetFolder: $(Build.ArtifactStagingDirectory)
      CleanTargetFolder: true

  - task: UniversalPackages@0
    displayName: Publish Universal Package
    inputs:
      command: publish
      publishDirectory: $(Build.ArtifactStagingDirectory)
      feedsToUsePublish: internal
      vstsFeedPublish: $(PROJECT_NAME)/$(FEED_NAME)
      vstsFeedPackagePublish: $(PACKAGE_NAME)
      versionOption: custom
      versionPublish: $(PACKAGE_VERSION)
      packagePublishDescription: Cloudnautic training package $(PACKAGE_VERSION)

  - task: UniversalPackages@0
    displayName: Download Published Package
    inputs:
      command: download
      feedsToUse: internal
      vstsFeed: $(PROJECT_NAME)/$(FEED_NAME)
      vstsFeedPackage: $(PACKAGE_NAME)
      vstsPackageVersion: $(PACKAGE_VERSION)
      downloadDirectory: $(DOWNLOAD_PATH)

  - script: |
      echo "Downloaded package contents:"
      find "$(DOWNLOAD_PATH)" -type f

      echo "Running downloaded application:"
      cd "$(DOWNLOAD_PATH)"
      python3 app.py

      echo "Running downloaded deployment script:"
      chmod +x deploy.sh
      ./deploy.sh
    displayName: Verify Downloaded Package

9. pipeline build run
10. feed - package check

11. Connect to feed

Get this package with the Azure DevOps Extension for Azure CLI

az artifacts universal download --organization "https://dev.azure.com/cloudnautic/" --project "9a6ab683-9cef-465e-84a6-2fe6b5ec0add" --scope project --feed "newfeed" --name "cloudnautic-tools" --version "1.0.383" --path .

```
