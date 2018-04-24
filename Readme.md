# Git-Test

an example project to test git-mirroring and redmine-integration

## gitolite

1. create a repository with write-access only to the user *gitlab-mirror-service*
2. create a user *gitlab-mirror-service* (if not exists) ssh-credentials and write-access to the repository
3. add the public key to the gitolite keydir as `gitlab-mirror-service.pub`

## gitlab

1. create the project
2. add a runner with docker-executor and network-connectivity to the gitolite-server
3. add a [secret variable] `GITLAB_MIRROR_SERVICE` with value as content of the private ssh-key of the user *gitlab-mirror-service*
4. add a [secret variable] `GITOLITE_SSH_FINGERPRINT` with the SSH-fingerprint of the gitolite-server
3. add a `.gitlab-ci.yml` with the following content:

```yaml
stages:
- mirror

gitolite_mirror:
  stage: mirror
  image:
    name: alpine/git
    entrypoint: [ "/bin/sh", "-c" ]
  variables:
    GIT_STRATEGY: clone
    GIT_CHECKOUT: "false"
  script: |
    mkdir -p ~/.ssh
    echo "${GITLAB_MIRROR_SERVICE}" > ~/.ssh/id_ecdsa
    echo "${GITOLITE_SSH_FINGERPRINT}" > ~/.ssh/known_hosts
    chmod go-rw -R ~/.ssh
    cd /tmp
    git clone --mirror ${CI_REPOSITORY_URL} project
    cd project
    git remote add gitolite git@git.ub.intern.uni-leipzig.de:git-test.git
    git push --mirror gitolite
  tags:
  - docker
```
_adjust the script-part according to your repository-name on the gitolite server_

Every push to the gitlab-repository now triggers the pipeline-job *gitolite-mirror* which mirror-pushes the entire repo to the gitolite-server.

[secret variable]: https://git.sc.uni-leipzig.de/help/ci/variables/README#secret-variables]