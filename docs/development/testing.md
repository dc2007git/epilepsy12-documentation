---
title: Testing
reviewers: Dr Marcus Baw
---

Tests for the Epilepsy12-specific parts of the platform are organised in a `epilepsy12/tests/` folder inside the epilepsy12 app. We have opted to use Pytest as it is well regarded in the Django community.

## Running `pytest`

When running tests it is important to understand that they will only run **inside** the Docker container (assuming you have used the Docker development setup). How you run the tests depends on whether you are using Docker Desktop (either through the native application or [VSCode extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker) where you can attach a shell terminal to the Docker environment) or Docker Compose. The following examples assume you are in the root of the project.

=== "Using Docker Desktop"
    Using the [integrated terminal](https://docs.docker.com/desktop/use-desktop/container/#integrated-terminal) in Docker Desktop:
    ```console
    pytest
    ```

=== "Using docker compose"
    Run the following command in your normal system terminal:
    ```console
    sudo docker compose -f docker-compose.dev-init.yml exec web pytest
    ```


### Skipping or selecting tests

Some of the more time consuming tests can be skipped for convenience, using the ['mark' feature](https://docs.pytest.org/en/7.1.x/how-to/mark.html) in Pytest. The following will **skip** tests marked 'examples'.

=== "Using Docker Desktop"
    ```console
    pytest -k "not examples"
    ```

=== "Using docker compose"
    ```console
    sudo docker compose -f docker-compose.dev-init.yml exec web pytest -k "not examples"
    ```


### Verbosity

You can increase the [verbosity of the Pytest output](https://docs.pytest.org/en/7.1.x/how-to/output.html#verbosity) using the `-v` flag. The more `-v` flags you add, the more verbose the output, to a limit of -vv. The following will run the tests with the maximum verbosity.

=== "Using Docker Desktop"
    ```console
    pytest -vv
    ```

=== "Using docker compose"
    ```console
    sudo docker compose -f docker-compose.dev-init.yml exec web pytest -vv
    ```


### Running a single test

Pytest allows a specific test to be run by specifying the path to the test file. This can be useful when debugging a specific test. The following will run only the test at the specified path.

=== "Using Docker Desktop"
    ```console
    pytest epilepsy12/tests/model_tests/test_antiepilepsy_medicine.py
    ```

=== "Using docker compose"
    ```console
    sudo docker compose -f docker-compose.dev-init.yml exec web pytest epilepsy12/tests/model_tests/test_antiepilepsy_medicine.py
    ```

To run a specific test within this file, use the `PATH_TO_TEST_FILE::TESTNAME` notation:

=== "Using Docker Desktop"
    ```console
    pytest epilepsy12/tests/model_tests/test_antiepilepsy_medicine.py::test_length_of_treatment
    ```

=== "Using docker compose"
    ```console
    sudo docker compose -f docker-compose.dev-init.yml exec web pytest epilepsy12/tests/model_tests/test_antiepilepsy_medicine.py::test_length_of_treatment
    ```


### Capturing `stdout` and `stderr`



If you have used `print()` statements in your code, you may want to capture the output of these statements in your tests. By default, Pytest captures the stdout from tests and displays them depending on certain conditions.

You can specify how stdout is displayed using either the `-s` or `-rP` flags.

The `-rP` flag defines how Pytest shows the  "short test summary info" content. It combines the `-r` option, which by default shows "failures and errors", with `P` which adds the captured output of passed tests.

=== "Using Docker Desktop"
    ```console
    pytest -rP
    ```

=== "Using docker compose"
    ```console
    sudo docker compose -f docker-compose.dev-init.yml exec web pytest -rP
    ```

The `-s` flag will tell Pytest to not capture the stdout, and instead prints it straight to the console:

=== "Using Docker Desktop"
    ```console
    pytest -s
    ```

=== "Using docker compose"
    ```console
    sudo docker compose -f docker-compose.dev-init.yml exec web pytest -s
    ```

!!! tip "`pytest -h` for help"
    Further detail on the options available can be found by running `pytest -h`, and by visiting the [Pytest documentation](https://docs.pytest.org/).

### `pytest.ini`

The `pytest.ini` file, found in the project root, defines configurations for our tests.

Taking this example:

```ini
[pytest]

# POINT PYTEST AT PROJECT SETTINGS
DJANGO_SETTINGS_MODULE = rcpch-audit-engine.settings

# SET FILENAME FORMATS OF TESTS
python_files = test_*.py


addopts =
    --reuse-db # RE USE TEST DB AS DEFAULT
    -k "not examples" # AVOID TESTS MARKED AS EXAMPLES

markers =
    examples: mark test as workshop-type / example test
    seed: mark test as 'meta test', just used for seeding
```

| Configuration Option      | Description                          |
| :---------- | :----------------------------------- |
| [`addopts`](https://docs.pytest.org/en/7.1.x/reference/reference.html?highlight=addopts#confval-addopts)       | Adds arguments to the `pytest` command by default. E.g. running `pytest` actually results in: <br>```pytest --reuse-db -k "not examples"```  |
| `markers`       | Define additional markers to selectively run tests. |

## Coverage

Using the `coverage` tool we can get some code analysis including the total test coverage. The following will run pytest through `coverage`. Then type `coverage report` to see the coverage report.

=== "Using Docker Desktop"
    ```console
    coverage run -m pytest
    coverage report
    ```

=== "Using docker compose"
    ```console
    sudo docker compose -f docker-compose.dev-init.yml exec web coverage run -m pytest
    sudo docker compose -f docker-compose.dev-init.yml exec web coverage report
    ```


## Factories and fixtures

Many tests require the use of an Epilepsy12 `User` model instance.

To create this model, we require multiple arguments to be set, along with other dependent models, such as `Organisation`.

Following the best-practice principle of DRY code, we use factories to define the creation of Epilepsy12 `User` instances, with default values set once, minimising repeated code. Then, we define multiple different Epilepsy12 `User` types, such as a normal user (`e12User_GOSH`) or a superuser (`e12User_GOSH_superuser`), as "fixtures". These fixtures can be passed into tests, enabling faster and more consistent test-driven development.

### E12 User Factory

Taking a more granular look at this workflow, with customisation options, we start with the [`e12user_factory.py`](https://github.com/rcpch/rcpch-audit-engine/blob/development/epilepsy12/tests/factories/e12user_factory.py), specifically looking at the default arguments set for our E12User fixtures:

```py title="e12user_factory.py"
...

@pytest.mark.django_db
@pytest.fixture()
def new_e12user_factory():
    """
    Factory fn which returns a new user. See definition for available options.
    """
    def create_e12user(
        email: str,
        password: str = 'password',
        first_name: str = "Mandel",
        surname: str = "Brot",
        is_active: bool = True,
        is_staff: bool = True,
        is_rcpch_audit_team_member: bool = True,
        is_superuser: bool = False,
        role: int = user_types.AUDIT_CENTRE_LEAD_CLINICIAN,
        email_confirmed: bool = True,
        organisation_employer = Organisation.objects.get(ODSCode='RP401'),
    ): 
    
    ...
```

Default values are set for every required argument, except for `email`, as this is a unique field, and will cause errors when creating multiple instances with the same email.

### E12 User fixture

We define fixtures to be used across an entire directory inside the [`confest.py` file](https://github.com/rcpch/rcpch-audit-engine/blob/development/epilepsy12/tests/conftest.py) (see [documentation](https://docs.pytest.org/en/7.1.x/reference/fixtures.html?highlight=conftest#conftest-py-sharing-fixtures-across-multiple-files) for more details).

One available 'global' fixture is the `e12User_GOSH` fixture, which is a normal E12 user, whose `Organisation` is Great Ormond Street Hospital:

```python title="conftest.py"
@pytest.mark.django_db
@pytest.fixture()
def e12User_GOSH(new_e12user_factory):
    """
    Creates a single authenticated E12 User object instance for tests.
    """

    return new_e12user_factory(first_name="Norm", email="normal.user@test.com")
```

- `#!python first_name` is overriden as the value `#!python "Norm"`
- `#!python email`, the only required argument, is defined as `#!python "normal.user@test.com"`

### Using the E12 User fixture in tests

Fixtures defined in `conftest.py` can be passed straight into test functions, **without requiring import**, as seen in [`test_visitactivity.py`](https://github.com/rcpch/rcpch-audit-engine/blob/development/epilepsy12/tests/model_tests/test_visitactivity.py):

```python title="test_visitactivity.py" hl_lines="2 7 13"
@pytest.mark.django_db
def test_visitActivity(e12User_GOSH, client):
    """
    Test that visit activity logging works, using e12User_GOSH fixture
    """

    client.force_login(e12User_GOSH)

    visit_activity = VisitActivity.objects.all()

    time.sleep(1)

    client.force_login(e12User_GOSH)

    assert len(visit_activity) == 2
    assert visit_activity[1].activity_datetime > visit_activity[0].activity_datetime
```