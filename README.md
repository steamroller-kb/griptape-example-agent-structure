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

- [Open AI Key](https://platform.openai.com/api-keys)
- [Griptape Cloud Key](https://cloud.griptape.ai/configuration/api-keys)

## Configuration

Save the following keys in you `.env` if running locally, or add them to the structure if running in Griptape Cloud.

```.env
OPENAI_API_KEY=<encrypted_value> # Fill in with your own key
GT_CLOUD_API_KEY=<encrypted_value> # Fill in with your own key
```
