BundleTester
============

A simple juju-deployer test runner for bundles.

This test runner uses the following pattern. A bundle has a URL.
The runner can be run directly from within a bundle checkout or
will pull the branch pointed at by the bundle url. From there
it will attempt to look for a 'tests' directory off the project 
root. Using the rules described below it will find and execute
each test within that directory and produce a report.

Example Usage
=============

From within the top level of a bundle 

    bundletester -l DEBUG -o result.json

Using a specific directory of tests

    bundletester -l DEBUG -o result --testdir `pwd`

Using the --testdir directive you can also run the tests
from a charm directory with all the default config.

    bundletester -l DEBUG -o result --testdir charm/tests


Test Directory
==============

The driver file, 'tests/tests.yaml' is used (by default) to control the overall
flow of how tests work. All values in this file and indeed the file itself are
optional. When not provided defaults will be used.

tests.yaml
----------

A sample `tests.yaml` file::

    bootstrap: true
    reset: true
    setup: script
    teardown: script
    tests: [\d+\w+ for d in dir if d is execuatble]
    virtualenv: true
    sources:
        - ppas, etc
    packages:
        - amulet
        - python-requests

Explanation of keys:

bootstrap: Allow bootstrap of current env, default: true

reset: Use juju-deployer to reset env between test, default: true

setup: optional name of script in test dir to run before each test

teardown: optional name of script to run after each test

tests: glob of files in testdir to treat as tests

virtualenv: create and activate a virutalenv for the running of all tests
defaults to true

sources: list of package sources to add automatically 

packages: list of packages to install automatically with apt


Found Tests
-----------

When tests.yaml's test pattern is executed it will yield test that should be run. Each
of these test files can optional have a control file of the same name but with a `.yaml`
extension. If present this file has a similar layout to tests.yaml. 

A sample `01-test.yaml` file:
    bundle: bundle.yaml
    setup: script
    teardown: script

Explanation of keys:

bundle: A bundle file in the tests directory. If bundle isn't specified
<BUNDLE_ROOT>/bundle.yaml will be used by default.

setup: optional script to be run before this test. This is called after the
tests.yaml setup if present.

teardown: optional script to be run after this test. This is called before the
tests.yaml teardown if present.

Setup/Teardown
--------------

If these scripts fail with a non-zero exit code the test will be recorded as a failure with
that exit code. 

Test Execution
--------------

Each test should be executable. If `virtualenv` is true each test will be executed with that 
environment activated. 

Each test will have its stdout and stderr captured.

Each tests exit code will be captured. A non-zero return indicates failure.

Reporting Results
-----------------

Any test failures will result in a non-zero exit code. 

By default a JSON string will be outputted on stdout. This string will 
contain at minimum the following structure (additional keys are possible)

    [{test result}, ...]

each test result is a dict with:

    {'test': test file, result: exit_code, 
      'exit': 'script name which returned exit code',
      'starttime': GMT,
      'endtime': GMT,
      'duration': timedelta in seconds}

`exit` will not be included if result is sucess (0)


TODO
====
Finish Test, started as TDD and then hulksmashed the end
Better runtime streaming
Cleaner reporting
More visability into process