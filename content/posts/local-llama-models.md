+++
title = "Local Ollama models"
date = 2024-02-11
draft = false
+++

How to install and run open source LLM's locally using [Ollama](https://ollama.com/) and integrate it into VSCode editor for assisted code completion etc.

<!--more-->

> Ollama is a tool that allows you to run open-source large language models (LLMs) locally on your machine to provide flexibility in working with different models

You can view a list of [supported models here](https://ollama.com/library).


## Running Ollama {#running-ollama}

For me running Ollama locally is a simple as executing the following in a terminal:

```bash
nix shell nixpkgs#ollama --command ollama serve
```

This will download Ollama and start the server. If all good at this point you should see in the terminal output that it is `Listening on 127.0.0.1:11434`. If you open that URL in a browser then you should see `Ollama is running`.
To run a specific model, browse to the [Ollama models library](https://ollama.com/library) and pick one that suits your needs. For example:

```bash
nix shell nixpkgs#ollama run llama2
```

> Llama 2 is released by Meta Platforms, Inc. This model is trained on 2 trillion tokens, and by default supports a context length of 4096. Llama 2 Chat models are fine-tuned on over 1 million human annotations, and are made for chat.


## Interaction {#interaction}

After running the model as shown above, once it finishes downloading it will provide you with a prompt to start chatting:

```bash
>>> hello
Hello! It's nice to meet you. Is there something I can help you with or would you like
to chat?

>>> Send a message (/? for help)
```

There is also an API available that you can send requests to:

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "llama2",
  "prompt": "Hello"
}'
```

You can [view the API docs here](https://github.com/ollama/ollama/blob/main/docs/api.md).

To install a different model, repeat the run command above, specifying a different model:

````bash
nix shell nixpkgs#ollama --command ollama run codellama "Write me a PHP function that outputs the fibonacci sequence"

Here is a PHP function that outputs the Fibonacci sequence:
```
function fibonacci($n) {
    if ($n <= 1) {
        return $n;
    } else {
        return fibonacci($n-1) + fibonacci($n-2);
    }
}
```
This function takes an integer `$n` as input and returns the `n`-th number in the
Fibonacci sequence. The function is based on the recurrence relation for the Fibonacci
sequence, which states that each number is equal to the sum of the previous two numbers.
The function uses a recursive approach, where it calls itself with the previous two
numbers as input until it reaches the desired output.

For example, if we call the function with `$n = 5`, it will return `8`, since `8` is the
fifth number in the Fibonacci sequence.
```
echo fibonacci(5); // Output: 8
```
Note that this function has a time complexity of O(`2^n`), which means that the running
time grows very quickly as the input increases. This is because each call to the
function creates a new stack frame, and the function calls itself with smaller inputs
until it reaches the base case. As a result, the function can become very slow for large
values of `n`.
````

To see a list of installed models:

````bash
nix shell nixpkgs#ollama --command ollama list
````


## Integration with VSCode {#integration-with-vscode}

Install and configure the `llama-coder` extension from the [VSCode marketplace](https://marketplace.visualstudio.com/items?itemName=ex3ndr.llama-coder).

> Llama Coder is a better and self-hosted Github Copilot replacement for VS Studio Code. Llama Coder uses Ollama and codellama to provide autocomplete that runs on your hardware.

I'll do a follow up on this with my findings later, after I have some more time using this and compared other models.
