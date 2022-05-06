# terraform-up-and-running
A repo to keep track of my progress from book "Terraform Up and Running"


## Terraform Cheatsheet

**If you don't like HCL (HashiCorp Configuration Language), you can also write terraform in JSON with extension as `main.tf.json`**


**Where does terraform store state?**

Defaults to localfilesystem as `terraform.tf.state`. It can be used with other backends such as s3, azure blob and google storage. 
You would run into chicken-egg problem where backend doesn't exist in cloud provider. This case you would need two terraform scripts/modules. One to initialize backend, second to use that backend and make it as source of truth.


**Can I lock terraform state remotely?**

Yes, but you would have to use another cloud resource. In AWS case, you use `dynamo_db` with a column called `LockID`



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

**Production Grade Terraform Module Folder Structure**
```bash
├── examples
│   └── minimal.tf
├── test
│   └── main_test.go
├── main.tf
├── outputs.tf
└── variables.tf
```

**Provisioners: remote-exec, local-exec**
```hcl
# remote-exec inside resource, would only works when terraform is running
resource "aws_instance" "example" {
  ami                    = "ami-0c55b159cbfafe1f0"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.instance.id]
  key_name               = aws_key_pair.generated_key.key_name

  provisioner "remote-exec" {
    inline = ["echo \"Hello, World from $(uname -smp)\""]
  }

  connection {
    type        = "ssh"
    host        = self.public_ip
    user        = "ubuntu"
    private_key = tls_private_key.example.private_key_pem
  }
}
```
```hcl
# local-exec as seperate block that runs everytime on 'terraform apply'
resource "null_resource" "example" {
  # Use UUID to force this null_resource to be recreated on every call to 'terraform apply'
  triggers = {
    uuid = uuid()
  }
  provisioner "local-exec" {
    command = "echo \"Hello, World from $(uname -smp)\""
  }
}
```


**for_each loop over map and generate list for roles/user**
```hcl
# this logic comes in handy when you want to add multiple users to a single role
# ex: add multiple users with Contributor access to Azure Resource Group

locals {
  users = [
    {
      "user_name" : "abc@gmail.com"
      "principal_id" : "123"
    },
    {
      "user_name" : "efg@gmail.com"
      "principal_id" : "456"
    }
  ]

}

output "test" {
  value = [for u in local.users : {
    # "APIM contributor" : {
    email     = u.user_name
    object_id = u.principal_id
  }]
}


```

