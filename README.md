# FAIRmat Hackathon - FAIR Data Management of Spectroscopic Simulations
Basic documentation for the FAIR Data Management of Spectroscopic Simulations in September 2024.

### How to launch locally for debugging

Clone and go into the documentation directory
```sh
git clone https://github.com/FAIRmat-NFDI/hackathon-fair-spectroscopy-data-2024.git
cd fairmat-tutorial-14-computational-plugins/
```

Create your own virtual environment with Python and activate it in your shell:
```sh
python3.11 -m venv .pyenvtuto
. .pyenvtuto/bin/activate
```
Always ensure that the environment is active. Else run the command above again.

Upgrade pip and install the requirements:
```sh
pip install --upgrade pip
pip install -r requirements.txt
```

You can launch locally the documentation:
```sh
mkdocs serve
```

The output on the terminal should have these lines:
```
...
INFO     -  Building documentation...
INFO     -  Cleaning site directory
INFO     -  Documentation built in 0.29 seconds
INFO     -  [14:31:29] Watching paths for changes: 'docs', 'mkdocs.yml'
INFO     -  [14:31:29] Serving on http://127.0.0.1:8000/
...
```
Then click on the http address to launch the MKDocs.
