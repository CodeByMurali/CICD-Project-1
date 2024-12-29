This project describes a Continuous Integration and Continuous Delivery pipeline for a Java application. The key components and processes involved are:

**Technologies:**

- **Jenkins:** A popular CI/CD tool used to orchestrate the pipeline.
- **Argo CD:** A Kubernetes-based CD tool that ensures consistency between the Git repository and the deployed application.
- **Maven:** A build tool for Java projects that automates the compilation, testing, and packaging of the application code.
- **SonarQube:** A static code analysis tool that identifies potential quality and security issues in the code.
- **Docker:** A containerization platform used to package the application into portable, isolated containers.
- **Argo Image Updater:** A tool that monitors the container registry and updates the application manifest (e.g., deployment.yaml) in the Git repository with the new image version.
- **GitHub:** A Git repository hosting both the application source code and the application manifests.


Architecture diagram

![image](https://github.com/user-attachments/assets/c3d42aa9-ed26-4b00-b446-9c0cbead2cbf)

