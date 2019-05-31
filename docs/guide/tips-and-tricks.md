## Custom Clone Command

By default Cirrus CI uses a [Git client implemented purely in Go](https://github.com/src-d/go-git) to perform a clone of
a single branch with full Git history. It is possible to control clone depth via `CIRRUS_CLONE_DEPTH` [environment variable](#behavioral-environment-variables).

Customizing clone behavior is a simple as overriding `clone_script`. For example, here an override to use a pre-installed
Git client (if your build environment has it) to do a shallow clone of a single branch:

```yaml
task:
  clone_script: |
    if [[ -z "$CIRRUS_PR" ]]; then
      git clone --recursive --branch=$CIRRUS_BRANCH https://x-access-token:${CIRRUS_REPO_CLONE_TOKEN}@github.com/${CIRRUS_REPO_FULL_NAME}.git $CIRRUS_WORKING_DIR
      git reset --hard $CIRRUS_CHANGE_IN_REPO
    else
      git clone --recursive https://x-access-token:${CIRRUS_REPO_CLONE_TOKEN}@github.com/${CIRRUS_REPO_FULL_NAME}.git $CIRRUS_WORKING_DIR
      git fetch origin pull/$CIRRUS_PR/head:pull/$CIRRUS_PR
      git reset --hard $CIRRUS_CHANGE_IN_REPO
    fi
  # ...
```

!!! note "`go-git` benefits"
    Using `go-git` made it possible to not require a pre-installed Git from an execution environment. For example, 
    most of `alpine`-based containers doesn't have Git pre-installed. Because of `go-git` you can even use distroless 
    containers with Cirrus CI which don't even have Operation System.

## Sharing configuration between tasks

You can use [YAML aliases](https://yaml.org/spec/1.2/spec.html#id2786196) to share configuration options between
multiple tasks. For example, here is a 2-task build which only runs for "master", PRs and tags, and installs some
framework:

```yaml
# Define a node anywhere in YAML file to create an alias. Make sure the name doesn't clash with an existing keyword.
regular_task_template: &REGULAR_TASK_TEMPLATE
  only_if: $CIRRUS_BRANCH == 'master' || $CIRRUS_TAG != '' || $CIRRUS_PR != ''
  env:
    FRAMEWORK_PATH: "${HOME}/framework"
  install_framework_script: curl https://example.com/framework.tar | tar -C "${FRAMEWORK_PATH}" -x

task:
  # This operator will insert REGULAR_TASK_TEMPLATE at this point in the task node.
  << : *REGULAR_TASK_TEMPLATE
  name: linux
  container:
    image: alpine:latest
  test_script: ls "${FRAMEWORK_PATH}"

task:
  << : *REGULAR_TASK_TEMPLATE
  name: osx
  osx_instance:
    image: mojave-xcode-10.2
  test_script: ls -w "${FRAMEWORK_PATH}"
```

## Long lines in configuration file

If you like your YAML file to fit on your screen, and some commands are just too long, you can split them across multiple
lines. YAML supports a [variety of options](https://yaml-multiline.info/) to do that, for example here's how you can split
ENCRYPTED values:

```yaml
  env:
    GOOGLE_APPLICATION_CREDENTIALS_DATA: "ENCRYPTED\
      [3287dbace8346dfbe98347d1954eca923487fd8ea7251983\
      cb6d5edabdf6fe5abd711238764cbd6efbde6236abd6f274]"
```
