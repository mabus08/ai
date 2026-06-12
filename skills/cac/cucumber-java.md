# Java — `cucumber-jvm`

## Dependencies

```xml
<!-- pom.xml -->
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-java</artifactId>
    <version>7.18.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>io.cucumber</groupId>
    <artifactId>cucumber-junit</artifactId>
    <version>7.18.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
```

## Layout

```
src/test/
├── resources/features/
│   ├── repository_discovery.feature
│   ├── browsing_commits.feature
│   └── reading_a_diff.feature
└── java/com/example/cac/
    ├── RunCucumberTest.java
    ├── World.java
    └── steps/
        ├── RepositoryDiscoverySteps.java
        ├── BrowsingCommitsSteps.java
        └── ReadingADiffSteps.java
```

## `RunCucumberTest.java`

```java
package com.example.cac;

import io.cucumber.junit.Cucumber;
import io.cucumber.junit.CucumberOptions;
import org.junit.runner.RunWith;

@RunWith(Cucumber.class)
@CucumberOptions(
    features = "src/test/resources/features",
    glue = "com.example.cac.steps"
)
public class RunCucumberTest {
    // Empty. JUnit + Cucumber discover features and step defs from the
    // annotations above.
}
```

## `World.java`

```java
package com.example.cac;

import java.nio.file.Path;
import java.util.ArrayList;
import java.util.List;
import java.util.HashMap;
import java.util.Map;

/**
 * Shared context across steps. Holds the test repo, SUT handle, and any
 * captured state. Steps mutate this; assertions read from it.
 *
 * Skeleton — the SUT handle, PTY harness, and frame parser are TODOs.
 * They turn into real code when the feature is implemented.
 */
public class World {
    public Path repoPath;
    public String stdout = "";
    public String stderr = "";
    public Integer exitCode;
    public Sut sut;
    public List<String> lastFrame = new ArrayList<>();
    public String activePanel;
    public Map<String, Integer> selected = new HashMap<>();

    public static class Sut {
        // TODO: PTY handle, child process, frame buffer.
    }
}
```

## `steps/RepositoryDiscoverySteps.java`

```java
package com.example.cac.steps;

import com.example.cac.World;
import io.cucumber.java.en.Given;
import io.cucumber.java.en.Then;
import io.cucumber.java.en.When;

public class RepositoryDiscoverySteps {
    private final World world = new World();

    @Given("the user is in a local git repository")
    public void user_in_a_local_repo() {
        // TODO: initialize a tempdir git repo with main + 3-commit feature branch
    }

    @When("the user runs {string}")
    public void user_runs_gitreview(String cmd) {
        // TODO: spawn the SUT in the tempdir via the PTY harness
    }

    @Then("gitreview starts without error")
    public void starts_without_error() {
        // TODO: assert the SUT is running and stderr is empty
    }
}
```

## Run

```sh
mvn test
# or
./gradlew test
```

## Notes

- One `@Given/@When/@Then` per step. Step text is matched by the regex inside the annotation string — `{string}`, `{int}`, `{word}` are built-in cucumber expressions.
- `glue` in `@CucumberOptions` points to the package that contains step classes. Sub-packages are scanned automatically.
- For TUI tests, the SUT harness needs a real PTY. See [tui-hints.md](tui-hints.md) for the language-agnostic approach and a worked Rust example; in Java, `pty4j` is the standard library.
