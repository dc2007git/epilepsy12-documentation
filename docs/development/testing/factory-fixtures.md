---
reviewers: Dr Anchit Chandran
---

Pytest factories and fixtures enable DRY code, allowing similar object instances to be defined once, and shared many times across the test directory, enabling faster and more consistent test-driven development.

For example, many tests require the use of an Epilepsy12 `User` model instance.

To create this model, we require multiple arguments to be set, along with other dependent models, such as `Organisation`.

Following the best-practice principle of DRY code, we use factories to define the creation of Epilepsy12 `User` instances, with default values set once, minimising repeated code. Then, we represent multiple different Epilepsy12 `User` types, such as a normal user (`e12User_GOSH`) or a superuser (`e12User_GOSH_superuser`), as "fixtures".

These fixtures are passed into tests and can be shared between them.

## E12 User Factory

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

Default values are set for every required argument, except for `email`, as this is a unique field and will cause errors when creating multiple instances with the same email.

## E12 User fixture

We define fixtures to be used across an entire directory inside the [`confest.py` file](https://github.com/rcpch/rcpch-audit-engine/blob/development/epilepsy12/tests/conftest.py) (see [documentation](https://docs.pytest.org/en/7.1.x/reference/fixtures.html?highlight=conftest#conftest-py-sharing-fixtures-across-multiple-files) for more details).

One available 'global' fixture is the `e12User_GOSH` fixture, which is a normal E12 user whose `Organisation` is Great Ormond Street Hospital:

```python title="conftest.py"
@pytest.mark.django_db
@pytest.fixture()
def e12User_GOSH(new_e12user_factory):
    """
    Creates a single authenticated E12 User object instance for tests.
    """

    return new_e12user_factory(first_name="Norm", email="normal.user@test.com")
```

- `#!python first_name` is overridden as the value `#!python "Norm"`
- `#!python email`, the only required argument, is defined as `#!python "normal.user@test.com"`

## Using the E12 User fixture in tests

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