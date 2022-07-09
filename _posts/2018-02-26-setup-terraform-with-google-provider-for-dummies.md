---
url: https://medium.com/notes-and-tips-in-full-stack-development/setup-terraform-with-google-provider-for-dummies-7da1e302ca85
canonical_url: https://medium.com/notes-and-tips-in-full-stack-development/setup-terraform-with-google-provider-for-dummies-7da1e302ca85
title: Setup Terraform with Google Provider
subtitle: This is article focused on routine steps how to start working with terraform
  and google cloud provider. For new clients/projects, we do the…
slug: setup-terraform-with-google-provider-for-dummies
description: ""
tags:
- google-cloud-platform
- terraform
- devops
- amazon
- tutorial
author: Michael Nikitochkin
username: miry
---

# Setup Terraform with Google Provider

![Photo by Jacob Miller on Unsplash](/assets/2018-02-26-setup-terraform-with-google-provider-for-dummies-1_XlcDf2s9-GCmqgi6WCw33Q.jpeg)

This is article focused on routine steps how to start working with terraform and google cloud provider. For new clients/projects, we do the same steps again and again. I will try to cover the whole process from scratch and record in this article all my steps.

* Create a google/gmail account.

* Register organisation or personal account in [https://cloud.google.com](https://cloud.google.com). It requires credit card information. Don’t worry, you will have $300 as credit for 12 months.

* Google cloud console authorization

* Set default application login solution

* Init a terraform project

* Create a google cloud project

# Google cloud console authorization

![Photo by Bernard Hermant on Unsplash](/assets/2018-02-26-setup-terraform-with-google-provider-for-dummies-1_i6D3J1KqW9DvczLGKhoZvw.jpeg)

Steps to authorize google account are based on official documentation [https://cloud.google.com/storage/docs/gsutil_install#creds-sdk](https://cloud.google.com/storage/docs/gsutil_install#creds-sdk). Install Google Cloud SDK and authorize:

```
$ brew install caskroom/cask/google-cloud-sdk
$ gcloud init # Will open browser and authorize your account
```

After this procedure, you should see the page [https://cloud.google.com/sdk/auth_success](https://cloud.google.com/sdk/auth_success) with “You are now authenticated with the Google Cloud SDK!”.

In the terminal you should pick the first default project on prompt:

```
You are logged in as: [user@example.com].
Pick cloud project to use: 
 [1] super-man-198503
 [2] Create a new project
Please enter numeric choice or text value (must exactly match list 
item):  1
```

To use terraform we can generate a separate service account or create a default application login

### Default application login solution

Reference: [https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login](https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login)

```
$ gcloud auth application-default login
Credentials saved to file: [/Users/dev/.config/gcloud/application_default_credentials.json]
...
```

This method would not require you to specify the credentials json path file in the terraform. All permissions would rely on a user who will use the terraform.

Another solution is to generate a separate service account: [https://cloud.google.com/docs/authentication/getting-started](https://cloud.google.com/docs/authentication/getting-started).

# Init terraform project

First, we need to test authorization to the Google API and configure Google provider for terraform: [https://www.terraform.io/docs/providers/google/index.html](https://www.terraform.io/docs/providers/google/index.html). In my case, I used an organization account.

Create a file `gcp.tf`:

```
// gcp.tf
data "google_organization" "goldminer" {
  domain = "example.com"
}
```

```
output "org_id" {
  value = "${data.google_organization.goldminer.id}"
}
```

Then we initialise terraform project and check that it works:

```
$ terraform init
Initializing provider plugins...
- Checking for available provider plugins on https://releases.hashicorp.com...
- Downloading plugin for provider "google" (1.6.0)...
...
* provider.google: version = "~> 1.6"
Terraform has been successfully initialized!
...
```

```
$ terraform apply -auto-approve
data.google_organization.goldminer: Refreshing state...
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
Outputs:
org_id = 123456789012
```

If you have a problem, try to generate a service account first your self with [https://cloud.google.com/docs/authentication/getting-started](https://cloud.google.com/docs/authentication/getting-started), download the credentials json file and specify a google provider [https://www.terraform.io/docs/providers/google/index.html](https://www.terraform.io/docs/providers/google/index.html) eg:

```
provider "google" {
  credentials = "${file("${path.module}/account.json")}"
  project     = "super-man-198503"
  region      = "us-central1-a"
}
```

# Create a Google cloud project

![Photo by Daniel Kainz on Unsplash](/assets/2018-02-26-setup-terraform-with-google-provider-for-dummies-1_LLYShBDnB-TsnVakB7BqzQ.jpeg)

We register our account to use Google cloud resources. By default, you will have a generated project “My First Project” with some random id, for example `super-man-198503`. There are no problems with default one, but I recommend to create projects with meaningful names.

Check the terraform `google_project` resource documentation [https://www.terraform.io/docs/providers/google/r/google_project.html](https://www.terraform.io/docs/providers/google/r/google_project.html). Base on it, let’s create a project resource:

```
// kubernetes_project.tf
resource "google_project" "kubernetes" {
  name = "kubernetes"
  project_id = "kubernetes-${data.google_organization.goldminer.directory_customer_id}"
  org_id = "${data.google_organization.goldminer.id}"
}
```

```
output "project_id" {
  value = "${google_project.kubernetes.id}"
}
```

Check that we are going to create a correct resource with plan and confirm it after:

```
$ terraform apply
...
Terraform will perform the following actions:
```

```
+ google_project.kubernetes
      id:          <computed>
      folder_id:   <computed>
      name:        "kubernetes"
      number:      <computed>
      org_id:      "123456789012"
      policy_data: <computed>
      policy_etag: <computed>
      project_id:  "kubernetes-a01bvf123"
      skip_delete: <computed>
```

```
Plan: 1 to add, 0 to change, 0 to destroy.
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.
```

```
Enter a value: yes
...
Outputs:
```

```
org_id = 123456789012
project_id = kubernetes-a01bvf123
```

For me, in web console of Google Cloud, this project did not appear at the same moment, so I tested with a command: `gcloud projects list`.

![](/assets/2018-02-26-setup-terraform-with-google-provider-for-dummies-1_191jZ4P6L-uGpX-7Fs1zGg.png)

**Michael Nikitochkin** *is a Lead Software Engineer. Follow him on [LinkedIn](https://www.linkedin.com/in/michaelnikitochkin/) or [GitHub](https://github.com/miry).*

> If you enjoyed this story, we recommend reading our [latest tech stories](https://jtway.co/latest) and [trending tech stories](https://jtway.co/trending).


