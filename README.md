<p align="center">
<img src="docs/aws_maintenance_lambda.png"/>
</p>

A lambda function to send alerts (to Slack, Hipchat) on AWS maintenance events. While the email from AWS includes only the instance id, the alert will include the Name of the instance and owner from the appropriate tags.

## Sample Notification on Slack
![](docs/slack-notification.png)

## Sample notification on HipChat
![](docs/hipchat-notification.png)

## Prerequisite

The lambda function assumes that all resources (EC2 instances) are tagged with a key `Owner` specifying the owner of the resource.

## Installation

Update `lambda/config.json` with necessary config for your environment. The keys are explained below:

`store.simpledb.domain` - The lambda function keeps track of processed events in AWS simbedb. This configures the simpledb domain to be used for this purpose.

`notification.hipchat`
  - `auth_token` - The Hipchat API token.
  - `room` - The room to send the notifications to.
  - `icon_url` - Icon to use for the bot that sends the notification.
  - `username` - Username of the bot that sends the notification.
  - `owners` - List of owners per tag. The keys here will be the value of the tag `Owner`. This maps the tag value to owners - for example - `"devops : { "owner": "@devops_team"}"`
    - `all` - this is a catchall owner that is used as default if the resource did not have the `Owner` tag.


`notification.slack`
  - `hook` - The slack hook url.
  - `channel` - The channel to send the notifications to.
  - `icon_url` - Icon to use for the bot that sends the notification.
  - `username` - Username of the bot that sends the notification.
  - `owners` - List of owners per tag. The keys here will be the value of the tag `Owner`. This maps the tag value to owners - for example - `"devops : { "owner": "@devops_team"}"`
    - `all` - this is a catchall owner that is used as default if the resource did not have the `Owner` tag.

### Manual
    
Once the `config.json` has been updated, the lambda function can be manually installed by doing a `npm install --production`, zipping up the entire lambda folder and uploading to AWS like any other lambda function.

### Terraform

The repo also has terraform plans to setup the lambda function - including the necessary IAM roles and lambda schedule (once an hour by default). A normal `terraform plan` and `terraform apply` should fully setup the lambda function. Requires  terraform 0.7.8+.

#### Using as a module

The terraform plans can also be used as a module within your existing terraform project. Add as a module with something like below:

```hcl
module "aws-maintenance-lambda" {
  source = "github.com/indix/aws-maintenance-lambda//terraform"

  lambda_prepared_source_dir = "${path.root}/aws-maintenance-lambda-temp/source"
  lambda_archive_path = "${path.root}/aws-maintenance-lambda-temp/dist/aws_maintenance_lambda.zip"
  config_json = "${path.root}/files/aws-maintenance-lambda-config.json"
}
```

## License

This is an open source project licensed under the [Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0).
