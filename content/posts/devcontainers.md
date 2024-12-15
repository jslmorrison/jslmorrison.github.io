+++
title = "Dev containers"
date = 2024-12-15
tags = ["docker", "containers", "vscode"]
draft = false
+++

How to leverage dev containers in VSCode, in a new or existing project, to help maintain consistent development environments and reduce the time and effort required when 'onboarding' to another team or project.

<!--more-->

> A Dev Container in Visual Studio Code (VS Code) is a tool that allows developers to define and manage development environments using Docker containers. Unlike normal Docker containers, Dev Containers integrate seamlessly with VS Code, providing a consistent, reproducible, and collaborative development experience.

While this article describes the process of getting started with dev containers in VS Code they can also be used with JetBrains editors/IDE's.


## Getting started {#getting-started}

Before setting up a dev container, ensure you have the following installed:

1.  Docker:
    Install Docker Desktop (Windows/macOS) or Docker Engine (Linux).
2.  Visual Studio Code:
    Download and install VS Code from <https://code.visualstudio.com/>.
3.  Dev Containers Extension:
    Install the 'Dev Containers' extension from the VS Code Extensions Marketplace.
4.  Create new or open existing project
5.  Open the Command Palette in VS Code `(Ctrl+Shift+P or Cmd+Shift+P)` and search for `Dev Containers: Add Development Container Configuration Files`.
6.  Choose a Predefined Template:
    VS Code provides templates for popular development stacks (e.g., PHP, Python, Java). Select the template that best suits your project.

Alternatively just create the file manually.

At this point you will have a `.devcontainer` folder in your project directory which will contain the following:

-   `devcontainer.json`: The main configuration file that defines the container’s settings.
-   `Dockerfile` (optional): for custom container configurations.

    You can now modify the `devcontainer.json` to include any tools, extensions, or configurations you or your team requires. For example:

<!--listend-->

```json
{
  "name": "A PHP/MariaDB container",
  "build": {
    "dockerfile": "Dockerfile"
  },
  "settings": {
    "terminal.integrated.shell.linux": "/bin/zsh"
  },
  "extensions": [
      "bmewburn.vscode-intelephense-client",
      "ikappas.phpcs",
      "phpstan.phpstan-vscode"
  ],
  "postCreateCommand": "php composer install"
}
```

You don't have to manually define the extensions - just use VS Code as normal by searching for extensions and right clicking them to install in `devcontainer`.

**N.B.** If you are using a `compose` file to orchestrate multiple services:

-   Define all services in a docker `compose.yaml` file.
-   Update the `devcontainer.json` to specify the service used for the development environment, for example:

<!--listend-->

```json
{
  "service": "php-service",
  "dockerComposeFile": ["../docker-compose.yml"]
}
```

When you have finished editing the `devcontainer.json` file you can reopen the project in devcontainer via the command palette or restarting VS Code.

-   VS Code will build the container image (if required) and start a new container instance.
-   Your project will now run inside the container, ensuring a consistent environment.


## Advantages {#advantages}

In summary, the advantages of using dev containers are:

-   Consistency:
    Every team member works with the same tools, dependencies and configurations, eliminating the 'works on my machine' problem.
-   Simplified onboarding:
    New developers can start contributing quickly by simply cloning the repository and opening it in a Dev Container. All necessary dependencies and tools are pre-configured.
-   Isolation:
    Dev Containers prevent environment conflicts by isolating dependencies from the host machine.
-   Integrated Workflow:
    Unlike standalone Docker containers, Dev Containers integrate tightly with VS Code, enabling features like IntelliSense, debugging and version control within the container.
-   Reproducibility:
    The configuration files ensure that the same environment can be reproduced anywhere, whether locally or in CI/CD pipelines.
-   Customizability:
    Dev Containers can be tailored to include specific extensions, tools, or even custom runtime environments, making them flexible for diverse projects.

Therefore, by leveraging dev containers, development teams can streamline collaboration, maintain consistent environments, and boost productivity. They bridge the gap between containerised workflows and developer friendly tools, making them an excellent choice for modern development practices.
