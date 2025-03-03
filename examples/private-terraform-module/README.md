# Private Terraform module

This example shows how to run Infracost actions with a Terraform project that uses a private Terraform module. This requires a secret to be added to your GitHub repository called `GIT_SSH_KEY` containing a private key so that Terraform can access the private repository.

[//]: <> (BEGIN EXAMPLE)
```yml
name: Private Terraform module
on: [pull_request]
jobs:
  private-terraform-module:
    name: Private Terraform module
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Install terraform
        uses: hashicorp/setup-terraform@v1
        with:
          terraform_wrapper: false # This is recommended so the `terraform show` command outputs valid JSON

      # IMPORTANT: add any required steps here to setup cloud credentials so Terraform can run

      # Add your git SSH key so Terraform can checkout the private modules
      - name: add GIT_SSH_KEY
        run: |
          mkdir -p .ssh
          echo "${{ secrets.GIT_SSH_KEY }}" > .ssh/git_ssh_key
          chmod 400 .ssh/git_ssh_key
          echo "GIT_SSH_COMMAND=ssh -i $(pwd)/.ssh/git_ssh_key -o 'StrictHostKeyChecking=no'" >> $GITHUB_ENV

      - name: Setup Infracost
        uses: infracost/actions/setup@v1
        with:
          api-key: ${{ secrets.INFRACOST_API_KEY }}

      - name: Run Infracost
        run: infracost breakdown --path=examples/private-terraform-module/code --format=json --out-file=/tmp/infracost.json

      - name: Post the comment
        uses: infracost/actions/comment@v1
        with:
          path: /tmp/infracost.json
          behavior: update # Create a single comment and update it. See https://github.com/infracost/actions/tree/master/comment for other options
```
[//]: <> (END EXAMPLE)
