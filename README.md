# Azure Artifacts Universal Packages – Complete Hands-on Guide

A complete hands-on guide to creating, publishing, downloading, and managing **Azure Artifacts Universal Packages** using **ArtifactTool**.

> **Note**
> During testing, the Azure CLI wrapper (`az artifacts universal publish`) consistently failed with **TF400813** despite correct permissions and PAT authentication. This guide uses the **verified working ArtifactTool commands**.

---

# Features

- Azure Artifacts Fundamentals
- Universal Packages
- Project Scoped Feed
- Package Versioning
- Publish Package
- Download Package
- Package Verification
- Best Practices
- Troubleshooting
- Complete Lab Exercise

---

# Prerequisites

- Azure DevOps Organization
- Azure DevOps Project
- Azure Artifacts Feed
- Personal Access Token (PAT)
- Azure CLI
- Azure DevOps Extension
- ArtifactTool

---

# Project Structure

```
Azure-Artifacts-Practice/
│
├── package/
│   ├── app.py
│   ├── config.json
│   ├── deploy.sh
│   └── .artifactignore
│
└── downloaded-package/
```

---

# Sample Python Application

```python
def hello() -> None:
    print("Hello from the Azure Artifacts package")


if __name__ == "__main__":
    hello()
```

---

# Configure Variables

```bash
ORGANIZATION="https://dev.azure.com/cloudnautic"

PROJECT="project"

FEED="newfeed"

PACKAGE_NAME="cloudnautic-tools"

PACKAGE_VERSION="1.0.9"

PACKAGE_PATH="./package"

DOWNLOAD_PATH="./downloaded-package"
```

---

# Authenticate

Export your Azure DevOps PAT.

```bash
export PATVAR="$AZURE_DEVOPS_EXT_PAT"
```

Verify

```bash
echo "PAT length: ${#PATVAR}"
```

Expected

```
PAT length: 84
```

---

# Locate ArtifactTool

```bash
ARTIFACT_TOOL=$(find ~/.azure/azuredevops/cli/tools/artifacttool \
-type f \
-name artifacttool \
| head -1)

echo "$ARTIFACT_TOOL"
```

Example

```
/Users/atul/.azure/azuredevops/cli/tools/artifacttool/ArtifactTool_osx-x64_0.2.565/artifacttool
```

---

# Publish Universal Package

```bash
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
```

Expected Output

```
Package does not yet exist

Pushing content...

Content pushed

Added package

Success
```

---

# Download Universal Package

```bash
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
```

Expected Output

```
Obtained package metadata

Download completed

Success
```

---

# Verify Download

```bash
tree downloaded-package
```

Expected

```
downloaded-package
├── app.py
├── config.json
└── deploy.sh
```

---

# Run Package

```bash
cd downloaded-package

python3 app.py
```

Output

```
Hello from the Azure Artifacts package
```

---

# Fix Permission

If you receive

```
permission denied: ./deploy.sh
```

Run

```bash
chmod +x deploy.sh

./deploy.sh
```

---

# Publish Next Version

Return to the project root.

```bash
cd ..
```

Update

```bash
PACKAGE_VERSION="1.0.10"
```

Publish

```bash
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
```

---

# Common Errors

## TF400813

```
TF400813: The user is not authorized.
```

### Solution

Use **ArtifactTool** instead of

```
az artifacts universal publish
```

---

## Package Already Exists

```
The package already exists
```

### Solution

Increase the package version.

```
1.0.9

↓

1.0.10
```

Universal Package versions are immutable.

---

## The path provided is invalid

Cause

Running publish command inside

```
downloaded-package/
```

while

```
PACKAGE_PATH="./package"
```

does not exist.

Solution

Return to the project root.

```bash
cd ..
```

---

## Permission Denied

```
permission denied: ./deploy.sh
```

Solution

```bash
chmod +x deploy.sh
```

---

# Best Practices

- Use Semantic Versioning (SemVer).
- Never overwrite an existing version.
- Store PAT securely.
- Keep `.artifactignore` updated.
- Publish only from the project root directory.
- Use meaningful package names.
- Test downloaded packages after publishing.

---

# Complete Lab

1. Create Azure Artifacts Feed.
2. Create sample package.
3. Configure PAT.
4. Locate ArtifactTool.
5. Publish version **1.0.9**.
6. Download version **1.0.9**.
7. Verify downloaded files.
8. Run the application.
9. Update package to **1.0.10**.
10. Publish again.
11. Verify the new version.

---

# Cleanup

```bash
unset PATVAR

unset AZURE_DEVOPS_EXT_PAT

unset AZURE_DEVOPS_EXT_ARTIFACTTOOL_PATVAR
```

---

# Summary

In this guide you learned how to:

- Configure Azure Artifacts
- Authenticate using a PAT
- Locate ArtifactTool
- Publish Universal Packages
- Download Universal Packages
- Verify package contents
- Handle package versioning
- Troubleshoot common errors
- Use the verified ArtifactTool workflow for Azure Artifacts

---

## Repository Structure

```
Azure-Artifacts-Practice
│
├── README.md
├── package/
│   ├── app.py
│   ├── config.json
│   ├── deploy.sh
│   └── .artifactignore
└── downloaded-package/
```

---

**Happy Learning! 🚀**
**Azure Artifacts | Azure DevOps | Universal Packages**
