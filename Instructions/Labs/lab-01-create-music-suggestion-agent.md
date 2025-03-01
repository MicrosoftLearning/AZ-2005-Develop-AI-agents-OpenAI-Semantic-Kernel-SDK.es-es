---
lab:
  title: 'Laboratorio: Desarrollo de agentes de inteligencia artificial mediante Azure OpenAI y el SDK de Kernel semántico'
  module: 'Module 01: Build your kernel'
---

# Laboratorio: Creación de un agente de recomendación de música de IA
# Manual de laboratorio para alumnos

En este laboratorio, creará el código para un agente de IA que puede administrar la biblioteca de música del usuario y proporcionar recomendaciones personalizadas de canciones y conciertos. Use el SDK de kernel semántico para compilar el agente de IA y conectarlo al servicio de modelo de lenguaje grande (LLM). El SDK de kernel semántico permite crear una aplicación inteligente que pueda interactuar con el servicio LLM y proporcionar recomendaciones personalizadas al usuario.

## Escenario de laboratorio

Pongamos que usted es desarrollador para un servicio internacional de streaming de audio. Se le ha encargado integrar el servicio con IA para proporcionar a los usuarios una experiencia más personalizada. La inteligencia artificial debe ser capaz de recomendar canciones y las próximas giras de artistas en función del historial y las preferencias de escucha del usuario. Decide usar el SDK de kernel semántico para crear un agente de IA que pueda interactuar con el servicio de modelo de lenguaje grande (LLM).

## Objetivos

Al completar este laboratorio, realizará lo siguiente:

* Crear un punto de conexión para el servicio de modelo de lenguaje grande (LLM)
* Compilar un objeto Kernel semántico
* Ejecutar indicaciones mediante el SDK de kernel semántico
* Crear funciones y complementos semánticos del kernel
* Habilitar llamadas a funciones automáticas para automatizar tareas

## Configuración del laboratorio

### Requisitos previos

Para completar el ejercicio, necesita que las herramientas siguientes estén instaladas en el sistema:

* [Visual Studio Code](https://code.visualstudio.com)
* [El SDK de .NET 8.0 más reciente](https://dotnet.microsoft.com/en-us/download/dotnet/8.0)
* La [extensión de C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) para Visual Studio Code.


### Preparación del entorno de desarrollo

Para estos ejercicios, hay disponible un proyecto de inicio para su uso. Siga estos pasos para configurar el proyecto de inicio:

> [!IMPORTANT]
> Debe tener instalado .NET Framework 8.0, así como las extensiones de VS Code para C# y el Administrador de paquetes NuGet.

1. Pega el siguiente vínculo en una nueva ventana del explorador:
   
     `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/01/Lab-01-Starter.zip`

1. Descarga el archivo ZIP haciendo clic en el botón `...` situado en la parte superior derecha de la página o presiona <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>S</kbd>.

1. Extraiga el contenido del archivo ZIP en una ubicación fácil de encontrar y recordar, como una carpeta en el escritorio.

1. Abra Visual Studio Code y seleccione **Archivo** > **Abrir carpeta**.

1. Vaya a la carpeta **Starter** que extrajo antes y elija **Seleccionar carpeta**.

1. Abra el archivo **Program.cs** en el editor de código.

> [!NOTE]
> Si se te pide que confíes en la carpeta, selecciona **Yes, I trust the authors** 

## Ejercicio 1: Ejecución de una indicación con el SDK de kernel semántico

En este ejercicio, creará un punto de conexión para el servicio de modelo de lenguaje grande (LLM). El SDK de kernel semántico usa este punto de conexión para conectarse a LLM y ejecutar solicitudes. El SDK de kernel semántico admite los LLM HuggingFace, OpenAI y Azure Open AI. En este ejemplo, se usa Azure Open AI.

**Tiempo de finalización del ejercicio estimado**: 10 minutos

### Tarea 1: Crear un recurso de Azure OpenAI

1. Vaya a [https://portal.azure.com](https://portal.azure.com).

1. Cree un nuevo recurso de Azure OpenAI con la configuración predeterminada.

    > [!NOTE]
    > Si ya tiene un recurso de Azure OpenAI, puede omitir este paso.

1. Tras crear el nuevo recurso, seleccione **Ir al recurso**.

1. En la página **Información general**, selecciona **Ir al portal de Azure AI Foundry**.

1. Selecciona **Crear nueva implementación** y después **desde modelos básicos**.

1. En la lista de modelos, selecciona **gpt-35-turbo-16k**.

1. Seleccione **Confirmar**

1. Escribe un nombre para la implementación y deja las opciones predeterminadas.

1. Cuando se complete la implementación, vuelve a tu recurso Azure OpenAI en Azure Portal.

1. En **Administración de recursos**, vaya a **Claves y puntos de conexión**.

    Usará los datos en la siguiente tarea para compilar el kernel. Recuerde mantener sus claves privadas y seguras.

### Tarea 2: Compilación del kernel

En este ejercicio, aprenderá a compilar el primer proyecto del SDK de kernel semántico. Aprenderá a crear un nuevo proyecto, agregar el paquete NuGet del SDK de kernel semántico y agregar una referencia al SDK de kernel semántico. Comencemos.

1. Vuelva a su proyecto de Visual Studio Code.

1. Abre el archivo **appsettings.json** y actualiza los valores con tu Id. de modelo, punto de conexión y clave de API de los servicios de Azure OpenAI.

    ```json
    {
        "modelId": "gpt-35-turbo-16k",
        "endpoint": "",
        "apiKey": ""
    }
    ```

1. Abra el terminal; para ello, seleccione **Terminal** > **Nuevo terminal**.

1. En terminal, ejecute el siguiente comando para instalar el SDK de kernel semántico:

    `dotnet add package Microsoft.SemanticKernel --version 1.30.0`

1. 1. Agrega las siguientes directivas `using` al archivo **Program.cs**:

    ```c#
    using Microsoft.SemanticKernel;
    using Microsoft.SemanticKernel.ChatCompletion;
    using Microsoft.SemanticKernel.Connectors.OpenAI;
    ```

1. Para crear el kernel, agrega el código siguiente a tu archivo **Program.cs**:
    
    ```c#
    // Create a kernel builder with Azure OpenAI chat completion
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(modelId, endpoint, apiKey);

    // Build the kernel
    var kernel = builder.Build();
    ```

1. Para verificar que el kernel y el punto de conexión funcionan, escriba el código siguiente:

    ```c#
    var result = await kernel.InvokePromptAsync("Who are the top 5 most famous musicians in the world?");
    Console.WriteLine(result);
    ```

1. Haz clic con el botón derecho del ratón en la carpeta **Starter** y selecciona **Abrir en terminal integrado**.

1. En el terminal, escribe `dotnet run` para ejecutar el código.

    Comprueba que ves una respuesta del modelo Azure Open AI que contiene los cinco músicos más famosos del mundo.

    La respuesta procede del modelo de Azure Open AI que ha pasado al kernel. El SDK de kernel semántico puede conectarse al modelo de lenguaje grande (LLM) y ejecutar la consulta. Observe lo rápido que pudo recibir respuestas de LLM. El SDK de kernel semántico hace que la creación de aplicaciones inteligentes sea fácil y eficaz.

## Ejercicio 2: Creación de complementos de biblioteca de música personalizados

En este ejercicio, creará complementos personalizados para la biblioteca de música. Creará funciones que pueden agregar canciones a la lista de reproducción reciente del usuario, obtener la lista de canciones reproducidas recientemente y proporcionar recomendaciones de canciones personalizadas. También crea una función que sugiere un concierto basado en la ubicación del usuario y sus canciones reproducidas recientemente.

**Tiempo de finalización del ejercicio estimado**: 15 minutos

### Tarea 1: Creación de un complemento de biblioteca de música

En esta tarea, creará un complemento que le permite agregar canciones a la lista de reproducción reciente del usuario y obtener la lista de canciones reproducidas recientemente. Por motivos de simplicidad, las canciones reproducidas recientemente se almacenan en un archivo de texto.

1. En la carpeta **Plugins**, crea un nuevo archivo **MusicLibraryPlugin.cs**

    En primer lugar, cree algunas funciones rápidas para obtener y agregar canciones a la lista de "Reproducciones recientes" del usuario.

1. Escriba el siguiente código:

    ```c#
    using System.Text.Json;
    using System.Text.Json.Nodes;
    using Microsoft.SemanticKernel;

    public class MusicLibraryPlugin
    {
        [KernelFunction("GetRecentPlays")]
        public static string GetRecentPlays()
        {
            string content = File.ReadAllText($"Files/RecentlyPlayed.txt");
            return content;
        }
    }
    ```

    En este código, se usa el decorador para declarar la función nativa `KernelFunction`. Usa un nombre descriptivo para la función para que la IA pueda llamarla correctamente. La lista de reproducciones recientes del usuario se almacena en un archivo de texto denominado "RecentlyPlayed.txt". A continuación, puede agregar código para agregar una canción a la lista.

1. Agregue el siguiente código a la clase `MusicLibraryPlugin`:

    ```c#
    [KernelFunction("AddToRecentPlays")]
    public static string AddToRecentlyPlayed(string artist,  string song, string genre)
    {
        // Read the existing content from the file
        string filePath = "Files/RecentlyPlayed.txt";
        string jsonContent = File.ReadAllText(filePath);
        var recentlyPlayed = (JsonArray) JsonNode.Parse(jsonContent)!;

        var newSong = new JsonObject
        {
            ["title"] = song,
            ["artist"] = artist,
            ["genre"] = genre
        };

        // Insert the new song
        recentlyPlayed.Insert(0, newSong);
        File.WriteAllText(filePath, JsonSerializer.Serialize(recentlyPlayed,
            new JsonSerializerOptions { WriteIndented = true }));

        return $"Added '{song}' to recently played";
    }
    ```

    En este código, se crea una función que acepta el artista, la canción y el género como cadenas. El archivo "RecentlyPlayed.txt" contiene una lista con formato JSON de canciones que el usuario ha reproducido recientemente. Este código lee el contenido existente del archivo, lo analiza y agrega la nueva canción a la lista. Después, la lista actualizada se vuelve a escribir en el archivo.

1. Actualice el archivo **Program.cs** con el código siguiente:

    ```c#
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();

    // Get chat completion service.
    var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();

    // Create a chat history object
    ChatHistory chatHistory = [];
    ```

    En este código, importarás el complemento al kernel y agregarás la configuración para la finalización del chat.

1. Agrega las siguientes solicitudes para invocar el complemento:

    ```c#
    chatHistory.AddSystemMessage("When a user has played a song, add it to their list of recent plays.");
    chatHistory.AddSystemMessage("The listener has just played the song Danse by Tiara. It falls under these genres: French pop, electropop, pop.");

    ChatMessageContent reply = await chatCompletionService.GetChatMessageContentAsync(
        chatHistory,
        kernel: kernel
    );
    Console.WriteLine(reply.ToString());
    chatHistory.AddAssistantMessage(reply.ToString());
    ```

1. Para ejecutar el código, escriba `dotnet run` en el terminal.

    Debería ver la salida siguiente:

    ```output
    Added 'Danse' to recently played
    ```

    Si abres "Files/RececentlyPlayed.txt", deberías ver la nueva canción agregada a la lista.

### Tarea 2: Proporcionar recomendaciones de canciones personalizadas

En esta tarea, creará una indicación que proporciona recomendaciones personalizadas de canciones al usuario en función de sus canciones reproducidas recientemente. La indicación combina las funciones nativas para generar una recomendación de canción. También puede crear una función a partir de una indicación para que sea reutilizable.

1. En tu archivo **MusicLibraryPlugin.cs**, agrega la siguiente función:

    ```c#
    [KernelFunction("GetMusicLibrary")]
    public static string GetMusicLibrary()
    {
        string dir = Directory.GetCurrentDirectory();
        string content = File.ReadAllText($"Files/MusicLibrary.txt");
        return content;
    }
    ```

    Esta función lee la lista de música disponible de un archivo denominado "MusicLibrary.txt". El archivo contiene una lista con formato JSON de canciones disponibles para el usuario.

1. Actualice el archivo **Program.cs** con el código siguiente:

    ```c#
    chatHistory.AddSystemMessage("When a user has played a song, add it to their list of recent plays.");
    
    string prompt = @"This is a list of music available to the user:
        {{MusicLibraryPlugin.GetMusicLibrary}} 

        This is a list of music the user has recently played:
        {{MusicLibraryPlugin.GetRecentPlays}}

        Based on their recently played music, suggest a song from
        the list to play next";

    var result = await kernel.InvokePromptAsync(prompt);
    Console.WriteLine(result);
    ```

    En primer lugar, puedes quitar el código que anexa una canción a la lista. Después, combinas las funciones nativas de tu complemento con una solicitud semántica. Las funciones nativas pueden recuperar los datos de usuario a los que el modelo de lenguaje de gran tamaño (LLM) no pudo acceder por sí mismo. Además, el LLM puede generar una recomendación de canción basada en la entrada de texto.

1. Para probar el código, escriba `dotnet run` en el terminal.

    Debería ver una respuesta similar a la siguiente salida:

    ```output 
    Based on the user's recently played music, a suggested song to play next could be "Sabry Aalil" by Yasemin since the user seems to enjoy pop and Egyptian pop music.
    ```

    >[!NOTE] La canción recomendada puede ser diferente de la que se muestra aquí.

1. Modifique el código para crear una función desde la indicación:

    ```c#
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

    En este código, creas una función desde tu solicitud y la agregas a los complementos del kernel.

1. Agrega el código siguiente para invocar automáticamente la función:

    ```c#
    OpenAIPromptExecutionSettings openAIPromptExecutionSettings = new() 
    {
        FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
    };

    chatHistory.AddUserMessage("What song should I play next?");

    reply = await chatCompletionService.GetChatMessageContentAsync(
        chatHistory,
        kernel: kernel,
        executionSettings: openAIPromptExecutionSettings
    );
    Console.WriteLine(reply.ToString());
    chatHistory.AddAssistantMessage(reply.ToString());
    ```

    En este código, creas la configuración para habilitar la llamada automática a funciones. Después, agregas una solicitud que invoque la función y recupere la respuesta.

### Tarea 3: Proporcionar recomendaciones de concierto personalizadas

En esta tarea, creas un complemento que pide al LLM que sugiera un concierto basándose en las últimas canciones reproducidas por el usuario y su ubicación 

1. En tu archivo **Program.cs**, agrega el complemento de conciertos de música al kernel:

    ```c#
    var kernel = builder.Build();    
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    ```

1. Agrega código para crear una función desde una solicitud:

    ```c#
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

    Esta solicitud de función obtiene la biblioteca de música y la información sobre los próximos conciertos, así como la ubicación del usuario, y proporciona una recomendación.

1. Agrega la siguiente solicitud para invocar la nueva función del complemento:

    ```c#
    chatHistory.AddUserMessage("Can you recommend a concert for me? I live in Washington");

    reply = await chatCompletionService.GetChatMessageContentAsync(
        chatHistory,
        kernel: kernel,
        executionSettings: openAIPromptExecutionSettings
    );
    Console.WriteLine(reply.ToString());
    chatHistory.AddAssistantMessage(reply.ToString());
    ```

1. En el terminal, escriba `dotnet run`.

    Debería ver una salida similar a la siguiente respuesta:

    ```output
    I recommend you attend the concert of Lisa Taylor. She will be performing in Seattle, Washington, USA on 2/22/2024. Enjoy the show!
    ```
    
    La respuesta del LLM puede variar. Pruebe a retocar la solicitud y la ubicación para ver qué otros resultados se pueden generar.

Ahora, tu agente puede realizar diferentes acciones automáticamente en función de la entrada del usuario. ¡Excelente trabajo!

### Revisar

En este laboratorio, ha creado un agente de inteligencia artificial que puede administrar la biblioteca de música del usuario y proporcionar recomendaciones personalizadas de canciones y conciertos. Use el SDK de kernel semántico para compilar el agente de IA y conectarlo al servicio de modelo de lenguaje grande (LLM). Has creado complementos personalizados para tu biblioteca de música, has habilitado la llamada automática a funciones para que tu agente responda dinámicamente a la entrada del usuario. Enhorabuena por completar este laboratorio.
