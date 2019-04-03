# Terraform Google Dataflow Module

The resources/services/activations/deletions that this module will create/trigger are:
- Create a  GCS bucket for temporary job data
- Create a Dataflow job 

## Usage

Before using this module, one should get familiar with the google_dataflow_job’s (Notes on “destroy”/”apply”)[https://www.terraform.io/docs/providers/google/r/dataflow_job.html#note-on-quot-destroy-quot-quot-apply-quot-] as the behavior is atypical when compared to other resources.

There are examples included in the [examples](./examples/) folder but simple usage is as follows:

```hcl
module "dataflow-job" {
  source      = "../../modules/dataflow"
  project_id  = "<project_id>"
  job_name = "<job_name>"
  on_delete = "cancel"
  zone = "us-central1-a"
  max_workers = 1
  template_gcs_path =  "gs://<path-to-template>"
  temp_gcs_location = "gs://<gcs_path_temp_data_bucket"
  parameters = {
        bar = "example string"
        foo = 123
  }
}
```

Then perform the following commands on the root folder:

- `terraform init` to get the plugins
- `terraform plan` to see the infrastructure plan
- `terraform apply` to apply the infrastructure build
- `terraform destroy` to destroy the built infrastructure


[^]: (autogen_docs_start)

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| bucket\_region | BUCKET REGION | string | `"us-central1"` | no |
| job\_name | (Required) The name of the dataflow job | string | n/a | yes |
| max\_workers | (Optional)  The number of workers permitted to work on the job. More workers may improve processing speed at additional cost. | string | `"1"` | no |
| on\_delete | (Optional) One of drain or cancel. Specifies behavior of deletion during terraform destroy. The default is cancel. | string | `"cancel"` | no |
| parameters | (Optional) Key/Value pairs to be passed to the Dataflow job (as used in the template). | map | `<map>` | no |
| project\_id | (Required) The project in which the resource belongs. If it is not provided, the provider project is used. | string | n/a | yes |
| service\_account\_email | (Optional) The Service Account email that will be used to identify the VMs in which the jobs are running | string | `""` | no |
| temp\_gcs\_location | (Required) A writeable location on GCS for the Dataflow job to dump its temporary data. | string | n/a | yes |
| template\_gcs\_path | (Required) The GCS path to the Dataflow job template. | string | n/a | yes |
| zone | (Optional) The zone in which the created job should run. If it is not provided, the provider zone is used. | string | `"us-central1-a"` | no |

## Outputs

| Name | Description |
|------|-------------|
| df\_job\_id | The unique Id of the newly created Dataflow job |
| df\_job\_name | The name of the dataflow job |
| df\_job\_state | The state of the newly created Dataflow job |
| df\_job\_template\_gcs\_path | The GCS path to the Dataflow job template. |
| temp\_gcs\_location | The GCS path for the Dataflow job's temporary data. |

[^]: (autogen_docs_end)

## Requirements

Before this module can be used on a project, you must ensure that the following pre-requisites are fulfilled:

1. Terraform is [installed](#software-dependencies) on the machine where Terraform is executed.
2. The Service Account you execute the module with has the right [permissions](#configure-a-service-account).
3. The necessary APIs are [active](#enable-apis) on the project.
4. A working Dataflow template in uploaded in a GCS bucket

The [project factory](https://github.com/terraform-google-modules/terraform-google-project-factory) can be used to provision projects with the correct APIs active.

### Software Dependencies
### Terraform
- [Terraform](https://www.terraform.io/downloads.html) 0.10.x
- [terraform-provider-google](https://github.com/terraform-providers/terraform-provider-google) plugin v1.8.0

### Configure a Service Account to execute the module
In order to execute this module you must have a Service Account with the
following project roles:
- roles/dataflow.admin
- roles/iam.serviceAccountUser

### Configure a Controller Service Account to create the job
If you want to use the service_account_email input to specify a service account that will identify the VMs in which the jobs are running, the service account will need the following project roles:
- roles/dataflow.worker
- roles/storage.objectAdmin

### Enable APIs
In order to launch a Dataflow Job, the Dataflow API must be enabled:

- Dataflow API - dataflow.googleapis.com

## Install

### Terraform
Be sure you have the correct Terraform version (0.10.x), you can choose the binary here:
- https://releases.hashicorp.com/terraform/

## File structure
The project has the following folders and files:

- /: root folder
- /examples: examples for using this module
- /helpers: Helper scripts
- /test: Folders with files for testing the module (see Testing section on this file)
- /main.tf: main file for this module, contains all the resources to create
- /variables.tf: all the variables for the module
- /output.tf: the outputs of the module
- /README.md: this file

## Testing

### Requirements
- [bundler](https://github.com/bundler/bundler)
- [gcloud](https://cloud.google.com/sdk/install)
- [terraform-docs](https://github.com/segmentio/terraform-docs/releases) 0.3.0

### Autogeneration of documentation from .tf files
Run
```
make generate_docs
```

### Integration test

Integration tests are run though [test-kitchen](https://github.com/test-kitchen/test-kitchen), [kitchen-terraform](https://github.com/newcontext-oss/kitchen-terraform), and [InSpec](https://github.com/inspec/inspec).

`test-kitchen` instances are defined in [`.kitchen.yml`](./.kitchen.yml). The test-kitchen instances in `test/fixtures/` wrap identically-named examples in the `examples/` directory.

#### Setup

1. Configure the [test fixtures](#test-configuration)
2. Download a Service Account key with the necessary permissions and put it in the module's root directory with the name `credentials.json`.
3. Build the Docker container for testing:

  ```
  make docker_build_kitchen_terraform
  ```
4. Run the testing container in interactive mode:

  ```
  make docker_run
  ```

  The module root directory will be loaded into the Docker container at `/cft/workdir/`.
5. Run kitchen-terraform to test the infrastructure:

  1. `kitchen create` creates Terraform state and downloads modules, if applicable.
  2. `kitchen converge` creates the underlying resources. Run `kitchen converge <INSTANCE_NAME>` to create resources for a specific test case.
  3. `kitchen verify` tests the created infrastructure. Run `kitchen verify <INSTANCE_NAME>` to run a specific test case.
  4. `kitchen destroy` tears down the underlying resources created by `kitchen converge`. Run `kitchen destroy <INSTANCE_NAME>` to tear down resources for a specific test case.

Alternatively, you can simply run `make test_integration_docker` to run all the test steps non-interactively.

#### Test configuration

Each test-kitchen instance is configured with a `variables.tfvars` file in the test fixture directory. For convenience, since all of the variables are project-specific, these files have been symlinked to `test/fixtures/shared/terraform.tfvars`.
Similarly, each test fixture has a `variables.tf` to define these variables, and an `outputs.tf` to facilitate providing necessary information for `inspec` to locate and query against created resources.

Each test-kitchen instance creates necessary fixtures to house resources.

### Autogeneration of documentation from .tf files
Run
```
make generate_docs
```

### Linting
The makefile in this project will lint or sometimes just format any shell,
Python, golang, Terraform, or Dockerfiles. The linters will only be run if
the makefile finds files with the appropriate file extension.

All of the linter checks are in the default make target, so you just have to
run

```
make -s
```

The -s is for 'silent'. Successful output looks like this

```
Running shellcheck
Running flake8
Running go fmt and go vet
Running terraform validate
Running hadolint on Dockerfiles
Checking for required files
Testing the validity of the header check
..
----------------------------------------------------------------------
Ran 2 tests in 0.026s

OK
Checking file headers
The following lines have trailing whitespace
```

The linters
are as follows:
* Shell - shellcheck. Can be found in homebrew
* Python - flake8. Can be installed with 'pip install flake8'
* Golang - gofmt. gofmt comes with the standard golang installation. golang
is a compiled language so there is no standard linter.
* Terraform - terraform has a built-in linter in the 'terraform validate'
command.
* Dockerfiles - hadolint. Can be found in homebrew
