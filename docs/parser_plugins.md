# Creating parser plugins

The role of a parser is to map between a structure or unstructure file into a well-defined, standardized, and (ideally) FAIR data schema. In our case, we are interested on mapping simulation data into the [`nomad-simulations` schema](nomad_simulations.md). When the sections defined in `nomad-simulations` are not enough, any user can extend and tune the schema to their specific needs, see [Extending the schema](extending_schema.md).

In NOMAD, when an user [creates an upload and push some files](https://nomad-lab.eu/prod/v1/staging/docs/examples/computational_data/uploading.html), a processing occurs. This processing involves several steps, being **matching** and **parsing** the most important and relevant ones:

1. [_Matching the file(s)_](#matching-parser) to the relevant parser class called `<ParserName>Parser`. Only a specific file is matched with the parser class, which receives the name of [_mainfile_](https://nomad-lab.eu/prod/v1/staging/docs/reference/glossary.html#mainfile). 
2. Call into the `<ParserName>Parser` class function `parse()`. 
4. The `parse()` function populates the [_archive_](https://nomad-lab.eu/prod/v1/staging/docs/reference/glossary.html#archive) from the mainfile and other _auxiliary_ files which might be relevant. This archive contains of the relevant metadata in a NOMAD [_entry_](https://nomad-lab.eu/prod/v1/staging/docs/reference/glossary.html#entry).

You can find more information about processing the NOMAD documentation page, see [Explanation - Processing](https://nomad-lab.eu/prod/v1/staging/docs/explanation/processing.html#processing-scenarios). Schematically, this processing can be visualized as:


<div class="click-zoom">
    <label>
        <input type="checkbox">
        <img src="../assets/parsing_illustration.png" alt="Symbolic, 3-step representation of the parsing process: 1. SELECT FILES; 2. EXTRACT DATA;  3. SCHEMA" width="80%" title="Click to zoom in">
    </label>
</div>


In this page, we are going to learn how to create a parser, its structure and basic functionalities. NOMAD parsers are [plugins](https://en.wikipedia.org/wiki/Plug-in_(computing)), and thus can be defined in their own repositories and be developed independently of the main software. In case of administrating a NOMAD Oasis, you can read more about connecting plugins in the NOMAD documentation, see [NOMAD Oasis - Install plugins](https://nomad-lab.eu/prod/v1/staging/docs/howto/oasis/plugins_install.html).


## Starting a plugin project {#start-plugin}

To create your own parser plugin, visit the [NOMAD plugin template](https://github.com/FAIRmat-NFDI/nomad-plugin-template) and click the “Use this template” button (you need a Github account to do so):

<div class="click-zoom">
    <label>
        <input type="checkbox">
        <img src="../assets/github_plugin_template.png" alt="Overview of the nomad-plugin-template repository." width="80%" title="Click to zoom in">
    </label>
</div>


You can decide where to host the parser plugin. Once this is done, in your local machine you can clone the generated repository. For the purpose of this example, we will use [JosePizarro3/example-plugin](https://github.com/JosePizarro3/example-plugin):
```sh
git clone https://github.com/JosePizarro3/example-plugin.git
```

All the steps we are going to do can be found in the branch `pyscf-new-template`.

Go to the `example-plugin` directory and create a virtual environment with Python 3.9, 3.10, or 3.11, and activate it:
```sh
cd example-plugin
python3.11 -m venv .pyenv
. .pyenv/bin/activate
```

Install `uv` (a very fast installer of Python packages) and `cruft`, and follow the instructions in the `README.md` to generate the parser:
```sh
pip install --upgrade pip
pip install uv cruft
cruft create https://github.com/FAIRmat-NFDI/cookiecutter-nomad-plugin
```

You will be prompted with some questions and information regarding the plugin. Make sure to include both `parser` and `schema_package` options (the latter will be use in the [Extending the Schema](extending_schema.md) part):
```sh
  [1/13] full_name (John Doe): <whatever-name>
  [2/13] email (john.doe@physik.hu-berlin.de): <whatever-email> 
  [3/13] github_username (Github organization or profile name, default: foo): <whatever-github-name>
  [4/13] plugin_name (foobar): nomad-parser-pyscf    
  [5/13] module_name (recommended: press enter to use the default module name) 
(nomad_parser_pyscf):     
  [6/13] short_description (Nomad example template): NOMAD parser plugin for PySCF simulations output in a log text file. 
  [7/13] version (0.1.0): 
  [8/13] Select license
    1 - MIT
    2 - BSD-3
    3 - GNU GPL v3.0+
    4 - Apache Software License 2.0
    Choose from [1/2/3/4] (1): <whatever-license>
  [9/13] include_schema_package [y/n] (y): y
  [10/13] include_normalizer [y/n] (y): n
  [11/13] include_parser [y/n] (y): y
  [12/13] include_app [y/n] (y): n
  [13/13] include_example_uploads [y/n] (y): n
```

You can use the script under the generated folder to move all files one level up:
```sh
sh nomad-parser-pyscf/move_template_files.sh
```

The structure of the plugin is (without including the `docs` and `mkdocs.yml` files):
```sh
example-plugin/
├── README.md
├── LICENSE
├── pyproject.toml
├── src/
│   └── nomad_parser_pyscf/
│       ├── __init__.py
│       ├── parsers/
│       │   ├── __init__.py
│       │   └── parser.py
│       └── schema_packages/
│           ├── __init__.py
│           └── schema_package.py
├── tests/
│   ├── conftest.py
│   ├── data/
│   │   ├── example.out
│   │   └── test.archive.yaml
│   ├── parsers/
│   │   └── test_parser.py
│   └── schema_packages/
│       └── test_schema_package.py
└── ... (other files)
```

You can read more about plugins in the NOMAD documentation page, see [How to get started with plugins](https://nomad-lab.eu/prod/v1/staging/docs/howto/plugins/plugins.html). The [_entry point_](https://nomad-lab.eu/prod/v1/staging/docs/reference/glossary.html#plugin-entry-point) for a parser plugin is defined in `src/nomad_parser_pyscf/parsers/__init__.py` file.


## Matching the files to a parser {#matching-parser}

We are going to consider an example file generated by the software [PySCF](https://pyscf.org/) ([click here for download](https://box.hu-berlin.de/f/6d369b46a7084a118e63/)), and use it to parse data into the `nomad-simulations` schema. If you want to learn more about extending the `nomad-simulations` schema when the information is not defined, see [Extending the Schema](extending_schema.md).

In the template, go to the entry point in `src/nomad_parser_pyscf/parsers/__init__.py` and change the content to:
```python
from nomad.config.models.plugins import ParserEntryPoint


class PySCFEntryPoint(ParserEntryPoint):

    def load(self):
        from nomad_parser_pyscf.parsers.parser import PySCFParser

        return PySCFParser(**self.dict())


parser_entry_point = PySCFEntryPoint(
    name='PySCFParser',
    description='Parser for PySCF output written in a log text file.',
    mainfile_name_re='.*\.log.*',
    mainfile_contents_re=r'PySCF version [\d\.]*',
)
```

??? tip "Matching other mainfiles"
    In the `ParserEntryPoint` class, there are several attributes that can be defined to match a file. These can be mainfile name, a regular expression (regex) at the beginning of the mainfile, binary file headers, etc. For your specific use-case, you need to identify which is the mainfile and use it to match it to your parser. You can read more of the options in the NOMAD documentation page [Reference - Configuration - ParserEntryPoint](https://nomad-lab.eu/prod/v1/staging/docs/reference/config.html#parserentrypoint).

Note that we deleted the configuration parameter field (as it is not relevant for the purpose of this example) and changed some naming of files, `src/nomad_parser_pyscf/parsers/parser.py` for `parser.py`, as well as slightly its content:
```python
# all the other imports are here

configuration = config.get_plugin_entry_point(
    'nomad_parser_pyscf.parsers:parser_entry_point'
)


class PySCFParser(MatchingParser):
    def parse(
        self,
        mainfile: str,
        archive: 'EntryArchive',
        logger: 'BoundLogger',
        child_archives: dict[str, 'EntryArchive'] = None,
    ) -> None:
        print('Hello world!')
```

Note that we have also changed the class imported in `tests/parsers/test_parser.py` from `NewParser` to `PySCFParser`.
Finally, as we are going to use the `nomad-simulations` package, we need to add it into our dependencies in `pyproject.toml`:
```toml
dependencies = [
    "nomad-lab>=1.3.0",
    "nomad-simulations>=0.0.3",
]
```

In order to test the parser, we can install parser plugin in editable mode (with the flag `-e`):
```sh
uv pip install -e '.[dev]' --index-url https://gitlab.mpcdf.mpg.de/api/v4/projects/2187/packages/pypi/simple
```

??? note "Installing nomad-lab"
    Until we have an official pypi NOMAD release with the plugins functionality make sure to include NOMAD's internal package registry (via `--index-url` in the above command). Alternatively, you can add the following lines to the `pyproject.toml`:
    ```toml
    [tool.uv]
    index-url = "https://gitlab.mpcdf.mpg.de/api/v4/projects/2187/packages/pypi/simple"
    ```
    And install the package with:
    ```sh
    uv pip install -e '.[dev]'
    ```


With these changes, you can parse the `glycine.log` file and show the archive in the terminal (note we put the `glycine.log` file in the `tests/data/` sub-folder):
```sh
nomad parse tests/data/glycine.log
```

If everything went well, you should see a message run by the `PySCFParser().parse()` function:
```sh
Hello world!
```

In Python (in a module, Jupyter notebook, or in the interactive terminal), you can test the parsing by importing `PySCFParser` and specifying the path to the file. Note we also need to instantiate an empty `EntryArchive` in order to pass it as an input of the `parse()` function:
```python
from nomad.datamodel import EntryArchive
from nomad_parser_pyscf.parsers.parser import PySCFParser


archive = EntryArchive()

PySCFParser().parse(mainfile='<path-to-mainfile>/tests/data/glycine.log', archive=archive, logger=None)

# Other operations / analyses here
```


## Mapping into `nomad-simulations` {#mapping-to-nomad-simulations}

Now that you know the basics of instantiating and populating the `nomad-simulations` schema (see Assignments in the [NOMAD-Simulations](nomad_simulations.md)) and the basics of setting up a parser, let's combine both concepts and populate the `nomad-simulations` schema **from the output mainfile**. The implementation of this plugin will allow you to manage data in a controlled way. You can further use it for analysis tools or include these functionalities in your research workflows. You can read more details in the NOMAD documentation page [Plugins - How to write a parser](https://nomad-lab.eu/prod/v1/staging/docs/howto/plugins/parsers.html). 

Depending on the format of the mainfile, extracting data will be different and you will need to implement the parsing slightly different. You might also need to manipulate the data in order to adapt to the definitions of `nomad-simulations` (shapes, types, etc.):

- Unstructured text &ndash; You need to specify the regular expression ([regex](https://regexr.com/)) to match the text in the file in order to use it in the `parser.py` module. This is the case of our example, so you can read below how to implement it.
- Structured formats (HDF5, JSON, XML) &ndash; You can use Python libraries to extract the data (e.g., `h5py` for HDF5 files) and map it into the `nomad-simulations` schema. In the specific case of XML, you can use the [`XMLParser`](https://nomad-lab.eu/prod/v1/staging/docs/howto/plugins/parsers.html#other-fileparser-classes).

In our example, we have an unstructured text file, `glycine.log`. For simplicity, we will start with the [`Program`](nomad_simulations.md/#program) information: `name` and `version`. For unstructure text, we can use the NOMAD implementation class, `TextParser`. We need to:

1. Define a class inheriting from `TextParser`.
2. Overwrite its method `init_quantities()` and define `self._quantities` as a list of `nomad.parsing.file_parser.Quantity`. Note this is a different class than the one used for defining a schema, and we will refer to it as `ParsedQuantity` from now on.
3. `ParsedQuantity()` has a `key: value` structure and can take several arguments, being the most important ones:
    - First argument is the `key` string to identify the parsed quantity.
    - Second argument is the regex associated with that `key` to match the text.
    - `repeats` is a boolean used to repeat multiple times. If `repeats=True` and the regex matches, it returns a list. Otherwise it returns nothing or a singular value.
4. Instantiate the class of point 1 in the `parse()` function and define the path to its mainfile.


We can implement these steps in `src/nomad_parser_pyscf/parsers/parser.py`. First, we add these lines before `PySCFParser(MatchingParser)`:
```python
from typing import (
    TYPE_CHECKING,
)

if TYPE_CHECKING:
    from nomad.datamodel.datamodel import (
        EntryArchive,
    )
    from structlog.stdlib import (
        BoundLogger,
    )

from nomad.config import config
from nomad.parsing.parser import MatchingParser
from nomad.parsing.file_parser import Quantity as ParsedQuantity, TextParser


configuration = config.get_plugin_entry_point(
    'nomad_parser_pyscf.parsers:parser_entry_point'
)


class LogParser(TextParser):
    def init_quantities(self):
        self._quantities = [
            ParsedQuantity(
                'program_version', r'PySCF *version *([\d\.]+)', repeats=False
            )
        ]
```

Note we defined `LogParser()` and its `self._quantities` to match the `version` of the PySCF run. The `name` will be anyways set up to be `'PySCF'` during parsing.

If the regex matches, then this should return the value appearing after the string `'PySCF version'`, i.e., `'2.2.1'`. We can test this by:
```python
class PySCFParser(MatchingParser):
    def parse(
        self,
        mainfile: str,
        archive: 'EntryArchive',
        logger: 'BoundLogger',
        child_archives: dict[str, 'EntryArchive'] = None,
    ) -> None:
        log_parser = LogParser(mainfile=mainfile, logger=logger)
        print(log_parser.get('program_version'))
```

If we run now `nomad parse tests/data/glycine.log` we obtain:
```log
2.2.1
```

Now, we can instantiate `Simulation` and `Program` sections, and add them to the `archive`. This can be done by:
```python
# other imports here
from nomad_simulations.schema_packages.general import Simulation, Program

# `configuration` and `LogParser` defined here


class PySCFParser(MatchingParser):
    def parse(
        self,
        mainfile: str,
        archive: 'EntryArchive',
        logger: 'BoundLogger',
        child_archives: dict[str, 'EntryArchive'] = None,
    ) -> None:
        
        log_parser = LogParser(mainfile=mainfile, logger=logger)
        
        simulation = Simulation()
        program = Program(
            name='PySCF', version=log_parser.get('program_version')
        )
        simulation.program = program

        # Add the `Simulation` activity to the `archive`
        archive.data = simulation
```

Now, this populated schema can be output adding a `print` statement with the path to the section or quantity, after `archive.data` is assigned in the parser:
```python
        print(archive.data.program.name, archive.data.program.version)
```
Or we can use the flag `--show-archive` when running the `nomad parse` command to print the archive in the terminal:
```sh
nomad parse --show-archive tests/data/glycine.log
```
And we obtain:
```sh
{
  "data": {
    "m_def": "nomad_simulations.schema_packages.general.Simulation",
    "program": {
      "name": "PySCF",
      "version": "2.2.1"
    }
  },
  # other metadata here
}
```
