# 1.2 RDFox Scripts

It's common to use scripts with RDFox to execute several RDFox commands with one input, either to perform repetitive actions while developing, for hands-free consistent setup, or to run several commands in one [transaction](https://docs.oxfordsemantic.tech/transactions.html#id1).

RDFox recognizes files of the type `.rdfox` as shell scripts, meaning these scripts can contain any RDFox commands that can be written in the shell.

Commands will be executed in line order.

## Running scripts

When running, RDFox will execute a script when it's file name is entering as a command. The file extension is not strictly necessary.

`file-name.rdfox`

or simply `file-name` will be accepted.

### Running commands (and scripts) on startup

The start-up command for RDFox can be modified to include shell commands (including scripts). A full stop is added after the RDFox mode, then the command(s) after that.

`./RDFox.exe sandbox . start`

**Have a look at the start.rdfox script we used on startup to see the very common startup commands.**
