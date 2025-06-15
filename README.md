

## Activating Jenkins CICD Pipeline

### Step 1: Deploy Infrastructure

Navigate to the GitHub repository `https://github.com/MeetingTeam/terraform` and run the following commands to deploy the necessary infrastructure:

```bash
terraform init
terraform apply
```

### Step 2: Configure Kubectl

Open Git Bash and run the following command to update your kubeconfig:

```bash
aws eks update-kubeconfig --name doan-cluster-<environment> --region ap-southeast-1
```

Replace `<environment>` with your specific environment (e.g., dev, prod).

### Step 3: Install Jenkins Helm Chart

Navigate to the directory containing the Jenkins Helm chart and run the following command:

```bash
helm install jenkins . -f values.custom.yaml -n jenkins
```

### Step 4: Obtain Kubernetes Token

Execute the following command to get the Kubernetes token for Jenkins:

```bash
kubectl create token jenkins -n jenkins --duration=8760h
```

### Step 5: Access Jenkins via Port Forwarding

Use kubectl to forward the Jenkins service port:

```bash
kubectl port-forward svc/jenkins 30808:8080 -n jenkins
```

Access Jenkins in your browser at `http://localhost:30808`.

### Step 6: Install Jenkins Plugins

Install the following plugins in Jenkins:

*   Kubernetes
*   Pipeline
*   Git
*   Configuration as Code Plugin

### Step 7: Add Credentials

Add your GitHub credentials and the Kubernetes token (obtained in Step 4) to Jenkins.

### Step 8: Create Pipeline

1.  **Navigate to Jenkins Dashboard:** Open Jenkins in your web browser (e.g., `http://localhost:30808`).
2.  **New Item:** Click on "New Item" on the left sidebar.
3.  **Enter Item Name:** Provide a name for your pipeline (e.g., "My-CICD-Pipeline").
4.  **Select Pipeline:** Choose "Pipeline" from the list of project types and click "OK".
5.  **Configure Pipeline:**
    *   **Description (Optional):** Add a description for your pipeline.
    *   **Pipeline Section:**
        *   **Definition:** Select "Pipeline script from SCM".
        *   **SCM:** Choose "Git".
        *   **Repository URL:** Enter the URL of your Kubernetes repository (e.g., `https://github.com/your-org/k8s-repo.git`).
        *   **Credentials:** Select the GitHub credentials you added in Step 7.
        *   **Branch Specifier:** Specify the branch to use (e.g., `*/main` or `*/develop`).
        *   **Script Path:** Enter the name of your Jenkinsfile (typically `Jenkinsfile`).
    *   **Save:** Click "Save" to create the pipeline.

### Configure Pipeline Parameters

Before running the pipeline for the first time, or if you want to change the deployment target, you should configure its parameters. The `Jenkinsfile` is set up to use the following parameters:

1.  **This project is parameterized:** In the pipeline configuration page (reached by clicking "Configure" on the pipeline's main page), ensure the "This project is parameterized" checkbox is checked.
2.  **Add Parameters:** If parameters are not already listed from the `Jenkinsfile` (Jenkins usually detects them), or if you need to modify their presentation (e.g., add descriptions), you can do so here. The parameters defined in the attached `Jenkinsfile` are:
    *   `ENVIRONMENT`:
        *   **Type:** Choice Parameter (Jenkins should infer this from the `Jenkinsfile`)
        *   **Name:** `ENVIRONMENT`
        *   **Choices** (as defined in `Jenkinsfile`):
            ```
            dev
            prod
            ```
        *   **Description:** (As defined in `Jenkinsfile`) "Chọn môi trường triển khai" (Select the deployment environment).
    *   `SERVICE`:
        *   **Type:** Choice Parameter
        *   **Name:** `SERVICE`
        *   **Choices** (as defined in `Jenkinsfile`):
            ```
            all
            chat-service
            frontend-service
            meeting-service
            team-service
            user-service
            websocket-service
            ```
        *   **Description:** (As defined in `Jenkinsfile`) "Chọn dịch vụ ứng dụng để triển khai" (Select the application service to deploy. Choose 'all' to deploy all configured services).
    *   `IMAGE_TAG`:
        *   **Type:** String Parameter
        *   **Name:** `IMAGE_TAG`
        *   **Default Value:** (As defined in `Jenkinsfile`) `latest`
        *   **Description:** (As defined in `Jenkinsfile`) "Docker image tag to deploy".

3.  **Save Configuration:** After reviewing or making changes to how parameters are presented (if necessary), click "Save".

When you click "Build with Parameters" (this option appears if the project is parameterized), Jenkins will prompt you to select or enter values for these parameters before starting the pipeline execution. The choices and default values will be pre-filled based on the `Jenkinsfile`.

### Step 9: Run and Monitor the Pipeline

1.  **Build Now:** After creating or configuring the pipeline, click on "Build Now" on the left sidebar of the pipeline page to trigger a new build.
2.  **Monitor Build Status:**
    *   **Build History:** You can see the build progress and status in the "Build History" section on the left sidebar. Click on a specific build number to see its details.
    *   **Console Output:** Click on "Console Output" for a specific build to see the live logs and detailed output of each stage in your `Jenkinsfile`.
    *   **Blue Ocean (Optional):** If you have the Blue Ocean plugin installed, it provides a more visual and user-friendly interface for monitoring pipeline executions.
