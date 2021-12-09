# Welcome to Documentation for Openxcell Cloud

## Endpoints

**Cloud API:**  [api.cloud.openxcell.dev](https://api.cloud.openxcell.dev)

**Cloud Apps:** [apps.openxcell.dev](https://*.apps.openxcell.dev)

**Cloud Console:** [console.openxcell.dev](https://console.openxcell.dev)

## Overview

> Making deployments fun and easy for the developers

Openxcell Cloud is built on top of Kubernetes with [cf-for-k8s](https://cf-for-k8s.io/) a distribution of [Cloudfoundry](https://cloudfoundry.org) managed by the operations team at Openxcell to provide a unified and self service platform for developers to deploy and manage their applications.

Integrated with technologies like [open service broker api](https://www.openservicebrokerapi.org/) and [paketo](https://paketo.io/) we bring you the best in class platform where you can manage applications and their backing services, easily and in a fun way. Install the CLI or use the [Stratos Console](https://console.openxcell.dev) to get started.

[Paketo Buildpacks](https://paketo.io/) are based on [cloud native buildpack](https://buildpacks.io/) standards to magically convert your source code into production ready container images, that just run.

- No Dockerfiles
- No 100's of lines of pipelines

### Embracing Devops

At Openxcell we strive to embrace a Devops culture and mindset, and this is one such attempt to bring a cultural shift to the organization, by creating consistent and streamlined processes across teams, shifting responsibility and accountability of the application development to their respective teams.

## Quick Setup

<!-- tabs:start -->

### **Windows**

[Download Installer](https://packages.cloudfoundry.org/stable?release=windows64&version=v8&source=github)

### **Linux**

- Debian/Ubuntu

```bash
curl -sSLk https://docs.apps.openxcell.dev/scripts/install-deb.sh | sudo bash
```

- RHEL/Fedora

```bash
curl -sSLk https://docs.apps.openxcell.dev/scripts/install-rhel.sh | sudo bash
```

### **Mac**

Using HomeBrew

```bash
brew install cloudfoundry/tap/cf-cli@8
```

or [Download Installer](https://packages.cloudfoundry.org/stable?release=macosx64&version=v8&source=github)

<!-- tabs:end -->

### Configure Cloudfoundry API Endpoint

```bash
cf api api.cloud.openxcell.dev
```

### Login to Cloudfoundry CLI

```bash
cf login --sso
```

**Output:**

```bash
API endpoint: https://api.cloud.openxcell.dev

Temporary Authentication Code ( Get one at https://uaa.cloud.openxcell.dev/passc
ode ): 

```

- Open the temporary passcode url in your browser.

- If you are already logged in on browser it will show the token right away copy and paste it on CLI

- Otherwise on the Cloudfoundry Login Page below `Login From:` Click on [Openxcell SSO](https://uaa.cloud.openxcell.dev/saml/discovery?returnIDParam=idp&entityID=cloudfoundry-saml-login&idp=SAML&isPassive=true) login to keycloak it will redirect to the passcode page.

- If you are already logged in to browser you can go directly to <https://uaa.cloud.openxcell.dev/passcode> and get the temporary passcode, which you can use to login to cli using the following command

```bash
cf login --sso-passcode PASSCODE
```

### Set Target Org and Space

```bash
cf target -o sandbox -s sandbox-<id>
```

!> sandbox org is a common sandbox area to test with the platform. Replace it with the space name provided by the Administrators, ususally it will be the name of the project.

> Please contact Administrators for more details.

### Start Deploying

Follow the docs for specific platforms to configure and deploy.
