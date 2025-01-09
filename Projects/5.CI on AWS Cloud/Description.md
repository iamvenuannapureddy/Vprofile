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
