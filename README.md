# Anthony Salame – Cloud Portfolio

Static portfolio site hosted on an **Azure Storage static website (France Central)**, infrastructure in **Terraform**, deployed by **GitHub Actions**.

```
site/                      → the website (HTML/CSS, no build step)
infra/                     → Terraform: resource group + storage account (static website)
.github/workflows/deploy.yml → deploys site/ on every push to main
```

---

## Setup guide (step by step)

### 0. What you need (all free)
- **Azure for Students** activated with your Ynov email → https://azure.microsoft.com/free/students
- A **GitHub** account
- On your computer: **Git**, **Azure CLI**, **Terraform**
  - Windows (PowerShell):
    ```powershell
    winget install Git.Git
    winget install Microsoft.AzureCLI
    winget install Hashicorp.Terraform
    ```
  - Close and reopen the terminal, then check: `git --version`, `az version`, `terraform -version`

### 1. Log in to Azure
```powershell
az login
az provider register --namespace Microsoft.Storage --wait
az account show --query id -o tsv     # copy this ID = your subscription ID
```

### 2. Create the infrastructure with Terraform
```powershell
cd infra
copy terraform.tfvars.example terraform.tfvars   # then paste your subscription ID inside
terraform init
terraform plan        # shows what will be created (resource group, random suffix, storage account, static website)
terraform apply       # type "yes"
```
Get your site URL and the connection string for GitHub:
```powershell
terraform output site_url
terraform output -raw storage_connection_string
```

### 3. Put the code on GitHub
1. Create a new **empty** repository on GitHub (e.g. `cloud-portfolio`).
2. From the project root:
   ```powershell
   git init
   git add .
   git commit -m "Initial portfolio: site + Terraform + CI/CD"
   git branch -M main
   git remote add origin https://github.com/<your-username>/cloud-portfolio.git
   ```
   ⚠️ Don't push yet, add the secret first (step 4).

### 4. Add the connection string as a GitHub secret
GitHub repo → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**
- Name: `AZURE_STORAGE_CONNECTION_STRING`
- Value: the result of `terraform output -raw storage_connection_string`

### 5. Push → automatic deployment
```powershell
git push -u origin main
```
Go to the **Actions** tab on GitHub and watch the workflow. After about a minute, open your `site_url`. 🎉

From now on: edit a file in `site/`, commit, push → the site updates by itself.

### 6. Safety net: budget alert
Azure portal → **Cost Management** → **Budgets** → **Add** → amount **€1** → alert at 100% → your email.

---

## Updating the content
- Text, experience, labs: `site/index.html`
- Colors and layout: `site/styles.css`
- CV: replace `site/CV_Anthony_Salame.pdf` (keep the same name)

## Rules
- Never commit `terraform.tfvars` or `*.tfstate` (already in `.gitignore`).
- Never paste the connection string in a file or chat. It lives only in GitHub Secrets.
- To delete everything: `terraform destroy`.
