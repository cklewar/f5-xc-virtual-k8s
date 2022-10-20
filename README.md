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

## vk8s module usage example

````hcl
module "vk8s" {
  source            = "./modules/f5xc/v8ks"
  f5xc_api_p12_file = var.f5xc_api_p12_file
  f5xc_api_token    = var.f5xc_api_token
  f5xc_api_url      = var.f5xc_api_url
  f5xc_namespace    = var.f5xc_virtual_k8s_namespace
  f5xc_tenant       = var.f5xc_tenant
  f5xc_vk8s_name    = format("%s-vk8s-%s", var.project_prefix, var.project_suffix)
}

module "credential" {
  source                     = "./modules/f5xc/api-credential"
  f5xc_tenant                = var.f5xc_tenant
  f5xc_api_url               = var.f5xc_api_url
  f5xc_api_token             = var.f5xc_api_token
  f5xc_namespace             = var.f5xc_namespace
  f5xc_virtual_k8s_name      = module.vk8s.vk8s["name"]
  f5xc_api_credential_type   = "KUBE_CONFIG"
  f5xc_api_credentials_name  = format("%s-vk8s-credential-%s", var.project_prefix, var.project_suffix)
  f5xc_virtual_k8s_namespace = var.f5xc_virtual_k8s_namespace
}

resource "local_file" "workload" {
  content  = local.workload_content
  filename = format("%s/%s", var.f5xc_vk8s_workload_file_path, var.f5xc_vk8s_workload_filename)
}

resource "local_file" "kubeconfig" {
  content  = base64decode(module.credential.api_credential["data"])
  filename = format("%s/%s", local.f5xc_vk8s_kubeconfig_file_path, var.f5xc_vk8s_kubeconfig_filename)
}

resource "null_resource" "apply_credentials" {
  depends_on = [module.vk8s, local_file.kubeconfig]
  triggers   = {
    kubectl_secret_name                   = var.kubectl_secret_name
    kubectl_secret_registry_type          = var.kubectl_secret_registry_type
    kubectl_secret_registry_server_type   = var.kubectl_secret_registry_server_type
    kubectl_secret_registry_server_name   = var.kubectl_secret_registry_server_name
    kubectl_secret_registry_username      = var.kubectl_secret_registry_username
    kubectl_secret_registry_username_type = var.kubectl_secret_registry_username_type
    kubectl_secret_registry_password      = var.kubectl_secret_registry_password
    kubectl_secret_registry_password_type = var.kubectl_secret_registry_password_type
    kubectl_secret_registry_email         = var.kubectl_secret_registry_email#
    kubectl_secret_registry_email_type    = var.kubectl_secret_registry_email_type
  }
  provisioner "local-exec" {
    command     = "kubectl create secret ${self.triggers.kubectl_secret_registry_type} ${self.triggers.kubectl_secret_name} ${self.triggers.kubectl_secret_registry_server_type}=${self.triggers.kubectl_secret_registry_server_name} ${self.triggers.kubectl_secret_registry_password_type}=${self.triggers.kubectl_secret_registry_password} ${self.triggers.kubectl_secret_registry_username_type}=${self.triggers.kubectl_secret_registry_username} ${self.triggers.kubectl_secret_registry_email_type}=${self.triggers.kubectl_secret_registry_email} --namespace=${var.f5xc_virtual_k8s_namespace}"
    environment = {
      KUBECONFIG = format("%s/%s", local.f5xc_vk8s_kubeconfig_file_path, var.f5xc_vk8s_kubeconfig_filename)
    }
  }
}

output "kube_config" {
  value     = base64decode(module.credential.api_credential["data"])
  sensitive = true
}

output "workload_config" {
  value = local.workload_content
}
````