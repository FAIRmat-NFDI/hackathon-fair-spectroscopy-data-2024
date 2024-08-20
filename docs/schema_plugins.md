# Extending the NOMAD-Simulations schema

As you develop your parser, you may find that the `nomad-simulations` package does not include some relevant quantities for your particular use case. You can easily extend upon the schema by adding your own custom schema under the `schema_packages/` directory in your parser plugin. For this, you should utilize and build upon `nomad-simulations` for consistency and future compatibility or integration. Below is a decision tree which illustrates the schema development process:

<div class="click-zoom">
    <label>
        <input type="checkbox">
        <img src="../assets/decision_tree.png" alt="Schema extension decision tree." width="80%" title="Click to zoom in">
    </label>
</div>


From this schematic, we can identify 3 distinct uses or extensions of `nomad-simulations`:

1. Direct use of existing section definitions (without extension):
    - See how to use them in [Parser Plugins](parser_plugins.md).
2. Semantic extension of section definitions:
    - Create sections that re-define the context through inheritance from existing sections. Quantities definitions within the inherited section can be overwritten.
    - Create brand new sections.
3. Extending section definitions for normalization functionalities:
    - The section normalizations (see [Extra: The `normalize()` class function](nomad_simulations.md#normalize-function)) can be overwritten to leverage certain tasks out of the parsers.


You can find more information about writing schemas packages in [How to write a schema package](https://nomad-lab.eu/prod/v1/docs/howto/plugins/schema_packages.html) of the general NOMAD documentation.

To start developing a custom schema and for the example in this page, add the schema packages. This will create a sub-folder `src/<parser_name>/schema_packages` and a Python file `mypackage.py` in your parser plugin project.

Suppose you are developing a parser for a well-defined schema specified in a structured file format, e.g., HDF5. The simulation data populates the [groups, datasets, and attributes](https://docs.h5py.org/en/stable/index.html) of this HDF5 file and we want to map this information into our `nomad-simulations` schema. The structured content of our file is:
```
<file.hdf5>
    +-- schema_version: Integer[3]
    \-- hdf5_generator
    |    +-- name: String[]
    |    +-- version: String[]
    \-- program
        +-- name: String[]
        +-- version: String[]
```

??? note "HDF5 syntax"
    `\-- item` &ndash;
    An object within a group, that is either a dataset or a group. If it is a group itself, the objects within the group are indented by five spaces with respect to the group name.

    `+-- attribute:` &ndash;
    An attribute, that relates either to a group or a dataset.

    `\-- data: <type>[dim1][dim2]` &ndash;
    A dataset with array dimensions `dim1` by `dim2` and of type `<type>` (following the HDF5 Datatype).


!!! abstract "Assignment 4.1"
    Which quantities within this HDF5 file can we store using existing sections within `nomad-simulations` and which require the creation of new schema sections? For the quantities that directly use the existing schema, write some code to demonstrate how your parser would populate the archive with this information.

??? success "Solution 4.1"
    The program information can be stored under the existing `Program` sections within `Simulation`. However the HDF5 file `version` and `hdf5_generator` information require some extensions to the schema.

    For populating the program information, the parser code would be:

    ```python
    import h5py
    from nomad_simulations.schema_packages.general import Simulation, Program


    class MyParser(MatchingParser):

        # other functions here
        # ...

        def parse(
            self,
            mainfile: str,
            archive: EntryArchive,
            logger: BoundLogger,
            child_archives: dict[str, EntryArchive] = None,
        ) -> None:
            # The specifics of opening a HDF5 file are not important for the purpose of this Assignment
            h5_data = h5py.File(mainfile, 'r')

            simulation = Simulation()
            simulation.program = Program(
                name=h5_data['program'].attrs['name'],
                version=h5_data['program'].attrs['version']
            )

            archive.data = simulation
    ```

The `hdf_generator` information is referring to a secondary software used to create the HDF5 file, so it makes sense to store this in a section very similar to `Program`. In this section, we have defined several quantities (see [NOMAD-Simulations: Program](nomad_simulations.md/#program)) that make sense in the context of _any_ software, `name` and `version` included. So, in this case we can simply reuse this section and add a new subsection to `Simulation`.

!!! abstract "Assignment 4.2"
    Extend the `Simulation` section by defining a new sub-section called `hdf_generator` of type `Program`. This new sub-section does not repeat.

??? success "Solution 4.2"
    We can use the `nomad-simulations` section `Simulation` to inherit in a new class called `HDF5Simulation` defined in `schema_packages/<parser_name>_schema.py`:
    ```python
    import nomad_simulations

    class HDF5Simulation(nomad_simulations.schema_packages.general.Simulation):
        pass
    ```

    We can define a new `SubSection` under our `HDF5Simulation` as:
    ```python
    import nomad_simulations
    from nomad.metainfo import SubSection


    class HDF5Simulation(nomad_simulations.schema_packages.general.Simulation):

        hdf5_generator = SubSection(
            sub_section=nomad_simulations.schema_packages.general.Program.m_def, repeats=False
        )
    ```

    The `sub_section` attribute of `SubSection` specifies that this new section under `Simulation` is of type `Program` and will thus inherit all of its attributes. The `repeats` attribute is not necessary, as it defaults to `False`, but we added it for pedagogical reasons.

    Now we can use this new section within our parser to store the `hdf_generator` information:
    ```python
    import h5py
    from nomad_simulations.schema_packages.general import Program
    from <parse_plugin>.schema_packages.<parser_name>_schema.py import HDF5Simulation


    class MyParser(MatchingParser):

        # other functions here
        # ...

        def parse(
            self,
            mainfile: str,
            archive: EntryArchive,
            logger: BoundLogger,
            child_archives: dict[str, EntryArchive] = None,
        ) -> None:
            # The specifics of opening a HDF5 file are not important for the purpose of this Assignment
            h5_data = h5py.File(mainfile, 'r')

            simulation = HDF5Simulation()
            simulation.program = Program(
                name=h5_data['program'].attrs['name'],
                version=h5_data['program'].attrs['version']
            )
            simulation.hdf5_generator = Program(
                name=h5_data['hdf_generator'].attrs['name'],
                version=h5_data['hdf_generator'].attrs['version']
            )

            archive.data = simulation
    ```


!!! abstract "Assignment 4.3"
    Add a quantity `hdf5_schema_version` with the appropriate `type`, `shape`, and `description` to your `HDF5Simulation` section in `schema_packages/<parser_name>_schema.py`.


??? success "Solution 4.3"
    In this case, we will use `Quantity` to define `hdf5_schema_version`:
    ```python
    import nomad_simulations
    from nomad.metainfo import Quantity, SubSection
    import numpy as np


    class HDF5Simulation(nomad_simulations.schema_packages.general.Simulation):

        hdf5_generator = SubSection(
            sub_section=nomad_simulations.schema_packages.general.Program.m_def, repeats=False
        )

        hdf5_schema_version = Quantity(
            type=np.int32,
            shape=[3],
            description="""
            Specifies the version of the HDF5 schema being followed, using sematic versioning.
            """,
        )
    ```

    Since the `schema_version` uses [semantic versioning](https://semver.org/), we need to define the `hdf5_schema_version` quantity as a list of 3 integers. In the parser:
    ```python
    import h5py
    from nomad_simulations.schema_packages.general import Program
    from <parse_plugin>.schema_packages.<parser_name>_schema.py import HDF5Simulation


    class MyParser(MatchingParser):

        # other functions here
        # ...

        def parse(
            self,
            mainfile: str,
            archive: EntryArchive,
            logger: BoundLogger,
            child_archives: dict[str, EntryArchive] = None,
        ) -> None:
            # The specifics of opening a HDF5 file are not important for the purpose of this Assignment
            h5_data = h5py.File(mainfile, 'r')

            simulation = HDF5Simulation()
            simulation.program = Program(
                name=h5_data['program'].attrs['name'],
                version=h5_data['program'].attrs['version']
            )
            simulation.hdf5_generator = Program(
                name=h5_data['hdf_generator'].attrs['name'],
                version=h5_data['hdf_generator'].attrs['version']
            )
            simulation.hdf5_schema_version = h5_data.attrs['version']

            archive.data = simulation
    ```