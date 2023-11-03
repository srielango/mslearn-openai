---
lab:
    title: 'Generate and improve code with Azure OpenAI Service'
---

# Generate and improve code with Azure OpenAI Service

The Azure OpenAI Service models can generate code for you using natural language prompts, fixing bugs in completed code, and providing code comments. These models can also explain and simplify existing code to help you understand what it does and how to improve it.

This exercise will take approximately **25** minutes.

## Provision an Azure OpenAI resource

If you don't already have one, provision an Azure OpenAI resource in your Azure subscription.

1. Sign into the [Azure portal](https://portal.azure.com) at `https://portal.azure.com`.
2. Create an **Azure OpenAI** resource with the following settings:
    - **Subscription**: *Select an Azure subscription that has been approved for access to the Azure OpenAI service*
    - **Resource group**: *Choose or create a resource group*
    - **Region**: *Make a random choice from any of the available regions*\*
    - **Name**: *A unique name of your choice*
    - **Pricing tier**: Standard S0

    > \* Azure OpenAI resources are constrained by regional quotas. Randomly choosing a region reduces the risk of a single region reaching its quota limit in scenarios where you are sharing a subscription with other users. In the event of a quota limit being reached later in the exercise, there's a possibility you may need to create another resource in a different region.

3. Wait for deployment to complete. Then go to the deployed Azure OpenAI resource in the Azure portal.

## Deploy a model

Azure OpenAI provides a web-based portal named **Azure OpenAI Studio**, that you can use to deploy, manage, and explore models. You'll start your exploration of Azure OpenAI by using Azure OpenAI Studio to deploy a model.

1. On the **Overview** page for your Azure OpenAI resource, use the **Go to Azure OpenAI Studio** button to open Azure OpenAI Studio in a new browser tab.
2. In Azure OpenAI Studio, on the **Deployments** page, view your existing model deployments. If you don't already have one, create a new deployment of the **gpt-35-turbo** model with the following settings:
    - **Model**: gpt-35-turbo
    - **Model version**: Auto-update to default
    - **Deployment name**: *A unique name of your choice*
    - **Advanced options**
        - **Content filter**: Default
        - **Tokens per minute rate limit**: 5K\*
        - **Enable dynamic quota**: Enabled

    > \* A rate limit of 5,000 tokens per minute is more than adequate to complete this exercise while leaving capacity for other people using the same subscription.

## Generate code in chat playground

Before using in your app, examine how Azure OpenAI can generate and explain code in the chat playground.

1. In [Azure OpenAI Studio](https://oai.azure.com), in the **Playground** section, select the **Chat** page. The **Chat** playground page consists of three main sections:
    - **Assistant setup** - used to set the context for the model's responses.
    - **Chat session** - used to submit chat messages and view responses.
    - **Configuration** - used to configure settings for the model deployment.
2. In the **Configuration** section, ensure that your model deployment is selected.
3. In the **Assistant setup** area, set the system message to `You are a programming assistant helping write code` and save the changes.
4. In the **Chat session**, submit the following query:

    ```
    Write a function in python that takes a character and a string as input, and returns how many times the character appears in the string
    ```

    The model will likely respond with a function, with some explanation of what the function does and how to call it.

5. Next, send the prompt `Do the same thing, but this time write it in C#`.

    The model likely responded very similarly as the first time, but this time coding in C#. You can ask it again for a different language of your choice, or a function to complete a different task such as reversing the input string.

6. Next, let's explore using AI to understand code. Submit the following prompt as the user message.

    ```
    What does the following function do?  
    ---  
    def multiply(a, b):  
        result = 0  
        negative = False  
          
        if a < 0 and b > 0:  
            a = -a  
            negative = True  
        elif a > 0 and b < 0:  
            b = -b  
            negative = True  
        elif a < 0 and b < 0:  
            a = -a  
            b = -b  
           
        while b > 0:  
            result += a  
            b -= 1  
            
        if negative:  
            return -result  
        else:  
            return result  
    
    ```

    The model should describe what the function does, which is to multiply two numbers together by using a loop.

7. Submit the prompt `Can you simplify the function?`.

    The model should write a simpler version of the function.

8. Submit the prompt: `Add some comments to the function.`

    The model adds comments to the code.

## Prepare to develop an app Visual Studio Code

Now let's explore how you could build a custom app that uses Azure OpenAI service to generate code. You'll develop your app using Visual Studio Code. The code files for your app have been provided in a GitHub repo.

> **Tip**: If you have aleady cloned the **mslearn-openai** repo, open it in Visual Studio code. Othewise, follow these steps to clone it to your development environment.

1. Start Visual Studio Code.
2. Open the palette (SHIFT+CTRL+P) and run a **Git: Clone** command to clone the `https://github.com/MicrosoftLearning/mslearn-openai` repository to a local folder (it doesn't matter which folder).
3. When the repository has been cloned, open the folder in Visual Studio Code.
4. Wait while additional files are installed to support the C# code projects in the repo.

    > **Note**: If you are prompted to add required assets to build and debug, select **Not Now**.

## Configure your application

Applications for both C# and Python have been provided, as well as a sample text file you'll use to test the summarization. Both apps feature the same functionality. First, you'll complete some key parts of the application to enable using your Azure OpenAI resource.

1. In Visual Studio Code, in the **Explorer** pane, browse to the **Labfiles/04-code-generation** folder and expand the **CSharp** or **Python** folder depending on your language preference. Each folder contains the language-specific files for an app into which you're you're going to integrate Azure OpenAI functionality.
2. Right-click the **CSharp** or **Python** folder containing your code files and open an integrated terminal. Then install the Azure AI Vision SDK package by running the appropriate command for your language preference:

    **C#**:

    ```
    dotnet add package Azure.AI.OpenAI --prerelease
    ```

    **Python**:

    ```
    pip install openai==0.28.1
    ```

3. In the **Explorer** pane, in the **CSharp** or **Python** folder, open the configuration file for your preferred language

    - **C#**: appsettings.json
    - **Python**: .env
    
4. Update the configuration values to include the **endpoint** and **key** from the Azure OpenAI resource you created (available on the **Keys and Endpoint** page for your Azure OpenAI resource in the Azure portal), as well as the name you specified for your model deployment (available in the **Deployments** page in Azure OpenAI Studio). Save the file.

## Add code to use your Azure OpenAI service model

Now you're ready to use the Azure OpenAI SDK to consume your deployed model.

1. In the **Explorer** pane, in the **CSharp** or **Python** folder, open the code file for your preferred language, and replace the comment ***Add Azure OpenAI package*** with code to add the Azure OpenAI SDK library:

    **C#**: Program.cs

    ```csharp
    // Add Azure OpenAI package
    using Azure.AI.OpenAI;
    ```

    **Python**: prompt-engineering.py

    ```python
    # Add Azure OpenAI package
    import openai
    ```

2. In the code file, find the comment ***Configure the Azure OpenAI client***, and add code to configure the Azure OpenAI client:

    **C#**: Program.cs

    ```csharp
    // Initialize the Azure OpenAI client
    OpenAIClient client = new OpenAIClient(new Uri(oaiEndpoint), new AzureKeyCredential(oaiKey));
    ```

    **Python**: code-generation.py

    ```python
    # Set OpenAI configuration settings
    openai.api_type = "azure"
    openai.api_base = azure_oai_endpoint
    openai.api_version = "2023-05-15"
    openai.api_key = azure_oai_key
    ```

3. In the function that calls the Azure OpenAI model, under the comment ***Format and send the request to the model***, add the code to format and send the request to the model.

    **C#**: Program.cs

    ```csharp
    // Format and send the request to the model
    var chatCompletionsOptions = new ChatCompletionsOptions()
    {
        Messages =
        {
            new ChatMessage(ChatRole.System, systemPrompt),
            new ChatMessage(ChatRole.User, userPrompt)
        },
        Temperature = 0.7f,
        MaxTokens = 1000,
    };

    // Get response from Azure OpenAI
    Response<ChatCompletions> response = await client.GetChatCompletionsAsync(
        oaiModelName,
        chatCompletionsOptions
    );

    ChatCompletions completions = response.Value;
    string completion = completions.Choices[0].Message.Content;
    ```

    **Python**

    ```python: code-generation.py
    # Format and send the request to the model
    messages =[
        {"role": "system", "content": system_message},
        {"role": "user", "content": user_message},
    ]
    
    # Call the Azure OpenAI model
    response = openai.ChatCompletion.create(
        engine=model,
        messages=messages,
        temperature=0.7,
        max_tokens=1000
    )
    ```

4. Save the changes to the code file.

## Run the application

Now that your app has been configured, run it to try generating code for each use case. The use case is numbered in the app, and can be run in any order.

> **Note**: Some users may experience rate limiting if calling the model too frequently. If you hit an error about a token rate limit, wait for a minute then try again.

1. In the **Explorer** pane, expand the **Labfiles/04-code-generation/sample-code** folder and review the function and the *go-fish* app for your language. These files will be used for the tasks in the app.
2. In the interactive terminal pane, ensure the folder context is the folder for your preferred language. Then enter the following command to run the application.

    - **C#**: `dotnet run`
    - **Python**: `python code-generation.py`

    > **Tip**: You can use the **Maximize panel size** (**^**) icon in the terminal toolbar to see more of the console text.

3. Choose option **1** to add comments to your code. Note, the response might take a few seconds for each of these tasks.

    The results will be put into `result/app.txt`. Open that file up, and compare it to the function file in `sample-code`.

4. Next, choose option **2** to write unit tests for that same function.

    The results will replace what was in `result/app.txt`, and details four unit tests for that function.

5. Next, choose option **3** to fix bugs in an app for playing Go Fish.

    The results will replace what was in `result/app.txt`, and should have very similar code with a few things corrected.

    - **C#**: Fixes are made on line 30 and 59
    - **Python**: Fixes are made on line 18 and 31

    The app for Go Fish in `sample-code` can be run if you replace the lines that contain bugs with the response from Azure OpenAI. If you run it without the fixes, it will not work correctly.
    
    > **Note**: It's important to note that even though the code for this Go Fish app was corrected for some syntax, it's not a strictly accurate representation of the game. If you look closely, there are issues with not checking if the deck is empty when drawing cards, not removing pairs from the players hand when they get a pair, and a few other bugs that require understanding of card games to realize. This is a great example of how useful generative AI models can be to assist with code generation, but can't be trusted as correct and need to be verified by the developer.

    If you would like to see the full response from Azure OpenAI, you can set the `printFullResponse` variable to `True`, and rerun the app.

## Clean up

When you're done with your Azure OpenAI resource, remember to delete the deployment or the entire resource in the [Azure portal](https://portal.azure.com).