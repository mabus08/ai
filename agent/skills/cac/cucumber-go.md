# Go — `godog`

## Dependencies

```go
// go.mod
require (
    github.com/cucumber/godog v0.15.0
    github.com/cucumber/godog/cucumber v0.15.0  // for expressions
)
```

## Layout

```
feature-tests/
├── world.go                       # test context, SUT, repo helpers
├── repository_discovery_test.go   # godog test suite per feature
├── browsing_commits_test.go
└── reading_a_diff_test.go
```

## `feature-tests/world.go`

```go
// Package cac_world holds the shared context for cucumber steps.
//
// Skeleton — the SUT harness and frame parser are TODOs.
package cac_world

import (
    "os"
    "path/filepath"
)

type Output struct {
    Stdout string
    Stderr string
}

type PanelSelection struct {
    Commit *int
    File   *int
}

type Sut struct {
    // TODO: PTY handle, child process, frame buffer.
}

func (s *Sut) Terminate() {
    // TODO: send SIGTERM and reap the child
}

type World struct {
    RepoPath     string
    Output       Output
    ExitCode     *int
    Sut          *Sut
    LastFrame    []string
    ActivePanel  string
    Selected     PanelSelection
}

func NewWorld() *World {
    return &World{}
}
```

## `feature-tests/repository_discovery_test.go`

```go
// Skeleton — these steps will fail until the feature is implemented.
// Do not change the .feature file to make this green. Implement the feature.
package cac

import (
    "testing"

    "github.com/cucumber/godog"

    "gitreview/feature-tests/cac_world"
)

func TestRepositoryDiscovery(t *testing.T) {
    suite := godog.TestSuite{
        ScenarioInitializer: InitializeRepositoryDiscoveryScenario,
        Options: &godog.Options{
            Format:   "pretty",
            Paths:    []string{"../feature-tests/repository_discovery.feature"},
            TestingT: t,
        },
    }
    if suite.Run() != 0 {
        t.Fatal("non-zero status returned, failed to run feature tests")
    }
}

type repositoryDiscoveryCtx struct {
    *cac_world.World
}

func InitializeRepositoryDiscoveryScenario(ctx *godog.ScenarioContext) {
    world := cac_world.NewWorld()
    rc := &repositoryDiscoveryCtx{World: world}

    ctx.Before(func(ctx context.Context, sc *godog.Scenario) (context.Context, error) {
        return ctx, nil
    })

    ctx.Step(`^the user is in a local git repository$`, rc.userInALocalRepo)
    ctx.Step(`^the user runs "gitreview"$`, rc.userRunsGitreview)
    ctx.Step(`^gitreview starts without error$`, rc.startsWithoutError)
}

func (rc *repositoryDiscoveryCtx) userInALocalRepo() error {
    // TODO: initialize a tempdir git repo with main + 3-commit feature branch
    return godog.ErrPending
}

func (rc *repositoryDiscoveryCtx) userRunsGitreview() error {
    // TODO: spawn the SUT in the tempdir via the PTY harness
    return godog.ErrPending
}

func (rc *repositoryDiscoveryCtx) startsWithoutError() error {
    // TODO: assert the SUT is running and stderr is empty
    return godog.ErrPending
}
```

## Run

```sh
go test ./feature-tests/... -v
```

## Notes

- `godog` step functions take a context and return `error`. `godog.ErrPending` signals a not-yet-implemented step (renders as `PENDING` in output). Use this for skeletons so the harness compiles and the steps are discovered.
- The step regex is a Go RE2 regex. `^...$` anchors are conventional.
- For TUI tests, the SUT harness needs a real PTY. See [tui-hints.md](tui-hints.md) for the language-agnostic approach and a worked Rust example; in Go, `creack/pty` is the standard library.
- One `Test<Feature>` Go test function per `.feature` file. Each gets its own `ScenarioContext` and step set.
