## Windows Containers

It is possible to run [Windows Containers](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/) the same way one can run [Linux containers](linux.md) on Windows Community Cluster. 
Simply use `windows_container` instead of `container` in `.cirrus.yml` files:

```yaml
windows_container:
  image: cirrusci/windowsservercore:2019
  
task:
  script: ...
```

Cirrus CI will execute [scripts instructions](writing-tasks.md#script-instruction) like **[Batch](https://en.wikipedia.org/wiki/Batch_file) scripts**.
    
## OS Versions

Cirrus CI assumes that the container image's host OS is Windows Server 2019.
Cirrus CI used to support `1709` and `1803` versions, but they are deprecated as of April 2021.

```yaml
windows_container:
  image: cirrusci/windowsservercore:2019

windows_task:
  install_script: choco install -y ...
  ...
```

### PowerShell support

By default Cirrus CI agent executed scripts using `cmd.exe`. It is possible to override default shell executor by providing
`CIRRUS_SHELL` [environment variable](writing-tasks.md#environment-variables):

```yaml
env:
  CIRRUS_SHELL: powershell
``` 

It is also possible to use *PowerShell* scripts inline inside of a script instruction by prefixing it with `ps`:

```yaml
windows_task:
  script:
    - ps: Get-Location
```

`ps: COMMAND` is just a syntactic sugar which transforms it to:

```bash
powershell.exe -NoLogo -EncodedCommand base64(COMMAND)
```

### Environment Variables

Some software installed with Chocolatey would update `PATH` environment variable in system settings and suggest using `refreshenv` to pull those changes into the current environment.
Unfortunately, using `refreshenv` will overwrite any environment variables set in Cirrus CI configuration with system-configured defaults.
We advise to make necessary changes using `env` and `environment` instead of using `refreshenv` command in scripts.

### Chocolatey

All `cirrusci/*` Windows containers like `cirrusci/windowsservercore:2016` have [Chocolatey](https://chocolatey.org/) pre-installed.
Chocolatey is a package manager for Windows which supports unattended installs of software, useful on headless machines.
