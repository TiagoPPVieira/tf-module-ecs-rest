# Terraform module to deploy an APP into ECS Fargate cluster.

This module:
* Create tasks.
* Create service.
* Zero downtime deploy.
* Create monitoring using CloudWatch.
* Receive an json with containers definitions.


## Usage

```terraform
module "app-deploy" {
  source                 = "git::http://<user>:<pass>@200.185.43.241:7990/scm/arq/tf-module-ecs-rest.git?ref=v0.6.1"
  containers_definitions = data.template_file.containers_definitions_json.rendered
  ecs_cluster_id         = data.terraform_remote_state.ecs_cluster.outputs.ecs_cluster_id
  app_project_name       = var.APP_PROJECT_NAME
  app_name               = var.APP_NAME
  app_port               = "80"
  cloudwatch_group_name  = "/ecs/${var.APP_PROJECT_NAME}_${var.APP_NAME}"
  vpc_tag_name           = var.vpc_tag_name
  subnet_private_tag_name= var.subnet_private_tag_name
  subnet_public_tag_name = var.subnet_public_tag_name
}

################   DATA   ################ 
data "terraform_remote_state" "ecs_cluster" {
    backend = "s3"
    config = {
       bucket = "dev-marisa-tfstate"
       key = "global/${var.APP_PROJECT_NAME}/terraform.state"
       region = "us-east-2"
    }
}

data "template_file" "containers_definitions_json" {
  template = file("./templates/containers_definitions.json")

  vars = {
    APP_VERSION = var.APP_VERSION
    APP_PROJECT_NAME = var.APP_PROJECT_NAME
    APP_NAME = var.APP_NAME
    LOG_GROUP_NAME = "/ecs/${var.APP_PROJECT_NAME}_${var.APP_NAME}"
    AWS_REGION  = var.aws_region
  }
}

################   VARIABLES   ################ 
variable "APP_PROJECT_NAME" {
  default = "dev-teste"
}

variable "APP_VERSION" {
  default = "latest"
}

variable "APP_NAME" {
  default = "app-name"
}

variable "aws_region" {
  default = "us-east-2"
}

variable "vpc_tag_name" {
  default = "arquitetura"
}

variable "subnet_public_tag_name" {
  default = "Public"
}

variable "subnet_private_tag_name" {
  default = "Private"
}
```

## Container Definition Sample

Create a folder called `templates` and a file called `containers_definitions.json` with the content bellow:

```json
[
  {
    "cpu": 256,
    "image": "584991916600.dkr.ecr.us-east-2.amazonaws.com/${APP_PROJECT_NAME}/${APP_NAME}:${APP_VERSION}",
    "memory": 512,
    "name": "${APP_NAME}-cnt",
    "networkMode": "awsvpc",
    "portMappings": [
      {
        "containerPort": 80,
        "hostPort": 80,
        "protocol":"tcp"
      }
    ],
    "environment": [
      {
        "name": "PROPERTIES_SPRING",
        "value": "application-hml.yml"
      },{
         "name": "JVM_MEMORY",
        "value": "512M"
      },{
         "name": "ENVIRONMENT_NEWRELIC",
        "value": "staging"
      }
    ],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "${LOG_GROUP_NAME}",
        "awslogs-region": "us-east-2",
        "awslogs-stream-prefix": "ecs"
      }
    }
  }
]
```

* The `name` of one container created on `containers_definitions.json` should be the same as `app_name` passed to module.
* You can configure almost everything as variable, and you probably should do this. 

## Multiple Container Definition Sample

```json
[
  {
    "cpu": 256,
    "image": "584991916600.dkr.ecr.us-east-2.amazonaws.com/${APP_IMAGE}_rest:${APP_VERSION}",
    "memory": 512,
    "name": "${APP_IMAGE}_rest",
    "networkMode": "awsvpc",
    "portMappings": [
      {
        "containerPort": 80,
        "hostPort": 80,
        "protocol":"tcp"
      }
    ],
    "environment": [
      {
        "name": "PROPERTIES_SPRING",
        "value": "application-hml.yml"
      },{
         "name": "JVM_MEMORY",
        "value": "512M"
      },{
         "name": "ENVIRONMENT_NEWRELIC",
        "value": "staging"
      }
    ]
  },
  {
    "cpu": 256,
    "image": "584991916600.dkr.ecr.us-east-2.amazonaws.com/${APP_IMAGE}_service:${APP_VERSION}",
    "memory": 512,
    "name": "${APP_IMAGE}_service",
    "networkMode": "awsvpc"
  },
  "environment": [
      {
        "name": "PROPERTIES_SPRING",
        "value": "application-hml.yml"
      },{
         "name": "JVM_MEMORY",
        "value": "512M"
      },{
         "name": "ENVIRONMENT_NEWRELIC",
        "value": "staging"
      }
  ]
]
```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| app\_count | Number of tasks that will be deployed for this app. | string | `"1"` | no |
| app\_project\_name | How your project will be called. The project name have to be the same name used to create the ecs_cluster | string | n/a | yes |
| app\_name | How your app will be called. | string | n/a | yes |
| app\_port | The PORT that will be used to communication between load balancer and container. | string | `"3000"` | no |
| aws\_region | The AWS region to create things in. | string | `"us-east-2"` | no |
| ecs\_cluster\_id | The cluster id created before to accomodate the app. | string | n/a | yes |
| cloudwatch\_group\_name | CloudWatch group name where to send the logs. | string | `"sample-group-name"` | no |
| containers\_definitions | A JSON with all container definitions that should be run on the task. For more http://bit.do/eKzfH | string | n/a | yes |
| fargate\_cpu | The maximum of CPU that the task can use. | string | `"256"` | no |
| fargate\_memory | The maximum of memory that the task can use. | string | `"512"` | no |
| fargate\_version | The fargate version used to deploy inside ECS cluster. | string | `"1.3.0"` | no |

# Wiki

Want to know more? 
# tf-module-ecs-rest
