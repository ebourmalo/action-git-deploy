# Koyeb Deploy GitHub Action

This action builds and deploys an application from a GitHub repository to Koyeb.

## Usage

To use this action, add the following steps to your workflow:

```yaml
- name: Install and configure the Koyeb CLI
  uses: koyeb-community/install-koyeb-cli@v2
  with:
    api_token: "${{ secrets.KOYEB_API_TOKEN }}"

- name: Build and deploy the application to Koyeb
  uses: koyeb/action-git-deploy@v1
  with:
    service-env: "PORT=8000"
    service-ports: "8000:http"
    service-routes: "/:8000"
```

The first step installs the CLI, and requires to set a Koyeb API token as a [secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets). To generate a token, go to https://app.koyeb.com/settings/api.

The service-env, service-ports, and service-routes parameters are optional, but you should set them to match the needs of your application. The defaults provided are unlikely to work for most applications.

## Optional Parameters

The following optional parameters can be added to the with block:

| Name	                | Description                                                                                                       | Default Value
|-----------------------|-------------------------------------------------------------------------------------------------------------------|--------------
| `app-name`            | The name of the application                                                                                       | `<repo>-<branch>`
| `service-name`        | Name of the koyeb service to be created	                                                                         | `${{ github.ref_name }}`
| `build-timeout`       | Number of seconds to wait for the build. After this timeout, the job fails	                                       | `900` (15 min)
| `healthy-timeout`     | Number of seconds to wait for the service to become healthy. After this timeout, the job fails                    | `900` (15 min)
| `git-url`             | The URL of the GitHub repository to build                                                                         | `github.com/<organization>/<repo>`
| `git-workdir`         | Directory inside the repository to clone                                                                          | Empty string, which represents the root directory
| `git-branch`          | The Git branch to deploy	                                                                                         | `${{github.ref_name}}`
| `git-build-command`   | Command to build the application	                                                                                 | Empty string
| `git-run-command`     | Command to run the application	                                                                                   | Empty string
| `service-env`         | A comma-separated list of KEY=value pairs to set environment variables for the service	                           | No env
| `service-ports`       | A comma-separated list of port:protocol pairs to specify the ports and protocols to expose for the service	       | `80:http`
| `service-routes`      | A comma-separated list of `<path>:<port>` pairs to specify the routes to expose for the service                   | `/:80`
| `service-checks`      | A comma-separated list of `<port>:http:<path>` or `<port>:tcp` pairs to specify the healthchecks for the service  | No healthchecks


## Example

```yaml
name: Deploy to Koyeb

on:
  push:
    branches:
      - '*'

jobs:
  deploy:
    runs-on: ubuntu-latest
    concurrency:
      group: "${{ github.ref_name }}"
      cancel-in-progress: true
    steps:
      - name: Install and configure the Koyeb CLI
        uses: koyeb-community/install-koyeb-cli@v2
        with:
          api_token: "${{ secrets.KOYEB_API_TOKEN }}"
      - name: Build and deploy the application to Koyeb
        uses: koyeb/action-git-deploy@v1
        with:
          app-name: my-koyeb-app
          service-name: my-koyeb-service
          service-env: FOO=bar,BAZ=qux
          service-ports: "80:http,8080:http"
          service-routes: "/api:80,/docs:8080"
          service-checks: "8000:http:/,8001:tcp"
```

## Cleaning up Services

After deploying a service to Koyeb, you may want to remove it when it is no longer needed. To do this, you can use the `koyeb/action-git-deploy/cleanup` action. Here's an example of how to use this action:

```yaml
- name: Install and configure the Koyeb CLI
  uses: koyeb-community/install-koyeb-cli@v2
  with:
    api_token: "${{ secrets.KOYEB_API_TOKEN }}"

- name: Clean up Koyeb Service
  uses: koyeb/action-git-deploy/cleanup@v1
```

The first step installs the CLI and requires you to set a Koyeb API token as a secret.

The second steps performs the cleanup. Optionally, you can provide the `app-name` parameter (which defaults to `<repo>/<branch>`) to specify the name of the application to remove.

### Removing a Service When a Ref is Deleted

To remove a Koyeb service when a branch or tag is deleted, you can use the delete event in your workflow file. Here's an example of how to do this:

```yaml
name: Cleanup Koyeb application

on:
  delete:
    branches:
      - '*'

jobs:
  cleanup:
    runs-on: ubuntu-latest
    steps:
      - name: Install and configure the Koyeb CLI
        uses: koyeb-community/install-koyeb-cli@v2
        with:
          api_token: "${{ secrets.KOYEB_API_TOKEN }}"

      - name: Cleanup Koyeb application
        uses: koyeb/action-git-deploy/cleanup@v1
```

In this example, the workflow listens for any branch or tag that is deleted using the `'*'` wildcard. When a delete event occurs, the cleanup job runs and uses the `koyeb/action-git-deploy/cleanup` action to remove the corresponding Koyeb service. Be sure to set `KOYEB_API_TOKEN` as a repository secret.
