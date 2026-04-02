# Day 20: Workflow for Deploying Application Code with Terraform

Part of my 30 Day Terraform Challenge journey with AWS AI/ML UserGroup Kenya,
Meru HashiCorp User Group, and EveOps.

## What This Covers

This repository simulates the complete seven-step application deployment
workflow applied to Terraform infrastructure code. It demonstrates feature
branching, saved plan files, pull request reviews with plan output, version
tagging, Terraform Cloud workspace setup, secure variable management, and
private registry module publishing.

## The Seven Steps

Step 1: Version control - all Terraform code in Git, state file never in Git.

Step 2: Run locally - terraform plan with -out flag saves the exact plan
for later apply.

Step 3: Make changes - feature branch per change, never commit directly to main.

Step 4: Submit for review - pull request with terraform plan output as a
comment so reviewers see exactly what will change in production.

Step 5: Run automated tests - GitHub Actions triggers terraform test on
every pull request.

Step 6: Merge and release - merge to main and tag the commit with a
semantic version.

Step 7: Deploy - terraform apply using the saved plan file, then verify
with curl.

## Prerequisites

AWS CLI installed and configured:
```bash
aws configure
```

Terraform installed:
```bash
terraform version
```

Terraform Cloud account at app.terraform.io (free tier available).

An existing S3 bucket and DynamoDB table for remote state if using the S3
backend. Update the bucket name in the backend block before running.

## Project Structure
```
day20-terraform/
├── main.tf
├── .gitignore
└── .github/
    └── workflows/
        └── terraform-test.yml
```

## How to Run the Seven-Step Workflow

Step 1: Clone and confirm version control is set up:
```bash
git clone https://github.com/Chenzie2/Day20-Terraform-Challenge.git
cd Day20-Terraform-Challenge
```

Step 2: Initialize and save the plan:
```bash
terraform init
terraform plan -var="cluster_name=your-cluster" -var="min_size=1" -var="max_size=1" -out=my.tfplan
```

Step 3: Create a feature branch and make your change:
```bash
git checkout -b my-feature-branch
# make your change to main.tf
git add .
git commit -m "Describe your change"
git push origin my-feature-branch
```

Step 4: Open a pull request on GitHub. Paste your terraform plan output
in the PR description so reviewers can see what will change.

Step 5: GitHub Actions runs terraform test automatically on the PR.

Step 6: Merge the PR to main and tag:
```bash
git checkout main
git pull origin main
git tag -a "v1.x.0" -m "Description of change"
git push origin v1.x.0
```

Step 7: Apply the saved plan and verify:
```bash
terraform apply my.tfplan
curl http://$(terraform output -raw alb_dns_name)
```

Always destroy when done:
```bash
terraform destroy
```

## How to Set Up Terraform Cloud

Create a free account at app.terraform.io. Create an organisation and a
workspace named webserver-cluster-dev.

Authenticate locally:
```bash
terraform login
```

Update the terraform block in main.tf to use the cloud block instead of
the S3 backend, replacing the organisation name with your own.

Re-initialize to migrate state:
```bash
terraform init
```

Add your AWS credentials as sensitive environment variables in the
Terraform Cloud workspace variables page. Add your Terraform variables
as workspace variables. Once configured, all runs use these variables
automatically with no credentials on any developer machine.

## Private Registry

The webserver cluster module is published to the Terraform Cloud private
registry. Teams reference it as:
```hcl
module "webserver_cluster" {
  source  = "app.terraform.io/grace-zawadi-tf/webserver-cluster/aws"
  version = "1.0.0"

  cluster_name  = "prod-cluster"
  instance_type = "t3.micro"
  min_size      = 1
  max_size      = 2
  environment   = "production"
}
```

The private registry provides versioning, documentation, and a consistent
source URL. Teams cannot accidentally use an unpinned GitHub URL that could
change at any time.

## Key Insight

Save your plan before applying it:
```bash
terraform plan -out=my.tfplan
terraform apply my.tfplan
```

Without -out, the state could change between plan and apply. You might
apply something different from what you reviewed. Always plan, review,
then apply the saved plan.

## Author

Grace Zawadi - Software Engineer
[LinkedIn](https://www.linkedin.com/in/gracezawadi) | [Medium](https://medium.com/@gracezawadi24)

Day 20 of the 30 Day Terraform Challenge. Application deployment workflow
mapped to Terraform. Seven steps from local change to production, Terraform
Cloud for state and variable management, private registry for internal
module sharing. Infrastructure as Code done properly looks exactly like
good software engineering.

#30DayTerraformChallenge #TerraformChallenge #Terraform #TerraformCloud
#DevOps #IaC #AWSUserGroupKenya #EveOps