Q1: why we need to generate an SSH key?
We generate an SSH key to securely authenticate with remote servers or services, like Bitbucket, without using passwords. It ensures encrypted communication and enhances security by verifying the user's identity.

Q2:how pom.xml and settings.xml ?
`pom.xml` defines project dependencies, build configuration, and repository URLs, enabling Maven to manage builds and integrate with CodeArtifact.  

`settings.xml` provides authentication details (e.g., credentials for CodeArtifact), ensuring secure access to private repositories during builds.

Q3:How buildspec.yml file ?
The `buildspec.yml` file is essential for defining the build process in AWS CodeBuild. It specifies the commands for installation, pre-build, build, and post-build phases, manages environment variables, integrates with tools like SonarCloud, and handles artifact storage. This ensures consistent and automated build execution.

Q4:what is sonarcloud and how it useful?
SonarCloud is a cloud-based code quality and security analysis tool. It helps identify bugs, code smells, and vulnerabilities, ensuring clean, maintainable, and secure code during development. Integrating it into CI/CD pipelines enhances software quality.

Q5:how does aws system manager useful for this project?
AWS Systems Manager is useful for managing and automating tasks in AWS. In this project, it helps by securely storing sensitive data (e.g., SonarCloud tokens) in Parameter Store, making it easy to retrieve and use them in build and deployment processes securely.
