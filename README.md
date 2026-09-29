# Task Cheatsheet

> A practical, copy-ready guide to [Task](https://taskfile.dev/)—the modern, cross-platform task runner written in Go.

[![Task](https://img.shields.io/badge/Task-v3-29BEB0?logo=task&logoColor=white)](https://taskfile.dev/)
[![Docs](https://img.shields.io/badge/docs-taskfile.dev-blue)](https://taskfile.dev/docs/)
[![License](https://img.shields.io/badge/license-MIT-green)](#license)

Task replaces long shell commands and repetitive setup instructions with readable, versioned workflows in a `Taskfile.yml`. Think of it as a friendlier, cross-platform alternative to Make.

```text
Install Task → create Taskfile.yml → list tasks → run tasks → automate everything
```

## Contents

- [1. Install](#1-install)
- [2. First workflow](#2-first-workflow)
- [3. Essential CLI commands](#3-essential-cli-commands)
- [4. Taskfile anatomy](#4-taskfile-anatomy)
- [5. Variables and environment](#5-variables-and-environment)
- [6. Dependencies and task composition](#6-dependencies-and-task-composition)
- [7. Conditions, validation, and prompts](#7-conditions-validation-and-prompts)
- [8. Avoid unnecessary work](#8-avoid-unnecessary-work)
- [9. Loops and matrices](#9-loops-and-matrices)
- [10. Includes and monorepos](#10-includes-and-monorepos)
- [11. Watch mode and development](#11-watch-mode-and-development)
- [12. Errors, cleanup, and timeouts](#12-errors-cleanup-and-timeouts)
- [13. Platforms and portability](#13-platforms-and-portability)
- [14. Output and debugging](#14-output-and-debugging)
- [15. Global and remote Taskfiles](#15-global-and-remote-taskfiles)
- [16. Configuration](#16-configuration)
- [17. CI/CD recipes](#17-cicd-recipes)
- [18. Production-ready starter](#18-production-ready-starter)
- [19. Best practices](#19-best-practices)
- [20. Troubleshooting](#20-troubleshooting)
- [21. Quick lookup](#21-quick-lookup)
- [Official references](#official-references)

---

## 1. Install

### Linux

Official repositories:

```bash
# Debian / Ubuntu
curl -1sLf 'https://dl.cloudsmith.io/public/task/task/setup.deb.sh' | sudo -E bash
sudo apt install task

# Fedora / RHEL
curl -1sLf 'https://dl.cloudsmith.io/public/task/task/setup.rpm.sh' | sudo -E bash
sudo dnf install task

# Alpine
curl -1sLf 'https://dl.cloudsmith.io/public/task/task/setup.alpine.sh' | sudo -E bash
sudo apk add task

# Arch Linux (community package)
sudo pacman -S go-task

# Snap
sudo snap install task --classic
```

Install script—useful locally and in CI:

```bash
# Install latest into ./bin
sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d

# Install latest for the current user
sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b ~/.local/bin

# Pin a version; arguments are order-sensitive
sh -c "$(curl --location https://taskfile.dev/install.sh)" -- \
  -d -b ~/.local/bin v3.53.0
```

Ensure the chosen directory is on `PATH`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

### macOS

```bash
brew install go-task/tap/go-task  # Official tap
# or
brew install go-task             # Homebrew core
```

### Windows

```powershell
winget install Task.Task
# or
choco install go-task
# or
scoop install task
```

### Cross-platform and language toolchains

```bash
npm install -g @go-task/cli
pip install go-task-bin
go install github.com/go-task/task/v3/cmd/task@latest
```

For a Go project, pin Task in `go.mod`:

```bash
go get -tool github.com/go-task/task/v3/cmd/task@latest
go tool task --version
go tool task build
```

### Verify and upgrade

```bash
task --version

# Upgrade with the package manager used to install it, for example:
brew upgrade go-task
sudo apt update && sudo apt upgrade task
```

### Shell completion

Add one command to your shell configuration:

```bash
# Bash: ~/.bashrc
eval "$(task --completion bash)"

# Zsh: ~/.zshrc
eval "$(task --completion zsh)"

# Fish: ~/.config/fish/config.fish
task --completion fish | source
```

PowerShell:

```powershell
# $PROFILE
Invoke-Expression (&task --completion powershell | Out-String)
```

See the [installation guide](https://taskfile.dev/docs/installation) for Nushell and completion-directory alternatives.

---

## 2. First workflow

Create a Taskfile:

```bash
mkdir my-project && cd my-project
task --init
```

Or create `Taskfile.yml` yourself:

```yaml
version: '3'

tasks:
  default:
    desc: Show available tasks
    cmds:
      - task --list

  hello:
    desc: Print a greeting
    vars:
      NAME: '{{.NAME | default "World"}}'
    cmds:
      - echo "Hello, {{.NAME}}!"
```

Run it:

```bash
task                 # Runs the special "default" task
task hello
task hello NAME=Ada  # Override a variable
task --list          # Discover documented tasks
```

Task searches the current directory and then parent directories for, in priority order:

```text
Taskfile.yml
taskfile.yml
Taskfile.yaml
taskfile.yaml
Taskfile.dist.yml
taskfile.dist.yml
Taskfile.dist.yaml
taskfile.dist.yaml
```

Use a committed `Taskfile.dist.yml` plus a gitignored local `Taskfile.yml` when developers need personal overrides.

---

## 3. Essential CLI commands

General form:

```text
task [options] [tasks...] [-- forwarded arguments...]
```

### Discover and inspect

```bash
task                          # Run the default task
task build                    # Run one task
task test lint                # Run tasks sequentially
task --parallel test lint     # Run tasks concurrently
task --list              # -l: tasks that have descriptions
task --list-all          # -a: all public tasks
task --list --silent     # -ls: names only, useful in scripts
task --list --json            # Machine-readable task metadata
task build --summary          # Description, commands, vars, requirements
task --help
task --version
```

### Control execution

```bash
task deploy --dry             # -n: print without executing
task build --force            # -f: ignore up-to-date checks
task test --failfast          # -F: stop scheduling after a failure
task test --concurrency 4     # -C: cap concurrent task count
task test --exit-code         # -x: return the failed command's exit code
task build --watch            # -w: rerun after watched files change
task build --status           # Exit 0 only when task is up to date
task deploy --yes             # -y: accept Task confirmation prompts
task deploy --interactive     # Prompt for missing required variables
```

### Choose location or Taskfile

```bash
task -d ./backend build
task --dir ./backend build
task -t ./ops/Taskfile.yml deploy
task --taskfile ./ops/Taskfile.yml deploy
task -g backup                # Use $HOME/Taskfile.yml
task -t - < Taskfile.yml      # Read Taskfile from stdin
```

### Output modes

```bash
task test -s                         # --silent: don't echo commands
task test -v                         # --verbose: detailed diagnostics
task test -o interleaved             # Default concurrent output
task test -o prefixed                # Prefix each line with task name
task test -o group                   # Keep each task's output together
task test --color=false
NO_COLOR=1 task test
```

### Pass arguments correctly

Use variables for structured inputs:

```bash
task deploy ENV=staging VERSION=1.4.0
```

Use `--` for raw arguments. Everything after it becomes `.CLI_ARGS`; newer Task versions also expose `.CLI_ARGS_LIST`.

```yaml
tasks:
  test:
    cmds:
      - go test ./... {{.CLI_ARGS}}

  show-args:
    cmds:
      - for:
          var: CLI_ARGS_LIST
        cmd: echo "{{.ITEM}}"
```

```bash
task test -- -run TestLogin -count=1
task show-args -- one "two words"
```

---

## 4. Taskfile anatomy

```yaml
version: '3'                 # Required schema/minimum Task version

output: prefixed             # interleaved | group | prefixed
silent: false
method: checksum             # checksum | timestamp | none
use_gitignore: true
set: [errexit, nounset, pipefail]
shopt: [globstar]

dotenv:
  - .env.local               # First file has highest priority
  - .env

vars:
  APP: example

env:
  APP_ENV: development

includes:
  api:
    taskfile: ./api/Taskfile.yml
    dir: ./api

tasks:
  build:
    desc: Build the application
    summary: |
      Compiles the application into ./dist.
      Override VERSION with: task build VERSION=1.2.3
    aliases: [b]
    dir: .
    vars:
      VERSION: '{{.VERSION | default "dev"}}'
    env:
      CGO_ENABLED: '0'
    sources: ['src/**/*']
    generates: ['dist/app']
    preconditions:
      - sh: command -v go
        msg: Go is required
    cmds:
      - go build -o dist/app ./cmd/app
```

Useful task fields:

- `desc` — short text shown by `task --list`.
- `summary` — detailed help shown by `task TASK --summary`.
- `aliases` — shorter names such as `b` for `build`.
- `cmd` / `cmds` — one command / ordered command list.
- `deps` — prerequisite tasks, run concurrently.
- `dir` — task working directory; created if missing.
- `vars`, `env`, `dotenv` — inputs and environment.
- `sources`, `generates`, `status` — up-to-date logic.
- `preconditions`, `requires`, `if` — validation and conditions.
- `prompt` — confirmation before execution.
- `internal` — callable by tasks but hidden from users.
- `platforms` — OS/architecture filters.
- `run` — `always`, `once`, or `when_changed`.
- `watch` — always launch that task in watch mode.
- `silent`, `ignore_error`, `timeout` — execution behavior.

YAML tips:

```yaml
tasks:
  short: echo "One command"

  also-short:
    - echo "First"
    - echo "Second"

  multiline:
    cmds:
      - |
        echo "Commands in one block share one shell"
        cd src
        pwd
```

Each separate `cmds` item runs in a separate shell. Use `dir`, or put dependent shell lines in the same `|` block—do not expect `cd` in one item to affect the next.

---

## 5. Variables and environment

### Define and override variables

```yaml
version: '3'

vars:
  APP: myapp
  PORT: 8080
  FEATURES: [auth, metrics]
  CONFIG:
    map:
      region: eu-west-1
      replicas: 3

tasks:
  info:
    vars:
      MODE: '{{.MODE | default "development"}}'
    cmds:
      - echo "{{.APP}} runs on {{.PORT}} in {{.MODE}}"
      - echo "First feature is {{index .FEATURES 0}}"
      - echo "Region is {{.CONFIG.region}}"
```

```bash
task info
task info MODE=production
MODE=production task info
```

Practical precedence, highest first:

1. Variables declared directly on the task.
2. Variables passed by another task call.
3. Included-Taskfile variables.
4. Variables passed by the include.
5. Root `vars`.
6. Environment variables.

To make a Taskfile value overridable, explicitly use `default`:

```yaml
vars:
  REGION: '{{.REGION | default "us-east-1"}}'
```

### Dynamic and typed variables

```yaml
vars:
  GIT_SHA:
    sh: git rev-parse --short HEAD
  FILES: [a.txt, b.txt]
  FILES_COPY:
    ref: .FILES
```

- `sh` captures command output and trims its final newline.
- `ref` preserves arrays, maps, numbers, and booleans.
- Normal `{{.VALUE}}` interpolation always produces text.

### Environment variables and dotenv

```yaml
dotenv:
  - .env.local
  - .env.{{.ENV | default "development"}}
  - .env

env:
  LOG_LEVEL: info

tasks:
  serve:
    dotenv: [.env.serve]
    env:
      PORT: '{{.PORT | default "3000"}}'
    cmds:
      - ./server
```

For duplicate dotenv keys, the first file wins. Explicit task `env` values override dotenv values. Do not commit secret `.env` files.

### Secret values

Task v3.52+ can mask secret variables in logs:

```yaml
vars:
  API_TOKEN:
    sh: secret-tool read api-token
    secret: true
```

Masking reduces accidental output, but it is not a secret store. Avoid placing secrets directly in committed Taskfiles or command-line arguments; prefer CI secrets, environment injection, or a secrets manager.

### Useful built-in variables

```yaml
tasks:
  paths:
    cmds:
      - echo "Task={{.TASK}}"
      - echo "Alias used={{.ALIAS}}"
      - echo "Task directory={{.TASK_DIR}}"
      - echo "Caller directory={{.USER_WORKING_DIR}}"
      - echo "Root directory={{.ROOT_DIR}}"
      - echo "Taskfile={{.TASKFILE}}"
      - echo "Root Taskfile={{.ROOT_TASKFILE}}"
      - echo "OS={{OS}} ARCH={{ARCH}}"
      - echo "Task version={{.TASK_VERSION}}"
```

Also useful: `.CLI_ARGS`, `.CLI_ARGS_LIST`, `.CLI_FORCE`, `.CLI_ASSUME_YES`, `.CHECKSUM`, `.TIMESTAMP`, and `.EXIT_CODE` inside deferred commands.

Task templates use Go `text/template` plus slim-sprig helpers:

```yaml
vars:
  TAG: '{{.TAG | default "latest" | lower}}'
  CLEAN_NAME: '{{.APP | trim | replace " " "-"}}'
```

See the [templating reference](https://taskfile.dev/docs/reference/templating) for all functions.

---

## 6. Dependencies and task composition

### Parallel prerequisites with `deps`

```yaml
tasks:
  verify:
    deps: [lint, test, security]

  lint:
    cmds: [golangci-lint run]

  test:
    cmds: [go test ./...]

  security:
    cmds: [govulncheck ./...]
```

Dependencies run concurrently. Use them only when tasks do not depend on each other's output.

```bash
task verify
task verify --concurrency 2
task verify --failfast
```

### Sequential task calls with `cmds`

```yaml
tasks:
  release:
    cmds:
      - task: test
      - task: build
        vars:
          MODE: release
      - task: publish
```

The list executes in order and stops on failure.

### Limit duplicate runs

```yaml
tasks:
  install:
    run: once
    cmds:
      - npm ci

  compile:
    run: when_changed
    vars:
      TARGET: '{{.TARGET}}'
    cmds:
      - echo "Compiling {{.TARGET}}"
```

- `always` — default; every call attempts to run.
- `once` — run once per Task invocation.
- `when_changed` — run once for each distinct variable set.

### Internal helper tasks

```yaml
tasks:
  deploy:
    cmds:
      - task: _upload
        vars: {ENV: production}

  _upload:
    internal: true
    cmds:
      - ./scripts/upload "{{.ENV}}"
```

Internal tasks do not appear in lists and cannot be invoked directly.

---

## 7. Conditions, validation, and prompts

Choose the behavior that matches your intent:

- `if` — condition false: skip successfully.
- `status` — checks succeed: task is already up to date.
- `preconditions` — check fails: stop with an error.
- `requires` — variable missing/invalid: stop before running.
- `prompt` — ask for confirmation; bypass with `--yes`.

### Optional execution with `if`

```yaml
tasks:
  upload-coverage:
    if: '[ "$CI" = "true" ]'
    cmds:
      - ./scripts/upload-coverage

  build:
    cmds:
      - cmd: echo "Production optimization"
        if: '{{eq .ENV "production"}}'
      - go build ./...
```

### Preconditions

```yaml
tasks:
  deploy:
    preconditions:
      - sh: command -v kubectl
        msg: Install kubectl before deploying
      - sh: test -f .env
        msg: Copy .env.example to .env and configure it
      - sh: test -n "$KUBE_CONTEXT"
        msg: KUBE_CONTEXT is required
    cmds:
      - kubectl --context "$KUBE_CONTEXT" apply -f deploy/
```

`--force` bypasses both up-to-date checks and failed preconditions, so do not treat a precondition as an unbreakable security boundary.

### Required and allowed variables

```yaml
tasks:
  deploy:
    requires:
      vars:
        - VERSION
        - name: ENV
          enum: [development, staging, production]
    cmds:
      - ./deploy --env "{{.ENV}}" --version "{{.VERSION}}"
```

```bash
task deploy ENV=staging VERSION=1.2.0
task deploy --interactive
```

Interactive mode can ask for missing values in a terminal. In CI, missing values fail instead of prompting.

### Destructive-operation prompts

```yaml
tasks:
  db:reset:
    prompt:
      - Reset the local database?
      - All local data will be deleted. Continue?
    cmds:
      - docker compose down --volumes
      - docker compose up -d db
```

Use `task db:reset --yes` only when intentional.

---

## 8. Avoid unnecessary work

### Fingerprint source and output files

```yaml
tasks:
  build:
    desc: Build only when inputs change
    sources:
      - 'cmd/**/*.go'
      - 'internal/**/*.go'
      - go.mod
      - go.sum
      - exclude: '**/*_test.go'
    generates:
      - dist/app
    cmds:
      - mkdir -p dist
      - go build -o dist/app ./cmd/app
```

Task defaults to content checksums. Cache metadata normally lives in `.task/`, so add this:

```gitignore
.task/
```

Available methods:

```yaml
method: checksum   # Content-based; reliable default
# method: timestamp  # Faster modification-time comparison
# method: none       # Disable fingerprint checks
```

Set `use_gitignore: true` to keep ignored build artifacts and caches out of source/glob fingerprinting.

### Programmatic status

If every `status` command exits successfully, Task skips the task:

```yaml
tasks:
  tools:install:
    status:
      - command -v golangci-lint
      - command -v govulncheck
    cmds:
      - ./scripts/install-tools
```

Check without changing anything:

```bash
task build --status
echo $?  # 0 = current; non-zero = needs work
```

Force a rerun:

```bash
task build --force
```

For variable-dependent outputs, add the variable to `label` so Task stores distinct fingerprints:

```yaml
tasks:
  build:
    label: 'build-{{.TARGET}}'
    sources: ['src/**/*']
    generates: ['dist/{{.TARGET}}/app']
    cmds:
      - ./build "{{.TARGET}}"
```

---

## 9. Loops and matrices

### Static list

```yaml
tasks:
  greet:
    cmds:
      - for: [Ada, Grace, Linus]
        cmd: echo "Hello {{.ITEM}}"
```

### Variable list with a custom item name

```yaml
tasks:
  lint:packages:
    vars:
      PACKAGES: [api, web, worker]
    cmds:
      - for:
          var: PACKAGES
          as: PACKAGE
        cmd: npm --prefix "packages/{{.PACKAGE}}" run lint
```

For a delimited string:

```yaml
vars:
  TARGETS: linux/amd64,linux/arm64,darwin/arm64

tasks:
  print-targets:
    cmds:
      - for:
          var: TARGETS
          split: ','
        cmd: echo "{{.ITEM}}"
```

### Source files

```yaml
tasks:
  format:
    sources: ['src/**/*.json']
    cmds:
      - for: sources
        cmd: jq . "{{.ITEM}}" > "{{.ITEM}}.tmp"
```

### Build matrix

```yaml
tasks:
  matrix:
    cmds:
      - for:
          matrix:
            OS: [linux, darwin, windows]
            ARCH: [amd64, arm64]
        cmd: echo "{{.ITEM.OS}}/{{.ITEM.ARCH}}"
```

### Loop over task calls

```yaml
tasks:
  test:all:
    deps:
      - for: [unit, integration, e2e]
        task: test:one
        vars:
          SUITE: '{{.ITEM}}'

  test:one:
    cmds:
      - ./test --suite "{{.SUITE}}"
```

---

## 10. Includes and monorepos

Split large workflows by domain:

```text
.
├── Taskfile.yml
├── taskfiles/
│   ├── Docker.yml
│   └── Release.yml
├── api/
│   └── Taskfile.yml
└── web/
    └── Taskfile.yml
```

Root `Taskfile.yml`:

```yaml
version: '3'

includes:
  api:
    taskfile: ./api/Taskfile.yml
    dir: ./api
    aliases: [a]

  web:
    taskfile: ./web/Taskfile.yml
    dir: ./web

  docker:
    taskfile: ./taskfiles/Docker.yml
    vars:
      REGISTRY: ghcr.io/example

  release:
    taskfile: ./taskfiles/Release.yml
    optional: true

  shared:
    taskfile: ./taskfiles/Shared.yml
    flatten: true
    excludes: [dangerous, 'debug:*']
```

```bash
task api:test
task a:test       # Namespace alias
task web:build
```

Key include controls:

- `dir` — working directory for included tasks.
- `vars` — configure a reusable Taskfile.
- `aliases` — alternate namespace names.
- `optional` — tolerate a missing include.
- `internal` — make all included tasks internal.
- `flatten` — expose tasks without a namespace.
- `excludes` — omit exact tasks or `namespace:*`.
- `checksum` — pin and verify included file contents.

From an included Taskfile, call a root task with a leading colon:

```yaml
cmds:
  - task: :root-task
```

Task also walks upward from subdirectories. Use `.USER_WORKING_DIR` when a root task should operate where the user invoked it:

```yaml
tasks:
  compose:up:
    dir: '{{.USER_WORKING_DIR}}'
    preconditions:
      - sh: test -f compose.yml
        msg: No compose.yml in the current directory
    cmds:
      - docker compose up -d
```

---

## 11. Watch mode and development

```yaml
tasks:
  dev:
    desc: Rebuild when source files change
    watch: true
    sources:
      - 'src/**/*'
      - package.json
    cmds:
      - npm run build
```

```bash
task dev
task build --watch
task build --watch --interval 500ms
```

`interval` is a Go duration such as `100ms`, `2s`, or `1m`. Current Task versions use filesystem notifications; the interval debounces repeated events.

Watch mode reruns the task when a matching source changes. Long-running development servers often have their own watcher; call that directly instead of wrapping it in Task's watcher.

---

## 12. Errors, cleanup, and timeouts

### Always clean up with `defer`

```yaml
tasks:
  integration:
    cmds:
      - docker compose up -d
      - defer: docker compose down --volumes
      - go test -tags=integration ./...
```

Deferred commands run when the task finishes, even after failure. Multiple defers run last-in, first-out.

Use the task exit code during cleanup:

```yaml
tasks:
  work:
    cmds:
      - defer: echo "Task exited with {{.EXIT_CODE}}"
      - ./possibly-failing-command
```

### Ignore only expected failures

```yaml
tasks:
  cleanup:
    cmds:
      - cmd: docker rm -f temporary-container
        ignore_error: true
      - echo "Continuing"
```

Prefer idempotent commands (`rm -f`, existence tests) over broad `ignore_error`.

### Bound hanging commands

Task v3.53+ supports per-command and dependency timeouts:

```yaml
tasks:
  deploy:
    deps:
      - task: fetch-assets
        timeout: 2m
    cmds:
      - cmd: npm run build
        timeout: 5m
      - cmd: ./deploy.sh
        timeout: 30m
      - defer: ./cleanup.sh
        timeout: 30s
```

Timeouts use Go durations and report exit code `124`.

### Shell safety

```yaml
set: [errexit, nounset, pipefail]
shopt: [globstar, nullglob]
```

Supported `set` options include `errexit`, `nounset`, `pipefail`, `xtrace`, `noexec`, `noglob`, and `allexport`. Use `set` globally, per task, or per command.

---

## 13. Platforms and portability

Restrict an entire task:

```yaml
tasks:
  build:windows:
    platforms: [windows/amd64]
    cmds:
      - go build -o dist/app.exe ./cmd/app

  build:unix:
    platforms: [linux, darwin]
    cmds:
      - go build -o dist/app ./cmd/app
```

Or restrict individual commands:

```yaml
tasks:
  open:
    cmds:
      - cmd: open http://localhost:3000
        platforms: [darwin]
      - cmd: xdg-open http://localhost:3000
        platforms: [linux]
      - cmd: start http://localhost:3000
        platforms: [windows]
```

Values use Go's `GOOS`, `GOARCH`, or `GOOS/GOARCH` names. Nonmatching commands are skipped, not failed.

Task includes common core utilities on recent Windows versions, but portable workflows should still:

- Prefer tools already required by the project.
- Quote paths and variable expansions.
- Use Task's `dir` instead of shell-specific `cd`.
- Use `platforms` when commands genuinely differ.
- Test on every supported OS in CI.

---

## 14. Output and debugging

```bash
task build --dry       # Inspect rendered commands
task build --summary   # Inspect task metadata and resolved structure
task build --verbose   # Trace Task's decisions
task --list-all        # Find tasks without descriptions
task --list --json | jq
```

For concurrent workflows:

```yaml
output: prefixed

# Or group output per task:
# output:
#   group:
#     begin: '▶ {{.TASK}}'
#     end: '✓ {{.TASK}}'
#     error_only: false
```

Per-task customization:

```yaml
tasks:
  quiet:
    silent: true
    cmds:
      - echo "Only command output is shown"

  labeled:
    label: 'test-{{.PACKAGE}}'
    prefix: '{{.PACKAGE}}'
    cmds:
      - go test "./{{.PACKAGE}}/..."
```

When debugging templates, quote the entire YAML command if it contains `:` or other YAML-significant characters:

```yaml
cmds:
  - 'echo "Image: {{.IMAGE}}"'
```

---

## 15. Global and remote Taskfiles

### Personal global tasks

Create `$HOME/Taskfile.yml`:

```yaml
version: '3'

tasks:
  update:
    desc: Update common development tools
    cmds:
      - echo "Update tools here"

  files:
    desc: List files in the directory where Task was invoked
    dir: '{{.USER_WORKING_DIR}}'
    cmds:
      - ls
```

Run from anywhere:

```bash
task -g update
task --global files
```

Global tasks run in `$HOME` unless `dir: '{{.USER_WORKING_DIR}}'` is set.

### Remote Taskfiles

```bash
# HTTPS
task -t https://example.com/Taskfile.yml hello

# Git repository, subpath, and pinned ref
task -t 'https://github.com/org/repo.git//automation/Taskfile.yml?ref=v1.2.0' build

# Use cache only
task -t https://example.com/Taskfile.yml --offline hello

# Refresh cache
task -t https://example.com/Taskfile.yml --download hello
```

Remote controls include:

```text
--offline             Never fetch from the network
--download            Ignore cache and fetch again
--expiry 24h          Set cache lifetime
--clear-cache         Clear remote files and checksums
--remote-cache-dir    Choose cache location
--trusted-hosts       Skip prompts for named trusted hosts
--timeout             Bound remote operations
--cacert              Custom CA certificate
--cert / --cert-key   Client mTLS credentials
--insecure            Allow insecure fetching (avoid when possible)
```

Security rules:

1. A remote Taskfile is executable code—read it before running it.
2. Pin a Git tag/commit or an include `checksum`; do not trust a moving branch for CI.
3. Limit trusted hosts to infrastructure you control.
4. Never pass credentials in a committed URL.

See [Remote Taskfiles](https://taskfile.dev/docs/experiments/remote-taskfiles) for cache, trust, Git, SSH, and TLS behavior.

---

## 16. Configuration

Task configuration priority is:

```text
configuration file < TASK_* environment variables < CLI flags
```

Task searches for `.taskrc.yml` up the directory tree and supports global configuration at `$XDG_CONFIG_HOME/task/taskrc.yml` or `$HOME/.taskrc.yml`.

Example `.taskrc.yml`:

```yaml
verbose: false
silent: false
color: true
concurrency: 4
failfast: false
interactive: true
disable-fuzzy: false
temp-dir: .task

remote:
  offline: false
  timeout: 30s
  expiry: 24h
  trusted-hosts:
    - raw.githubusercontent.com
```

Common environment equivalents:

```bash
TASK_VERBOSE=1 task build
TASK_SILENT=1 task build
TASK_CONCURRENCY=4 task verify
TASK_FAILFAST=1 task verify
TASK_INTERACTIVE=1 task deploy
TASK_TEMP_DIR=.task task build
TASK_OUTPUT=group task verify
NO_COLOR=1 task build
```

Use project config for shared behavior and user config for personal preferences. Avoid requiring a personal config for builds to succeed.

---

## 17. CI/CD recipes

### GitHub Actions

```yaml
name: CI

on:
  push:
  pull_request:

jobs:
  verify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install Task
        uses: go-task/setup-task@v1
        with:
          version: '3.53.0'

      - name: Verify
        run: task ci
```

### Portable install script

```yaml
- name: Install Task
  run: |
    sh -c "$(curl --location https://taskfile.dev/install.sh)" -- \
      -d -b "$HOME/.local/bin" v3.53.0
    echo "$HOME/.local/bin" >> "$GITHUB_PATH"
```

### CI-oriented Taskfile

```yaml
version: '3'

output:
  group:
    begin: '::group::{{.TASK}}'
    end: '::endgroup::'
    error_only: false

tasks:
  ci:
    desc: Run all CI checks
    deps: [lint, test, build]

  lint:
    cmds:
      - golangci-lint run

  test:
    cmds:
      - go test -race -coverprofile=coverage.out ./...

  build:
    sources: ['**/*.go', go.mod, go.sum]
    generates: [dist/app]
    cmds:
      - mkdir -p dist
      - go build -trimpath -o dist/app ./cmd/app
```

CI guidance:

- Pin Task's version for reproducible pipelines.
- Keep CI thin: CI calls `task ci`; the Taskfile owns workflow logic.
- Provide every required variable explicitly—CI never answers interactive prompts.
- Use command timeouts for network and deployment operations.
- Use `--exit-code` only when the underlying tool's exact exit code matters.
- Cache language dependencies separately from Task's `.task` metadata.

---

## 18. Production-ready starter

Copy this and adapt the actual tool commands:

```yaml
version: '3'

silent: false
output: prefixed
use_gitignore: true
set: [errexit, nounset, pipefail]

dotenv:
  - .env.local
  - .env

vars:
  APP: '{{.APP | default "myapp"}}'
  ENV: '{{.ENV | default "development"}}'
  VERSION:
    sh: git describe --tags --always --dirty 2>/dev/null || echo dev

tasks:
  default:
    desc: Show available tasks
    cmds:
      - task --list

  setup:
    desc: Install project dependencies
    sources: [package.json, package-lock.json]
    generates: [node_modules/.package-lock.json]
    cmds:
      - npm ci

  dev:
    desc: Start the development server
    deps: [setup]
    cmds:
      - npm run dev

  format:
    desc: Format source files
    cmds:
      - npm run format

  lint:
    desc: Run static analysis
    deps: [setup]
    cmds:
      - npm run lint

  test:
    desc: Run tests; pass extra arguments after --
    deps: [setup]
    cmds:
      - npm test -- {{.CLI_ARGS}}

  build:
    desc: Build production artifacts
    deps: [setup]
    sources:
      - 'src/**/*'
      - package.json
      - package-lock.json
    generates: [dist/**/*]
    env:
      NODE_ENV: production
    cmds:
      - npm run build

  verify:
    desc: Run all checks
    deps: [format:check, lint, test, build]

  format:check:
    internal: true
    deps: [setup]
    cmds:
      - npm run format:check

  clean:
    desc: Remove generated files
    prompt: Remove all generated files?
    cmds:
      - rm -rf dist coverage .task

  deploy:
    desc: Deploy a version to an environment
    summary: |
      Required:
        ENV=staging|production
        VERSION=release identifier

      Example:
        task deploy ENV=staging VERSION=v1.2.0
    requires:
      vars:
        - VERSION
        - name: ENV
          enum: [staging, production]
    prompt: 'Deploy {{.VERSION}} to {{.ENV}}?'
    preconditions:
      - sh: command -v deployctl
        msg: Install deployctl first
    cmds:
      - defer: ./scripts/deploy-cleanup.sh
        timeout: 1m
      - cmd: ./scripts/deploy.sh "{{.ENV}}" "{{.VERSION}}"
        timeout: 30m
```

Suggested repository files:

```text
Taskfile.yml            Shared workflow; commit it
.taskrc.yml             Optional shared Task settings
.env.example            Document expected environment variables
.env                    Local values; gitignore it
.task/                  Task fingerprints; usually gitignore it
taskfiles/              Split domain-specific tasks here
scripts/                Put complex logic in testable scripts
```

---

## 19. Best practices

### Design a discoverable interface

- Make `default` run `task --list`.
- Add `desc` to every user-facing task.
- Add `summary` to risky or parameter-heavy tasks.
- Use namespaces such as `db:migrate`, `docker:build`, and `release:publish`.
- Keep names verb-oriented and predictable: `build`, `test`, `lint`, `clean`.

### Keep behavior reliable

- Make tasks idempotent whenever possible.
- Use `deps` for independent parallel work; task calls for ordered work.
- Use `sources`/`generates` only when they accurately describe all inputs and outputs.
- Validate required tools and files with helpful `preconditions`.
- Validate inputs with `requires` and enums.
- Add `prompt` to destructive or production actions.
- Use `defer` to release containers, locks, ports, and temporary files.
- Pin tool and remote Taskfile versions in CI.

### Keep Taskfiles maintainable

- Put orchestration in Task; put complex business logic in scripts.
- Prefer short, readable commands over deeply nested templates.
- Quote interpolated values passed to a shell.
- Mark implementation details `internal`.
- Split large files with namespaced `includes`.
- Test with `task --dry`, then run in CI on supported platforms.

### Security checklist

- Never commit credentials.
- Treat remote Taskfiles and included scripts as executable dependencies.
- Pin remote references and verify checksums.
- Remember that masked secret variables can still leak through child programs.
- Avoid `--insecure` and overly broad `--trusted-hosts`.
- Do not rely on prompts or preconditions as authorization controls.

---

## 20. Troubleshooting

### `task: command not found`

```bash
task --version
command -v task
echo "$PATH"
```

Add the installation directory—often `$HOME/.local/bin` or `$HOME/go/bin`—to `PATH`, then restart the shell.

### `No Taskfile found` (exit 100)

```bash
pwd
ls
task -t /absolute/path/Taskfile.yml --list
```

Check the supported filenames and directory ownership boundaries.

### YAML parse failure (exit 102)

- Use spaces, never tabs.
- Quote commands containing `:`, `{}`, `#`, or leading special characters.
- Validate YAML in your editor.
- Add the schema directive below for completion and validation.

```yaml
# yaml-language-server: $schema=https://taskfile.dev/schema.json
version: '3'
```

### Variable is empty

```bash
task my-task --summary
task my-task --verbose
task my-task VALUE=test --dry
```

Remember that task-level variables outrank CLI/environment values unless they use `default`.

### A command cannot see a previous `cd` or `export`

Each command item gets its own shell. Use `dir`, `env`, or one multiline command:

```yaml
tasks:
  example:
    dir: ./subdir
    env:
      MODE: test
    cmds:
      - |
        export EXTRA=value
        ./first
        ./second
```

### Task says “up to date” unexpectedly

```bash
task build --status
task build --verbose
task build --force
```

Then check `sources`, `generates`, exclusions, `status`, `method`, `label`, and `.gitignore` interaction.

### Parallel output is unreadable

```bash
task verify --output prefixed
task verify --output group
task verify --concurrency 2
```

### Exit-code map

- `0` — success.
- `1` — unknown/general error.
- `100` — no Taskfile.
- `101` — Taskfile already exists during `--init`.
- `102` — invalid/unparseable Taskfile.
- `103–106` — remote download/trust/security/cache errors.
- `107` — Taskfile has no schema version.
- `200` — task not found.
- `201` — command failed.
- `202` — direct call to an internal task.
- `203` — duplicate task or alias.
- `204` — recursion limit.
- `205` — cancelled.
- `206` — required variable missing.
- `207` — variable value invalid.

Use `--exit-code` to pass through the failed command's own exit code.

---

## 21. Quick lookup

### Commands

```bash
task                          # Default task
task NAME                     # Run task
task A B                      # Run A then B
task -p A B                   # Run A and B in parallel
task -l / task -a             # List documented / all tasks
task NAME --summary           # Detailed task help
task NAME VAR=value           # Pass variable
task NAME -- args             # Forward raw arguments
task NAME -n / -v / -s        # Dry / verbose / silent
task NAME -f                  # Force
task NAME -w                  # Watch
task NAME --status            # Check freshness
task -d DIR NAME              # Run from directory
task -t FILE NAME             # Use another Taskfile
task -g NAME                  # Use global Taskfile
```

### Execution decision guide

```text
Must happen before, and independent?       deps
Must happen in strict order?               cmds with task calls
Run only when a condition is true?         if
Fail unless a condition is true?           preconditions
Skip when work already exists?             status
Skip when source/output files unchanged?   sources + generates
Require or constrain an input?              requires
Ask before dangerous work?                 prompt
Always clean up?                           defer
Run once per invocation?                   run: once
Run once per distinct variable set?        run: when_changed
Hide an implementation helper?             internal: true
```

### Task skeleton

```yaml
version: '3'

tasks:
  name:
    desc: What this task does
    summary: More detailed usage
    aliases: [n]
    dir: .
    deps: [other]
    vars:
      VALUE: '{{.VALUE | default "fallback"}}'
    env:
      MODE: development
    requires:
      vars: [VALUE]
    preconditions:
      - sh: command -v required-tool
        msg: Install required-tool
    sources: ['src/**/*']
    generates: ['dist/output']
    cmds:
      - command --value "{{.VALUE}}"
```

---

## Official references

- [Task home](https://taskfile.dev/)
- [Installation](https://taskfile.dev/docs/installation)
- [Getting started](https://taskfile.dev/docs/getting-started)
- [Guide](https://taskfile.dev/docs/guide)
- [CLI reference](https://taskfile.dev/docs/reference/cli)
- [Taskfile schema](https://taskfile.dev/docs/reference/schema)
- [Templating reference](https://taskfile.dev/docs/reference/templating)
- [Taskfile versions](https://taskfile.dev/docs/taskfile-versions)
- [Configuration](https://taskfile.dev/docs/configuration)
- [Remote Taskfiles](https://taskfile.dev/docs/experiments/remote-taskfiles)
- [JSON schema](https://taskfile.dev/schema.json)
- [GitHub repository](https://github.com/go-task/task)
- [Release notes](https://taskfile.dev/docs/changelog)

This guide targets Task v3 and includes features available through v3.53. If compatibility with older Task v3 releases matters, set `version` to the minimum SemVer your Taskfile needs instead of only `'3'`.

## License

You may reuse and adapt this cheatsheet under the [MIT License](LICENSE).
