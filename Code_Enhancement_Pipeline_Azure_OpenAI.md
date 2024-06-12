
# Implementing Code Enhancement Pipeline with Azure OpenAI

## Step 1: Set Up Azure OpenAI Service

1. **Create an Azure Account**: If you don't already have one, sign up for an Azure account.
2. **Create an Azure OpenAI Service**:
   - Go to the Azure portal.
   - Create a new resource and search for "Azure OpenAI Service".
   - Follow the prompts to set up the service.

## Step 2: Install Required Libraries

You'll need to install the Azure and OpenAI SDKs. You can do this using pip:

```bash
!pip install openai --quiet
```

## Step 3: Authentication and Setup

You'll need to authenticate with Azure and set up the OpenAI client.

```python
api_key = "c3cbe9bf946c4f449b4de783ad3043bb"
api_version = "2023-07-01-preview" # "2023-05-15" OR "2024-02-01"
azure_endpoint = "https://anshugptoo7.openai.azure.com/"
model_name = "gpt-35-turbo"

from openai import AzureOpenAI

# gets the API Key from environment variable AZURE_OPENAI_API_KEY
client = AzureOpenAI(
    # https://learn.microsoft.com/en-us/azure/ai-services/openai/reference#rest-api-versioning
    api_version=api_version,
    # https://learn.microsoft.com/en-us/azure/cognitive-services/openai/how-to/create-resource?pivots=web-portal#create-a-resource
    azure_endpoint=azure_endpoint,
    api_key = api_key)

```

## Step 4: Create the Code Enhancement Pipeline

Now, we'll create the code enhancement pipeline. The pipeline will:
1. Take a code snippet as input.
2. Send it to the OpenAI model for enhancement.
3. Return the enhanced code.

```python
def enhance_code(code_snippet):
    # Define the prompt
    prompt = f"""Enhance the following 
    Only provide code as a response, no other preamable text, explanation or anything else. The response should be only code in string format.
    Example:
    input:
    def hello_world():
        print('Hellooo, world!)
    output:

    def hello_world():
    return 'Hello, world!'

    code:\n\n{code_snippet}"""

    # Call the OpenAI API
    response = client.chat.completions.create(
        messages=[{"role":"system",'content':"You are an expert programmer, you follow standard best practices for answering coding questions."},
            {"role":"user",'content':prompt}],
        model = model_name,
      temperature=0.7,)
    

    # Extract the enhanced code from the response
    enhanced_code = response.choices[0].message.content.strip()
    return enhanced_code


# Example usage
code_snippet = """
def hello_world():
    print("Hello, world!")
"""
enhanced_code = enhance_code(code_snippet)
print("Enhanced Code:\n", enhanced_code)
```

## Step 5: Integrate with Azure Functions (Optional)

To make this pipeline accessible via a web service, you can deploy it as an Azure Function.

1. **Create a new Azure Function**:
   - Go to the Azure portal and create a new Azure Function.
   - Choose a Python runtime stack.

2. **Write the Azure Function Code**:

```python
import azure.functions as func
import logging

app = func.FunctionApp(http_auth_level=func.AuthLevel.FUNCTION)

api_key = "c3cbe9bf946c4f449b4de783ad3043bb"
api_version = "2023-07-01-preview" # "2023-05-15" OR "2024-02-01"
azure_endpoint = "https://anshugptoo7.openai.azure.com/"
model_name = "gpt-35-turbo"

from openai import AzureOpenAI

# gets the API Key from environment variable AZURE_OPENAI_API_KEY
client = AzureOpenAI(api_version=api_version,azure_endpoint=azure_endpoint, api_key = api_key)

@app.route(route="http_trigger2", auth_level=func.AuthLevel.ANONYMOUS)
def http_trigger2(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Python HTTP trigger function processed a request.')

    code_snippet = req.params.get('code')
    if not code_snippet:
        try:
            req_body = req.get_json()
        except ValueError:
            pass
        else:
            code_snippet = req_body.get('code')

    if code_snippet:
        response = enhance_code(code_snippet)
        return func.HttpResponse(f"response")
    else:
        return func.HttpResponse(
             "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response.",
             status_code=200
        )

def enhance_code(code_snippet):
    # Define the prompt
    prompt = f"""Enhance the following 
    Only provide code as a response, no other preamable text, explanation or anything else. The response should be only code in string format.
    Example:
    input:
    def hello_world():
        print('Hellooo, world!)
    output:

    def hello_world():
    return 'Hello, world!'

    code:\n\n{code_snippet}"""

    # Call the OpenAI API
    response = client.chat.completions.create(
        messages=[{"role":"system",'content':"You are an expert programmer, you follow standard best practices for answering coding questions."},
            {"role":"user",'content':prompt}],
        model = model_name,
      temperature=0.7,)
    

    # Extract the enhanced code from the response
    enhanced_code = response.choices[0].message.content.strip()
    return enhanced_code
```

## Step 6: Testing and Iteration

- **Test the pipeline**: Ensure that the code enhancement works as expected by testing with various code snippets.
- **Iterate and improve**: Adjust parameters like max_tokens and temperature for better results. You might also need to fine-tune the model if you have specific requirements.

```python
import requests
import json

url = "https://codepipeline.azurewebsites.net"
path = "/api/http_trigger2"

code ="""
def print_hello()
    print("Hello, world!)
"""

data = json.dumps({"code":code})
headers = {'Content-Type': 'application/json'}

response = requests.post(url+path, data=data, headers=headers)

print(response.text)
```
