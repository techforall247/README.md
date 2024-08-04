## Streamline Your Cloud Infrastructure: Automate S3 Bucket Creation and Deployment with GitHub Actions

Managing cloud infrastructure can often be a cumbersome process, especially when it comes to automating tasks like creating S3 buckets and deploying containerized applications. In this blog post, we will guide you through automating these processes using Terraform and GitHub Actions. This approach not only simplifies your workflow but also ensures consistency and efficiency in your deployments.

### 1. Automate S3 Bucket Creation with Terraform

Terraform is an excellent tool for managing infrastructure as code. By defining your infrastructure in Terraform configuration files, you can easily create and manage AWS resources in a repeatable and reliable manner.

#### Terraform Configuration

To create an S3 bucket using Terraform, you need to write a configuration file that specifies the details of your S3 bucket. Here’s a sample configuration:

**`s3.tf`**

```hcl
resource "aws_s3_bucket" "bucket" {
  bucket = "my-s3-bucket-via-github"
  acl    = "private"

  tags = {
    Name        = "My bucket"
    Environment = "Dev"
  }
}
```

In this configuration:
- `bucket` specifies the name of your S3 bucket. Ensure that the name is globally unique.
- `acl` defines the access control list for the bucket. Setting it to `"private"` means that the bucket is not accessible to the public.
- `tags` allows you to add metadata to your bucket. Tags can be useful for organizing and managing your resources.

#### Running Terraform Commands

To apply this configuration, you need to run a few Terraform commands:
1. **Initialize Terraform**: `terraform init` – This command initializes the working directory containing Terraform configuration files.
2. **Validate Configuration**: `terraform validate` – This checks whether your configuration is syntactically valid.
3. **Plan Changes**: `terraform plan` – This shows what actions Terraform will take based on your configuration.
4. **Apply Configuration**: `terraform apply` – This applies the changes required to reach the desired state of the configuration.

### 2. Deploy Container Images Using GitHub Actions

GitHub Actions enables you to automate workflows directly from your GitHub repository. We can use it to build and deploy container images to Amazon ECS (Elastic Container Service) whenever there is a push to the main branch.

#### GitHub Actions Workflow Configuration

Here's an example GitHub Actions workflow that builds a Docker image, pushes it to Amazon ECR (Elastic Container Registry), and deploys it to Amazon ECS.

**`.github/workflows/aws.yml`**

```yaml
# This workflow will build and push a new container image to Amazon ECR,
# and then will deploy a new task definition to Amazon ECS, when there is a push to the main branch.

# Setup Instructions:
# 1. Create an ECR repository to store your images.
# 2. Create an ECS task definition, ECS cluster, and ECS service.
# 3. Store your ECS task definition as a JSON file in your repository.
# 4. Store an IAM user access key in GitHub Actions secrets named `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.

name: Deploy to Amazon ECS

on:
  push:
    branches:
      - main

env:
  AWS_REGION: eu-west-1                       # Set this to your preferred AWS region
  ECR_REPOSITORY: MY_ECR_REPOSITORY           # Set this to your Amazon ECR repository name
  ECS_SERVICE: MY_ECS_SERVICE                 # Set this to your Amazon ECS service name
  ECS_CLUSTER: MY_ECS_CLUSTER                 # Set this to your Amazon ECS cluster name
  ECS_TASK_DEFINITION: MY_ECS_TASK_DEFINITION # Set this to the path to your Amazon ECS task definition file
  CONTAINER_NAME: MY_CONTAINER_NAME           # Set this to the name of the container in your task definition

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}
    
    - name: Terraform Init
      id: init
      run: terraform init

    - name: Terraform Validate
      id: validate
      run: terraform validate -no-color

    - name: Terraform Plan
      id: plan
      run: terraform plan -no-color
      continue-on-error: true

    - uses: actions/github-script@0.9.0
      if: github.event_name == 'pull_request'
      env:
        PLAN: "terraform\n${{ steps.plan.outputs.stdout }}"
      with:
        github-token: ${{ secrets.GITHUB_TOKEN }}
        script: |
          const output = `#### Terraform Format and Style 🖌\`${{ steps.fmt.outcome }}\`
          #### Terraform Initialization ⚙️\`${{ steps.init.outcome }}\`
          #### Terraform Validation 🤖\`${{ steps.validate.outputs.stdout }}\`
          #### Terraform Plan 📖\`${{ steps.plan.outcome }}\`
          
          <details><summary>Show Plan</summary>
          
          \`\`\`\n
          ${process.env.PLAN}
          \`\`\`
          
          </details>
          
          *Pusher: @${{ github.actor }}, Action: \`${{ github.event_name }}\`, Working Directory: \`${{ env.tf_actions_working_dir }}\`, Workflow: \`${{ github.workflow }}\`*`;
          
          github.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: output
          })
```

#### Workflow Breakdown

- **Checkout Code**: Retrieves your code from the repository.
- **Configure AWS Credentials**: Sets up AWS credentials for use in the workflow.
- **Terraform Init**: Initializes Terraform in your working directory.
- **Terraform Validate**: Validates the Terraform configuration.
- **Terraform Plan**: Creates an execution plan for Terraform.
- **GitHub Script Action**: Posts a comment with the Terraform plan details to the pull request.

By automating the creation of S3 buckets and deploying container images using Terraform and GitHub Actions, you streamline your cloud infrastructure management. This setup reduces manual intervention, minimizes errors, and ensures that your deployments are consistent and repeatable.

For a complete example and to explore the code, check out our GitHub repository: [s3bucketcreation](https://github.com/techforall247/s3bucketcreation).

Feel free to explore these tools further and adapt the workflows to fit your specific needs. If you have any questions or need further assistance, don’t hesitate to reach out!