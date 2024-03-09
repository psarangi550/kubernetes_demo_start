# Introduction to HashiCorp Vault on Kubernetes 

- how to get `hashicorp key vault` up and running in `kubernetes` ?

- we will be looking at all the `key component` that makes a `hashicorp key vault` ?

- we will also be looking at the `configuration of the key vault` ?

- **Why do we need a Key Vault ?**

- lets suppose we have an `application` and a `Database` with `password`

- we can usually put the `password` of the `Database` inside the `kubernetes secrets`

- we can `put that secret` in to the `key vault` , which can be seemed `skeptical` because we are trading `one secret with another secret`

- here we are `trading one key for another key` preveiously we need the `key of the database password` now we need the `key of the hashicorp key vault`

- if `somebody have to get the key` then they can access `all the database password` thats been saved to the `key vault` not just one `database password`

- 