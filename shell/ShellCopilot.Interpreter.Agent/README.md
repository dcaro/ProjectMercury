# Interpreter Agent

![GIF showing demo of Interpreter Agent](./assets/InterpreterAgentDemoSpedUp.gif)

## Description

An agent that specializes in completing code related tasks. This agent will take a user's task to be done, write a plan, generate code, execute code, and move on to the next step of the plan until
the task is complete while correcting itself for any errors. Currently only supports running PowerShell and Python code.

## Table of Contents

  - [Usage](#usage)
	 - [Set-Up](#set-up)
  - [Features](#features)
  - [Architecture/Flow](#architectureflow)
  - [Known limitations](#known-limitations)

## Usage

1. You can use this agent by selecting it after you run the `aish` executable. It currently supports generating and running PowerShell and Python code. The `aish` executable can be called in any shell.
>```
>Shell Copilot
>v0.1.0-preview.1
>
>Please select an agent to use:
>
>  az-ps
>  az-cli
> >interpreter
>  openai-gpt
>```
2. Enter a task in natural language and the agent will generate code to complete the task. We have only tested using English but may work with other languages.
3. Type 'y' and press enter or just press enter (prompt defaults to 'y') to run the code generated by the agent or 'n' to not run the code.
	a. Auto execution is configurable in the settings file, where the agent will run the code without user confirmation.
4. Sit back and watch the agent do the work.

### Set-Up

1. Clone the repository															
2. Build the project by running the following command in the root directory of the project:
>```pwsh
>.\build.ps1
>```
3. Install Python and/or PowerShell 7 
4. Get an OpenAI or AzureOpenAI key You can get one by signing up at [OpenAI](https://platform.openai.com/signup) or [AzureOpenAI](https://azure.microsoft.com/en-us/services/cognitive-services/openai/).
5. Your first chat with the agent will prompt you to enter your key.
>```
>aish:1> hello world in python
>
>NOTE: Some required information is missing for the GPT:
>
>    Type :       AzureOpenAI
>    Endpoint :   https://pscopilot.azure-api.net
>    Deployment : gpt4
>    Model :      gpt-4-0613
>
>NOTE: The access key is missing.
> > The model uses the default ShellCopilot endpoint.
> > You can apply an access key for it by following the instructions in our doc.
>
>Enter key:
>```

6. To change endpoints, keys, model, or agent settings run the `/agent config` command in the interpreter agent. This will open the `interpreter.agent.json` file.
>```
> aish:1> /agent config
>```

a. Default `interpreter.agent.json` file:

>```json
>{
>      "Endpoint" : "https://pscopilot.azure-api.net",
>      "Deployment" : "gpt4",
>      "ModelName" : "gpt-4-0613",   // required field to infer properties of the service, such as token limit.
>      "AutoExecution" : false,
>      "DisplayErrors" : true,
>      "Key" : null
>
>    // To use the public OpenAI as the AI completion service:
>    // - Ignore the `Endpoint` and `Deployment` keys.
>    // - Set `Key` to be the OpenAI access token.
>    // Replace the above with the following:
>    /*
>      "ModelName": "gpt-4-0613",
>      "AutoExecution": false,
>      "DisplayErrors": true,
>      "Key": null
>    */
>}
>```
b. To use Azure.OpenAI as the AI completion service specify the endpoint and deployment keys in the `interpreter.agent.json` file.
>```json
>{
>      "Endpoint" : "https://pscopilot.azure-api.net",
>      "Deployment" : "gpt4",
>      "ModelName" : "gpt-4-0613",   // required field to infer properties of the service, such as token limit.
>      "AutoExecution" : false,
>      "DisplayErrors" : true,
>      "Key" : null
>}
>```
c. To use OpenAI as the AI completion service, ignore the `Endpoint` and `Deployment` keys in the `interpreter.agent.json` file.
>```json
>{
>      "ModelName": "gpt-4-0613",
>      "AutoExecution": false,
>      "DisplayErrors": true,
>      "Key": null
>}
>```
d. Specify the model and set the `ModelName` key in the `interpreter.agent.json` file. Model names can be found at https://platform.openai.com/docs/models/gpt-4-turbo-and-gpt-4. Function calling models are specified at https://platform.openai.com/docs/guides/function-calling.
>```
> "ModelName": "gpt-4-0613"
>```


## Features

- Takes natural language and generates code
- Can execute the code generated in a sandboxed persistent environment
- Attempts to correct any errors it encounters when running that code
- Aware of past commands it ran in a session
- Auto-execution feature
>```
> "AutoExecution" : true
>```
- Error hiding, can hide errors ran in the code but continue to correct behind the scenes
>```
> "DisplayErrors" : false
>```

## Architecture

<img src="./assets/Interpreter-Agent-Architecture.png" alt="Interpreter-Agent" width="500"/>

### Architecture Description
1. The agent is initialized using the `interpreter.agent.json` file.
2. The user enters a task in natual language.
3. TaskCompletionChat is the main loop of the program, the API request is sent here.
4. The type of GPT (function calling or text based) is determined from the Settings file in TaskCompletionChat.
5. If the GPT response does not contain a function call/code, the main program loop is terminated.
6. If the GPT response contains a function call/code, the code is executed in the sandboxed environment.
7. Code output is displayed, collected, and returned to the GPT.				
8. If GPT detects any errors in the code, it will attempt to correct them and re-rerun code.
9. The main loop will continue until task is complete or GPT requires more information. 

### Flow

<img src="./assets/InterpreterAgentFlowChart.png" alt="Interpreter-Agent" width="500"/>

### Flow Description
1. The user enters a task in natural language.
2. The task is sent to the GPT model.
3. The GPT model generates a response to complete the task.
4. Response is scanned for presence of code.
5. If there is no code, end the main-loop. 
4. If there is code, it is executed in a sandboxed environment.
5. The output of the code is returned to the GPT model.
6. The GPT model corrects any errors in the code.	
7. Repeat steps 3-6 until task is complete.

## Known limitations
- A.I generated code still can product incorrect code and responses
- Agent could produce hallucinations for code results if there is no output to verify the code.
- Chat history is reduced to git token limits per model. When chat is too long, agent will lose context of earliest results.
- Agent cannot handle concurrent tasks.
- Interactive prompts in code will not work properly and will make the execution seem "hung", try pressing any key to see if it will continue execution.