# Adding Parameters

Let's improve our Agent by giving the user the option of specifying some parameters. For example, let's give the user the ability to change the model being used from "gpt-4o" to "gpt-3.5-turbo" or 'gpt-4o-mini".

## Update the code

### Import OpenAiChatPromptDriver

First we'll need to import the ChatPromptDriver.

```python
from griptape.drivers import OpenAiChatPromptDriver
```

### Add ChatPromptDriver to the Agent creation

```python
    Agent(
        prompt_driver=OpenAiChatPromptDriver(model="gpt-4o"), tools=[DateTimeTool()]
    ).run(*args)
```

This will still execute exactly the same as before, but now we're specifying the model in the driver.

The next step will be to provide the ability to pass arguments.

## Passing arguments

There are a number of libraries available in Python for passing arguments. You can use the default [`argparse`](https://docs.python.org/3/library/argparse.html), [`click`](https://click.palletsprojects.com/en/stable/), or [`typer`](https://typer.tiangolo.com/).

All have their merits - `argparse` of course coming with Python by default, but it's rather complicated to set up. Both `click` and `typer` are easier to use, so it's really up to you which you'd like.

Since we're all very advanced Python developers - let's go with `typer` because it's more advanced, has better help documentation, and seems to be called out on redit quite a bit.

## Getting Started with Typer

First, let's install Typer:

```bash
pip install typer
```

### Step 1: Basic Typer Setup

Let's convert your script to use Typer in its simplest form:

First, we'll import it.

```python
import typer
```

Next, we'll create our `app` instance so typer knows we're making an application.

```python
app = typer.Typer(add_completion=False)
```

Then, we'll create a function that will be our `command`. With typer this is done with an @app.command() decorator. This function will take our parameters. Notice we add a type to the parameter.

```python
@app.command()
def run(prompt: str):
    """Run the agent with a prompt."""
    setup_cloud_listener()
    Agent(
        prompt_driver=OpenAiChatPromptDriver(model="gpt-4o"), tools=[DateTimeTool()]
    ).run(prompt)
```

Finally, we'll call `app`.

```python
if __name__ == "__main__":
    app()
```

Here's the whole code:

```python
import os
import typer
from griptape.drivers import GriptapeCloudEventListenerDriver, OpenAiChatPromptDriver
from griptape.events import EventBus, EventListener
from griptape.structures import Agent
from griptape.tools import DateTimeTool

def setup_cloud_listener():
    # Are we running in a managed environment?
    if "GT_CLOUD_STRUCTURE_RUN_ID" in os.environ:
        # If so, the runtime takes care of loading the .env file
        EventBus.add_event_listener(
            EventListener(
                event_listener_driver=GriptapeCloudEventListenerDriver(),
            )
        )
    else:
        # If not, we need to load the .env file ourselves
        from dotenv import load_dotenv
        load_dotenv()

app = typer.Typer(add_completion=False)

@app.command()
def run(prompt: str):
    """Run the agent with a prompt."""
    setup_cloud_listener()
    Agent(
        prompt_driver=OpenAiChatPromptDriver(model="gpt-4o"), tools=[DateTimeTool()]
    ).run(prompt)

if __name__ == "__main__":
    app()

```

You can now run this like:

```bash
python structure.py "What's the current time?"
```

The cool thing is you get help for free! Try:

```bash
python structure.py --help
```

Heres the help you'll see:

```
 Usage: structure.py [OPTIONS] PROMPT

 Run the agent with a prompt.

╭─ Arguments ───────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ *    prompt      TEXT  [default: None] [required]                                                                         │
╰───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─ Options ─────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ --help          Show this message and exit.                                                                               │
╰───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯

```

### Step 2: Adding a Model Parameter

We'll add the model as a parameter with a simple string, and adjust the `OpenAiChatPromptDriver` to take it.

```python
@app.command()
def run(
    prompt: str,
    model: str = typer.Option("gpt-4o", "--model", "-m", help="The model to use")
):
    """Run the agent with a prompt and specified model."""
    setup_cloud_listener()
    Agent(
        prompt_driver=OpenAiChatPromptDriver(model=model),
        tools=[DateTimeTool()]
    ).run(prompt)
```

Now you can run it like:

```bash
python structure.py "What time is it?" --model gpt-3.5-turbo
# or
python structure.py "What time is it?" -m gpt-3.5-turbo
```

### Step 3: Making it Better with Enums

Let's make the model parameter more robust using an Enum:

First import and create the Enum class:

```python
from enum import Enum

class Model(str, Enum):
    GPT4 = "gpt-4o"
    GPT35 = "gpt-3.5-turbo"
    GPT4_MINI = "gpt-4o-mini"
```

Then, modify the parameter to use it:

```python
@app.command()
def run(
    prompt: str,
    model: Model = typer.Option(Model.GPT4, "--model", "-m", help="The model to use")
):
```

Now when you run `--help`, you'll see the available model choices:

```bash
python structure.py --help
```

And if you try to use an invalid model, Typer will show an error:

```bash
python structure.py "Hello" --model invalid-model  # This will fail with a helpful error
```

The valid ways to run it are:

```bash
python structure.py "Hello"  # Uses default (gpt-4o)
python structure.py "Hello" --model gpt-3.5-turbo
python structure.py "Hello" -m gpt-4o-mini
```

What's cool about this progression is how Typer handles more and more complexity for us:

1. First version: Basic command-line argument handling
2. Second version: Added optional parameter with a default
3. Final version: Added type safety with enums and automatic validation

## Update requirements.txt

Before we push this code, we'll need to update `requirements.txt` to include `typer`, otherwise our code won't run correctly when it's deployed to Griptape Cloud.

```
griptape
python-dotenv
typer
```

## Check in your changes

Next, let's check in the changes we've made. In VSCode, go to the sidebar and click on the `source control` button. This should show you what files have changed.

![source control](../images/source_control_button.png)

We need to **stage** the changes. Hover over the **Changes** header and you'll see a `+` icon. That lets you move the changes up so you'll be ready to commit them.

![ready_to_stage](../images/ready_to_stage.png)

After you click the button, you'll see them up in the staged area.

![alt text](../images/staged.png)

Then enter a commit message and click **Commit**.

![commit message](../images/commit_message.png)

## Sync

Once you've hit commit, you will be asked to sync your changes to the server. This will push them up to GitHub and update the repo.

![sync](../images/sync_changes.png)

Click the button.

If you go back over to GitHub now, you should see the updated changes in your GitHub repo.

![alt text](../images/github_repo_updated.png)

## Check the Structure

Your structure should automatially be re-deployed based on this updated repo. It may take a few minutes because the `requirements.txt` file was updated, so the docker image will need to be regenerated.

Once it's successfuly updated, you are ready to test it with a new run.

## Run the structure

To run this updated repo and pass a parameter, you can choose **Create run**

Enter the Arguments with one on each line:

![arguments](../images/arguments.png)

You can see the Input/Output info in the tab for that run. Notice how each line is a separate argument:

![input/output](../images/input_output_params.png)

Try an example where you input a bad model name:

![not a real model](../images/not_a_real_model.png)

You'll get a failed result, and when you check out the logs you'll see why:

![why failed](../images/why_failed.png)

## Update README.md

This is great - you now have the ability to pass a parameter to the structure, and you get a proper error message if it's done incorrectly. However, the README hasn't been updated to correctly reflect the new functionality. Let's do that.

### Update the README.md in VSCode

Jump back over to Visual Studio Code and select `README.md`.

Then modify the readme to be what you'd like.

````
# Griptape Demo Assistant

This is an example structure that can be used to run a Griptape Agent. The agent takes a string as an input, and returns a string after being sent to an LLM. It uses OpenAI, and allows you to specify the model using the `--model` parameter.

## Example:

**Input**: `Hi, how are you doing?`

**Output**: `Fantastic, thank you for asking. How can I help?`

## Paramters

You can pass a `model` parameter to the structure, specifying what OpenAI model is used.

```
--model  -m      [gpt-4o|gpt-3.5-turbo|gpt-4o-mini]  The model to use [default: gpt-4o]
```

## Running this Structure

### Griptape Cloud

You can create a run via the API or in the UI.

**Arguments**:
```
Hello, how are you?
```

If you want to pass the model as a parameter, add the `--model` parameter and then the model you'd like to use, each on a separate line:

```
Hello, how are you?
--model
gpt-3.5-turbo
```

### Locally

You can run this locally by passing an argument to your script.

```python
python structure.py "Hello, how are you?" --model gpt-3.5-turbo
```

## Requirements

The structure requires two API keys:

* [Open AI Key](https://platform.openai.com/api-keys)
* [Griptape Cloud Key](https://cloud.griptape.ai/configuration/api-keys)

## Configuration

Save the following keys in you `.env` if running locally, or add them to the structure if running in Griptape Cloud.

```.env
OPENAI_API_KEY=<encrypted_value> # Fill in with your own key
GT_CLOUD_API_KEY=<encrypted_value> # Fill in with your own key
```

````

### Sync changes

Once you're finished, push your changes back up to the repo the same way you did before. Because this change is only affecting the README, the deployment should be very fast.

---

Amazing! You've now got a structure that's deployed in Griptape Cloud. That structure is pointing to a GitHub repo that you can push changes to either through the GitHub web ui, or by making changes locally.

Great work! Let's now look at other ways we can use a structure.

---

[← Working locally](working_locally.md) | [Using Structures - Framework →](using_structures_framework.md)
