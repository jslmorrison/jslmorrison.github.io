+++
title = "Automated testing with Playwright"
date = 2024-06-18
tags = ["nixos", "testing", "playwright"]
draft = false
+++

[Playwright](https://playwright.dev/python/) enables reliable end-to-end testing for modern web apps - any browser, any platform, one API.

Read on to get it installed and running on nixos.

<!--more-->


## Installation {#installation}

> Playwright was created specifically to accommodate the needs of end-to-end testing. Playwright supports all modern rendering engines including Chromium, WebKit, and Firefox.
>
> The Playwright library can be used as a general purpose browser automation tool, providing a powerful set of APIs to automate web applications, for both sync and async Python.

To begin, create `flake.nix` file with the following content:

```nix
{
  description = "A dev shell for running playwright e2e tests";

  inputs.nixpkgs.url = "github:nixos/nixpkgs/nixpkgs-unstable";

  outputs = { self, nixpkgs }:
    let
      system = "x86_64-linux";
      pkgs = import nixpkgs { inherit system; };
    in
    {
      devShells.${system}.default = pkgs.mkShell {
        buildInputs = with pkgs; [
          playwright-driver
          python312Full
          python312Packages.playwright
          python312Packages.pip
          python312Packages.pytest
        ];

        shellHook = ''
            export PLAYWRIGHT_BROWSERS_PATH=${pkgs.playwright-driver.browsers}
            export PLAYWRIGHT_SKIP_VALIDATE_HOST_REQUIREMENTS=true
        '';
      };
    };
}
```

then enter the development shell and create new virtual env:

```bash
nix develop
```

and create and activate new virtual env:

```bash
python -m venv .venv
source .venv/bin/activate
```

Put the pip requirements in a file named `requirements.txt` with the following content:

```text
pytest-playwright
```

and install requirements with:

```bash
pip install -r requirements.txt
```


## Create test(s) {#create-test--s}

We are now ready to start writing tests with Python. Create a new test file with the following content:

```python
# example_test.py
import re
from playwright.sync_api import Page, expect

page.goto("https://google.co.uk/about")
# Expect a title "to contain" a substring.
expect(page).to_have_title(re.compile("About Google"))
```


## Run test(s) {#run-test--s}

From the terminal:

```bash
pytest --tracing on
```

You should see some output in the terminal about the number of tests ran, passed or failed etc. Given that you have passed the `--tracing` flag when running the tests, the traces for the tests can be found in the `test-results/` directory and these can be viewed by running another command in the terminal:

```bash
playwright show trace test-results/test-example-py-test-has-title-chromium/trace.zip
```

Note how the trace includes the test and the browser as part of the directory structure. `chromium` is the default browser here.

That is a brief overview of getting started with Playwright for e2e testing. It can do so much more. Read more in the [official docs](https://playwright.dev/python/docs/writing-tests).
