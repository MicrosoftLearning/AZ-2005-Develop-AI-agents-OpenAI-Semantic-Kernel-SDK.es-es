---
lab:
  title: 'Laboratorio: Desarrollo de agentes de inteligencia artificial mediante Azure OpenAI y el SDK de Kernel semántico'
  module: 'Module 01: Build your kernel'
---

# Laboratorio: Creación de un asistente de recomendación de música de IA
# Manual de laboratorio para estudiantes

En este laboratorio, crearás el código para un asistente de IA que puede administrar la biblioteca de música del usuario y proporcionar recomendaciones personalizadas de canciones y conciertos. Usarás el SDK de kernel semántico para crear el asistente de IA y conectarlo al servicio de modelo de lenguaje grande (LLM). El SDK de kernel semántico permite crear una aplicación inteligente que pueda interactuar con el servicio LLM y proporcionar recomendaciones personalizadas al usuario.

## Escenario de laboratorio

Pongamos que eres desarrollador para un servicio internacional de streaming de audio. Se te ha encargado integrar el servicio con IA para proporcionar a los usuarios una experiencia más personalizada. La inteligencia artificial debe ser capaz de recomendar canciones y las próximas giras de artistas en función del historial y las preferencias de escucha del usuario. Decides usar el SDK de kernel semántico para crear un asistente de IA que pueda interactuar con el servicio de modelo de lenguaje grande (LLM).

## Objetivos

Al completar este laboratorio, realizarás lo siguiente:

* Crear un punto de conexión para el servicio de modelo de lenguaje grande (LLM)
* Compilar un objeto Kernel semántico
* Ejecutar indicaciones mediante el SDK de kernel semántico
* Crear funciones y complementos semánticos del kernel
* Habilitar llamadas a funciones automáticas para automatizar tareas

## Configuración del laboratorio

### Requisitos previos

Para completar el ejercicio, necesitas que las herramientas siguientes estén instaladas en el sistema:

* [Visual Studio Code](https://code.visualstudio.com)
* [El SDK de .NET 8.0 más reciente](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
* La [extensión de C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) para Visual Studio Code.


### Preparación del entorno de desarrollo

Para estos ejercicios, hay disponible un proyecto de inicio para su uso. Sigue estos pasos para configurar el proyecto de inicio:

> [!IMPORTANT]
> Debes tener instalado .NET Framework 8.0, así como las extensiones de VS Code para C# y el Administrador de paquetes NuGet.

1. Pega el siguiente vínculo en una nueva ventana del explorador:
   
     `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/01/Lab-01-Starter.zip`

1. Descarga el archivo ZIP haciendo clic en el botón `...` situado en la parte superior derecha de la página o presiona <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>S</kbd>.

1. Extrae el contenido del archivo ZIP en una ubicación fácil de encontrar y recordar, como una carpeta en el escritorio.

1. Abre Visual Studio Code y selecciona **Archivo** > **Abrir carpeta**.

1. Ve a la carpeta **Starter** que extraíste antes y elige **Seleccionar carpeta**.

1. Abre el archivo **Program.cs** en el editor de código.

> [!NOTE]
> Si se te pide que confíes en la carpeta, selecciona **Yes, I trust the authors** 

## Ejercicio 1: Ejecución de una indicación con el SDK de kernel semántico

En este ejercicio, crearás un punto de conexión para el servicio de modelo de lenguaje grande (LLM). El SDK de kernel semántico usa este punto de conexión para conectarse a LLM y ejecutar solicitudes. El SDK de kernel semántico admite los LLM HuggingFace, OpenAI y Azure Open AI. En este ejemplo, se usa Azure Open AI.

**Tiempo de finalización del ejercicio estimado**: 10 minutos

### Tarea 1: Creación de un recurso de Azure OpenAI

1. Ve a [https://portal.azure.com](https://portal.azure.com).

1. Crea un nuevo recurso de Azure OpenAI con la configuración predeterminada.

    > [!NOTE]
    > Si ya tienes un recurso de Azure OpenAI, puedes omitir este paso.

1. Tras crear el nuevo recurso, selecciona **Ir al recurso**.

1. En la página **Información general**, selecciona **Ir al Portal de la Fundición de IA de Azure**.

1. Selecciona **Crear nueva implementación** y después **desde modelos básicos**.

1. En la lista de modelos, selecciona **gpt-35-turbo-16k**.

1. Selecciona **Confirmar**

1. Escribe un nombre para la implementación y deja las opciones predeterminadas.

1. Cuando se complete la implementación, vuelve a tu recurso de Azure OpenAI en Azure Portal.

1. En **Administración de recursos**, ve a **Claves y puntos de conexión**.

    Usarás los datos en la siguiente tarea para compilar el kernel. Recuerda mantener tus claves privadas y seguras.

### Tarea 2: Compilación del kernel

En este ejercicio, aprenderás a compilar el primer proyecto del SDK de kernel semántico. Aprenderás a crear un nuevo proyecto, agregar el paquete NuGet del SDK de kernel semántico y agregar una referencia al SDK de kernel semántico. Comencemos.

1. Vuelve al proyecto de Visual Studio Code.

1. Abre el archivo **appsettings.json** y actualiza los valores con tu Id. de modelo, punto de conexión y clave de API de los servicios de Azure OpenAI.

    ```json
    {
        "modelId": "gpt-35-turbo-16k",
        "endpoint": "",
        "apiKey": ""
    }
    ```

1. Abre el terminal; para ello, selecciona **Terminal** > **Nuevo terminal**.

1. En terminal, ejecuta el siguiente comando para instalar el SDK de kernel semántico:

    `dotnet add package Microsoft.SemanticKernel --version 1.30.0`

1. 1. Agrega las siguientes directivas `using` al archivo **Program.cs**:

    ```c#
    using Microsoft.SemanticKernel;
    using Microsoft.SemanticKernel.ChatCompletion;
    using Microsoft.SemanticKernel.Connectors.OpenAI;
    ```

1. Agrega el código siguiente en el comentario **Creación de un generador de kernel con la finalización del chat de Azure OpenAI**:

    ```c#
    // Create a kernel builder with Azure OpenAI chat completion
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(modelId, endpoint, apiKey);
    ```

1. Compila el kernel agregando este código en el comentario **Compilar el kernel**:

    ```c#
    // Build the kernel
    var kernel = builder.Build();
    ```

1. Agrega el siguiente código en el comentario **Comprobar punto de conexión y ejecutar una indicación**:

    ```c#
    // Verify the endpoint and run a prompt
    var result = await kernel.InvokePromptAsync("Who are the top 5 most famous musicians in the world?");
    Console.WriteLine(result);
    ```

1. Haz clic con el botón derecho del ratón en la carpeta **Starter** y selecciona **Abrir en terminal integrado**.

1. En el terminal, escribe `dotnet run` para ejecutar el código.

    Comprueba que ves una respuesta del modelo Azure Open AI que contiene los cinco músicos más famosos del mundo.

    La respuesta procede del modelo de Azure Open AI que ha pasado al kernel. El SDK de kernel semántico puede conectarse al modelo de lenguaje grande (LLM) y ejecutar la consulta. Observa lo rápido que pudiste recibir respuestas de LLM. El SDK de kernel semántico hace que la creación de aplicaciones inteligentes sea fácil y eficaz.

Puedes quitar el código de verificación una vez que confirmes la respuesta.

## Ejercicio 2: Creación de complementos de biblioteca de música personalizados

En este ejercicio, crearás complementos personalizados para la biblioteca de música. Crearás funciones que pueden agregar canciones a la lista de reproducción reciente del usuario, obtener la lista de canciones reproducidas recientemente y proporcionar recomendaciones de canciones personalizadas. También crea una función que sugiere un concierto basado en la ubicación del usuario y sus canciones reproducidas recientemente.

**Tiempo de finalización del ejercicio estimado**: 15 minutos

### Tarea 1: Creación de un complemento de biblioteca de música

En esta tarea, crearás un complemento que te permite agregar canciones a la lista de reproducción reciente del usuario y obtener la lista de canciones reproducidas recientemente. Por motivos de simplicidad, las canciones reproducidas recientemente se almacenan en un archivo de texto.

1. En la carpeta **Plugins**, abre el archivo **MusicLibraryPlugin.cs**

1. En el comentario **Crear una función de kernel para reproducir canciones recientemente**, agrega el decorador de la función kernel:


    ```c#
    // Create a kernel function to get recently played songs
    [KernelFunction("GetRecentPlays")]
    public static string GetRecentPlays()
    ```

    El decorador `KernelFunction` declara la función nativa. Usa un nombre descriptivo para la función para que la IA pueda llamarla correctamente. La lista de reproducciones recientes del usuario se almacena en un archivo de texto denominado "RecentlyPlayed.txt".

1. En el comentario **Crear una función kernel para agregar una canción a la lista reproducida recientemente**, agrega el decorador de la función kernel:

    ```c#
    // Create a kernel function to add a song to the recently played list
    [KernelFunction("AddToRecentPlays")]
    public static string AddToRecentlyPlayed(string artist,  string song, string genre)
    ```

    Ahora, cuando se agrega esta clase de complemento al kernel, podrá identificar e invocar las funciones.

1. Ve al archivo **Program.cs**.

1. Agrega el código siguiente en el comentario **Importar complementos al kernel**:

    ```c#
    // Import plugins to the kernel
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    ```

1. En el comentario **Crear configuración de ejecución de indicación**, agrega el código siguiente para invocar automáticamente la función:

    ```c#
    // Create prompt execution settings
    OpenAIPromptExecutionSettings openAIPromptExecutionSettings = new() 
    {
        FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
    };
    ```

    El uso de esta configuración permitirá que el kernel invoque automáticamente las funciones sin necesidad de especificarlas en la indicación.

1. Agrega el código siguiente en el comentario **Obtener servicio de finalización del chat**:

    ```c#
    // Get chat completion service.
    var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();
    ChatHistory chatHistory = [];
    ```

1. Agrega el código siguiente en el comentario **Crear una función auxiliar para esperar y generar la respuesta desde el servicio de finalización del chat**:

    ```c#
    // Create a helper function to await and output the reply from the chat completion service
    async Task GetAssistantReply() {
        ChatMessageContent reply = await chatCompletionService.GetChatMessageContentAsync(
            chatHistory,
            kernel: kernel,
            executionSettings: openAIPromptExecutionSettings
        );
        chatHistory.AddAssistantMessage(reply.ToString());
        Console.WriteLine(reply.ToString());
    }
    ```


1. Agrega el código siguiente en el comentario **Agregar mensajes del sistema al chat**:

    ```c#
    // Add system messages to the chat
    chatHistory.AddSystemMessage("When a user has played a song, add it to their list of recent plays.");
    chatHistory.AddSystemMessage("The listener has just played the song Danse by Tiara. It falls under these genres: French pop, electropop, pop.");
    await GetAssistantReply();
    ```

1. Para ejecutar el código, escribe `dotnet run` en el terminal.

    Las indicaciones del sistema que has agregado deben invocar las funciones del complemento y deben ver la siguiente salida:

    ```output
    Added 'Danse' to recently played
    ```

    Si abres **Files/RececentlyPlayed.txt**, deberías ver la nueva canción agregada a la lista.

### Tarea 2: Proporcionar recomendaciones de canciones personalizadas

En esta tarea, crearás una indicación que proporciona recomendaciones personalizadas de canciones al usuario en función de sus canciones reproducidas recientemente. La indicación combina las funciones nativas para generar una recomendación de canción. También puedes crear una función a partir de una indicación para que sea reutilizable.

1. En el archivo **Program.cs**, agrega el código siguiente en el comentario **Crear una función del proveedor de sugerencias de canciones mediante una indicación**:

    ```c#
    // Create a song suggester function using a prompt
    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"This is a list of music available to the user:
        {{MusicLibraryPlugin.GetMusicLibrary}} 

        This is a list of music the user has recently played:
        {{MusicLibraryPlugin.GetRecentPlays}}

        Based on their recently played music, suggest a song from
        the list to play next",
        functionName: "SuggestSong",
        description: "Suggest a song to the user"
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);
    ```

    En este código, creas una función desde tu indicación y la agregas a los complementos del kernel.

1. En el comentario **Invocar la función del proveedor de sugerencias de canciones con una indicación del usuario***, agrega el código siguiente:

    ```c#
    // Invoke the song suggester function with a prompt from the user
    chatHistory.AddUserMessage("What song should I play next?");
    await GetAssistantReply();
    ```

    Ahora la aplicación puede invocar automáticamente las funciones del complemento según la solicitud del usuario. Vamos a ampliar este código para proporcionar recomendaciones de conciertos, así como en función de la información del usuario.

### Tarea 3: Proporcionar recomendaciones de concierto personalizadas

En esta tarea, creas un complemento que pide al LLM que sugiera un concierto basándose en las últimas canciones reproducidas por el usuario y su ubicación 

1. En el archivo **Program.cs**, importa el complemento de conciertos en el comentario **Importar complementos al kernel**:

    ```c#
    // Import plugins to the kernel 
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    ```

1. Agrega el código siguiente en el comentario **Crear una función del proveedor de sugerencias de conciertos mediante una indicación**:

    ```c#
    // Create a concert suggester function using a prompt
    var concertSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"This is a list of the user's recently played songs:
        {{MusicLibraryPlugin.GetRecentPlays}}

        This is a list of upcoming concert details:
        {{MusicConcertsPlugin.GetConcerts}}

        Suggest an upcoming concert based on the user's recently played songs. 
        The user lives in {{$location}}, 
        please recommend a relevant concert that is close to their location.",
        functionName: "SuggestConcert",
        description: "Suggest a concert to the user"
    );

    kernel.Plugins.AddFromFunctions("SuggestConcertPlugin", [concertSuggesterFunction]);
    ```

    Esta indicación de función obtiene la biblioteca de música y la información sobre los próximos conciertos, así como la ubicación del usuario, y proporciona una recomendación.

1. Agrega la siguiente solicitud para invocar la nueva función del complemento:

    ```c#
    // Invoke the concert suggester function with a prompt from the user
    chatHistory.AddUserMessage("Can you recommend a concert for me? I live in Washington");
    await GetAssistantReply();
    ```

1. En el terminal, escribe `dotnet run`.

    Deberías ver una salida similar a la siguiente respuesta:

    ```output
    I recommend you attend the concert of Lisa Taylor. She will be performing in Seattle, Washington, USA on 2/22/2024. Enjoy the show!
    ```
    
    La respuesta del LLM puede variar. Prueba a retocar la indicación y la ubicación para ver qué otros resultados se pueden generar.

Ahora, tu asistente puede realizar diferentes acciones automáticamente en función de la entrada del usuario. ¡Excelente trabajo!

### Revisar

En este laboratorio, has creado un asistente de IA que puede administrar la biblioteca de música del usuario y proporcionar recomendaciones personalizadas de canciones y conciertos. Usa el SDK de kernel semántico para crear el asistente de IA y conectarlo al servicio de modelo de lenguaje grande (LLM). Has creado complementos personalizados para tu biblioteca de música, has habilitado la llamada automática a funciones para que tu asistente responda dinámicamente a la entrada del usuario. Enhorabuena por completar este laboratorio.
