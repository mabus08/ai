# Python — `pytest-bdd` or `behave`

Both are valid. `pytest-bdd` integrates with pytest (recommended for projects that already use pytest). `behave` is closer to "pure cucumber".

## `pytest-bdd`

### Dependencies

```toml
# pyproject.toml
[project.optional-dependencies]
test = ["pytest>=8", "pytest-bdd>=7"]
```

### Layout

```
feature-tests/
├── __init__.py
├── conftest.py                    # shared fixtures (world, sut, ...)
├── world.py
├── repository_discovery.feature
├── browsing_commits.feature
└── reading_a_diff.feature

tests/
├── __init__.py
├── test_repository_discovery.py   # one test module per feature
├── test_browsing_commits.py
└── test_reading_a_diff.py
```

### `feature-tests/conftest.py`

```python
import pytest
from feature_tests.world import World, Sut


@pytest.fixture
def world() -> World:
    return World()


@pytest.fixture
def sut():
    s = Sut()
    yield s
    s.terminate()
```

### `feature-tests/world.py`

```python
"""Shared context for pytest-bdd steps.

Skeleton — the SUT harness and frame parser are TODOs.
"""
from dataclasses import dataclass, field
from pathlib import Path
from typing import Optional


@dataclass
class Sut:
    # TODO: PTY handle, child process, frame buffer.
    pass

    def terminate(self):
        # TODO: send SIGTERM and reap the child
        pass


@dataclass
class World:
    repo_path: Optional[Path] = None
    stdout: str = ""
    stderr: str = ""
    exit_code: Optional[int] = None
    sut: Optional[Sut] = None
    last_frame: list[str] = field(default_factory=list)
    active_panel: Optional[str] = None
    selected_commit: Optional[int] = None
    selected_file: Optional[int] = None
```

### `tests/test_repository_discovery.py`

```python
# Skeleton — these steps will fail until the feature is implemented.
# Do not change the .feature file to make this green. Implement the feature.

from pathlib import Path

from pytest_bdd import given, scenario, then, when

from feature_tests.world import World


@scenario(
    "../feature-tests/repository_discovery.feature",
    "Open a repository in the current working directory",
)
def test_open_repo_in_cwd():
    pass  # body is provided by the @given/@when/@then decorators below


@given("the user is in a local git repository")
def user_in_a_local_repo(world: World):
    # TODO: initialize a tempdir git repo with main + 3-commit feature branch
    raise NotImplementedError("see CAC: Open a repository in the current working directory")


@when('the user runs "gitreview"')
def user_runs_gitreview(world: World):
    # TODO: spawn the SUT in the tempdir via the PTY harness
    raise NotImplementedError


@then("gitreview starts without error")
def starts_without_error(world: World):
    # TODO: assert the SUT is running and stderr is empty
    raise NotImplementedError
```

### Run

```sh
pytest tests/
```

---

## `behave`

If the project prefers `behave`, the layout is:

```
features/
├── environment.py                 # shared hooks (before/after scenario)
├── repository_discovery.feature
├── browsing_commits.feature
├── reading_a_diff.feature
└── steps/
    ├── __init__.py
    ├── world.py
    ├── repository_discovery_steps.py
    ├── browsing_commits_steps.py
    └── reading_a_diff_steps.py
```

Steps look like:

```python
from behave import given, when, then
from features.steps.world import World

@given("the user is in a local git repository")
def step_impl(context: World):
    # TODO
    raise NotImplementedError
```

Run with `behave features/`.

## Notes

- `pytest-bdd` step decorators (`@given`, `@when`, `@then`) must be paired with `@scenario(...)` per scenario. The decorator is empty but pytest-bdd uses it to register the test.
- `behave` matches step text to step definitions by regex parsing the decorator string. `{name}`, `{name:type}` cucumber expressions are supported.
- For TUI tests, the SUT harness needs a real PTY. See [tui-hints.md](tui-hints.md) for the language-agnostic approach and a worked Rust example; in Python, `pexpect` or `ptyprocess` are the standard libraries.
