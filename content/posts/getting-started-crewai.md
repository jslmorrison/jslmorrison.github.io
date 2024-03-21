+++
title = "Getting started with crewAI"
date = 2024-03-21
tags = ["nixos", "ai"]
categories = ["nixos"]
draft = false
+++

A basic outline in getting started with [crewAI](https://www.crewai.io/) framework in order to evaluate it further.

<!--more-->

What is crewAI and how does it help me? According to their [documentation website](https://docs.crewai.com/), crewAI is a:

> Cutting-edge framework for orchestrating role-playing, autonomous AI agents. By fostering collaborative intelligence, CrewAI empowers agents to work together seamlessly, tackling complex tasks.

That is probably better explained by an example and some are listed on the site, but I want to try implementing the introductory example.


## Setup {#setup}


#### Installing requirements {#installing-requirements}

I need the following available in a dev env shell:

-   python
-   pip

I'll create a `flake.nix` file in order to have those available after executing `nix develop` in the terminal:

```nix
{
  description = "A dev env with Python 3 and pip";

  inputs.nixpkgs.url = "github:nixos/nixpkgs/nixpkgs-unstable";

  outputs = { self, nixpkgs }:
    let
      system = "x86_64-linux";
      pkgs = import nixpkgs { inherit system; };
    in
    {
      devShells.${system}.default = pkgs.mkShell {
        buildInputs = with pkgs; [
          python311Full
          python3Packages.pip
        ];
      };
    };
}
```


#### Create a python virtual env {#create-a-python-virtual-env}

```bash
python -m venv .venv
```


#### Activate the virtual env {#activate-the-virtual-env}

```bash
source .venv/bin/activate
```

Check the path to the python executable:

```nil
which python
/home/john/dev/crewai/.venv/bin/python
```

and I can see it has changed as expected from the previous path within the nix store.
The required packages can now be installed and I list them in a `requirements.txt` file:

```txt
crewai
crewai[tools]
duckduckgo-search
langchain-community
python-dotenv
```

```bash
pip install -r requirements.txt
```


## Assemble the agents {#assemble-the-agents}

We can now start writing the python script that will define the agents roles and capabilities and the agent specific tasks.

To be continued...
