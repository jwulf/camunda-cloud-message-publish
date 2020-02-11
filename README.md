# Camunda Cloud Message Publish

This repository has a GitHub Workflow that uses [jwulf/zeebe-action](https://github.com/jwulf/zeebe-action) to act as a Zeebe message publisher for Camunda Cloud.

The GitHub Action can be triggered by the CAMUNDA-HTTP task worker in Camunda Cloud by posting a Repository Dispatch event.

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
