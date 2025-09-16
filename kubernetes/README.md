# Kubernetes Instana agent version pinning
This guide demonstrates how to control Instana agent updates by pinning agent versions in Kubernetes environments using ArgoCD and environment variables. For more information, check the official documentation on [configuring updates](https://www.ibm.com/docs/en/instana-observability/current?topic=agents-configuring-updates-dynamic-host#version-pinning).

## Pinning of the agent in operator and helm chart installation
1. Verify `kubectx` points to the right cluster
1. `kubectl` installed
1. Verify ArgoCD is installed in the cluster
    ```
    kubectl get pods -n argocd
    ```
    If it's not installed, follow the instructions at https://argo-cd.readthedocs.io/en/stable/getting_started/.
2. Construct the `instana-agent.customresource.yaml` for the operator installation, [example](instana/instana-agent-operator/dynamic-agent/instana-agent.customresource.yaml) or `values.yaml` for the helm-chart installation, [example](instana/helm-chart/dynamic-agent/values.yaml). The available environment variables for configuring the updates are `INSTANA_AGENT_UPDATES_TIME`, `INSTANA_AGENT_UPDATES_FREQUENCY`, and `INSTANA_AGENT_UPDATES_VERSION`. For the available agent, check https://github.com/instana/agent-updates/tags. See [Agent Configuration](https://www.ibm.com/docs/en/SSE1JP5_current/src/pages/setup_and_manage/host_agent/configuration/index.html#agent-configuration-file) documentation for more information about these settings.

    Once you create the file with desired configuration, commit and push the changes to the remote repository.
3. Ensure that the GitHub repo is either a public repository or that Argo CD has appropriate access to pull from it.
Access can be configured using a GitHub access token or the SSH key authentication. First, create a secret in the cluster.
    For the GitHub access token, run:
    ```
    kubectl create secret generic git-credentials --from-literal=username=<git-username> --from-literal=password=<git-token> -n argocd
    ```
    For the SSH key authentication, first generate the key. Once the SSH key is generated, determine the path of the private key (`~/.ssh/id_rsa` by default), and add the key to the Github repository settings. Then, create the `git-credentials` secret:
    ```
    kubectl create secret generic git-credentials \
        --from-file=sshPrivateKey=<path-to-your-ssh-private-key> \
        -n argocd
    ```
    Then, follow the steps to add the Git repo to ArgoCD:
    ```
    kubectl port-forward svc/argocd-server -n argocd 8080:443
    ```
    ```
    argocd login localhost:8080
    ```
    When promted to enter the credentials for logging in, enter `admin` for the username. To get the password, run:
    ```
    kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
    ```
    Once logged in, add the GitHub repo with `argocd` command.
    If you are using the GitHub access token, run:
    ```
    argocd repo add <github-repo-url> \
     --username <git-username> --password <git-password>
    ```
    Alternatively, if you are using the SSH authentication, run:
    ```
    argocd repo add git@github.com:<your-org/your-repo>.git \
     --ssh-private-key-path <private-key-path> \
     --name <your-repo-name>
    ```


1. Create an Argo CD `Application` manifest that instructs ArgoCD to monitor and apply any changes from the GitHub repository to the cluster. For the example for the operator CR, please check the [operator example](instana/instana-agent-operator/argo/argo-application.yaml) and  for the helm chart installation, please chec the [values.yaml example](instana/helm-chart/argo/argo-application.yaml)).
   * use `spec.target.revision` to specify which branch should be applied to the cluster. It's possible to have a branch per environment, e.g. `test`, `stage` and `prod`, so that each Instana agent version is tested out before using it in production.
2. If running the operator installation, run the following command. Otherwise, skip this step.
    Deploy the operator. For example, for the manual installation, run:
    ```
    kubectl apply -f https://github.com/instana/instana-agent-operator/releases/latest/download/instana-agent-operator.yaml
    ```
3. Apply the Argo CD `Application` manifest to the cluster. This is a manual one-time step.
    ```
    kubectl apply -f argo-application.yaml -n argocd

    ```
 4. Verify that everything is configured properly by updating a pinned version in CR or values.yaml and pushing it to the remote repository. Argo CD should automatically detect changes and sync them to the cluster.