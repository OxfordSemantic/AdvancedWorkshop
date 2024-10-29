# RDFox Advanced Reasoning Workshop

Welcome to the RDFox Advanced Reasoning Workshop!

In this workshop you will learn how to:
- write, debug and optimise advanced rules using all of RDFox's features for faster reasoning performance,
- manipulate sets of rule to take advantage of the most advanced functionality of reasoning,
- tips and tricks to improve your ability and understanding of reasoning with RDFox.

Contained within each notebook is:
- a simple example of a complex concept with details explanations of each part,
- a challenging rule-writing exercise,
- real world applications that are enabled by the kind of reasoning described.

## Get the workshop working

1. Open the **Advanced Reasoning Workshop** folder in [VS Code](https://code.visualstudio.com/) - this should be your working directory.

2. Create a virtual environment using `python3 -m venv venv` or `python -m venv venv`

3. Activate the virtual environment with one of the following:

| Platform | Shell | Command to activate virtual environment |
|----------|-------|-----------------------------------------|
| Mac/Linux | bash/zsh | `source venv/bin/activate` |
| Windows | cmd | `venv\Scripts\activate` |
| Windows | PowerShell | `venv\Scripts\Activate` |


4. Install the relevant dependencies by running `pip install -r requirements.txt`.

## How to use the workshop material

1. Start RDFox using the `start.rdfox` script from the `AdvancedWorkshop` working directory.

- Instructions can be found in `0-1_StartRDFox.ipynb` in the `exercises` folder.

3. Open an exercise notebook and follow the instructions within.

4. IMPORTANT: Run every cell in order within a given notebook

- They depend heavily on one another.

5. If, when you run a cell it does not stop, refer to `0-2_Help.ipynb`.

## Useful tips & tricks

1. Get the [RDFox VS Code Extension](https://marketplace.visualstudio.com/items?itemName=rdfox.rdfox-rdf) for syntax highlighting and in-IDE features.

2. When completing an exercise, split your editors viewport between the notebook and the relevant rule file.

## Helpful Resources

1. [Download the latest version of RDFox here](https://www.oxfordsemantic.tech/download)

2. [Request a free trial here](https://www.oxfordsemantic.tech/free-trial)

3. [See the RDFox Docs here](https://docs.oxfordsemantic.tech)
