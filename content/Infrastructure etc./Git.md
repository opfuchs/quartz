
# Git Credential Manager

https://github.com/git-ecosystem/git-credential-manager

NOTE: Only works with GitHub and Azure Repos

**Basic Usage tl;dr**

1. Set a secret store (e.g., the freedesktop Secret Service) via either the `GCM_CREDENTIAL_STORE` environment variable (`export GCM_CREDENTIAL_STORE=<store>)` or git configuration settings (`git config --global credential.credentialStore <store>`). On Linux, `secretservice` is the best default choice in most circumstances.
2. Run `git-credential-manager <provider> login`. This can be either `github` or `azure-repos`

That's it. Most of the complexity in the official documentation is good to know especially for more serious dev projects, but the above is all that's needed to get going.