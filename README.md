# Camunda Cloud Message Publish

This repository has a GitHub Workflow that uses [jwulf/zeebe-action](https://github.com/jwulf/zeebe-action) to act as a Zeebe message publisher for Camunda Cloud.

The GitHub Action can be triggered by the CAMUNDA-HTTP task worker in Camunda Cloud by posting a Repository Dispatch event. 
