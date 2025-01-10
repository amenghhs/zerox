Use streamlit to add UI interface for Zerox OCR.

Usage:
```
streamlit run app.py
```

## Zerox OCR

A dead simple way of OCR-ing a document for AI ingestion. Documents are meant to be a visual representation after all. With weird layouts, tables, charts, etc. The vision models just make sense!

The general logic:

- Pass in a file (pdf, docx, image, etc.)
- Convert that file into a series of images
- Pass each image to GPT and ask nicely for Markdown
- Aggregate the responses and return Markdown

Try out the hosted version here: <https://getomni.ai/ocr-demo>

## Getting Started

Zerox is available as both a Node and Python package.

- [Node Readme](#node-zerox) - [npm package](https://www.npmjs.com/package/zerox)
- [Python Readme](#python-zerox) - [pip package](https://pypi.org/project/py-zerox/)

## Python Zerox

(Python SDK - supports vision models from different providers like OpenAI, Azure OpenAI, Anthropic, AWS Bedrock etc)

### Installation

- Install **poppler** on the system, it should be available in path variable. See the [pdf2image documentation](https://pdf2image.readthedocs.io/en/latest/installation.html) for instructions by platform.
- Install py-zerox:

```sh
pip install py-zerox
```

The `pyzerox.zerox` function is an asynchronous API that performs OCR (Optical Character Recognition) to markdown using vision models. It processes PDF files and converts them into markdown format. Make sure to set up the environment variables for the model and the model provider before using this API.

Refer to the [LiteLLM Documentation](https://docs.litellm.ai/docs/providers) for setting up the environment and passing the correct model name.

### Usage

```python
from pyzerox import zerox
import os
import json
import asyncio

### Model Setup (Use only Vision Models) Refer: https://docs.litellm.ai/docs/providers ###

## placeholder for additional model kwargs which might be required for some models
kwargs = {}

## system prompt to use for the vision model
custom_system_prompt = None

# to override
# custom_system_prompt = "For the below pdf page, do something..something..." ## example

###################### Example for OpenAI ######################
model = "gpt-4o-mini" ## openai model
os.environ["OPENAI_API_KEY"] = "" ## your-api-key


###################### Example for Azure OpenAI ######################
model = "azure/gpt-4o-mini" ## "azure/<your_deployment_name>" -> format <provider>/<model>
os.environ["AZURE_API_KEY"] = "" # "your-azure-api-key"
os.environ["AZURE_API_BASE"] = "" # "https://example-endpoint.openai.azure.com"
os.environ["AZURE_API_VERSION"] = "" # "2023-05-15"


###################### Example for Gemini ######################
model = "gemini/gpt-4o-mini" ## "gemini/<gemini_model>" -> format <provider>/<model>
os.environ['GEMINI_API_KEY'] = "" # your-gemini-api-key


###################### Example for Anthropic ######################
model="claude-3-opus-20240229"
os.environ["ANTHROPIC_API_KEY"] = "" # your-anthropic-api-key

###################### Vertex ai ######################
model = "vertex_ai/gemini-1.5-flash-001" ## "vertex_ai/<model_name>" -> format <provider>/<model>
## GET CREDENTIALS
## RUN ##
# !gcloud auth application-default login - run this to add vertex credentials to your env
## OR ##
file_path = 'path/to/vertex_ai_service_account.json'

# Load the JSON file
with open(file_path, 'r') as file:
    vertex_credentials = json.load(file)

# Convert to JSON string
vertex_credentials_json = json.dumps(vertex_credentials)

vertex_credentials=vertex_credentials_json

## extra args
kwargs = {"vertex_credentials": vertex_credentials}

###################### For other providers refer: https://docs.litellm.ai/docs/providers ######################

# Define main async entrypoint
async def main():
    file_path = "https://omni-demo-data.s3.amazonaws.com/test/cs101.pdf" ## local filepath and file URL supported

    ## process only some pages or all
    select_pages = None ## None for all, but could be int or list(int) page numbers (1 indexed)

    output_dir = "./output_test" ## directory to save the consolidated markdown file
    result = await zerox(file_path=file_path, model=model, output_dir=output_dir,
                        custom_system_prompt=custom_system_prompt,select_pages=select_pages, **kwargs)
    return result


# run the main function:
result = asyncio.run(main())

# print markdown result
print(result)
```

### Parameters

```python
async def zerox(
    cleanup: bool = True,
    concurrency: int = 10,
    file_path: Optional[str] = "",
    maintain_format: bool = False,
    model: str = "gpt-4o-mini",
    output_dir: Optional[str] = None,
    temp_dir: Optional[str] = None,
    custom_system_prompt: Optional[str] = None,
    select_pages: Optional[Union[int, Iterable[int]]] = None,
    **kwargs
) -> ZeroxOutput:
  ...
```

Parameters

- **cleanup** (bool, optional):
  Whether to clean up temporary files after processing. Defaults to True.
- **concurrency** (int, optional):
  The number of concurrent processes to run. Defaults to 10.
- **file_path** (Optional[str], optional):
  The path to the PDF file to process. Defaults to an empty string.
- **maintain_format** (bool, optional):
  Whether to maintain the format from the previous page. Defaults to False.
- **model** (str, optional):
  The model to use for generating completions. Defaults to "gpt-4o-mini".
  Refer to LiteLLM Providers for the correct model name, as it may differ depending on the provider.
- **output_dir** (Optional[str], optional):
  The directory to save the markdown output. Defaults to None.
- **temp_dir** (str, optional):
  The directory to store temporary files, defaults to some named folder in system's temp directory. If already exists, the contents will be deleted before zerox uses it.
- **custom_system_prompt** (str, optional):
  The system prompt to use for the model, this overrides the default system prompt of zerox.Generally it is not required unless you want some specific behaviour. When set, it will raise a friendly warning. Defaults to None.
- **select_pages** (Optional[Union[int, Iterable[int]]], optional):
  Pages to process, can be a single page number or an iterable of page numbers, Defaults to None
- **kwargs** (dict, optional):
  Additional keyword arguments to pass to the litellm.completion method.
  Refer to the LiteLLM Documentation and Completion Input for details.

Returns

- ZeroxOutput:
  Contains the markdown content generated by the model and also some metadata (refer below).

### Example Output (Output from "azure/gpt-4o-mini")

`Note: The output is mannually wrapped for this documentation for better readability.`

````Python
ZeroxOutput(
    completion_time=9432.975,
    file_name='cs101',
    input_tokens=36877,
    output_tokens=515,
    pages=[
        Page(
            content='| Type    | Description                          | Wrapper Class |\n' +
                    '|---------|--------------------------------------|---------------|\n' +
                    '| byte    | 8-bit signed 2s complement integer   | Byte          |\n' +
                    '| short   | 16-bit signed 2s complement integer  | Short         |\n' +
                    '| int     | 32-bit signed 2s complement integer  | Integer       |\n' +
                    '| long    | 64-bit signed 2s complement integer  | Long          |\n' +
                    '| float   | 32-bit IEEE 754 floating point number| Float         |\n' +
                    '| double  | 64-bit floating point number         | Double        |\n' +
                    '| boolean | may be set to true or false          | Boolean       |\n' +
                    '| char    | 16-bit Unicode (UTF-16) character    | Character     |\n\n' +
                    'Table 26.2.: Primitive types in Java\n\n' +
                    '### 26.3.1. Declaration & Assignment\n\n' +
                    'Java is a statically typed language meaning that all variables must be declared before you can use ' +
                    'them or refer to them. In addition, when declaring a variable, you must specify both its type and ' +
                    'its identifier. For example:\n\n' +
                    '```java\n' +
                    'int numUnits;\n' +
                    'double costPerUnit;\n' +
                    'char firstInitial;\n' +
                    'boolean isStudent;\n' +
                    '```\n\n' +
                    'Each declaration specifies the variable’s type followed by the identifier and ending with a ' +
                    'semicolon. The identifier rules are fairly standard: a name can consist of lowercase and ' +
                    'uppercase alphabetic characters, numbers, and underscores but may not begin with a numeric ' +
                    'character. We adopt the modern camelCasing naming convention for variables in our code. In ' +
                    'general, variables must be assigned a value before you can use them in an expression. You do not ' +
                    'have to immediately assign a value when you declare them (though it is good practice), but some ' +
                    'value must be assigned before they can be used or the compiler will issue an error.\n\n' +
                    'The assignment operator is a single equal sign, `=` and is a right-to-left assignment. That is, ' +
                    'the variable that we wish to assign the value to appears on the left-hand-side while the value ' +
                    '(literal, variable or expression) is on the right-hand-side. Using our variables from before, ' +
                    'we can assign them values:\n\n' +
                    '> 2 Instance variables, that is variables declared as part of an object do have default values. ' +
                    'For objects, the default is `null`, for all numeric types, zero is the default value. For the ' +
                    'boolean type, `false` is the default, and the default char value is `\\0`, the null-terminating ' +
                    'character (zero in the ASCII table).',
            content_length=2333,
            page=1
        )
    ]
)
````

## Supported File Types

We use a combination of `libreoffice` and `graphicsmagick` to do document => image conversion. For non-image / non-pdf files, we use libreoffice to convert that file to a pdf, and then to an image.

```js
[
  "pdf", // Portable Document Format
  "doc", // Microsoft Word 97-2003
  "docx", // Microsoft Word 2007-2019
  "odt", // OpenDocument Text
  "ott", // OpenDocument Text Template
  "rtf", // Rich Text Format
  "txt", // Plain Text
  "html", // HTML Document
  "htm", // HTML Document (alternative extension)
  "xml", // XML Document
  "wps", // Microsoft Works Word Processor
  "wpd", // WordPerfect Document
  "xls", // Microsoft Excel 97-2003
  "xlsx", // Microsoft Excel 2007-2019
  "ods", // OpenDocument Spreadsheet
  "ots", // OpenDocument Spreadsheet Template
  "csv", // Comma-Separated Values
  "tsv", // Tab-Separated Values
  "ppt", // Microsoft PowerPoint 97-2003
  "pptx", // Microsoft PowerPoint 2007-2019
  "odp", // OpenDocument Presentation
  "otp", // OpenDocument Presentation Template
];
```

## Credits

- [Litellm](https://github.com/BerriAI/litellm): <https://github.com/BerriAI/litellm> | This powers our python sdk to support all popular vision models from different providers.
- [Streamlit](https://github.com/streamlit/streamlit): <https://github.com/streamlit/streamlit> | Streamlit — A faster way to build and share data apps.

### License

This project is licensed under the MIT License.
