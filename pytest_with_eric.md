# All about Pytesting in SDLC 

## Common testing patterns in software development lifecycle

**Unit tests**
- Focus on individual components of code in isolation
- Testing e.g., function, class, module, assertion, math functions, string manipulation, data processing, file i/o, etc.

**Integration tests**
- Focus on interaction between different units of code
- Testing e.g., DB, API, UI, Hardware, etc.
  
**End-to-end tests**
- Focus on simulating the actions and behaviours of an end-user, from start of interaction to the end
- Testing e.g., e-commerce website checkout, banking application transactions, flight booking system, social media app, etc.

**Security tests**
- Focus on penetration testing, vulnerability scanning, and security code analysis. 
- Testing e.g., web application security (SQL injection, cross-site-scripting, cross site request forgery), network security (open ports, weak encryption protocols, unauthorized access), mobile application security (data leakage, data storage), cloud security (misconfigured servers), etc.

**Performance tests**
- Focus on load testing, stress, stress testing, and endurance testing to improve user experience (slow page load times) and ensure that application can handle a harge influx of users or transactions (increased scalability)without experiencing performance degradation or failures (reduced downtime).
- Testing e.g., website performance, DB performance, network performance, server performance, etc.

**Regression tests**
- Focus on re-running previously executed test cases to ensure app still works correctly after changes and does not cause unexpected side effects
- Testing e.g., unit regression, GUI, integration regression, e2e regression, performance, etc.

Property-based testing is a complementary approach to traditional unit testing where test cases are generated based on properties or constraints that the code should satisfy. **Hypothesis** (Python lib) addresses this limitation by automatically generating test data based on specified strategies

### Python unit testing best practices
Without proper practices, tests can become a source of frustration and inefficiency bottlenecks. From minor issues like changing your tests often to larger issues like broken CI Pipelines blocking releases, common issues include inefficient testing (slow/complex tests delay dev cycles), lack of clarity (what is being tested and why test fail), fragile tests (tightly coupled with implementation), test redundancy (writing tests that duplicate implementation logic can lead to bloated test suites), non-deterministic tests (flaky tests that produce inconsistent results undermine trust in the testing suite and can mask real issues), inadequate coverage (missing key scenarios, especially edge cases, can leave significant gaps in the test suite, allowing bugs to go undetected)

To enhance the effectiveness of the testing process, leading to more reliable and maintainable Python applications:
- **Tests should be fast**: Fast tests foster regular testing,  a key aspect in continuous integration environments where tests may accompany every commit which in turn speeds up bug discovery. To maintain a strong development workflow, prioritize testing concise code segments and minimize dependencies and large setups.
- **Tests should be independent**: Each test should be able to run individually and as part of the test suite, irrespective of the sequence in which it’s executed. Adhering to this rule necessitates that each test starts with a fresh dataset and often requires some form of post-test cleanup (fixture setup/teardown).
- **Each test should test one thing**: Write separate tests for each functionality, which aids in pinpointing specific issues and ensures clarity.
```py
# Bad practice: Testing multiple functionalities in one test  
def test_user_creation_and_email_sending():  
    user = createUser("test@example.com")  
    assert user.email == "test@example.com"  
    sendWelcomeEmail(user)  
    assert emailService.last_sent_email == "test@example.com"

# Good practice: Testing one functionality per test  
def test_user_creation():  
    user = createUser("test@example.com")  
    assert user.email == "test@example.com"  
  
def test_welcome_email_sending():  
    user = User("test@example.com")  
    sendWelcomeEmail(user)  
    assert emailService.last_sent_email == "test@example.com"
```
- **Tests should be readable**: Readable tests act as documentation. The use of descriptive names and straightforward logic ensures that anyone reviewing the code can understand the test’s purpose and function.
- **Tests should be deterministic**: Deterministic tests give consistent results under the same conditions, ensuring test predictability and trustworthiness. To manage non-deterministic tests, it’s crucial to drop randomness or external dependencies where possible. If randomness is essential, consider using fixed seeds for random number generators. For external dependencies like databases or APIs, use mock objects or fixtures to simulate predictable responses. Libraries like Faker, Model Bakery and Hypothesis allow you to test your code against a variety of sample input data.
- **Tests should not include implementation details**: Tests should verify behaviour (what the code does), not implementation details (how the code does). This distinction is crucial for creating robust, maintainable tests that remain valid even when the underlying implementation changes.
```py
# What not to do
def test_add():  
    assert 2 + 3 == 5  # includes implementation detail

# What to do
 def test_add():  
    assert add(2, 3) == 5  # Implementation detail abstracted
```
- **Tests should not part of commit and build process**: It enforces a standard of code health and functionality, serving as a crucial checkpoint before deployment. You can use pre-commit hooks to run tests before each commit, ensuring that only passing code is committed. `pip install pre-commit`. You can set up Pytest to run with CI tooling like GitHub Actions, Bamboo, CircleCI, Jenkins and so on that runs your test after deployment thus helping your release with confidence
- **Use fake data/db in testing**: Using fake data or databases in testing ensures consistency in test environments and allows you to simulate various scenarios without relying on actual databases, which can be unpredictable and slow down tests.
- **Use dependency injection**: DI involves providing objects with their dependencies from outside, rather than having them create dependencies internally. This method is particularly valuable in testing, as it allows for easy substituting of real dependencies with mocks or stubs. For instance, a class that requires a database connection can be injected with a mock database during testing, isolating the class’s functionality from external systems.
```py
# Without Dependency Injection
class DatabaseConnector:  
    def connect(self):    
        return "Database connection established"  
  
class DataManager:  
    def __init__(self):  
        self.db = DatabaseConnector()  
  
    def data_operation(self):  
        return self.db.connect() + " - Data operation done"

# With Dependency Injection (DI)
class DatabaseConnector:  
    def connect(self):  
        return "Database connection established"  
  
class DataManager: 
    '''
    In this scenario, testing DataManager‘s data_operation method is challenging because it’s tightly coupled with DatabaseConnector.
    ''' 
    def __init__(self, db_connector):  
        self.db = db_connector  
  
    def data_operation(self):  
        return self.db.connect() + " - Data operation done"  
  
# During testing  
class MockDatabaseConnector:
    '''
    This allows us to inject a MockDatabaseConnector during testing, focusing the test on DataManager‘s behaviour without relying on the actual database connection logic.
    '''  
    def connect(self):  
        return "Mock database connection"  
  
# Test  
def test_data_operation():  
    mock_db = MockDatabaseConnector()  
    data_manager = DataManager(mock_db)  
    result = data_manager.data_operation()  
    assert result == "Mock database connection - Data operation done"
```
- **Use setup and teardown To isolate test dependencies**: The setup method prepares the test environment before each test, ensuring a clean state. Conversely, the teardown method cleans up after tests, removing any data or state alterations. This process prevents interference between tests, maintaining their independence. Fixtures, in Pytest, provide a flexible way to set up and tear down code for many tests.
```py
import pytest  
  
@pytest.fixture  
def resource_setup():  
    # Setup code: Create resource  
    resource = "some resource"  
    print("Resource created")  
      
    # This yield statement is where the test execution will take place  
    yield resource  
  
    # Teardown code: Release resource  
    print("Resource released")  
  
def test_example(resource_setup):  
    assert resource_setup == "some resource"
```
- **Group related tests**: Organizing related tests into modules or classes is a cornerstone of good unit testing in Python. For instance, all tests related to a user might reside in a `TestUser` class.
- **Mock where necessary (with caution)**: Over-mocking can lead to tests that pass despite bugs in production code, as they might not accurately represent real-world interactions.
- **Use test framework features to your advantage**: Key among these in Pytest are `conftest.py`, fixtures, and parameterization.
- **Practice test driven development (TDD)**: The cycle of “Red-Green-Refactor” — writing a failing test, making it pass, and then refactoring — guides development, ensuring that your codebase is thoroughly tested and designed with testing in mind.
- **Regularly review and refactor tests**: Refactoring might include removing redundancy, updating tests to reflect code changes, or improving test organization and naming. This proactive approach keeps the test suite streamlined, improves its reliability, and ensures it continues to support ongoing development and codebase evolution.
- **Analyze test coverage**: Coverage analysis involves measuring the extent to which your test suite exercises your codebase. High coverage means you’ve tested a larger part of your code. While this is helpful, it doesn’t mean your code is free from bugs. It means you’ve tested the implemented logic. Testing edge cases and against a variety of test data to uncover potential bugs should be your top priority.
- **Test for security issues as part of your unit tests**: Security testing involves crafting tests to uncover vulnerabilities like injection flaws, broken authentication, and data exposure. Using libraries like `Bandit`, you can automate the detection of common security issues. For instance, if your application involves user input processing, your tests should include cases that check for common security issues like SQL injection or cross-site scripting (XSS) such as:
```py
# Consider a function that queries a database
def get_user_details(user_id):  
    query = f"SELECT * FROM users WHERE id = {user_id}"  
    # Execute query and return results

# To test for this security issue, you can write a unit test that attempts to exploit the vulnerability of SQL injection
import pytest  
from mymodule import get_user_details  
  
def test_sql_injection_vulnerability():  
    malicious_input = "1; DROP TABLE users"  
    with pytest.raises(SecurityException):  
        get_user_details(malicious_input)
```
_Recommendation: Against using raw SQL in your code unless absolutely necessary and instead use an ORM (Object Relational Mapping) tool like `SQLAlchemy` or `SQL Model` to perform database operations_

## Running Pytest
- `pytest` : run all the tests from the root dir
- `-v` : verbose output
- `q` : concise output
- `-s` : live console logging
- `-rs` : print skip reason
- `pytest tests/unit` : run tests in a directory
- `pytest tests/unit/test_func.py` : run tests in a module
- `pytest tests/unit/test_func.py::TestUnit` : run tests of a specific class
- `pytest tests/unit/test_func.py::test_add_nums` : run tests by node ID
- `pytest -m unit` : run all tests within the unit marker
- `pytest --collect-only` : see what tests Pytest is collecting
- `pytest --ignore=tests/test_code.py`: exclude test file from the test run

### Run tests by marker expression
Run specific group of tests, exclude tests, and prioritize tests to get more value from your test suite. Use `-m` flag followed by a marker name.
```py
@pytest.mark.end_to_end
def test_func():
    pass

@pytest.mark.skip
def test_func1():
    pass

@pytest.mark.unit
def test_func2():
    pass
```
Define markers in `pytest.ini` to avoid errors in the terminal
```py
[pytest]
markers = 
    unit: unit tests
    end_to_end: end to end tests
    skip: slow tests
```

### Run tests as per verbosity levels
The verbosity level controls the amount of information displayed in the test results. This information is extremely useful for debugging tests and understanding what’s going on during test execution.

- **Low verbosity `pytest -v`**: Lists each test function with its result (PASSED or FAILED). Provides a more detailed failure report than the default, but omits some common items in the failure diff.

- **Medium verbosity `pytest -vv`**: It displays additional information about test collection and execution such as the names of collected tests, setup and teardown steps, etc.

- **High verbosity `pytest -vvv`**: It displays extensive information about each test, including captured stdout/stderr, local variables, and more.

**Practical use cases of `-v`**: CI/CD tools easily parse test results and identify failures. The detailed output aids in quickly spotting issues during automated builds; Debugging; Regression testing; Resource management because when tests involve the allocation and release of resources (e.g., databases, files), it helps ensure that fixtures are set up and torn down correctly.

## Python exception handling
`excinfo` is an `ExceptionInfo` instance, a wrapper around the actual exception raised. The main attributes of interest are `.type`, `.value`, `.traceback`. 

### Assert when an exception is raised
```py
with  pytest.raises(EXCEPTION) as excinfo:
    func()
assert str(excinfo.value) == "Exception"
```
- If the code raises an exception, pytest will pass.
- E.g. `ValueError`, `TypeError`, `FileNotFoundError`, `ZeroDivisionError`, custom exceptions, etc.

One can pass a `match` keyword param to context-manager as `pytest.raises(ValueError, match=r".*123.*")` 
to test that a regular expression matches on the string representation of an exception

### Assert when NO exception is raised
```py
try:
    func()
except Exception as excinfo:
    pytest.fail(f"Unexpected exception raised: {excinfo}")
```
- Run desired function in a try/except, catch any unexpected exceptions and fail the test if an exception is raised.

### Custom explanation for failed assertions
Consider adding the following hook in `conftest.py` which provides an alternative explanation
```py
def pytest_assertrepr_compare(op, left, right):
    if isinstance(left, Foo) and isinstance(right, Foo) and op == "==":
        return ["Comparing Foo instances:",  " vals: {} != {}".format(left.val, right.val)]
```

## Pytest skip test and xfail
**Usecases**: Incompatible python or pkg version, platform specific (windows, mac, linux), external resource dependency (DB or APIs), local dependencies pre-commit hooks & local AWS profiles

- **Unconditional skipping**

    Use a marker (function/class level) and add a reason arg (highly recommended): `@pytest.mark.skip(reason="abc")`

- **Conditional skipping**

    Skip if Python version > X: `@pytest.mark.skipif(sys.version_info >= (3, 10), reason="requires lower than python 3.10")`
    
    Skip all tests - Add pytest marker at top of the test file: `pytestmark = pytest.mark.skip("Tests in WIP")`

    Skip if module import fails: `fake_library = pytest.importerskip("fake_library")`

    Skip on platform: `pytestmark = pytest.mark.skipif(sys.platform == "darwin", reason="tests for windows only")`

- **Intentional failure: XFail**

    Valid reason: bug fix or new feature yet to be released: `pytest.mark.xfail(reason="abc")`

Failing tests intentionally is preferred over skipping them because failed tests show up in Pytest coverage report.

## Avoiding ModuleNotFound/Import Errors
An essential part of Pytest's conf is managing Pytest `PYTHONPATH`

**Run Pytest as a module** `python -m pytest` 

Treats our unit test like a Python module, looks in the current directory and executes it. Works but not convenient.

**Correctly specify PYTHONPATH**
- Use INIT files
- Use Pytest config files - pytest.ini
- Define in the test
- Use environment variables

## Parameterized tests
Pytest parametrized testing is a powerful technique that allows you to efficiently run same tests with multiple sets of input data eliminating the need for reduntant code by using decorators such as `@pytest.mark.parametrize`. It is possible to parametrize functions, class, and even fixtures too.

To use markers like `xfail` or `skip` while using parametrize approach:
```py
@pytest.mark.parametrize("input, expected", [
    ("hello", "hello"),
    pytest.param(None, 42, marks=pytest.mark.xfail(reason="bug")),
])
def test_string(input, expected):
    assert clean_str(input) == expected
```

## Pytest plugin `pytest_generate_tests`
This hook allows to dynamically generate test cases based on variety of conditions or inputs specific to project's req. It runs before any test is executed.
```py
def pytest_generate_tests(metafunc):
    if 'test_input in metafunc.fixturenames:
        metafunc.parametrize("test_input, expected_output", test_data)
```
```py
def test_addition(test_input, expected):
    res = add(*test_input)
```

**Limitations**: Not a best fit to 
- generate test cases with extensive data manipulation
- handle fixture dependencies
- identify source of failure (debugging is trickier)

## Pytest Fixture
Pytest Fixtures allow you to set up necessary prerequisites for running your tests, such as managing resources, data, and dependencies, before executing tests. They help provide a well-organized and efficient testing environment. Fixtures are defined using the `@pytest.fixture` decorator.
```
from src.network_manager import network_manager

@pytest.fixture
def network_connection():
    net_connect = network_manager()
    yield net_connect
    # Teardown code

# Function to check network
def test_network_connection(network_connection):
    assert network_connection == True
```

In some cases, the Pytests cache can lead fixture fixture-related issues and result in fixture not found error. So clear the cache with the command defined below and try again. `pytest --cache-clear`

File organization is essential for Pytest. Pytest looks for files starting with test_* under the /tests directory. You can override this using the --rootdir option or specifying the directory in the pytest.ini file.

### Autouse
Sometimes there are fixtures on which all your tests depends. You can make these fixtures available to all test models by setting `autouse=True`. It offers a convenient way to have all tests automatically request and utilize them.

### Scopes
Pytest fixtures also allow you to determine the lifetime and availability of a fixture. These are done via fixture scopes:
- `function` - This is the default scope. It defines that the fixture will be created at the beginning of a test function and torn down at the end of the function.
- `class` - This scope defines that the fixture will be created at the beginning of a class and torn down at the end of the class.
- `module` - This scope defines that the fixture will be created at the beginning of a module and torn down at the end of the module.
- `session` - This scope defines that the fixture will be created at the beginning of a session and torn down at the end of the session.

## Pytest Mocker
Mocking is a technique to allow parts of a system to be replaced with controlled stand-ins, enhancing test isolation and control. The `mocker` fixture is a fixture provided by Pytest, built atop Python’s built-in `unittest.mock` method. It is available in Pytest via the `pytest-mock` plugin. Instead of relying on direct imports, this fixture is delivered through dependency injection.

To resolve "Fixture `Mocker` Not Found" error:
- Ensure you’ve installed it using the command `pip install pytest-mock` or just stick it in your `requirements.txt` file to be installed when creating your virtual environment.
- As silly as this sounds, misspelling the fixture name- using `mock` instead of `mocker` can cause issues.
- You don’t need to import it explicitly in your test files. It will be automatically recognized by pytest once the plugin is installed.

## Pytest Logging
Enhance logging experience by utilizing Pytest’s verbosity feature. Pytest logging empowers to not only create but also override log handlers within the Pytest framework. Add following snippet in Pytest config file `pytest.ini`:
```py
[pytest]
addopts = -v
log_cli = True
log_cli_level = INFO
```

### References
- https://pytest-with-eric.com/
