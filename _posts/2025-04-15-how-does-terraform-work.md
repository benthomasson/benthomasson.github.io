---
title:  "How does Terraform work?"
date:   2025-04-15 05:08:00 -0500
categories: terraform code-review
---

Terraform is an automation tool that excels at provisioning infrastructure. How do Terraform providers work?  Let's
take a close look at the example [hashicups provider](https://github.com/hashicorp/terraform-provider-hashicups). 

Terraform providers are Go programs.  After checking out this repo and setting up a Go environment they can be compiled
as normal:

    git checkout https://github.com/hashicorp/terraform-provider-hashicups.git
    cd terraform-provider-hashicups
    go build -o terraform-provider-hashicups






* Outline

[link][https://benthomasson.com]

```python
print('Hello world')
```
