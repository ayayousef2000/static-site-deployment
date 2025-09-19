### **Report on Deploying a Static Site from GitHub to Yandex Cloud with Analysis of Alternative Tools**

**Objective:** To automate the deployment of a static website, generated using MkDocs, from a GitHub repository to Yandex Object Storage, and to analyze alternative technologies and approaches.

**Process Overview:**
To achieve this objective, a CI/CD pipeline was implemented using GitHub Actions. This process automatically builds the site upon every push to the main branch of the repository and uploads the resulting files to a public Yandex Object Storage bucket configured for website hosting.

---

#### **Step 1: Repository and Local Environment Setup**

First, a repository was created on GitHub and then cloned to a local machine. Inside the project folder, a Python virtual environment named `env` was created to isolate dependencies.

Within this environment, the `MkDocs-material` library was installed. The `mkdocs serve` command was used to preview the site locally and verify its appearance.

---

#### **Step 2: Yandex Cloud Infrastructure Configuration**

The next step was to prepare the cloud infrastructure to host the website files.

1.  **Create a Yandex Object Storage Bucket:**
    *   A new bucket named `aya-yousef` was created in the Yandex Cloud console.
    *   Public access was configured for the bucket to ensure the website is accessible over the internet.

2.  **Configure the Bucket for Website Hosting:**
    *   In the settings for the `aya-yousef` bucket, the "Website" feature was enabled.
    *   `index.html` was specified as the index document, and `error.html` was set as the error document.

3.  **Create a Service Account:**
    *   To grant GitHub Actions permission to upload files to the bucket, a service account named `github-actions-deployer` was created.
    *   This account was assigned the `storage.editor` role, which provides the necessary permissions to manage objects in the bucket.

4.  **Generate a Static Access Key:**
    *   To authenticate GitHub Actions with Yandex Cloud, a static access key (Key ID) and a secret key were generated for the `github-actions-deployer` service account.

---

#### **Step 3: GitHub Repository Configuration**

The **Secrets** feature in GitHub was used to securely store the Yandex Cloud credentials.

*   In the repository settings under **Settings > Secrets and variables > Actions**, the following secrets were created:
    *   `YC_ACCESS_KEY_ID`: To store the `Key ID`.
    *   `YC_SECRET_ACCESS_KEY`: To store the `Secret key`.
    *   `YC_BUCKET_NAME`: To store the bucket name (`aya-yousef`).

---

#### **Step 4: Create the GitHub Actions Workflow (CI/CD)**

The core of the automation is the workflow file that defines the steps for building and deploying the site.

1.  **Create the Workflow File:**
    *   A `.github/workflows/` directory was created in the repository structure, and a file named `deploy.yml` was placed inside it.

2.  **Configure the Workflow:**
    *   A YAML configuration was added to the `deploy.yml` file, describing the sequence of actions: trigger on a push to the `main` branch, install dependencies, build the site using MkDocs, and synchronize the resulting files with the `aya-yousef` bucket in Yandex Cloud.

3.  **Commit and Push Changes:**
    *   The `deploy.yml` file was committed and pushed to the remote repository on GitHub.

---

#### **Step 5: Trigger Deployment and Verify the Result**

Pushing the `.github/workflows/deploy.yml` file to the repository automatically triggered the first run of the workflow.

*   The execution process was monitored in real-time under the **Actions** tab of the GitHub repository.
*   Upon successful completion of all steps, the workflow finished, and the static site became available at the public URL: [https://aya-yousef.website.yandexcloud.net](https://aya-yousef.website.yandexcloud.net).

---

#### **Step 6: Final Changes**

In the final stage, the last adjustments were made:
*   An `error.html` file was created and added to display a proper error page.
*   The configuration file `mkdocs.yml` and the main page `docs/index.md` were updated to populate the site with content.

These changes were pushed to the repository, which triggered the CI/CD pipeline again and automatically updated the site in Yandex Cloud.

---

### **Analytical Section**

#### **1. Using Domestic CDNs to Accelerate Content Delivery**

To speed up the loading of static content (images, styles, scripts) for users in different geographical regions, it is advisable to use a CDN (Content Delivery Network). A CDN caches content on its servers located close to end-users, which significantly reduces response times.

**Domestic (Russian) CDN Providers:**

*   **Yandex Cloud CDN:** Integrates seamlessly with Yandex Object Storage. It allows content to be served from Yandex's edge servers, ensuring high speed for users in Russia and the CIS.
*   **Selectel CDN:** Offers CDN services with a wide network of presence points in Russia and abroad.
*   **G-Core Labs:** An international provider with powerful infrastructure in Russia.

**Integration Process:** Typically, integration involves creating a CDN resource in the provider's control panel and specifying the content source (in this case, the Object Storage bucket). After this, a special domain provided by the CDN service is used to access the files.

#### **2. CI/CD Capabilities of Gitverse**

**Gitverse** is a domestic (Russian) platform for hosting Git repositories. Currently, its CI/CD functionality is in active development and does not yet match the capabilities of mature solutions like GitHub Actions or GitLab CI/CD.

*   **Current Capabilities:** Gitverse allows for code storage, version control, and collaborative development.
*   **Future Prospects:** It is expected that the platform will be enhanced with built-in tools for automatic building, testing, and deployment of applications, which will enable the creation of CI/CD pipelines similar to the one described in this report.
*   **Alternative:** Until native CI/CD is implemented in Gitverse, automation can be achieved using external tools like **Jenkins** or **TeamCity**, configured to monitor changes in the Gitverse repository.

#### **3. Options for Deploying a Static Site to Production and Required Tools**

There are several popular approaches to deploying static websites.

**1. Cloud Storage Hosting (S3-compatible)**
This method involves uploading the site's files to a cloud storage bucket configured to serve a website. It is a highly reliable and scalable approach. The required tools include a cloud platform (like Yandex Cloud, AWS, or Selectel) and a file synchronization tool (such as aws-cli, s3cmd, or rclone). The deployment process detailed in this report, using Yandex Object Storage, is a prime example of this method.

**2. Static Hosting Platforms (Jamstack Providers)**
These are specialized services that integrate with Git repositories to automatically build and deploy the site. They often provide essential features like CDN integration and SSL certificates out-of-the-box. The necessary tools are an account on a platform (such as Netlify, Vercel, or Cloudflare Pages) and a Git repository (hosted on GitHub, GitLab, etc.). For example, you could connect a GitHub repository to Netlify, specify the build command (`mkdocs build`) and the output directory (`site`), and Netlify would automatically handle the build and deployment process for every new commit.

**3. Deployment to a Self-Hosted Server (VPS/VDS)**
This traditional approach involves copying the static files to a virtual or dedicated server where a web server (e.g., Nginx or Apache) is configured to serve them. This requires a VPS/VDS server, a web server, file copy tools (like scp or rsync), and an automation system for CI/CD (such as Ansible or Jenkins). An example would be configuring Nginx on a server to serve files from the `/var/www/my-site` directory, then adding a step in a CI/CD pipeline to copy the built site files to this directory via SSH after each update.

**4. Using Containerization**
With this method, the static site is packaged into a Docker container along with a web server (like Nginx). This self-contained unit can then be deployed consistently in any environment that supports Docker. The required tools include Docker, a web server like Nginx, and potentially an orchestration system (like Docker Swarm or Kubernetes) for managing deployments. For instance, you could create a `Dockerfile` that copies the built site files into an Nginx image and then run the resulting container on a server using the `docker run` command.