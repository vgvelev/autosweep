.. AutoSweep documentation master file, created by
   sphinx-quickstart on Mon Jul 10 16:19:21 2023.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to AutoSweep's documentation!
===========================================

The AutoSweep project is designed to run automated testing of silicon photonic devices. It's
designed to be highly modular, giving a framework with which to organize test sequences and manage instruments, while
keeping track of device metadata, saving raw data, generating reports and handling errors.

Installation
------------

After cloning the repository, you can install it using ``pip install .``. If you want to develop features within Test
Automation, you'll find the pip ``-e`` option useful, ``pip install -e .``, this allows you to make edits to the code
and execute tests from the python environment the package was installed in without reinstalling it via pip.

After installation, execute the python script in ``tests/test_exec/run_test_exec.py``. You'll see a a folder of virtual
test data generated in the folder ``tests/test_exec/data``. Open the script and browse the test data to become more
familiar with the operation of the code.

Station Configuration
---------------------

The station configuration contains information about the test setup itself, including any instrumentation connected to
it. The simplest way to import a configuration is via JSON file, though any ``dict`` with the correct formatting will
work.

For loading a file ``station_config = StationConfig.read_json(path=Path('station_config.json'))``. For creating a
``StationConfig`` from a ``dict``, use ``station_config = StationConfig({...})``.

+-----------------+------------+---------------------------------------------------------------------------------------+
| Key             | Value Type | Value Description                                                                     |
+=================+============+=======================================================================================+
| ``station_id``  | ``str``    | A single unique identifier of the station itself.                                     |
+-----------------+------------+---------------------------------------------------------------------------------------+
| ``paths``       | ``dict``   | The contents of this field are the locations to read and save data. Two keys are      |
|                 |            | supported; ``base``, whose value is the parent directory for all other directories    |
|                 |            | listed, and ``data``, whose value is the name of the directory where all raw data is  |
|                 |            | saved.                                                                                |
+-----------------+------------+---------------------------------------------------------------------------------------+
| ``instruments`` | ``dict``   | The information about connected instruments is contained in this section. The keys of |
|                 |            | this dictionary are the instance names of the instruments. These instance names must  |
|                 |            | match those in the recipe for the instrument to be used. The value of these keys is   |
|                 |            | another dictionary that contains the instrument class and information about           |
|                 |            | connecting to its com port.
+-----------------+------------+---------------------------------------------------------------------------------------+


Recipe
------

A recipe represents the tests to run over a device under test. The simplest way create a recipe is by writing a JSON
file, though any ``dict`` with the correct formatting can be loaded.

For loading a file ``recipe = Recipe.read_json(path=Path('recipe.json'))``. For creating a ``Recipe`` from a ``dict``,
use ``recipe = Recipe({...})``.

.. code-block:: json

  "instruments": ["smu", "ps"],

This element lists instrument instance names that are needed to execute this recipe. Every instrument needed by the all
the tests in the recipe need to be included. If the recipe requires both an SMU and a power supply, for example, the
the entry might look as above. These instance names must match those in
the Station Configuration (make this a link to the relevant section above). The station configuration determines which
SMU to use and how to connect to it. Not all instruments present in a station need to be listed, only those needed for
this specific recipe execution, however, if an instrument instance does not exist, testing will fail during
initialization.

Individual tests are defined in the ``test`` block, an example is shown below:

.. code-block:: json
  :linenos:

  "tests": [
    [
      "abc",
      {
        "class": "VirtualTest",
        "init": {},
        "acquire": {},
        "analysis": {
          "report_headings": ["Virtual IV"]
        }
      }
    ],
    [
      "bcd",
      {
        "class": "NewTest",
        "init": {},
        "acquire": {},
        "analysis": {
            "report_headings": ["New Virtual IV"]
        }
      }
    ]
  ]

The ``tests`` value is of type ``list[list[str, dict]]``. The tests are executed in the order listed. Each element of
the ``tests`` list contains two parts, a ``str``, which is the test instance name, and must be unique (TODO: CHECK) and
a dictionary with the following elements:

+--------------+------------+------------------------------------------------------------------------------------------+
| Key          | Value Type | Value Description                                                                        |
+==============+============+==========================================================================================+
| ``class``    | ``str``    | A valid test class registered within the framework. See the section on                   |
|              |            | (TODO: adding tests)                                                                     |
+--------------+------------+------------------------------------------------------------------------------------------+
| ``init``     | ``dict``   | A ``kwargs`` for the initialization of the test class. The recipe can pass additional    |
|              |            | information to the test this way. If nothing is needed, the value should be a ``{}``.    |
+--------------+------------+------------------------------------------------------------------------------------------+
| ``acquire``  | ``dict``   | A ``kwargs`` for the data acquisition portion of the test execution. If the TestExec is  |
|              |            | run in ``analysis_only=True`` mode, the data acquisition portion will not execute so     |
|              |            | these options will not be used. If nothing is needed, the value should be a ``{}``.      |
+--------------+------------+------------------------------------------------------------------------------------------+
| ``analysis`` | ``dict``   | A ``kwargs`` for the data analysis portion of the test execution. If nothing is needed,  |
|              |            | the value should be a ``{}``.                                                            |
+--------------+------------+------------------------------------------------------------------------------------------+

Adding a Test
-------------

To add a new test, you need to create a class which inherits from ``autosweep.tests.abs_test.AbsTest``. Remember that
the TestExec only calls the ``__init__``, ``run_acquire``, and ``run_analysis`` methods during recipe execution.

The script that runs the TestExec must also register the test with AutoSweep before running the recipe by calling the
function ``register_classes`` with the argument being the module that contains the new test.

Adding an Instrument
--------------------

To add a new instrument, you need to create a class which inherits from
``autosweep.instruments.abs_instr.AbsInstrument``.


.. toctree::
   :maxdepth: 2
   :caption: Contents:

   api


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
