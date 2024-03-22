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
onnxruntime
python-dotenv
```

```bash
pip install -r requirements.txt
```


## Assemble the agents {#assemble-the-agents}

We can now start writing the python script that will define the agents roles and capabilities and the agent specific tasks. Basically I defined 2 agents - a 'researcher' and a 'writer', each with their own specific task. I want them to research usage of crewAI in software development and output the result to a markdown file.

```python
researcher = Agent(
    role='Senior Research Analyst',
    goal='Uncover cutting-edge developments in use of AI agents and crewAI specifically in software development',
    backstory="""You are a Senior Research Analyst at a leading tech think tank.
        Your expertise lies in identifying emerging trends and technologies in AI and utilising AI agents
        software development. You have a knack for dissecting complex data and presenting
        actionable insights.""",
    verbose=True,
    allow_delegation=False,
    tools=[search_tool]
)
writer = Agent(
    role='Tech Content Strategist',
    goal='Craft compelling content on tech advancements',
    backstory="""You are a renowned Tech Content Strategist, known for your insightful
    and engaging articles on technology and innovation. With a deep understanding of
    the tech industry, you transform complex concepts into compelling narratives.""",
    verbose=True,
    # (optional) llm=ollama_llm, If you wanna use a local modal through Ollama, default is GPT4 with temperature=0.7
    allow_delegation=True
)

research_task = Task(
  description=(
    "Identify the next big trend in {topic}."
    "Focus on identifying pros and cons and the overall narrative."
    "Your final report should clearly articulate the key points"
    "its market opportunities, and potential risks."
  ),
  expected_output='A comprehensive 3 paragraphs long report on the latest AI trends.',
  tools=[search_tool],
  agent=researcher,
)
write_task = Task(
  description=(
    "Compose an insightful article on {topic}."
    "Focus on the latest trends and how it's impacting the industry."
    "This article should be easy to understand, engaging, and positive."
  ),
  expected_output='A 4 paragraph article on {topic} advancements formatted as markdown.',
  tools=[search_tool],
  agent=writer,
  async_execution=False,
  output_file='crewai-report.md'  # Example of output customization
)
```

[View full source code here.](https://github.com/jslmorrison/crewai-example)


## Summary {#summary}

This is just scratching the surface of the capabilities of the crewAI framework. Its relatively easy to get started and fun to work with. I would recommend it to others and I hope to use it more in future on more advanced tasks.

Finally, this was the result of the agents hard work:

> CrewAI: Transforming Software Development
>
> CrewAI, an open-source framework for orchestrating role-playing and autonomous AI agents, is revolutionizing the software development industry. By allowing engineers to assemble AI agents into cohesive, high-performing teams, it streamlines processes across multiple sectors. This innovative tool mirrors the dynamics of a real-world crew; AI agents assume roles, delegate tasks, and share goals, creating an ecosystem of efficiency and collaboration. The practical applications are limitless: imagine a 'Bug Tester' agent working harmoniously with a 'Code Reviewer' agent to enhance software quality.
>
> As CrewAI continues to evolve, we can expect more advanced features like hierarchical and consensual task processes, further enhancing the framework's utility. The diverse customization options for AI agents and task management have made CrewAI an efficient tool for both engineers and creatives. This versatility has contributed to its rising reputation as an effective solution for complex problems in the industry.
>
> However, the adoption of CrewAI is not without challenges. The learning curve can be steep, particularly for those new to AI-driven content creation or digital marketing analytics. Users need to invest significantly in training and upskilling to fully harness the capabilities of CrewAI. Cost considerations also factor in, with the pricing of CrewAI's services varying based on usage and feature requirements.
>
> Despite these challenges, the potential of CrewAI is undeniable. It presents an exciting opportunity for enhanced collaboration and efficiency in the software development industry. As with any technology, users must navigate the challenges to fully realize its potential. Through this balance, we can anticipate the pace and scale of CrewAI's adoption in the software development industry, transforming it for the better.
