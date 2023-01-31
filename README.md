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
module "f5xc_namespace" {
  source              = "./modules/f5xc/namespace"
  f5xc_namespace_name = format("%s-namespace-%s", var.project_prefix, var.project_suffix)
  providers           = {
    volterra = volterra.default
  }
}

module "vk8s_reference" {
  source                     = "./modules/f5xc/v8ks"
  f5xc_tenant                = var.f5xc_tenant
  f5xc_api_url               = var.f5xc_api_url
  f5xc_api_token             = var.f5xc_api_token
  f5xc_vk8s_name             = format("%s-vk8s-reference-%s", var.project_prefix, var.project_suffix)
  f5xc_virtual_site_refs     = ["virtual-site-a", "virtual-site-b"]
  f5xc_vsite_refs_namespace  = var.f5xc_vsite_refs_namespace
  f5xc_virtual_k8s_namespace = module.f5xc_namespace.namespace["name"]
  providers                  = {
    volterra = volterra.default
  }
}

module "credential_by_reference" {
  source                     = "./modules/f5xc/api-credential"
  f5xc_tenant                = var.f5xc_tenant
  f5xc_api_url               = var.f5xc_api_url
  f5xc_api_token             = var.f5xc_api_token
  f5xc_virtual_k8s_name      = module.vk8s_reference.vk8s["name"]
  f5xc_api_credential_type   = "KUBE_CONFIG"
  f5xc_api_credentials_name  = format("%s-vk8s-credential-%s", var.project_prefix, var.project_suffix)
  f5xc_virtual_k8s_namespace = module.f5xc_namespace.namespace["name"]
}

output "kube_config_reference" {
  value     = module.credential_by_reference.api_credential["k8s_conf"]
  sensitive = false
}

output "workload_config_reference" {
  value = local.workload_content
}
````

## vk8s create kubeconf inline module usage example 

````hcl
module "f5xc_namespace" {
  source              = "./modules/f5xc/namespace"
  f5xc_namespace_name = format("%s-namespace-%s", var.project_prefix, var.project_suffix)
  providers           = {
    volterra = volterra.default
  }
}

module "f5xc_virtual_site" {
  source                                = "./modules/f5xc/site/virtual"
  f5xc_namespace                        = "shared"
  f5xc_virtual_site_name                = format("%s-regression-%s", var.project_prefix, var.project_suffix)
  f5xc_virtual_site_type                = "CUSTOMER_EDGE"
  f5xc_virtual_site_selector_expression = ["site_mesh_group in (regression-sites)"]
  providers                             = {
    volterra = volterra.default
  }
}

module "vk8s_inline" {
  source                     = "./modules/f5xc/v8ks"
  f5xc_tenant                = var.f5xc_tenant
  f5xc_api_url               = var.f5xc_api_url
  f5xc_api_token             = var.f5xc_api_token
  f5xc_vk8s_name             = format("%s-vk8s-inline-%s", var.project_prefix, var.project_suffix)
  f5xc_create_k8s_creds      = true
  f5xc_virtual_k8s_name      = module.vk8s_reference.vk8s["name"]
  f5xc_virtual_site_refs     = [module.f5xc_virtual_site.virtual-site["name"]]
  f5xc_vsite_refs_namespace  = var.f5xc_vsite_refs_namespace
  f5xc_virtual_k8s_namespace = module.f5xc_namespace.namespace["name"]
  f5xc_k8s_credentials_name  = format("%s-vk8s-inline-creds-%s", var.project_prefix, var.project_suffix)
  providers                  = {
    volterra = volterra.default
  }
}

output "kube_config_inline" {
  value = module.vk8s_inline.vk8s["k8s_conf"]
  sensitive = true
}
````