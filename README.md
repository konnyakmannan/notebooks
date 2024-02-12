# notebooks

This repository provides notebooks for my blog.

## Getting Started

Follow these steps to set up your environment:

```shell
# Create a virtual environment using Python 3.11
py -3.11 -m venv .venv

# Activate the virtual environment
# On Unix or MacOS:
source .venv/bin/activate
# On Windows:
.\.venv\Scripts\activate

# Upgrade pip to the latest version
python -m pip install --upgrade pip

# Install pip-tools for managing dependencies
python -m pip install pip-tools

# Synchronize the environment with the requirements.txt file
pip-sync
```

You can then run the notebook in a subdirectory.

```
# Change to the subdirectory
cd ./subdirectory

# Convert Markdown files to Jupyter Notebook files
jupytext --to ipynb index.md

# Start Jupyter Lab without opening a browser
jupyter lab --no-browser
```

