
<p align="center">
  <a href="https://github.com/actions/checkout"><img alt="GitHub Actions status" src="https://github.com/actions/checkout/workflows/test-local/badge.svg"></a>
</p>

# Sign with CodeSignTool

CodeSignTool is a secure, privacy-oriented multi-platform Java command line utility for remotely signing Microsoft Authenticode and Java code objects with eSigner EV code signing certificates. Hashes of the files are sent to SSL.com for signing so that the code itself is not sent. This is ideal where sensitive files need to be signed, but should not be sent over the wire for signing. CodeSignTool is also ideal for automated batch processes for high volume signings or integration into existing CI/CD pipeline workflows.

This action provides the sign artifacts from your build.

# Usage

<!-- start usage -->
```yaml
- name: Sign Artifact with CodeSignTool
  uses: sslcom/actions-codesigner@develop
  with:
    # CodeSignTool Commands:
    # - get_credential_ids: Output the list of eSigner credential IDs associated with a particular user.
    # - credential_info: Output key and certificate information related to a credential ID.
    # - sign: Sign and timestamp code object.
    # - batch_sign: Sign and timestamp multiple code objects with one OTP.
    # - hash: Pre-compute hash(es) for later use with batch_hash_sign command.
    # - batch_sign_hash: Sign hash(es) pre-computed with hash command.
    command: sign

    # SSL.com account username
    username: ${{secrets.ES_USERNAME}}

    # SSL.com account password.
    password: ${{secrets.ES_PASSWORD}}

    # Credential ID for signing certificate.
    credential_id: ${{secrets.CREDENTIAL_ID}}

    # OAuth TOTP Secret (https://www.ssl.com/how-to/automate-esigner-ev-code-signing)
    totp_secret: ${{secrets.ES_TOTP_SECRET}}

    # Path of code object to be signed.
    file_path: ${GITHUB_WORKSPACE}/test/src/build/HelloWorld.jar

    # Directory where signed code object(s) will be written.
    output_path: ${GITHUB_WORKSPACE}/artifacts
```
<!-- end usage -->

## Dotnet Code Signing Example Workflow

<!-- start dotnet usage -->
```yml
# The name of the workflow.
name: Dotnet Core Build and Sign

# Trigger this workflow on a push
on: push

# Create an environment variable
env:
  PROJECT_NAME: HelloWorld
  PROJECT_VERSION: 0.0.1
  DOTNET_VERSION: 3.1

# Defines a single job named "codesigner-dotnet"
jobs:
  codesigner-dotnet:
    # Run job on Ubuntu Runner
    runs-on: ubuntu-latest
    # When the workflow runs, this is the name that is logged
    name: Run CodeSigner Action on Dotnet
    steps:
      # 1) Check out the source code so that the workflow can access it.
      - name: Checkout Repository
        uses: actions/checkout@v2

      # 2) Set up the .NET CLI environment for the workflow to use.
      - name: Setup Dotnet Core
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: '3.1.x'

      # 3) Create Artifact Directory to store signed and unsigned artifact files
      - name: Create Artifacts Directory
        shell: bash
        run: |
          mkdir ${GITHUB_WORKSPACE}/artifacts
          mkdir ${GITHUB_WORKSPACE}/packages

      # 4) Build a dotnet project or solution and all of its dependencies.
      #    After it has been created dll or exe file, copy to 'packages' folder for siging
      - name: Compile Dotnet Library
        shell: bash
        run: |
          dotnet build dotnet/${{env.PROJECT_NAME}}.csproj -c Release
          cp dotnet/bin/Release/netcoreapp${{env.DOTNET_VERSION}}/${{env.PROJECT_NAME}}-${{env.PROJECT_VERSION}}.dll ${GITHUB_WORKSPACE}/packages/${{env.PROJECT_NAME}}.dll

      # 5) This is the step where the created DLL or EXE (artifact) files will be signed with CodeSignTool.
      - name: Sign Artifact with CodeSignTool
        uses: bayrakmustafa/actions-codesigner@develop
        with:
          # Sign and timestamp code object.
          command: sign
          # SSL.com account username
          username: ${{secrets.ES_USERNAME}}
          # SSL.com account password.
          password: ${{secrets.ES_PASSWORD}}
          # Credential ID for signing certificate.
          credential_id: ${{secrets.CREDENTIAL_ID}}
          # OAuth TOTP Secret (https://www.ssl.com/how-to/automate-esigner-ev-code-signing)
          totp_secret: ${{secrets.ES_TOTP_SECRET}}
          # Path of code object to be signed. (DLL, JAR, EXE, MSI files vb... )
          file_path: ${GITHUB_WORKSPACE}/packages/${{env.PROJECT_NAME}}.dll
          # Directory where signed code object(s) will be written.
          output_path: ${GITHUB_WORKSPACE}/artifacts

        # 6) This uploads artifacts from your workflow allowing you to share data between jobs and store data once a workflow is complete
      - name: Upload Signed Files
        uses: actions/upload-artifact@v2
        with:
          name: ${{env.PROJECT_NAME}}.dll
          path: ./artifacts/${{env.PROJECT_NAME}}.dll
```
<!-- end dotnet usage -->

## Java Code (Maven) Signing Example Workflow

<!-- start maven usage -->
```yml
# The name of the workflow.
name: Java with Maven Build and Sign

# Trigger this workflow on a push
on: push

# Create an environment variable
env:
  PROJECT_NAME: HelloWorld
  PROJECT_VERSION: 0.0.1
  MAVEN_VERSION: 3.8.5
  JAVA_VERSION: 17

# Defines a single job named "codesigner-maven"
jobs:
  codesigner-maven:
    # Run job on Ubuntu Runner
    runs-on: ubuntu-latest
    # When the workflow runs, this is the name that is logged
    name: Run CodeSigner Action on Java with Maven
    steps:
      # 1) Check out the source code so that the workflow can access it.
      - name: Checkout Repository
        uses: actions/checkout@v2

      # 2) Set up the Java and Maven environment for the workflow to use.
      - name: Setup Java and Maven
        uses: s4u/setup-maven-action@v1.3.1
        with:
          java-version: '${{env.JAVA_VERSION}}'
          maven-version: '${{env.MAVEN_VERSION}}'

      # 3) Create Artifact Directory to store signed and unsigned artifact files
      - name: Create Artifacts Directory
        shell: bash
        run: |
          mkdir ${GITHUB_WORKSPACE}/artifacts
          mkdir ${GITHUB_WORKSPACE}/packages

      # 4) Build a dotnet project or solution and all of its dependencies.
      #    After it has been created jar file, copy to 'packages' folder for siging
      - name: Compile Java Library with Maven
        shell: bash
        run: |
          mvn clean install -f java/pom.xml
          cp java/target/${{env.PROJECT_NAME}}-${{env.PROJECT_VERSION}}.jar ${GITHUB_WORKSPACE}/packages/${{env.PROJECT_NAME}}.jar

      # 5) This is the step where the created JAR (artifact) files will be signed with CodeSignTool.
      - name: Sign Artifact with CodeSignTool
        uses: bayrakmustafa/actions-codesigner@develop
        with:
          # Sign and timestamp code object.
          command: sign
          # SSL.com account username
          username: ${{secrets.ES_USERNAME}}
          # SSL.com account password.
          password: ${{secrets.ES_PASSWORD}}
          # Credential ID for signing certificate.
          credential_id: ${{secrets.CREDENTIAL_ID}}
          # OAuth TOTP Secret (https://www.ssl.com/how-to/automate-esigner-ev-code-signing)
          totp_secret: ${{secrets.ES_TOTP_SECRET}}
          # Path of code object to be signed. (DLL, JAR, EXE, MSI files vb... )
          file_path: ${GITHUB_WORKSPACE}/packages/${{env.PROJECT_NAME}}.jar
          # Directory where signed code object(s) will be written.
          output_path: ${GITHUB_WORKSPACE}/artifacts

        # 6) This uploads artifacts from your workflow allowing you to share data between jobs and store data once a workflow is complete
      - name: Upload Signed Files
        uses: actions/upload-artifact@v2
        with:
          name: ${{env.PROJECT_NAME}}.jar
          path: ./artifacts/${{env.PROJECT_NAME}}.jar
```
<!-- end maven usage -->

## Java Code (Gradle) Signing Example Workflow

<!-- start gradle usage -->
```yml
# The name of the workflow.
name: Java with Gradle Build and Sign

# Trigger this workflow on a push
on: push

# Create an environment variable
env:
  PROJECT_NAME: HelloWorld
  PROJECT_VERSION: 0.0.1
  GRADLE_VERSION: 7.3
  JAVA_VERSION: 17

# Defines a single job named "codesigner-gradle"
jobs:
  codesigner-gradle:
    # Run job on Ubuntu Runner
    runs-on: ubuntu-latest
    # When the workflow runs, this is the name that is logged
    name: Run CodeSigner Action on Java with Gradle
    steps:
      # 1) Check out the source code so that the workflow can access it.
      - name: Checkout Repository
        uses: actions/checkout@v2

      # 2) Set up the Java environment for the workflow to use.
      - uses: actions/setup-java@v2
        with:
          distribution: zulu
          java-version: ${{env.JAVA_VERSION}}

      # 3) Set up the Gradle environment for the workflow to use.
      - name: Setup Gradle
        uses: gradle/gradle-build-action@v2
        with:
          gradle-version: ${{env.GRADLE_VERSION}}

      # 4) Create Artifact Directory to store signed and unsigned artifact files
      - name: Create Artifacts Directory
        shell: bash
        run: |
          mkdir ${GITHUB_WORKSPACE}/artifacts
          mkdir ${GITHUB_WORKSPACE}/packages

      # 5) Build a dotnet project or solution and all of its dependencies.
      #    After it has been created jar file, copy to 'packages' folder for siging
      - name: Compile Java Library with Maven
        shell: bash
        run: |
          gradle clean build -p java
          cp java/build/libs/${{env.PROJECT_NAME}}-${{env.PROJECT_VERSION}}.jar ${GITHUB_WORKSPACE}/packages/${{env.PROJECT_NAME}}.jar

      # 6) This is the step where the created JAR (artifact) files will be signed with CodeSignTool.
      - name: Sign Artifact with CodeSignTool
        uses: bayrakmustafa/actions-codesigner@develop
        with:
          # Sign and timestamp code object.
          command: sign
          # SSL.com account username
          username: ${{secrets.ES_USERNAME}}
          # SSL.com account password.
          password: ${{secrets.ES_PASSWORD}}
          # Credential ID for signing certificate.
          credential_id: ${{secrets.CREDENTIAL_ID}}
          # OAuth TOTP Secret (https://www.ssl.com/how-to/automate-esigner-ev-code-signing)
          totp_secret: ${{secrets.ES_TOTP_SECRET}}
          # Path of code object to be signed. (DLL, JAR, EXE, MSI files vb... )
          file_path: ${GITHUB_WORKSPACE}/packages/${{env.PROJECT_NAME}}.jar
          # Directory where signed code object(s) will be written.
          output_path: ${GITHUB_WORKSPACE}/artifacts

        # 7) This uploads artifacts from your workflow allowing you to share data between jobs and store data once a workflow is complete
      - name: Upload Signed Files
        uses: actions/upload-artifact@v2
        with:
          name: ${{env.PROJECT_NAME}}.jar
          path: ./artifacts/${{env.PROJECT_NAME}}.jar
```
<!-- end gradle usage -->

# CodeSignTool Guide

* https://www.ssl.com/guide/esigner-codesigntool-command-guide
