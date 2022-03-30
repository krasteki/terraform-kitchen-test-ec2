### This repo contains terraform configuration

I. Prerequisites

An AWS Account
An AWS Access Key ID
An AWS Secret Key
An AWS Keypair
Terraform installed
Bundler installed
Ruby = 3.1.1

The default security group on your account must allow SSH access from your IP address.

II. Building from scratch

1. Add in the Kitchen-Terraform gem like below (substituting your Ruby version if necessary):

```
ruby '3.1.1'

source 'https://rubygems.org/' do
  gem 'kitchen-terraform', '>= 5.7'
end
```

2. Run bundler to install these gems:
```
$ bundle install
```

3. Create `.kitchen.yaml`

  3.1. First the driver - the driver we are using is called terraform.
  ```
  ---
  driver:
    name: terraform
    variable_files:
      - testing.tfvars
  ```

  3.2. Add the provisioner. The provisioner we will use is also called terraform.
  ```
  ---
  driver:
    name: terraform
    variable_files:
      - testing.tfvars

  provisioner:
    name: terraform
  ```

  3.3. Add in a platform. Although our terraform config will determine what OS we use, we need to include at least one value here
  ```
  ---
  driver:
    name: terraform
    variable_files:
      - testing.tfvars

  provisioner:
    name: terraform

  platforms:
    - name: ubuntu
  ```

  3.4. Add another verifier section. he verifier is what will verify whether your Test Kitchen instances match what is laid out in your inspec files.
  ```
  ---
  driver:
    name: terraform
    variable_files:
      - testing.tfvars

  provisioner:
    name: terraform

  platforms:
    - name: ubuntu

  verifier:
    name: terraform
    systems:
      - name: default
        controls:
          - operating_system
        backend: ssh
        user: ubuntu
        key_files:
          - ~/path/to/your/private/aws/key.pem
        hosts_output: public_dns
        reporter:
          - documentation
  ```

  3.5. Add a test suite at the end of the `.kitchen.yml`
  ```
  suites:
  - name: default
  ```



