### Setting Up AWS CodeArtifact Repository

#### Overview
AWS CodeArtifact allows the creation of private artifact repositories to comply with security or compliance requirements by avoiding internet-based repositories. Maven can download dependencies from these private repositories instead of public repositories like Maven Central.

---

### Step-by-Step Process

#### 1. Create AWS CodeArtifact Repository
1. **Navigate to CodeArtifact:**
   - Go to the AWS Console.
   - Search for **CodeArtifact** or access it via the Developer Tools section.

2. **Create Repository:**
   - Click **Getting Started** and review the architecture.
   - Click **Create Repository** and fill in the following:
     - **Name:** `vprofile-maven-repo`
     - **Description:** Custom Maven repository
     - **Upstream Repository:** Select **Maven Central Store**.
   - Create a **Domain Name:** Example: `HC-Infotech`.
   - Review and click **Create Repository**.

3. **Verify Repository:**
   - Navigate to **Repositories** to confirm:
     - `Maven Central Store`
     - `vprofile-maven-repo`

4. **Connection Instructions:**
   - Select **Package Manager:** Maven.
   - Generate authentication token command (to be used in environment variables).

---

#### 2. Configure Maven
1. **Edit `pom.xml`:**
   - Add repository details at the end of the file:
     ```xml
     <repositories>
       <repository>
         <id>codeartifact</id>
         <url>YOUR-CODEARTIFACT-URL</url>
       </repository>
     </repositories>
     ```

2. **Edit `settings.xml`:**
   - Define repository authentication details:
     ```xml
     <servers>
       <server>
         <id>codeartifact</id>
         <username>AWS</username>
         <password>${env.CODEARTIFACT_AUTH_TOKEN}</password>
       </server>
     </servers>
     ```
   - Add repository mirror settings:
     ```xml
     <mirrors>
       <mirror>
         <id>codeartifact</id>
         <url>YOUR-CODEARTIFACT-URL</url>
         <mirrorOf>*</mirrorOf>
       </mirror>
     </mirrors>
     ```

3. **Token Authentication:**
   - Use the provided `export` command to set the authentication token in your environment:
     ```bash
     export CODEARTIFACT_AUTH_TOKEN=$(aws codeartifact get-authorization-token ...)
     ```

4. **Copy `settings.xml` to Maven Default Location:**
   - Ensure `settings.xml` is available at `~/.m2/settings.xml`.

---

#### 3. AWS CodeBuild Configuration
1. **Buildspec File:** Define `buildspec.yml` in YAML format with the following structure:
   - **Phases:**
     - **Install:**
       - Define runtime versions (e.g., Java 17).
       - Copy `settings.xml` to the `.m2` directory.
       - Export the authentication token.
     - **Pre-Build:**
       - Update package manager and install dependencies like Maven and SonarScanner.
     - **Build:**
       - Execute commands such as `mvn test`, `mvn checkstyle`, and `mvn sonar`.
   - Example:
     ```yaml
     phases:
       install:
         commands:
           - runtime-versions:
               java: corretto17
           - cp settings.xml ~/.m2/settings.xml
           - export CODEARTIFACT_AUTH_TOKEN=$(aws codeartifact get-authorization-token ...)
       pre_build:
         commands:
           - apt-get update && apt-get install -y jq
           - wget MAVEN_URL && tar -xzf maven.tar.gz
           - export PATH=$PATH:/path/to/maven
       build:
         commands:
           - mvn test
           - mvn checkstyle
           - mvn sonar
     ```

2. **Parameter Store:** Store sensitive values like SonarCloud tokens securely and reference them in the buildspec file.

---

#### 4. Key Points
1. **Use AWS CLI:** To retrieve and set CodeArtifact tokens dynamically.
2. **Validate Build Spec:**
   - Test locally before running in CodeBuild.
3. **Integration with Other Tools:**
   - Jenkins or other CI/CD tools can also point to the CodeArtifact repository.

---

By following these steps, you can successfully configure AWS CodeArtifact to manage dependencies securely and integrate with Maven-based projects.

