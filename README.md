### This repo contains terraform configuration

I. Prerequisites

An AWS Account
An AWS Access Key ID
An AWS Secret Key
An AWS Keypair
Terraform installed
Bundler installed
Ruby >= 2.4, < 2.8
The default security group on your account must allow SSH access from your IP address.

II. Building from scratch

1. Add in the Kitchen-Terraform gem like below (substituting your Ruby version if necessary):

```
ruby '3.1.1'

source 'https://rubygems.org/' do
  gem 'kitchen-terraform', '>= 5.7'
end
```

2. run bundler to install these gems:
```
$ bundle install
```

