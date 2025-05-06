---
lab:
  title: Creación de un asistente de IA con kernel semántico
  description: Aprende a usar kernel semántico para crear un asistente de IA generativa que pueda realizar tareas de DevOps.
---

# Creación de un asistente de IA con kernel semántico

En este laboratorio, desarrollarás el código para un asistente impulsado por inteligencia artificial diseñado para automatizar las operaciones de desarrollo y ayudar a simplificar las tareas. Usarás el SDK de kernel semántico para crear el asistente de IA y conectarlo al servicio de modelo de lenguaje grande (LLM). El SDK de kernel semántico te permite crear una aplicación inteligente que pueda interactuar con el servicio LLM, responder a consultas en lenguaje natural y proporcionar información personalizada al usuario. Para este ejercicio, se proporcionan funciones simuladas para representar tareas de DevOps típicas. Comencemos.

Este ejercicio dura aproximadamente **30** minutos.

## Implementación de un modelo de finalización de chat

1. Vaya a [https://portal.azure.com](https://portal.azure.com).

1. Crea un nuevo recurso de Azure OpenAI con la configuración predeterminada.

1. Tras crear el nuevo recurso, selecciona **Ir al recurso**.

1. En la página **Información general**, selecciona **Ir al portal de Azure Foundry**.

1. Selecciona **Crear nueva implementación** y después **desde modelos base**.

1. Busca **gpt-4o** en la lista de modelos, selecciona y confirma.

1. Escribe un nombre para la implementación y deja las opciones predeterminadas.

1. Cuando se complete la implementación, vuelve a tu recurso de Azure OpenAI en Azure Portal.

1. En **Administración de recursos**, ve a **Claves y puntos de conexión**.

    Usarás los datos en la siguiente tarea para compilar el kernel. Recuerda mantener tus claves privadas y seguras.

## Preparación de la configuración de aplicación

1. Abre una nueva pestaña del explorador (mantén el Portal de la Fundición de IA de Azure abierto en la pestaña existente). En la nueva pestaña, explora [Azure Portal](https://portal.azure.com) en `https://portal.azure.com` e inicia sesión con tus credenciales de Azure, si se te solicita.

    Cierra las notificaciones de bienvenida para ver la página principal de Azure Portal.

1. Usa el botón **[\>_]** situado a la derecha de la barra de búsqueda en la parte superior de la página para crear una nueva instancia de Cloud Shell en Azure Portal, para lo que deberás seleccionar un entorno de ***PowerShell*** sin almacenamiento en tu suscripción.

    Cloud Shell proporciona una interfaz de la línea de comandos en un panel situado en la parte inferior de Azure Portal. Puedes cambiar el tamaño o maximizar este panel para facilitar el trabajo.

    > **Nota**: si has creado anteriormente una instancia de Cloud Shell que usa un entorno de *Bash*, cámbiala a ***PowerShell***.

1. En la barra de herramientas de Cloud Shell, en el menú **Configuración**, selecciona **Ir a la versión clásica** (esto es necesario para usar el editor de código).

    **<font color="red">Asegúrate de que has cambiado a la versión clásica de Cloud Shell antes de continuar.</font>**

1. En el panel de Cloud Shell, escribe los siguientes comandos para clonar el repositorio de GitHub que contiene los archivos de código de este ejercicio (escribe el comando o cópialo en el Portapapeles y, a continuación, haz clic con el botón derecho en la línea de comandos y pega como texto sin formato):

    ```
    rm -r semantic-kernel -f
    git clone https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK semantic-kernel
    ```

    > **Sugerencia**: al pegar comandos en CloudShell, la salida puede ocupar una gran cantidad del búfer de pantalla. Puedes despejar la pantalla al escribir el comando `cls` para que te resulte más fácil centrarte en cada tarea.

1. Una vez clonado el repo, ve a la carpeta que contiene los archivos de código de la aplicación de chat:

    > **Nota**: sigue los pasos del lenguaje de programación elegido.

    **Python**
    ```
    cd semantic-kernel/Allfiles/Labs/Devops/python
    ```

    **C#**
    ```
    cd semantic-kernel/Allfiles/Labs/Devops/c-sharp
    ```

1. En el panel de la línea de comandos de Cloud Shell, escribe el siguiente comando para instalar las bibliotecas que vas a usar:

    **Python**
    ```
    python -m venv labenv
    ./labenv/bin/Activate.ps1
    pip install python-dotenv azure-identity semantic-kernel[azure] 
    ```

    **C#**
    ```
    dotnet add package Microsoft.Extensions.Configuration
    dotnet add package Microsoft.Extensions.Configuration.Json
    dotnet add package Microsoft.SemanticKernel
    dotnet add package Microsoft.SemanticKernel.PromptTemplates.Handlebars
    ```

1. Escribe el siguiente comando para editar el archivo de configuración que se ha proporcionado:

    **Python**
    ```
    code .env
    ```

    **C#**
    ```
    code appsettings.json
    ```

    El archivo se abre en un editor de código.

1. Actualiza los valores con tu Id. de modelo, punto de conexión y clave de API de los servicios de Azure OpenAI.

    **Python**
    ```python
    MODEL_DEPLOYMENT=""
    BASE_URL=""
    API_KEY="
    ```

    **C#**
    ```json
    {
        "modelName": "",
        "endpoint": "",
        "apiKey": ""
    }
    ```

1. Una vez actualizados los valores, usa el comando **CTRL+S** para guardar los cambios y después usa el comando **CTRL+Q** para cerrar el editor de código mientras mantienes abierta la línea de comandos de Cloud Shell.

## Creación de un complemento de kernel semántico

1. Escribe el siguiente comando para editar el archivo de código que se ha proporcionado:

    **Python**
    ```
    code devops.py
    ```

    **C#**
    ```
    code Program.cs
    ```

1. Agrega el código siguiente en el comentario **Creación de un generador de kernel con la finalización del chat de Azure OpenAI**:

    **Python**
    ```python
    # Create a kernel builder with Azure OpenAI chat completion
    kernel = Kernel()
    chat_completion = AzureChatCompletion(
        deployment_name=deployment_name,
        api_key=api_key,
        base_url=base_url,
    )
    kernel.add_service(chat_completion)
    ```
    **C#**
     ```c#
    // Create a kernel builder with Azure OpenAI chat completion
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(modelId, endpoint, apiKey);
    var kernel = builder.Build();
    ```

1. Cerca de la parte inferior del archivo, busca el comentario **Crear una función de kernel para compilar el entorno de fase**, y agrega el siguiente código para crear una función de complemento ficticio que compilará el entorno de ensayo:

    **Python**
    ```python
    # Create a kernel function to build the stage environment
    @kernel_function(name="BuildStageEnvironment")
    def build_stage_environment(self):
        return "Stage build completed."
    ```

    **C#**
    ```c#
    // Create a kernel function to build the stage environment
    [KernelFunction("BuildStageEnvironment")]
    public string BuildStageEnvironment() 
    {
        return "Stage build completed.";
    }
    ```

    El decorador `KernelFunction` declara la función nativa. Usa un nombre descriptivo para la función para que la IA pueda llamarla correctamente. 

1. Ve al comentario **Importar complementos al kernel** y agrega el siguiente código:

    **Python**
    ```python
    # Import plugins to the kernel
    kernel.add_plugin(DevopsPlugin(), plugin_name="DevopsPlugin")
    ```

    **C#**
    ```c#
    // Import plugins to the kernel
    kernel.ImportPluginFromType<DevopsPlugin>();
    ```


1. En el comentario **Crear configuración de ejecución de indicación**, agrega el código siguiente para invocar automáticamente la función:

    **Python**
    ```python
    # Create prompt execution settings
    execution_settings = AzureChatPromptExecutionSettings()
    execution_settings.function_choice_behavior = FunctionChoiceBehavior.Auto()
    ```

    **C#**
    ```c#
    // Create prompt execution settings
    OpenAIPromptExecutionSettings openAIPromptExecutionSettings = new() 
    {
        FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
    };
    ```

    El uso de esta configuración permitirá que el kernel invoque automáticamente las funciones sin necesidad de especificarlas en la indicación.

1. Agrega el siguiente código bajo el comentario **Crear historial de chats**:

    **Python**
    ```python
    # Create chat history
    chat_history = ChatHistory()
    ```

    **C#**
    ```c#
    // Create chat history
    var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();
    ChatHistory chatHistory = [];
    ```

1. Quita la marca de comentario del bloque de código ubicado después del comentario **Lógica de interacción del usuario**

1. Usa el comando **CTRL+S** para guardar los cambios en el archivo de código.

## Ejecución del código del asistente de DevOps

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para iniciar sesión en Azure.

    ```
    az login
    ```

    **<font color="red">Debes iniciar sesión en Azure, aunque la sesión de Cloud Shell ya esté autenticada.</font>**

    > **Nota**: en la mayoría de los escenarios, el uso de *inicio de sesión de az* será suficiente. Sin embargo, si tienes suscripciones en varios inquilinos, es posible que tengas que especificar el inquilino mediante el parámetro *--tenant*. Consulta [Inicio de sesión en Azure de forma interactiva mediante la CLI de Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively) para obtener más información.

1. Cuando se te solicite, sigue las instrucciones para abrir la página de inicio de sesión en una nueva pestaña y escribe el código de autenticación proporcionado y las credenciales de Azure. Después, completa el proceso de inicio de sesión en la línea de comandos y selecciona la suscripción que contiene tu centro de Fundición de IA de Azure si se te solicita.

1. Después de iniciar sesión, escribe el siguiente comando para ejecutar la aplicación:


    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. Cuando se te solicite, escribe la siguiente indicación `Please build the stage environment`

1. Debería ver una respuesta similar a la siguiente salida:

    ```output
    Assistant: The stage environment has been successfully built.
    ```

1. Después, escribe la siguiente indicación `Please deploy the stage environment`

1. Debería ver una respuesta similar a la siguiente salida:

    ```output
    Assistant: The staging site has been deployed successfully.
    ```

1. Presione <kbd>ENTRAR</kbd> para finalizar el programa.

## Creación de una función de kernel desde un símbolo del sistema

1. Agrega el siguiente código bajo el comentario `Create a kernel function to deploy the staging environment`

     **Python**
    ```python
    # Create a kernel function to deploy the staging environment
    deploy_stage_function = KernelFunctionFromPrompt(
        prompt="""This is the most recent build log:
        {{DevopsPlugin.ReadLogFile}}

        If there are errors, do not deploy the stage environment. Otherwise, invoke the stage deployment function""",
        function_name="DeployStageEnvironment",
        description="Deploy the staging environment"
    )

    kernel.add_function(plugin_name="DeployStageEnvironment", function=deploy_stage_function)
    ```

    **C#**
    ```c#
    // Create a kernel function to deploy the staging environment
    var deployStageFunction = kernel.CreateFunctionFromPrompt(
    promptTemplate: @"This is the most recent build log:
    {{DevopsPlugin.ReadLogFile}}

    If there are errors, do not deploy the stage environment. Otherwise, invoke the stage deployment function",
    functionName: "DeployStageEnvironment",
    description: "Deploy the staging environment"
    );

    kernel.Plugins.AddFromFunctions("DeployStageEnvironment", [deployStageFunction]);
    ```

1. Usa el comando **CTRL+S** para guardar los cambios en el archivo de código.

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para ejecutar la aplicación:

    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. Cuando se te solicite, escribe la siguiente indicación `Please deploy the stage environment`

1. Debería ver una respuesta similar a la siguiente salida:

    ```output
    Assistant: The stage environment cannot be deployed because the earlier stage build failed due to unit test errors. Deploying a faulty build to stage may cause eventual issues and compromise the environment.
    ```

    La respuesta del LLM puede variar, pero aún así te impide implementar el sitio de fase.

## Creación de una indicación de handlebars

1. Agrega el código siguiente en el comentario **Create a handlebars prompt**:

    **Python**
    ```python
    # Create a handlebars prompt
    hb_prompt = """<message role="system">Instructions: Before creating a new branch for a user, request the new branch name and base branch name/message>
        <message role="user">Can you create a new branch?</message>
        <message role="assistant">Sure, what would you like to name your branch? And which base branch would you like to use?</message>
        <message role="user">{{input}}</message>
        <message role="assistant">"""
    ```

    **C#**
    ```c#
    // Create a handlebars prompt
    string hbprompt = """
        <message role="system">Instructions: Before creating a new branch for a user, request the new branch name and base branch name/message>
        <message role="user">Can you create a new branch?</message>
        <message role="assistant">Sure, what would you like to name your branch? And which base branch would you like to use?</message>
        <message role="user">{{input}}</message>
        <message role="assistant">
        """;
    ```

    En este código, crearás una solicitud de pocas etapas con el formato de plantilla de Handlebars. La solicitud guiará al modelo para recuperar más información del usuario antes de crear una nueva rama.

1. Agrega el código siguiente en el comentario **Create the prompt template config using handlebars format**:

    **Python**
    ```python
    # Create the prompt template config using handlebars format
    hb_template = HandlebarsPromptTemplate(
        prompt_template_config=PromptTemplateConfig(
            template=hb_prompt, 
            template_format="handlebars",
            name="CreateBranch", 
            description="Creates a new branch for the user",
            input_variables=[
                InputVariable(name="input", description="The user input", is_required=True)
            ]
        ),
        allow_dangerously_set_content=True,
    )
    ```

    **C#**
    ```c#
    // Create the prompt template config using handlebars format
    var templateFactory = new HandlebarsPromptTemplateFactory();
    var promptTemplateConfig = new PromptTemplateConfig()
    {
        Template = hbprompt,
        TemplateFormat = "handlebars",
        Name = "CreateBranch",
    };
    ```

    En este código, crearás una configuración de plantilla de Handlebars a partir de la solicitud. Puedes usarlo para crear una función de complemento.

1. Agrega el código siguiente en el comentario **Create a plugin function from the prompt**: 

    **Python**
    ```python
    # Create a plugin function from the prompt
    prompt_function = KernelFunctionFromPrompt(
        function_name="CreateBranch",
        description="Creates a branch for the user",
        template_format="handlebars",
        prompt_template=hb_template,
    )
    kernel.add_function(plugin_name="BranchPlugin", function=prompt_function)
    ```

    **C#**
    ```c#
    // Create a plugin function from the prompt
    var promptFunction = kernel.CreateFunctionFromPrompt(promptTemplateConfig, templateFactory);
    var branchPlugin = kernel.CreatePluginFromFunctions("BranchPlugin", [promptFunction]);
    kernel.Plugins.Add(branchPlugin);
    ```

    Este código crea una función de complemento para la solicitud y la agrega al kernel. Ahora estás preparado para invocar tu función.

1. Usa el comando **CTRL+S** para guardar los cambios en el archivo de código.

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para ejecutar la aplicación:

    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. Cuando se te solicite, escribe el siguiente texto: `Please create a new branch`

1. Debería ver una respuesta similar a la siguiente salida:

    ```output
    Assistant: Could you please provide the following details?

    1. The name of the new branch.
    2. The base branch from which the new branch should be created.
    ```

1. Introduce el siguiente texto `feature-login main`

1. Debería ver una respuesta similar a la siguiente salida:

    ```output
    Assistant: The new branch `feature-login` has been successfully created from `main`.
    ```

## Requerimiento del consentimiento del usuario para acciones

1. Cerca de la parte inferior del archivo, busca el comentario **Crear un filtro de función**, y agrega el siguiente código:

    **Python**
    ```python
    # Create a function filter
    async def permission_filter(context: FunctionInvocationContext, next: Callable[[FunctionInvocationContext], Awaitable[None]]) -> None:
        await next(context)
        result = context.result
        
        # Check the plugin and function names
    ```

    **C#**
    ```c#
    // Create a function filter
    class PermissionFilter : IFunctionInvocationFilter
    {
        public async Task OnFunctionInvocationAsync(FunctionInvocationContext context, Func<FunctionInvocationContext, Task> next)
        {
            // Check the plugin and function names
            
            await next(context);
        }
    }
    ```

1. Agrega el siguiente código bajo el comentario **Comprobación de los nombres de complemento y función** para detectar cuándo se invoca la función `DeployToProd`:

     **Python**
    ```python
    # Check the plugin and function names
    if context.function.plugin_name == "DevopsPlugin" and context.function.name == "DeployToProd":
        # Request user approval
        
        # Proceed if approved
    ```

    **C#**
    ```c#
    // Check the plugin and function names
    if ((context.Function.PluginName == "DevopsPlugin" && context.Function.Name == "DeployToProd"))
    {
        // Request user approval

        // Proceed if approved
    }
    ```

    Este código usa el objeto `FunctionInvocationContext` para determinar qué complemento y función se invocaron.

1. Agrega la siguiente lógica para solicitar el permiso del usuario para reservar el vuelo:

     **Python**
    ```python
    # Request user approval
    print("System Message: The assistant requires approval to complete this operation. Do you approve (Y/N)")
    should_proceed = input("User: ").strip()

    # Proceed if approved
    if should_proceed.upper() != "Y":
        context.result = FunctionResult(
            function=result.function,
            value="The operation was not approved by the user",
        )
    ```

    **C#**
    ```c#
    // Request user approval
    Console.WriteLine("System Message: The assistant requires an approval to complete this operation. Do you approve (Y/N)");
    Console.Write("User: ");
    string shouldProceed = Console.ReadLine()!;

    // Proceed if approved
    if (shouldProceed != "Y")
    {
        context.Result = new FunctionResult(context.Result, "The operation was not approved by the user");
        return;
    }
    ```

1. Ve al comentario **Agregar filtros al kernel** y agrega el siguiente código:

    **Python**
    ```python
    # Add filters to the kernel
    kernel.add_filter('function_invocation', permission_filter)
    ```

    **C#**
    ```c#
    // Add filters to the kernel
    kernel.FunctionInvocationFilters.Add(new PermissionFilter());
    ```

1. Usa el comando **CTRL+S** para guardar los cambios en el archivo de código.

1. En el panel de línea de comandos de Cloud Shell, escribe el siguiente comando para ejecutar la aplicación:

    **Python**
    ```
    python devops.py
    ```

    **C#**
    ```
    dotnet run
    ```

1. Escribe un mensaje para implementar la compilación en producción. Debería ver una respuesta similar a la siguiente:

    ```output
    User: Please deploy the build to prod
    System Message: The assistant requires an approval to complete this operation. Do you approve (Y/N)
    User: N
    Assistant: I'm sorry, but I am unable to proceed with the deployment.
    ```

### Revisar

En este laboratorio, creaste un punto de conexión para el servicio de modelo de lenguaje grande (LLM), creaste un objeto de kernel semántico y ejecutaste indicaciones mediante el SDK de kernel semántico. También has creado complementos y has aprovechado los mensajes del sistema para guiar al modelo. Enhorabuena por completar este laboratorio.