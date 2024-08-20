# Developing parsers for FAIR computational data storage of spectroscopic simulations using the NOMAD-Simulations package

This documentation page will provide foundational knowledge for creating parser plugins in NOMAD to manage computational data of spectroscopic simulations. We will use the [`nomad-simulations`](https://pypi.org/project/nomad-simulations/) package to populate our data schema and extend it when needed. The steps are more general and can be used to create _any_ parser plugin, but in here we will use examples specifically related with theoretical spectroscopy simulations.

## Hackathon information

The FAIRmat Hackathon - FAIR Data Management of Spectroscopic Simulations, presented by FAIRmat Area C Computation, will take place from Wednesday, September 4, 2024 to Friday September 6, 2024. Check your email for the link if you have already registered, or register at the [Event Page](https://events.fairmat-nfdi.eu/event/25/){:target="_blank"}

To help facilitate discussions and provide prolonged assistance beyond the tutorial, we have created a [Hackathon FAIR spectra Sep2024 event channel](https://discord.gg/qgpHtPZwkt) in the NOMAD Discord server.

The structure of this documentation page is divided in two parts:

1. Understanding the [NOMAD-Simulations schema](nomad_simulations.md), what are its strengths and weaknesses, as well as learning how to extend it.
2. [How to create a parser plugin](parser_plugins.md), importing the data schema from the NOMAD-Simulations package and matching the files to be parsed.

You can find Assignments throughout these documentation pages which will help you understand the main concepts.


## General background

[NOMAD](https://nomad-lab.eu/nomad-lab/){:target="_blank"} is an open-source, community-driven data infrastructure, focusing on materials science data. Originally built as a repository for data from DFT calculations, the NOMAD software can automatically extract data from the output of a large variety of simulation codes. 

The key advantages of the NOMAD schema are summed up in **FAIR**mat's core values:

- **F**indable: a wide selection of the extracted data is indexed in a database, powering a the search with highly customizable queries and modular search parameters.
- **A**ccessible: the same database specifies clear API and GUI protocols on how retrieve the _full_ data extracted.
- **I**nteroperable: we have a diverse team of experts who interface with various materials science communities, looking into harmonizing data representations and insights among them. Following the NOMAD standard also opens up the (meta)data to the "NOMAD apps" ecosystem. <!-- Repeated in Parsers intro -->
- **R**eproducible: data is not standalone, but has a history, a vision, a workflow behind it. Our schema aims to capture the full context necessary for understanding and even regenerating via metadata.

We have presented and prepared several resources that can be visited after this Hackathon. You can find more information in the [Domain-specific NOMAD documentation page](https://nomad-lab.eu/prod/v1/staging/docs/examples/overview.html). If you want to know more, we also recommend you to check our latest Tutorial 14, the [Youtube playlist](https://www.youtube.com/watch?v=Al_wY2eqn6g&list=PLrRaxjvn6FDXiHpOKpRN_Phv14Qdqg7lp) as well as its [documentation page](https://fairmat-nfdi.github.io/fairmat-tutorial-14-computational-plugins/).