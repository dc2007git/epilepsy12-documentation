---
title: Testing
reviewers: Dr Marcus Baw
---

Tests for the Epilepsy12-specific parts of the platform are organised in a `epilepsy12/tests/` folder inside the epilepsy12 app. We have opted to use Pytest as it is well regarded in the Django community.

## Running `pytest`

When running tests it is important to understand that they will only run **inside** the Docker container (assuming you have used the Docker development setup). How you run the tests depends on whether you are using Docker Desktop or Docker Compose. The following examples assume you are in the root of the project.

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


### Capturing `stdout` and `stderr`

If you have used `print()` statements in your code, you may want to capture the output of these statements in your tests. This can be done using the `-s` flag. The following will print the stdout and stderr to the console.


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


## Factories

<!-- Anchit to add -->