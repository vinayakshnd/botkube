# BotKube
[![Build Status](https://travis-ci.org/infracloudio/botkube.svg?branch=master)](https://travis-ci.org/infracloudio/botkube) [![Go Report Card](https://goreportcard.com/badge/github.com/infracloudio/botkube)](https://goreportcard.com/report/github.com/infracloudio/botkube) [![BotKube website](https://img.shields.io/badge/docs-botkube.io-blue.svg)](https://botkube.io) [![Slack](https://botkube-slack.herokuapp.com/badge.svg)](http://join.botkube.io/)

For complete documentation visit www.botkube.io

BotKube integration with Slack or Mattermost helps you monitor your Kubernetes cluster, debug critical deployments and gives recommendations for standard practices by running checks on the Kubernetes resources.
You can also ask BotKube to execute kubectl commands on k8s cluster which helps debugging an application or cluster.

![](botkube-title.jpg)

## Getting started
Please follow [this](https://www.botkube.io/installation/) for complete BotKube installation guide.
### Install BotKube app to your Slack workspace
Click the "Add to Slack" button provided to install `BotKube` Slack application to your workspace. Once you authorized the application, you will be provided a BOT Access token. Kindly note down that token which will be required while deploying BotKube controller to your cluster

<a href="https://slack.com/oauth/authorize?scope=commands,bot&client_id=12637824912.515475697794"><img alt="Add to Slack" height="40" width="139" src="https://platform.slack-edge.com/img/add_to_slack.png" srcset="https://platform.slack-edge.com/img/add_to_slack.png 1x, https://platform.slack-edge.com/img/add_to_slack@2x.png 2x" /></a>

### Add BotKube to a Slack channel
After installing BotKube app to your Slack workspace, you could see new bot user with name 'BotKube' create in your workspace. Add that bot to a Slack channel you want to receive notification in. (You can add it by inviting using `@BotKube` message in a required channel)

### Installing BotKube controller to your Kubernetes cluster

#### Using helm

- We will be using [helm](https://helm.sh/) to install our k8s controller. Follow [this](https://docs.helm.sh/using_helm/#installing-helm) guide to install helm if you don't have it installed already
- Clone the BotKube github repository.
```bash
$ git clone https://github.com/infracloudio/botkube.git
```

- Update default **config** in **helm/botkube/values.yaml** to watch the resources you want. (by default you will receive **create**, **delete** and **error** events for all the resources in all the namespaces.)
If you are not interested in events about particular resource, just remove it's entry from the config file.
- Deploy BotKube controller using **helm install** in your cluster.
```bash
$ helm install --name botkube --namespace botkube \
--set config.communications.slack.enabled=true \
--set config.communications.slack.channel={SLACK_CHANNEL_NAME} \
--set config.communications.slack.token={SLACK_API_TOKEN_FOR_THE_BOT} \
--set config.settings.clustername={CLUSTER_NAME} \
--set config.settings.allowkubectl={ALLOW_KUBECTL} \
helm/botkube
```

  where,<br>
  **SLACK_CHANNEL_NAME** is the channel name where @BotKube is added<br>
  **SLACK_API_TOKEN_FOR_THE_BOT** is the Token you received after installing BotKube app to your Slack workspace<br>
  **CLUSTER_NAME** is the cluster name set in the incoming messages<br>
  **ALLOW_KUBECTL** set true to allow kubectl command execution by BotKube on the cluster<br>

  Configuration syntax is explained [here](https://www.botkube.io/configuration) 

- Send **@BotKube ping** in the channel to see if BotKube is running and responding.

#### Using kubectl

- Make sure that you have kubectl cli installed and have access to Kubernetes cluster
- Download deployment specs yaml

```bash
$ wget -q https://raw.githubusercontent.com/infracloudio/botkube/master/deploy-all-in-one.yaml
```

- Open downloaded **deploy-all-in-one.yaml** and update the configuration.
  Set *SLACK_CHANNEL*, *SLACK_API_TOKEN*, *clustername*, *allowkubectl* and update the resource events configuration you want to receive notifications for in the configmap.

  where,<br>
  **SLACK_CHANNEL** is the channel name where @BotKube is added<br>
  **SLACK_API_TOKEN** is the Token you received after installing BotKube app to your Slack workspace<br>
  **clustername** is the cluster name set in the incoming messages<br>
  **allowkubectl** set true to allow kubectl command execution by BotKube on the cluster<br>

  Configuration syntax is explained [here](https://www.botkube.io/configuration) 

- Create **botkube** namespace and deploy resources

```bash
$ kubectl create ns botkube && kubectl create -f deploy-all-in-one.yaml -n botkube
```

- Check pod status in botkube namespace. Once running, send **@BotKube ping** in the Slack channel to confirm if BotKube is responding correctly.

- To deploy with TLS, download and use deploy-all-in-one-tls.yaml. Replace **ENCODED_CERTIFICATE** with your base64 encoded certificate value in the secret. To get a base64 encoded value of your certificate, use below command and replace <YOUR_CERTIFICATE> with the certificate name.

```bash
$ cat <YOUR_CERTIFICATE> | base64 -w 0
```

## Add BotKube to your Mattermost team
Before adding `BotKube` to your Mattermost team, please make sure you have all the pre-requisites working.

### Pre-requisites
**Step 1**: Login with System Admin account, and in the Menu proceed to System console -> Integrations -> Custom Integrations and enable `Personal Access Token`.<br><br>
**Step 2**: To create a Botkube user, if not already created, proceed to menu and Get team invite link. Logout from admin account and paste the link in the address bar and create a user with the username `BotKube`.<br><br>
**Step 3**: To create a Botkube alerts channel, if not already created, click on `Create new channel` and fill the details of the channel. Add BotKube user to the Channel and use the same Channel name in the configuration parameters for Botkube.<br><br>
**Step 4**: Login as System Admin and in the Menu proceed to System console -> Users. For `BotKube` user, Manage Roles and allow tokens and post_all access.<br><br>
**Step 5**: Login as BotKube user, in the Menu proceed to Account Settings -> Security -> Personal Access Token -> Create and save the token.

### Configurations
In the helm chart, config.yaml or deploy-all-in-one.yaml update the following fields for enabling Mattermost support and providing Mattermost config parameters.

**MATTERMOST_SERVER_URL** is the URL where Mattermost is running<br>
**MATTERMOST_CERT** is the SSL certificate file for HTTPS connection. Place it in Helm directory and specify the path<br>
**MATTERMOST_TOKEN** is the Token you received after installing BotKube user and creating Personal Access Token<br>
**MATTERMOST_TEAM** is the team name where BotKube will be added<br>
**MATTERMOST_CHANNEL** is the channel name where BotKube will be added<br>

#### Using helm

```bash
$ helm install --name botkube --namespace botkube \
--set config.communications.mattermost.enabled=true \
--set config.communications.mattermost.url={MATTERMOST_SERVER_URL} \
--set config.communications.mattermost.cert={MATTERMOST_CERT} \
--set config.communications.mattermost.token={MATTERMOST_TOKEN} \
--set config.communications.mattermost.team={MATTERMOST_TEAM} \
--set config.communications.mattermost.channel={MATTERMOST_CHANNEL} \
--set config.settings.clustername={CLUSTER_NAME} \
--set config.settings.allowkubectl={ALLOW_KUBECTL} \
helm/botkube
```

## Architecture
![](/botkube_arch.jpg)
- **Informer Controller:** Registers informers to kube-apiserver to watch events on the configured k8s resources. It forwards the incoming k8s event to the Event Manager
- **Event Manager:** Extracts required fields from k8s event object and creates a new BotKube event struct. It passes BotKube event struct to the Filter Engine
- **Filter Engine:** Takes the k8s object and BotKube event struct and runs Filters on them. Each filter runs some validations on the k8s object and modifies the messages in the BotKube event struct if required.
- **Event Notifier:** Finally, notifier sends BotKube event over the configured communication channel.
- **Bot Interface:** Bot interface takes care of authenticating and managing connections with communication mediums like Slack, Mattermost. It reads/sends messages from/to commucation mediums. 
- **Executor:** Executes BotKube or kubectl command and sends back the result to the Bot interface.

Visit www.botkube.io for Configuration, Usage and Examples.
