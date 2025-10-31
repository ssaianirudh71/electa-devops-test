# DevOps Practical Assessment: Docker Edition

[cite_start]This project fulfills the requirements of the DevOps Practical Assessment [cite: 1] [cite_start]using Docker and GitHub Actions, as permitted by the instructions[cite: 6, 7].

It deploys two `nginx` containers (as stand-ins for Windows VMs) on a custom Docker network, demonstrating Infrastructure as Code (IaC), CI/CD, and configuration management.

## Project Structure

```
.
├── .github/
│   └── workflows/
│       └── main.yml        # CI/CD Pipeline
├── .env                    # Local variables (gitignored)
├── .gitignore              # Ignores .env
├── docker-compose.yml      # Infrastructure as Code (IaC)
└── README.md               # This file
```

## "Outputs" (Assessment Equivalents)

* [cite_start]**Resource Group Name**[cite: 33]: `elekta-devops-test` (This is the logical name of our project).
* [cite_start]**VM Names**[cite: 30]: `vm-test-1`, `vm-test-2` (Defined in `.env` and used by `docker-compose.yml`).
* [cite_start]**Public IP Addresses**[cite: 31]: Access is via `localhost` on the host machine.
    * `vm-test-1` is accessible at `http://localhost:8081`
    * `vm-test-2` is accessible at `http://localhost:8082`
* [cite_start]**Private IP Addresses**[cite: 32]: You can find these by running `docker inspect <container_name>`. [cite_start]The containers can ping each other directly using their service names (`vm-test-1`, `vm-test-2`) because they share the `elekta-net` network[cite: 27].

---

## How to Deploy (Local)

1.  **Prerequisites:**
    * Git
    * Docker
    * Docker Compose

2.  **Clone the repository:**
    ```sh
    git clone <your-repo-url>
    cd <your-repo-name>
    ```

3.  **Create your local configuration:**
    [cite_start]Create a file named `.env` and copy the contents from the `.env` example in this guide.

4.  **Deploy the "Infrastructure":**
    ```sh
    docker-compose up -d
    ```

5.  **Verify:**
    * Open `http://localhost:8081` in your browser. You should see the "Welcome to nginx!" page.
    * Open `http://localhost:8082` in your browser. You should see the same page.
    * Run `docker ps` to see your two containers, `vm-test-1` and `vm-test-2`, running.

---

## [cite_start]CI/CD Pipeline (GitHub Actions) [cite: 59]

This repository contains a CI/CD pipeline in `.github/workflows/main.yml`.

### 1. Setup

To make the `deploy` job work, you must:

1.  **Set up a Self-Hosted Runner:**
    * Go to your GitHub Repo > **Settings** > **Actions** > **Runners**.
    * Click **"New self-hosted runner"** and follow the instructions to install it on your machine (or the server where you want to deploy this).
    * **Important:** The machine running the runner *must* have Docker and Docker Compose installed.

2.  [cite_start]**Add Repository Secrets**[cite: 48]:
    * Go to your GitHub Repo > **Settings** > **Secrets and variables** > **Actions**.
    * Click **"New repository secret"** for each of the following:
        * `VM1_NAME`: `vm-test-1`
        * `VM2_NAME`: `vm-test-2`
        * `ADMIN_USER`: `Elekta`
        * `ADMIN_PASS`: `ElektaDevopsTest123!`

### 2. Pipeline Jobs

* [cite_start]**`validate`**[cite: 60]: This job runs on every push or pull request. It uses `docker-compose config` to check the `docker-compose.yml` file for any syntax errors.
* [cite_start]**`deploy`**[cite: 61]: This job runs *only* after `validate` succeeds on a push to `main`. It runs on your self-hosted runner, checks out the code, and executes `docker-compose up -d`, applying any changes to your "infrastructure."

## [cite_start]Assumptions & Design Decisions [cite: 65]

* [cite_start]**Azure VMs -> Docker Containers**: The two Windows VMs [cite: 23] are represented by two `nginx:latest` containers. This is a lightweight way to prove the IaC and networking principles.
* [cite_start]**Azure VNet -> Docker Network**: The VNet/Subnet [cite: 19, 20] is represented by a Docker `bridge` network named `elekta-net`.
* [cite_start]**NSG / Public IP -> Port Mapping**: RDP access and Public IPs [cite: 21, 28] are simulated by mapping ports `8081` and `8082` from the host to the containers.
* [cite_start]**Azure DevOps -> GitHub Actions**: GitHub Actions is used as the CI/CD platform.
* [cite_start]**State Management**: Terraform state [cite: 44, 47] is not required, as Docker Compose manages the state of containers directly via the Docker daemon. Secrets and variables are managed via GitHub Secrets.