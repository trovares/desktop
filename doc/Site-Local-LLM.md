# Site-Local LLM Configuration Guide

This guide provides instructions for system administrators to configure site-local Large Language Models (LLMs) for your application.  The two categories of information you will need for each LLM are API details and any authentication that may be required for each user.

## How the Trovares Desktop calls an LLM

There is a Python dictionary called `LLM_CONFIG` that is used by the Trovares Desktop (TD) to produce the choices available for the LLM selection box.
Each dictionary entry contains:

  - **key**:  used as the display name of the LLM throughout the TD.
  - **value**:  a Python dictionary containing enough information to allow TD to send requests to the LLM and retrieve responses.

Here is a sample `LLM_CONFIG` data structure:
```python
LLM_CONFIG = {
    'LLM-Number-1': {
        'fields': [],
        'model': 'gpt-1o',
        'callback': 'my_site_local_llm_1',
    }
}
```

The choice of LLMs will include 'LLM-Number-1'.  When selected in the LLM selection box and a call to the selected LLM is requested, The callback function is invoked with the following function parameters:

```python
def my_site_local_llm_1(llm_config:Dict, llm_credentials:Dict={},
                        question:str=None, prompt:str=None,
                        history:list=[]):
```

Note that the first parameter, `llm_config`, is the value of the `LLM_CONFIG` dictionary for the selected llm string as the key in the dictionary.
The `fields` array and the `model` name are expected to exist in the `llm_config`.
Site administrators are free to add any additional information to the llm_config entry such as `temperature` and `max_tokens`.

## Parameters passed to the callback function

The parameter values are populated differently depending upon where in the TD the request for the LLM occurred.

  - The `llm_config` has already been described above.
  - The `llm_credentials` are retrieved from the user's persistent store for any credentials described in the `fields` array of the config.
These credentials contain the values provided by the user on the `Settings` page.
  - The `question` is whatever the user typed into the related box on the page.  This could be the question box on the Data Explorer page, the Schema Guidance box on the Schema Explorer page, or the message box on the Chat popup.
  - The `prompt` is automatically provided by TD based on the context of the LLM request.  In many cases, this prompt will include descriptions of graph schemas.
  - The `history` array is populated with recent history of questions and responses.


# Configuration Steps

## Step 1:  Obtain API details for your LLM

Understanding the API details for your LLM is necessary to write Python scripting that calls your LLM and is able to extract responses for processing in the TD.
A critical part of this is understanding the response JSON structure so your Python code can retrieve the LLM answer.

Sample LLM API Structure:

  - **URL**: https://my.local.llm:6502/api/v1/query
  - **Method**: POST
  - **Headers**:
    - Authorization: Bearer <API_KEY>
    - Content-Type: application/json
  - **Request Payload**:
    - model: The model identifier (e.g., "gpt-3")
    - prompt: The input text or question
    - max_tokens: Maximum number of tokens to generate
    - temperature: Sampling temperature (range 0.0 to 1.0)
    - history: List of previous interactions (optional)
  - **Response**:
    ```json
    {
        "id": "cmpl-5b8b8b8b8b8b8b8b8b8b8b8b8b8b8b8",
        "object": "text_completion",
        "created": 1612304000,
        "model": "gpt-3",
        "choices": [
            {
            "text": "The capital of France is Paris.",
            "index": 0,
            "logprobs": null,
            "finish_reason": "length"
            }
        ]
    }
    ```

## Step 2:  Obtain required LLM credentials

This step involves understand what credentials are needed to authenticate with the LLM, not the actual values.
The values are supplied by each user for their settings.
Configuring the LLM consists of naming each credential and understanding its data types or constraints.
For example, if an authentication key is needed, its type is a string, but you could add information such as a regular expression that allows the TD to confirm that the user has entered this setting information in a proper format.

There can be any number of these credential settings for each LLM.
Each of authentication credentials are defined in the `fields` array within the `LLM_CONFIG` entry for the LLM.

It may be that an on-premises LLM has access controlled by a firewall within an enterprise network, in which has no authentication credentials are needed and the `fields` array can be an empty array:  `'fields': [],`.

Here is an example for OpenAI API Key:
```python
'fields': [
    {
        'name': 'openai_api_key',
        'label': 'OpenAI API Key',
        'type': 'text',
        'pattern': r'^sk-([a-zA-Z0-9]{20}|[a-zA-Z0-9\-]{42})T3BlbkFJ([a-zA-Z0-9]{20}|[a-zA-Z0-9\-]{42})$',
        'mask': True
    },
],
```

  - **name**: the variable name used in the Python script to refer to this credential.
  - **label**: the text to display on the TD for this credential.
  - **type**: the data type for the python variable.
  - **pattern**: the regular expression (RE) to use to validate the correct format of this credential.
    Note that this RE is provided here in Python, but is used in the javascript front-end, so the RE must be expressed in a valid javascript format.
  - **mask**: controls whether to mask or hide part of the credential after it is has been added by a user in  their settings.

## Step 3:  Decide LLM configuration strategy

After preparing steps 1 and 2 above for each LLM to be configured, it is time to insert them into the operational `LLM_CONFIG` used by TD.

There are several LLMs that come pre-configured with the TD that require access to the open internet; some also require AWS account keys.

If the open internet is not available from your enterprise network, then you should simply replace all values in the `LLM_CONFIG` data structure with your site-local values.  This is done by including `LLM_CONFIG_local_replace` in your site-local configuration file.

```python
# Replace existing (all the default configuration) with these
LLM_CONFIG_local_replace = {
    'LLM-Number-1': {
        # your LLM configuration
    },
}
```

You may prefer to add to the existing LLMs that are pre-configured and optionally remove some of the existing LLMs.  This strategy involves including `LLM_CONFIG_local_add` and optionally `LLM_CONFIG_local_delete` in your site-local configuration file.


```python
# Add these site-local LLM configurations
LLM_CONFIG_local_add = {
    'LLM-Number-1': {
        'fields': [],
        'model': 'Openai-gpt1o',
        'callback': 'call_onprem_llm',
    },
}

# Delete these LLM names from the default configuration
LLM_CONFIG_local_delete = [
    'Example-LLM-name'
]
```

## Step 4:  Create site_local_config.py

This step is to create the `site_local_config.py` file for setting up all of a site's LLMs.
This can be done in one Python file that includes both updates to the `LLM_CONFIG` and callback functions that coordinate the calls out to the LLMs.
The content of the callback functions largely depend upon the site-local LLM APIs.

There is a template inside the docker image for `trovares/desktop_backend`, or you can use the template shown here:

```python
def call_onprem_llm(llm_config: dict, llm_credentials: dict = {},
                    question: str = None, prompt: str = None, history: list = [],
                    **kwargs) -> str:
    import requests
    model = llm_config.get('model', "")
    headers = {"User-Agent": "LLM Client"}
    prompt = f"<s> {prompt}\n[INST]{question}[/INST]</s>"

    payload = {
        "prompt": prompt,
        "model": model,
        "temperature": 0.05,
        "max_tokens": 5000,
        "history": history,
    }
    response = requests.post('https://my.local.llm:6502/api/v1/query',
                             headers=headers, json=payload)
    response_json = response.json()
    if response.status_code != 200:
        current_app.logger.error(f"Error: {response_json['error']}")
    response_text = response_json['choices'][0]['text'].strip()
    print(f"> Query: {prompt}", flush=True)
    print(f"> Response: {response_text}", flush=True)
    return response_text


# Add these site-local LLM configurations
LLM_CONFIG_local_add = {
    'OnPremLLM': {
        'fields': [],
        'model': 'Trovares',
        'callback': 'call_onprem_llm',
    },
}

# Delete these LLM names from the default configuration
LLM_CONFIG_local_delete = [
    'Example-LLM-name'
]

# Replace existing (all the default configuration) with these
# LLM_CONFIG_local_replace = {
#     'Your-Onprem-LLM-name': {
#     },
# }
```

## Step 5:  Restart Trovares Desktop with site-local configuration

A convenient way to do this configuration and have it enabled every time the app is restarted, place the `site_local_config.py` file in the same directory as your `docker-compose.yml` file.  Then add this line to the `docker-compose.yml` file in the “volumes” section of the “backend” service:

```yaml
     - ${PWD}/site_local_config.py:/app/site_local_config.py:ro
```

Now you can do this sequence of shell commands:

```bash
$ docker compose down

$ docker compose up -d
```

You can confirm that the site-local configuration took effect by going to the user Settings, the data explorer, or the schema explorer of the Trovares Desktop and click on the LLM pull-down menu.  If authentication values are required for your site-local LLM, then you will need to enter those for yourself in the user Settings, and all your users will need to do that for their login accounts as well.