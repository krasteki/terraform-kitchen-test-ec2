### This repo contains terraform configuration to spin up an Amazon Web Services (AWS) EC2 instance using Inspec and Kitchen-Terraform from scratch.

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

  3.4. Add another verifier section. The verifier is what will verify whether your Test Kitchen instances match what is laid out in your inspec files.
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

III. Add the test

1. Set up an inspec directory within our test directories
```
$ mkdir -p test/integration/default/controls
```
This will be our default group of tests. Now we need to provide a yml file with the name of that group within the group directory.
```
$ vim test/integration/default/inspec.yml
```
Add the following lines in the `inspec.yml
```
---
name: default
```

2. Create EC2 instance and the rest of the resources in `main.tf`
3. Add `variables.tf`
4. Add `testing.tfvars.`
And add in this content (substitute in the appropriate values for your AWS account, region, etc. key_name will be the existing key pair name already existing in AWS)
5. Add `outputs.tf`

6. Create test Kitchen instances
```
$ bundle exec kitchen converge
```
7. Check the output
```
Outputs:

public_dns = your_aws_instance_public_ip
```

8.  Try running the tests as is
```
$ bundle exec kitchen verify
```

If it is successful the output should be like:
```
Finished in 0.00292 seconds (files took 6.57 seconds to load)
0 examples, 0 failures
```

9. Add basic test to make sure we are running on an Ubuntu system

```
$ vim test/integration/default/controls/operating_system_spec.rb
```

10. Run the test
```
$ bundle exec kitchen verify
```
output
```
Command: `lsb_release -a`
  stdout
    is expected to match /Ubuntu/

Finished in 0.21235 seconds (files took 4.37 seconds to load)
1 example, 0 failures
```

IV. destroy current test kitchen 
```
$ bundle exec kitchen destroy
```

1. Change the content of `testing.tfvars` - the `region` and the `ami` (new aws key pair is needed in the new region)

2. Now create the test kitchen instance again
```
$ bundle exec kitchen converge
```

3. Run the test
```
$ bundle exec kitchen verify
```
Transport error, can't connect to 'ssh' backend: SSH session could not be established
Exception
 Class: Kitchen::ActionFailed
 Message: 1 actions failed.
```

The username we have specified for our test is ubuntu - this is fine for an ubuntu instance, but for an Amazon Linux instance, we need to use the ec2-user username

4. Edit the `.kitchen.yml`
```
verifier:
  name: terraform
  systems:
    - name: default
      controls:
        - operating_system
      backend: ssh
      key_files:
        - ~/.ssh/ncs-laptop.pem
      hosts_output: public_dns
      user: ec2-user
      reporter:
        - documentation
```

5. Run the test again
```
$ bundle exec kitchen verify
```

And now we see a test failure:

```
Command: `lsb_release -a`
  stdout
    is expected to match /Ubuntu/ (FAILED - 1)

Failures:

  1) Command: `lsb_release -a` stdout is expected to match /Ubuntu/
     Failure/Error: DEFAULT_FAILURE_NOTIFIER = lambda { |failure, _opts| raise failure }

  
Finished in 0.20723 seconds (files took 6.56 seconds to load)
1 example, 1 failure

Failed examples:

rspec   Command: `lsb_release -a` stdout is expected to match /Ubuntu/

```

Alright, that means our test is failing when it is supposed to be failing, and passing when it is supposed to pass.

6. Change the username back to ubuntu in the `kitchen.yml` and the `ami` back to ubuntu.

7. Destroy the infra
```
$ bundle exec kitchen destroy
```
