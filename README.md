# Camunda Cloud Message Publish

This repository has a GitHub Workflow that uses [jwulf/zeebe-action](https://github.com/jwulf/zeebe-action) to act as a Zeebe message publisher for [Camunda Cloud](https://camunda.io).

The GitHub Action is triggered by the CAMUNDA-HTTP task worker in Camunda Cloud.

The messaging [publishing workflow](.github/workflows/publish_message.yml) looks like this:

```
name: CI

on: [repository_dispatch]

jobs:
  publish_message:

    runs-on: ubuntu-latest

    steps:
    
    - name: Publish message to Camunda Cloud
      uses: jwulf/zeebe-action@master
      with:
        zeebe_address: ${{ secrets.ZEEBE_ADDRESS }}
        zeebe_client_id: ${{ secrets.ZEEBE_CLIENT_ID }}
        zeebe_authorization_server_url: ${{ secrets.ZEEBE_AUTHORIZATION_SERVER_URL }}
        zeebe_client_secret: ${{ secrets.ZEEBE_CLIENT_SECRET }}
        operation: publishMessage
        message_name: ${{ github.event.client_payload.message_name }}
        variables: ${{ github.event.client_payload.variables }}
 ```

## Setup

### This repo

1. Fork this repo.
2. Grab your client connection info from the [Camunda Cloud console](https://console.cloud.camunda.io/).
3. Set the various values as secrets in your forked repo (Repo Settings > Secrets) - i.e: `ZEEBE_ADDRESS`, etc...

### Your GitHub account

1. Create a new GitHub token.
  1. Go to Account Settings > Developer settings > Personal Access Tokens > Generate New Token
  2. Create a token and give it `repo` scope.
2. Go to your [Camunda Cloud console](https://console.cloud.camunda.io/).
3. Click on the cluster.
4. Click `Worker Variables`.
5. Add a new variable `GitHubToken` and paste the token in as the value.

## Using the message publisher

Here is the minimal example, to publish a message back to Zeebe in Camunda Cloud, for example, to trigger the message start event of a workflow, with no variables.

Create a service task in a Zeebe BPMN model like this, replacing the value for `url` with the URL for your forked repository, and `startWorkflowX_MSG` with the name of the message that you want to send ([see here](https://github.com/zeebe-io/zeebe-http-worker/issues/45#issuecomment-577532830):

```
<bpmn:serviceTask id="Task_0ozskvn" name="Build new Release images">
  <bpmn:extensionElements>
    <zeebe:taskDefinition type="CAMUNDA-HTTP" />
    <zeebe:taskHeaders>
      <zeebe:header key="method" value="post" />
      <zeebe:header key="url" value="https://api.github.com/repos/jwulf/camunda-cloud-message-publish/dispatches" />
      <zeebe:header key="body" value="{&#34;message_name&#34;: &#34;startWorkflowX_MSG&#34;}" />
    </zeebe:taskHeaders>
    <zeebe:ioMapping>
      <zeebe:input source="githubAuthorization" target="authorization" />
    </zeebe:ioMapping>
  </bpmn:extensionElements>
  <bpmn:incoming>SequenceFlow_0zgov2g</bpmn:incoming>
  <bpmn:outgoing>SequenceFlow_0desyou</bpmn:outgoing>
</bpmn:serviceTask>
```

When you start an instance of a workflow, you need to include the following key with your variables:

```
{
   githubAuthorization: "Bearer {{GitHubToken}}"
}
```

This makes the GitHub token available to the CAMUNDA-HTTP worker. The token will be templated into this variable at runtime by Zeebe. The I/O mapping in the service task will cause the service worker to use this as the authorization header when it  posts the repository dispatch event to trigger the GitHub action.
