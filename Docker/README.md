
Docker allows you to bundle up all the installation procedures into a text file.
This text file will be executed using the docker interpreter, creating a virtual-machine-like environment that will contain all the settings that the creator of the docker file specified.

__Docker basically requires root access, which is the main reason why it is often not supported in scientific computing cluster environments! You may want to look into Singular (e.g., [here](http://geekyap.blogspot.ch/2016/11/docker-vs-singularity-vs-shifter-in-hpc.html) or [here](https://www.hpcwire.com/2017/05/04/singularity-hpc-container-technology-moves-lab/) instead.

A very good step-by-step introduction to docker by [Andrew Odewahn of O'Reilly](http://odewahn.github.io/docker-jumpstart/introduction.html)

A briefer, but still somewhat granular overview may be found at our [Hackathon's page](https://github.com/NCBI-Hackathons/Cancer_Epitopes_CSHL/blob/master/doc/Docker.md) where we supplied a docker image to run our pipeline.
