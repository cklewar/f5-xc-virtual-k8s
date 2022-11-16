# F5-XC-V8KS
This repository consists of Terraform templates to bring up a F5XC Virtual Kubernetes environment.

## Usage

- Clone this repo with: `git clone --recurse-submodules https://github.com/cklewar/f5-xc-vk8s`
- Enter repository directory with: `cd f5-xc-vk8s`
- Obtain F5XC API certificate file from Console and save it to `cert` directory
- Pick and choose from below examples and add mandatory input data and copy data into file `main.tf.example`.
- Rename file __main.tf.example__ to __main.tf__ with: `rename main.tf.example main.tf`
- Initialize with: `terraform init`
- Apply with: `terraform apply -auto-approve` or destroy with: `terraform destroy -auto-approve`

## vk8s create credential by reference module usage example 

````hcl
module "vk8s" {
  source                    = "./modules/f5xc/v8ks"
  f5xc_tenant               = var.f5xc_tenant
  f5xc_api_url              = var.f5xc_api_url
  f5xc_api_token            = var.f5xc_api_token
  f5xc_vk8s_name            = format("%s-vk8s-%s", var.project_prefix, var.project_suffix)
  f5xc_vsite_refs_namespace = var.f5xc_namespace
  providers                 = {
    volterra = volterra.default
  }
}

module "credential" {
  source                    = "./modules/f5xc/api-credential"
  f5xc_tenant               = var.f5xc_tenant
  f5xc_api_url              = var.f5xc_api_url
  f5xc_api_token            = var.f5xc_api_token
  f5xc_namespace            = var.f5xc_namespace
  f5xc_virtual_k8s_name     = module.vk8s.vk8s["name"]
  f5xc_api_credential_type  = "KUBE_CONFIG"
  f5xc_api_credentials_name = format("%s-vk8s-credential-%s", var.project_prefix, var.project_suffix)
}

output "kube_config" {
  value     = module.credential.api_credential["data"]
  sensitive = true
}

output "workload_config" {
  value = local.workload_content
}
````

## vk8s create kubeconf inline module usage example 

````hcl
module "vk8s" {
  source                    = "./modules/f5xc/v8ks"
  f5xc_tenant               = var.f5xc_tenant
  f5xc_api_url              = var.f5xc_api_url
  f5xc_api_token            = var.f5xc_api_token
  f5xc_vk8s_name            = format("%s-vk8s-%s", var.project_prefix, var.project_suffix)
  f5xc_vsite_refs_namespace = var.f5xc_namespace
  providers                 = {
    volterra = volterra.default
  }
}

module "credential" {
  source                    = "./modules/f5xc/api-credential"
  f5xc_tenant               = var.f5xc_tenant
  f5xc_api_url              = var.f5xc_api_url
  f5xc_api_token            = var.f5xc_api_token
  f5xc_namespace            = var.f5xc_namespace
  f5xc_virtual_k8s_name     = module.vk8s.vk8s["name"]
  f5xc_api_credential_type  = "KUBE_CONFIG"
  f5xc_api_credentials_name = format("%s-vk8s-credential-%s", var.project_prefix, var.project_suffix)
}

output "kube_config" {
  value     = module.credential.api_credential["data"]
  sensitive = true
}

output "workload_config" {
  value = local.workload_content
}
````