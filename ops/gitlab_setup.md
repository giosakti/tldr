# Gitlab Setup

## Requirements

- Bare metal or VM with at least 4GB of RAM
- Install Ubuntu 16.04 LTS
  - Guided partitioning with LVM
  - Standard system utilities only

## Installation

1. Update Apt and install prerequisites

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install -y curl openssh-server ca-certificates
```

2. Option 1: Install using apt-get (preferred)

```
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
sudo EXTERNAL_URL="http://gitlab.local" apt-get install gitlab-ee
```

   Option 2: Install usign dpkg

```
curl -LJO https://packages.gitlab.com/gitlab/gitlab-ee/packages/ubuntu/trusty/gitlab-ee_10.4.0-ee.0_amd64.deb/download
sudo EXTERNAL_URL="http://gitlab.local" dpkg -i gitlab-ee_10.4.0-ee.0_amd64.deb
```

3. Install docker (when using docker for runner)

```
sudo apt-get update
sudo apt-get install \
  apt-transport-https \
  ca-certificates \
  curl \
  software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
sudo apt-get update
sudo apt-get install docker-ce
```

4. Install Gitlab runner

```
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
sudo apt-get install gitlab-runner

sudo gitlab-runner register
# specify gitlab host
# get token from gitlab
```

modify `/etc/gitlab-runner/config.toml` by adding host as necessary
```
  [runners.docker]
    tls_verify = false
    image = "ruby:ruby:2.4"
    ...
    extra_hosts = ["gitlab.local:192.168.33.10"]
```

## References

- https://docs.gitlab.com/runner/install/linux-repository.html
- https://docs.gitlab.com/runner/register/index.html
- https://docs.gitlab.com/ce/ci/runners/README.html
