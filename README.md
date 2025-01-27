# Griptape Example Agent Structure

This is an example structure that can be used to run a Griptape Agent. The agent takes a string as an input, and returns a string after being sent to an LLM. It uses OpenAI's `gpt-4o`.

## Example:

**Input**: `Hi, how are you doing?`

**Output**: `Fantastic, thank you for asking. How can I help?`

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

## Running this Structure

### Griptape Cloud

You can create a run via the API or in the UI. 

**Input**: "Hello, how are you?"

### Locally

You can run this locally by passing an argument to your script.

```python
python structure.py "Hello, how are you?"
```
