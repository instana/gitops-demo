# Sample repository for Instana Git-based configuration management

This repository provides samples of how to configure your Git or GitHub repository to make use of the Git-based Configuration Management capabilities of the Instana host agent.

## Repository structure

The Git repository you use to store configurations for the Instana host agent should have the same structure as the host agent's `<agent_installation_dir>/etc` folder.
In the large majority of cases, you will likely need to version only files in the `instana/` subfolder, although some settings like repository configurations for advanced update management scenarios will require you to modify files like `mvn-settings.xml` in the root of the repository.

Only the files you version will override existing configurations present in the agent, so feel free to version the minimal amount of configurations you need.
(At Instana we work really hard to ensure you need to configure anyhow as little ads possible :-) .)

## Git Hooks

The [.githooks/post-receive](.githooks/post-receive) file provides you a sample of how to configure your Git repository so that, when a branch is updated, it will notify the Instana backend that some Git-enabled agents should update their configurations.

## GitHub Action

The [.github/workflows/instana-gitops-update.yml](.github/workflows/instana-gitops-update.yml) file shows you how to wire the Instana GitHub action for GitOps as a GitHub Workflow, to be executed when a branch is updated.

## Further reading

* [Instana's Git-based Configuration Management documentation](https://www.instana.com/docs/setup_and_manage/host_agent/configuration/git_ops#with-the-api)
* [Instana GitHub action for GitOps](https://github.com/instana/github-action-update-agent-configurations)
* [Githooks documentation](https://git-scm.com/docs/githooks)