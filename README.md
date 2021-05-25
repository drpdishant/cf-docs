# Welcome to Documentation for Openxcell Cloud

## Overview 

Openxcell Cloud is a distribution of [Cloudfoundry](https://cloudfoundry.org) Managed by DevOps Team to provide self service app deployment platform for development teams.

> cf push .. and your app is live

## Quick Setup

<!-- tabs:start -->

### **Windows**

[Download Installer](https://packages.cloudfoundry.org/stable?release=windows64&version=v7&source=github)

### **Linux**

- Debian/Ubuntu 

```bash
curl -sSLk https://docs.cloud.openxcell.dev/scripts/install-deb.sh | sudo bash
```

- RHEL/Fedora

```bash
curl -sSLk https://docs.cloud.openxcell.dev/scripts/install-rhel.sh | sudo bash
```
### **Mac**

Using HomeBrew

```bash
brew install cloudfoundry/tap/cf-cli@7
```

or [Download Installer](https://packages.cloudfoundry.org/stable?release=macosx64&version=v7&source=github)

<!-- tabs:end -->

### Configure Cloudfoundry API Endpoint

```bash
cf api api.cloud.openxcell.dev
```

### Login to Cloudfoundry CLI

```bash
cf auth USERNAME PASSWORD
```

> Replace testing with your username and login with password provided by the platform administrators
