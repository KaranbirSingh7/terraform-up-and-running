# terraform-up-and-running
A repo to keep track of my progress from book "Terraform Up and Running"


## Terraform Cheatsheet

**If you don't like HCL (HashiCorp Configuration Language), you can also write terraform in JSON with extension as `main.tf.json`**


**Where does terraform store state?**

Defaults to localfilesystem as `terraform.tf.state`. It can be used with other backends such as s3, azure blob and google storage. 
You would run into chicken-egg problem where backend doesn't exist in cloud provider. This case you would need two terraform scripts/modules. One to initialize backend, second to use that backend and make it as source of truth.


**Can I lock terraform state remotely?**

Yes, but you want to use another cloud resource. In AWS case, you use `dynamo_db` with a column called `LockID`



**String interpolation using `${...}`:**
```hcl
    user_data = <<-EOF
                #!/bin/bash
                echo "hello world" > index.html
                nohup busybox httpd -f -p ${var.server_port} &
                EOF
```


**Every resource in terraform supports `lifecycle` method.
This takes care of create/update/delete of resource.
Scenario: Delete resource only if new one is created successfully and is stable.**
```hcl
resource "aws_ec2" "example" {
    ami = "xXXXxx"

    lifecycle {
        create_before_destroy = true
    }
}

```


**Specify value for a variable using `-var` flag:**
```bash
terraform plan -var "server_port" 8080
```


**Specify value for a variable using environment var:**
```bash
export TF_VAR_server_port=8080
terraform plan
```


**Specify value file for a plan:**
```bash
terraform plan -var-file dev_vars.tf
```


**Show outputs:**
```bash
terraform output
```

**Show specific variable output:**
```bash
# terraform output <VARIABLE-NAME>
terraform output public_ip
```
