<h1>Continuous Integration on AWS Cloud</h1>

### 1. **Bitbucket**  
**a. Create Account & Repository on Bitbucket**  
   1. Sign up at [Bitbucket](https://bitbucket.org/).  
   2. Log in to your account.  
   3. Create a new repository:
      - Navigate to "Repositories" > "Create Repository."  
      - Enter repository name and visibility (private/public).  
      - Choose project type and create the repository.  

**b. SSH Authentication from Local to Bitbucket Account**  
   1. Generate an SSH key (if not already done):
      ```bash
      ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
      ```
   2. Copy the SSH public key:
      ```bash
      cat ~/.ssh/id_rsa.pub
      ```
   3. Add the SSH key to Bitbucket:
      - Go to "Personal Settings" > "SSH Keys" > "Add Key."
   4. Test the SSH connection:
      ```bash
      ssh -T git@bitbucket.org
      ```

**c. Migrate vProfile Project Source Code from GitHub to Bitbucket**  
   1. Clone the GitHub repository locally:
      ```bash
      git clone https://github.com/<username>/<repo>.git
      ```
   2. Add the Bitbucket remote:
      ```bash
      cd <repo>
      git remote add bitbucket ssh://git@bitbucket.org/<username>/<repo>.git
      ```
   3. Push code to Bitbucket:
      ```bash
      git push bitbucket master
      ```

---

### 2. **AWS CodeArtifact**  
**a. Create CodeArtifact Repository**  
   1. Open the AWS Management Console > CodeArtifact.  
   2. Create a repository:
      - Click "Create Repository."
      - Enter repository name and domain name.  
   3. Note the repository endpoint.

**b. Configure `pom.xml` and `settings.xml`**  
   1. Update your `pom.xml` with the CodeArtifact repository URL:
      ```xml
      <repositories>
        <repository>
          <id>codeartifact-repo</id>
          <url>https://<domain>.d.codeartifact.<region>.amazonaws.com/maven/<repo-name>/</url>
        </repository>
      </repositories>
      ```
   2. Update `settings.xml` for authentication:
      ```xml
      <servers>
        <server>
          <id>codeartifact-repo</id>
          <username>aws</username>
          <password>${env.CODEARTIFACT_AUTH_TOKEN}</password>
        </server>
      </servers>
      ```

**c. Understand `buildspec.yml` for Code Analysis Build Job**  
   - Include commands to install dependencies and trigger Sonar analysis.

Let's go through this configuration step by step to understand its purpose and functionality. The file appears to define a build process in **AWS CodeBuild** using **Buildspec version 0.2**.

---

### **Header Section**

```yaml
version: 0.2
```

This specifies the Buildspec file version. Version `0.2` is the latest as of now, allowing more flexible build phases and environment specifications.

---

### **Environment Variables**

```yaml
env:
  parameter-store:
    LOGIN: LOGIN
    HOST: HOST
    Organization: Organization
    Project: Project
    #CODEARTIFACT_AUTH_TOKEN: CODEARTIFACT_AUTH_TOKEN
```

- **env.parameter-store**: These environment variables are securely stored in AWS Parameter Store. They are fetched at runtime to keep sensitive information (e.g., login credentials, tokens) secure.
  - **LOGIN**: SonarQube/SonarCloud login key.
  - **HOST**: The URL for the SonarQube/SonarCloud server.
  - **Organization**: The organization identifier in SonarQube/SonarCloud.
  - **Project**: The key representing the project in SonarQube/SonarCloud.
  - **#CODEARTIFACT_AUTH_TOKEN**: (Commented out) Token for AWS CodeArtifact authentication. Uncomment if needed.

---

### **Phases**

The process is divided into **three key phases**: `install`, `pre_build`, and `build`.

#### 1. **Install Phase**

```yaml
install:
  runtime-versions:
    java: corretto17
  commands:
    - cp ./settings.xml /root/.m2/settings.xml
    - export CODEARTIFACT_AUTH_TOKEN=`aws codeartifact get-authorization-token --domain ArtifactDomain --domain-owner ID --region REGION --query authorizationToken --output text`
```

- **runtime-versions.java: corretto17**: Specifies the Java runtime version (Amazon Corretto 17) for the build environment.
- **commands**:
  - `cp ./settings.xml /root/.m2/settings.xml`: Copies the Maven `settings.xml` file to the default Maven configuration directory for dependency management.
  - `export CODEARTIFACT_AUTH_TOKEN=...`: Fetches an authentication token from AWS CodeArtifact and assigns it to the environment variable `CODEARTIFACT_AUTH_TOKEN`. Replace placeholders like `ArtifactDomain`, `ID`, and `REGION` with actual values.

---

#### 2. **Pre-Build Phase**

```yaml
pre_build:
  commands:
    - apt-get update
    - apt-get install -y jq checkstyle
    - wget https://dlcdn.apache.org/maven/maven-3/3.9.4/binaries/apache-maven-3.9.4-bin.tar.gz
    - tar xzvf apache-maven-3.9.4-bin.tar.gz
    - ln -s apache-maven-3.9.4 maven
    - wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.3.0.1492-linux.zip
    - unzip ./sonar-scanner-cli-3.3.0.1492-linux.zip
    - export PATH=$PATH:/sonar-scanner-3.3.0.1492-linux/bin/
```

- **commands**:
  - `apt-get update`: Updates the package list to ensure the latest versions of packages are fetched.
  - `apt-get install -y jq checkstyle`: Installs `jq` (for JSON parsing) and `checkstyle` (for static code analysis).
  - `wget ...maven-3.9.4-bin.tar.gz`: Downloads the specified version of Apache Maven.
  - `tar xzvf apache-maven-3.9.4-bin.tar.gz`: Extracts the downloaded Maven tarball.
  - `ln -s apache-maven-3.9.4 maven`: Creates a symbolic link for easier Maven access.
  - `wget ...sonar-scanner-cli...`: Downloads the SonarQube Scanner CLI tool.
  - `unzip ...sonar-scanner-cli...`: Unzips the SonarQube Scanner CLI.
  - `export PATH=$PATH:/sonar-scanner-3.3.0.1492-linux/bin/`: Updates the system's `PATH` to include the SonarQube Scanner binary.

---

#### 3. **Build Phase**

```yaml
build:
  commands:
    - mvn test
    - mvn checkstyle:checkstyle
    - mvn sonar:sonar -Dsonar.login=$LOGIN -Dsonar.host.url=$HOST -Dsonar.projectKey=$Project -Dsonar.organization=$Organization -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ -Dsonar.junit.reportsPath=target/surefire-reports/ -Dsonar.jacoco.reportsPath=target/jacoco.exec -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
    - sleep 5
    - curl https://sonarcloud.io/api/qualitygates/project_status?projectKey=$Project >result.json
    - cat result.json
    - if [ $(jq -r '.projectStatus.status' result.json) = ERROR ] ; then $CODEBUILD_BUILD_SUCCEEDING -eq 0 ;fi
```

- **commands**:
  1. `mvn test`: Runs unit tests using Maven.
  2. `mvn checkstyle:checkstyle`: Executes the Checkstyle plugin to analyze code style issues and generate a report.
  3. `mvn sonar:sonar ...`: Runs the SonarQube analysis with various options:
     - `-Dsonar.login=$LOGIN`: Uses the login token.
     - `-Dsonar.host.url=$HOST`: Specifies the SonarQube server URL.
     - `-Dsonar.projectKey=$Project`: Uses the project key for identification.
     - `-Dsonar.organization=$Organization`: Targets the organization.
     - Other flags specify paths for Java binaries, JUnit test reports, Jacoco coverage reports, and Checkstyle results.
  4. `sleep 5`: Waits for 5 seconds to ensure the SonarQube analysis results are ready.
  5. `curl https://sonarcloud.io/api/qualitygates/project_status?projectKey=$Project >result.json`: Fetches the project quality gate status from SonarCloud and saves it to `result.json`.
  6. `cat result.json`: Displays the quality gate results.
  7. `if [ $(jq -r '.projectStatus.status' result.json) = ERROR ] ; then $CODEBUILD_BUILD_SUCCEEDING -eq 0 ;fi`: Checks the quality gate status. If it's `ERROR`, the build fails by setting the `$CODEBUILD_BUILD_SUCCEEDING` variable to `0`.

---

This configuration ensures:
1. Dependencies and tools are installed in the `install` and `pre_build` phases.
2. Code is analyzed, tested, and validated against SonarQube quality gates during the `build` phase.
3. Builds fail automatically if SonarQube detects critical issues.

---

### 3. **SonarCloud**  
**a. Create Account and Set Project Details**  
   1. Sign up at [SonarCloud](https://sonarcloud.io/).  
   2. Create a new project and link it to your Bitbucket repository.  
   3. Obtain the project token and configure it in your build tool.

---

### 4. **AWS Parameter Store**  
**a. Store SonarCloud Details into Parameter Store**  
   1. Open AWS Systems Manager > Parameter Store.  
   2. Create a new parameter:
      - Name: `/sonarcloud/token`  
      - Value: SonarCloud project token.  

**b. Mention Parameter Store Details in `buildspec.yml` File**  
   - Retrieve the token using AWS CLI in `buildspec.yml`:  
      ```bash
      aws ssm get-parameter --name "/sonarcloud/token" --with-decryption --query Parameter.Value --output text
      ```

---

### 5. **AWS CodeBuild Job (Code Analysis)**  
**a. Understand `buildspec.yml` File for CodeBuild Job**  
   - Include pre-build and build phases to run tests and trigger Sonar analysis.

**b. Update `pom.xml` and `settings.xml` for CodeArtifact Repo Details**  
   - Ensure repository and credentials for CodeArtifact are updated.  

**c. Create a Build Job for Sonar Code Analysis**  
   1. Open AWS CodeBuild > Create Build Project.  
   2. Configure the source to use your Bitbucket repository.  
   3. Use the `buildspec.yml` file in your repository.  

**d. Execute & Test**  
   - Start the build and ensure it completes successfully.

---

### 6. **Build Job for Artifact**  
**a. Understand `buildspec.yml` File**  
   - Define stages for building the artifact and uploading it to S3.  

**b. Create S3 Bucket for Artifact Storage**  
   1. Open AWS S3 > Create Bucket.  
   2. Name the bucket and configure permissions.  

**c. Create AWS CodeBuild Job**  
   - Configure a build project to compile the code and upload artifacts to S3.  

**d. Execute & Test**  
   - Run the build and confirm the artifact is stored in S3.

---

### 7. **AWS CodePipeline**  
**a. Create SNS Notifications**  
   1. Open AWS SNS > Create Topic.  
   2. Subscribe an email to the topic.  

**b. Create AWS CodePipeline**  
   1. Open AWS CodePipeline > Create Pipeline.  
   2. Configure source as Bitbucket and build as CodeBuild.  
   3. Add S3 as a deployment step (for artifacts).  

**c. Execute & Test**  
   - Trigger the pipeline and ensure all stages complete successfully.  

--- 

This step-by-step process will help you execute the entire workflow from setting up the environment to deploying artifacts and ensuring notifications.
