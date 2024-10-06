# Prompty Specification

Prompty assets are represented by files with a `.prompty` extension and must follow the schema outlined in the [Prompty Specification](https://github.com/microsoft/prompty/blob/main/Prompty.yaml). 

---

## 1. Specification: Schema

We can now revisit the [Prompty Specification](https://github.com/microsoft/prompty/blob/main/Prompty.yaml) to get the full picture of the Prompty asset schema. Let's start by recongizing the top-level sections:

- `$schema` - the schema version used for validation
- `$id` - the unique identifier for the schema
- `title` - the title of the schema
- `description` - a brief description of the schema
- `type` - the type of the schema (here, "object")
- `additionalProperties` - whether additional properties are allowed (here, "false")
- `properties` - the properties of the schema
- `definitions` - the definitions used in the schema


??? info "Specificaton file: `prompty.yaml`"

    ```yaml
        # This schema represents the specification file for the prompty
    # _frontmatter_, not the content section.

    $schema: http://json-schema.org/draft-07/schema#  
    $id: http://azureml/sdk-2-0/Prompty.yaml  
    title: Prompty front matter schema specification  
    description: A specification that describes how to provision a new prompty using definition frontmatter.
    type: object 

    additionalProperties: false

    properties:
    $schema:
        type: string
    # metadata section
    model:
        type: object
        additionalProperties: false
        properties:
        api:
            type: string
            enum: 
            - chat
            - completion
            description: The API to use for the prompty -- this has implications on how the template is processed and how the model is called.
            default: chat

        configuration:
            oneOf:
            - $ref: "#/definitions/azureOpenaiModel"
            - $ref: "#/definitions/openaiModel"
            - $ref: "#/definitions/maasModel"
    
        parameters:
            $ref: "#/definitions/parameters"

        response: 
            type: string
            description: This determines whether the full (raw) response or just the first response in the choice array is returned.
            default: first 
            enum:
            - first
            - full


    name:
        type: string
        description: Name of the prompty
    description:
        type: string
        description: Description of the prompty
    version:
        type: string
        description: Version of the prompty
    authors:
        type: array
        description: Authors of the prompty
        items:
        type: string
    tags:
        type: array
        description: Tags of the prompty
        items:
        type: string

    # not yet supported -- might add later
    # base:
    #   type: string
    #   description: The base prompty to use as a starting point

    sample: 
        oneOf:
        - type: object
            description: The sample to be used in the prompty test execution
            additionalProperties: true
        - type: string
            description: The file to be loaded to be used in the prompty test execution

    # the user can provide a single sample in a file or specify the data inline
    # sample:
    #   messages: 
    #     - role: user
    #       content: where is the nearest coffee shop?
    #     - role: system
    #       content: I'm sorry, I don't know that. Would you like me to look it up for you?
    # or point to a file
    # sample: sample.json
    # also the user can specify the data on the command line
    # pf flow test --flow p.prompty --input my_sample.json
    # if the user runs this command, the sample from the prompty will be used
    # pf flow test --flow p.prompty   
        
    inputs:
        type: object
        description: The inputs to the prompty

    outputs:
        type: object
        description: The outputs of the prompty

    # currently not supported -- might not be needed
    # init_signature:
    #   type: object
    #   description: The signature of the init function

    template:
        type: string
        description: The template engine to be used can be specified here. This is optional.
        enum: [jinja2]
        default: jinja2

    definitions:
    ```

Here, 

- **properties** describe valid "keys" that can be present in a `.prompty` asset file,
- **definitions** describe "reusable" schema definitions" for quick reference in properties

Let's take a look at Prompty schema properties and definitions in more detail.

---

### 1.1 Specification: Properties

The _properties_ schema describes these valid "keys" that can be present in a `.prompty` asset file:

- `$schema` - (string) the schema version used for validation
- `model` - (object) the model configuration for the asset
    - `api` - (string) the API type (`chat` (default) or `completion`)
    - `configuration` - (object) one of _definitions_ provided
    - `parameters` - (object) from _definitions_ provided
    - `response` - (string) whether to return `full` or `first` response (default: first)
- `name` - (string) the name of the asset
- `description` - (string) a brief description of the asset
- `version` - (string) the version of the asset
- `authors` - (array) a list of authors who contributed to the asset
- `tags` - (array) a list of tags for the asset
- `sample` - test data for validation - can be object (inline) or string (filename)
- `inputs` - (object) defining request properties for prompty asset
- `outputs` - (object) defining response properties for prompty asset
- `template` - (string) the template engine to be used (default: `jinja2`)

---

### 1.2 Specification: Definitions

The `definitions` section of the schema describes reusable schema sections that can then be referenced within the `properties` section with the `$ref` keyword. For example, this is in `model` properties:

```yaml
    configuration:
        oneOf:
        - $ref: "#/definitions/azureOpenaiModel"
        - $ref: "#/definitions/openaiModel"
        - $ref: "#/definitions/maasModel"

    parameters:
        $ref: "#/definitions/parameters"
```

This lets us define complex schema once and reference it in multiple places without duplication of content - helping keep the specification readable and maintainable.

The specification has 3 definitions for **Model: Configuration**:

??? example "Definition: `openaiModel` configuration for Model"
    
    ```yaml
    # vanilla openai models
    openaiModel:
        type: object
        description: Model used to generate text
        properties:
        type:
            type: string
            description: Type of the model
            const: openai
        name:
            type: string
            description: Name of the model
        organization:
            type: string
            description: Name of the organization
        additionalProperties: false
    ```

??? example "Definition: `azurOpenaiModel` configuration for Model"
    
    ```yaml
    # azure openai models
    azureOpenaiModel:
        type: object
        description: Model used to generate text
        properties:
        type:
            type: string
            description: Type of the model
            const: azure_openai
        api_version:
            type: string
            description: Version of the model
        azure_deployment:
            type: string
            description: Deployment of the model
        azure_endpoint:
            type: string
            description: Endpoint of the model
        additionalProperties: false
    ```
??? example "Definition: `maasModel` configuration for Model"
    
    ```yaml
    # for maas models
    maasModel:
        type: object
        description: Model used to generate text
        properties:
        type:
            type: string
            description: Type of the model
            const: azure_serverless
        azure_endpoint:
            type: string
            description: Endpoint of the model
        additionalProperties: false
    ```


The specification has 1 definition for **Model: Parameters**. For now, it defines these as _common_ to all models (where individual models may process each differently). 

The paramters are: *response_format, seed, max_tokens, temperature, tools_choice, tools, frequency_penalty, presence_penalty, stop, and top_p*. Click to expand for details.

??? example "Definition: `parameters` for Model"
    
    ```yaml
    # parameters for the model -- for now these are not per model but the same for all models
    parameters:
        type: object
        description: Parameters to be sent to the model 
        additionalProperties: true
        properties: 
        response_format: 
            type: object
            description: >
            An object specifying the format that the model must output. Compatible with
            `gpt-4-1106-preview` and `gpt-3.5-turbo-1106`.
            Setting to `{ "type": "json_object" }` enables JSON mode, which guarantees the
            message the model generates is valid JSON.

        seed:
            type: integer
            description: > 
            This feature is in Beta. If specified, our system will make a best effort to
            sample deterministically, such that repeated requests with the same `seed` and
            parameters should return the same result. Determinism is not guaranteed, and you
            should refer to the `system_fingerprint` response parameter to monitor changes
            in the backend.

        max_tokens:
            type: integer
            description: The maximum number of [tokens](/tokenizer) that can be generated in the chat completion.

        temperature:
            type: number
            description: What sampling temperature to use, 0 means deterministic.

        tools_choice:
            oneOf:
            - type: string
            - type: object
            
            description: > 
            Controls which (if any) function is called by the model. `none` means the model
            will not call a function and instead generates a message. `auto` means the model
            can pick between generating a message or calling a function. Specifying a
            particular function via
            `{"type": "function", "function": {"name": "my_function"}}` forces the model to
            call that function.

            `none` is the default when no functions are present. `auto` is the default if
            functions are present.

        tools:
            type: array
            items:
            type: object

        frequency_penalty:
            type: number
            description: What sampling frequency penalty to use. 0 means no penalty.
        
        presence_penalty:
            type: number
            description: What sampling presence penalty to use. 0 means no penalty.
        
        stop:
            type: array
            items:
            type: string
            description: > 
            One or more sequences where the model should stop generating tokens. The model
            will stop generating tokens if it generates one of the sequences. If the model
            generates a sequence that is a prefix of one of the sequences, it will continue
            generating tokens.
        
        top_p:
            type: number
            description: > 
            What nucleus sampling probability to use. 1 means no nucleus sampling. 0 means
            no tokens are generated.
    ```


---

## 2. Asset: Example

Use the Prompty Visual Studio Code extension to generate the default `.prompty` starter file as follows:

- open the file-explorer window (left) in Visual Studio Code
- right-click on the folder where you want to create the `.prompty` file
- select `New Prompty File` from the context menu

You should see something like the example below. Let's deconstruct this in the next few sections to get a sense of the Prompty specification by example.


??? info "Asset file: `basic.prompty`"

    ```markdown
    ---
    name: ExamplePrompt
    description: A prompt that uses context to ground an incoming question
    authors:
    - Seth Juarez
    model:
        api: chat
        configuration:
            type: azure_openai
            azure_endpoint: ${env:AZURE_OPENAI_ENDPOINT}
            azure_deployment: <your-deployment>
            api_version: 2024-07-01-preview
        parameters:
            max_tokens: 3000
    sample:
        firstName: Seth
        context: >
            The Alpine Explorer Tent boasts a detachable divider for privacy, 
            numerous mesh windows and adjustable vents for ventilation, and 
            a waterproof design. It even has a built-in gear loft for storing 
            your outdoor essentials. In short, it's a blend of privacy, comfort, 
            and convenience, making it your second home in the heart of nature!
        question: What can you tell me about your tents?
    ---

    system:
    You are an AI assistant who helps people find information. As the assistant, 
    you answer questions briefly, succinctly, and in a personable manner using 
    markdown and even add some personal flair with appropriate emojis.

    # Customer
    You are helping {{firstName}} to find answers to their questions.
    Use their name to address them in your responses.

    # Context
    Use the following context to provide a more personalized response to {{firstName}}:
    {{context}}

    user:
    {{question}}
    ```
---

### 2.1 Asset: Frontmatter

The _frontmatter_ section of the asset occurs between the `---` delimiters (lines 1-24 above) and **contains metadata about the asset**. Let's take a look at just this segment below.

We see the following properties used in the frontmatter:

- `name` - default name is "ExamplePrompt" (change it)
- `description` - default description given (change it)
- `authors` - default author given (change it)
- `model` - the **model configuration** for the asset
    - `api` - set to default ("chat")
    - `configuration` - set to default ("azure_openai")
    - `parameters` - sets `max_tokens` to 3000
- `sample` - set to inline object (not filename) with 3 properties
    - `firstName` - default requestor name (change it)
    - `context` - gives example "grounding" data (change it)
    - `question` - default user question (change it)

The _frontmatter_ is used by Prompty tooling and runtime to understand the asset and its requirements for execution. Later, we'll review the Prompty specification to understand this and other frontmatter properties in more detail.

??? example "Asset Frontmatter: `basic.prompty`"

    ```markdown
    ---
    name: ExamplePrompt
    description: A prompt that uses context to ground an incoming question
    authors:
    - Seth Juarez
    model:
        api: chat
        configuration:
            type: azure_openai
            azure_endpoint: ${env:AZURE_OPENAI_ENDPOINT}
            azure_deployment: <your-deployment>
            api_version: 2024-07-01-preview
        parameters:
            max_tokens: 3000
    sample:
        firstName: Seth
        context: >
            The Alpine Explorer Tent boasts a detachable divider for privacy, 
            numerous mesh windows and adjustable vents for ventilation, and 
            a waterproof design. It even has a built-in gear loft for storing 
            your outdoor essentials. In short, it's a blend of privacy, comfort, 
            and convenience, making it your second home in the heart of nature!
        question: What can you tell me about your tents?
    ---
    ```

---

### 2.2 Asset: Template

The _template_ section of the asset occurs below the second `---` delimiters (lines 24-end in example above) and **provides the content for the prompt template**. Let's take a look at just this segment below.

A prompt template typically defines the _persona_, _instructions_ and _primary content_ (e.g., cues, examples) for the _inference task_ that we want the model to execute. We see the following properties used in the template section for this purpose:

- `system` - establishes default persona (AI assistant to help ..) and instructions (answer briefly ...)
    - `Customer` -  data shape for "Customer" with instructions ("address them by name")
    - `Context` - data shape for "Grounding Context" with instructions ("personalize response")
- `user` - defines the user prompt (actual incoming request)

The `{{<variable-name>}}` syntax used in the template denotes _placeholders_ for data that will be bound in, when the template is _instantiated_ at runtime. Note how each of those variables typically has a value defined in the **sample** object, for testing purposes. This allows us to get a sense for the _shape of data_ we expect to retrieve and augment (from third-party services or from flow orchestration) as we iterate and engineer the prompt.


??? example "Asset Template: `basic.prompty`"

    ```markdown
    system:
    You are an AI assistant who helps people find information. As the assistant, 
    you answer questions briefly, succinctly, and in a personable manner using 
    markdown and even add some personal flair with appropriate emojis.

    # Customer
    You are helping {{firstName}} to find answers to their questions.
    Use their name to address them in your responses.

    # Context
    Use the following context to provide a more personalized response to {{firstName}}:
    {{context}}

    user:
    {{question}}
    ```

---