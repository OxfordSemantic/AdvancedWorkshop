# Starting RDFox

To run RDFox:

1. Open a terminal in VS Code and navigate to the **Advanced Reasoning Workshop** working directory.

2. Use the following command go start RDFox using the **start.rdfox** script (see 1.2 for details). The command you need will vary slightly depending on your system OS and CLI.

`<rdfox_filepath> sandbox . start`


### MacOS & Linux

E.g.
`./Users/Thomas/Desktop/RDFox/RDFox-macOS-arm64-7.2b/RDFox sandbox . start`

### Windows (PowerShell)

E.g.
`& /Users/Thomas/Desktop/RDFox/RDFox-macOS-arm64-7.2b/RDFox sandbox . start`

### Windows (CMD)

E.g.
`Users/Thomas/Desktop/RDFox/RDFox-macOS-arm64-7.2b/RDFox.exe sandbox . start`

## Sanity check

If everything has gone smoothly, you will see a message like this:


----------------------------------------
Source code for RDFox v1.0 Copyright 2013 Oxford University Innovation Limited and subsequent improvements Copyright 2017-2024 by Oxford Semantic Technologies Limited.

This system is equipped with 8.5 GB of RAM, and RDFox is configured to use at most 7.7 GB (89.9% of the total).
Currently, 105.4 MB (1.3% of the amount allocated to RDFox) appear to be available on the system.
Since RDFox is a RAM-based system, its performance can suffer when other running processes use a lot of memory.

Access control has been initialized by creating the first role with name "guest".
This copy of RDFox is licensed for Developer use to Tom Vout (tom.vout@oxfordsemantic.tech) of OST until 13-Oct-2025 01:00:00.

A new server connection was opened as role 'guest' and stored with name 'sc1'.
output = "out"
The REST endpoint was successfully started at port number/service name 12110 with 8 threads.
WARNING: The RDFox endpoint is running with no transport layer security (TLS). This could allow attackers to steal
         information including role passwords or authorization tokens. See the endpoint.channel variable and related
         variables in the description of the RDFox endpoint for details of how to set up TLS.

A new data store 'default' was created and initialized.

----------------------------------------