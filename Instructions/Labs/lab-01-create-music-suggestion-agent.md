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
* Uso del plan Handlebars para automatizar tareas

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

1. En la página **Información general**, seleccione **Ir a Azure OpenAI Studio**.

:::image type="content" source="../media/model-deployments.png" alt-text="Captura de pantalla de la página Implementaciones de Azure OpenAI.":::

1. Seleccione **Crear nueva implementación** y, después, **Implementar modelo**.

1. En **Seleccionar un modelo**, selecciona **gpt-35-turbo-16k**.

    Uso de la versión predeterminada del modelo

1. Escriba un nombre para la implementación

1. Cuando se complete la implementación, vuelva al recurso de Azure OpenAI.

1. En **Administración de recursos**, vaya a **Claves y puntos de conexión**.

    Usará los datos en la siguiente tarea para compilar el kernel. Recuerde mantener sus claves privadas y seguras.

### Tarea 2: Compilación del kernel

En este ejercicio, aprenderá a compilar el primer proyecto del SDK de kernel semántico. Aprenderá a crear un nuevo proyecto, agregar el paquete NuGet del SDK de kernel semántico y agregar una referencia al SDK de kernel semántico. Comencemos.

1. Vuelva a su proyecto de Visual Studio Code.

1. Abra el terminal; para ello, seleccione **Terminal** > **Nuevo terminal**.

1. En terminal, ejecute el siguiente comando para instalar el SDK de kernel semántico:

    `dotnet add package Microsoft.SemanticKernel --version 1.2.0`

1. Para crear el kernel, agregue el código siguiente al archivo "Program.cs":
    
    ```c#
    using Microsoft.SemanticKernel;

    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(
        "your-deployment-name",
        "your-endpoint",
        "your-api-key",
        "deployment-model");
    var kernel = builder.Build();
    ```

    Asegúrese de reemplazar los marcadores de posición por los valores del recurso de Azure.

1. Para verificar que el kernel y el punto de conexión funcionan, escriba el código siguiente:

    ```c#
    var result = await kernel.InvokePromptAsync(
        "Who are the top 5 most famous musicians in the world?");
    Console.WriteLine(result);
    ```

1. Introduce `dotnet run` para ejecutar el código y comprueba que ves una respuesta del modelo Azure Open AI que contiene los cinco músicos más famosos del mundo.

    La respuesta procede del modelo de Azure Open AI que ha pasado al kernel. El SDK de kernel semántico puede conectarse al modelo de lenguaje grande (LLM) y ejecutar la consulta. Observe lo rápido que pudo recibir respuestas de LLM. El SDK de kernel semántico hace que la creación de aplicaciones inteligentes sea fácil y eficaz.

## Ejercicio 2: Creación de complementos de biblioteca de música personalizados

En este ejercicio, creará complementos personalizados para la biblioteca de música. Creará funciones que pueden agregar canciones a la lista de reproducción reciente del usuario, obtener la lista de canciones reproducidas recientemente y proporcionar recomendaciones de canciones personalizadas. También crea una función que sugiere un concierto basado en la ubicación del usuario y sus canciones reproducidas recientemente.

**Tiempo de finalización del ejercicio estimado**: 15 minutos

### Tarea 1: Creación de un complemento de biblioteca de música

En esta tarea, creará un complemento que le permite agregar canciones a la lista de reproducción reciente del usuario y obtener la lista de canciones reproducidas recientemente. Por motivos de simplicidad, las canciones reproducidas recientemente se almacenan en un archivo de texto.

1. Cree una nueva carpeta en el directorio "Lab01-Project" y asígnela el nombre "Complementos".

1. En la carpeta "Plugins", cree un nuevo archivo "MusicLibrary.cs"

    En primer lugar, cree algunas funciones rápidas para obtener y agregar canciones a la lista de "Reproducciones recientes" del usuario.

1. Escriba el siguiente código:

    ```c#
    using System.ComponentModel;
    using System.Text.Json;
    using System.Text.Json.Nodes;
    using Microsoft.SemanticKernel;

    public class MusicLibraryPlugin
    {
        [KernelFunction, 
        Description("Get a list of music recently played by the user")]
        public static string GetRecentPlays()
        {
            string content = File.ReadAllText($"Files/RecentlyPlayed.txt");
            return content;
        }
    }
    ```

    En este código, se usa el decorador para declarar la función nativa `KernelFunction`. También se usa el decorador `Description` para agregar una descripción de lo que hace la función. La lista de reproducciones recientes del usuario se almacena en un archivo de texto denominado "RecentlyPlayed.txt". A continuación, puede agregar código para agregar una canción a la lista.

1. Agregue el siguiente código a la clase `MusicLibraryPlugin`:

    ```c#
    [KernelFunction, Description("Add a song to the recently played list")]
    public static string AddToRecentlyPlayed(
        [Description("The name of the artist")] string artist, 
        [Description("The title of the song")] string song, 
        [Description("The song genre")] string genre)
    {
        // Read the existing content from the file
        string filePath = "Files/RecentlyPlayed.txt";
        string jsonContent = File.ReadAllText(filePath);
        var recentlyPlayed = (JsonArray) JsonNode.Parse(jsonContent);

        var newSong = new JsonObject
        {
            ["title"] = song,
            ["artist"] = artist,
            ["genre"] = genre
        };

        recentlyPlayed.Insert(0, newSong);
        File.WriteAllText(filePath, JsonSerializer.Serialize(recentlyPlayed,
            new JsonSerializerOptions { WriteIndented = true }));

        return $"Added '{song}' to recently played";
    }
    ```

    En este código, se crea una función que acepta el artista, la canción y el género como cadenas. Además de la `Description` de la función, también se agregan descripciones de las variables de entrada. El archivo "RecentlyPlayed.txt" contiene una lista con formato JSON de canciones que el usuario ha reproducido recientemente. Este código lee el contenido existente del archivo, lo analiza y agrega la nueva canción a la lista. Después, la lista actualizada se vuelve a escribir en el archivo.

1. Actualice el archivo 'Program.cs' con el código siguiente:

    ```c#
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();

    var result = await kernel.InvokeAsync(
        "MusicLibraryPlugin", 
        "AddToRecentlyPlayed", 
        new() {
            ["artist"] = "Tiara", 
            ["song"] = "Danse", 
            ["genre"] = "French pop, electropop, pop"
        }
    );
    
    Console.WriteLine(result);
    ```

    En este código, se importa el `MusicLibraryPlugin` al kernel usando `ImportPluginFromType`. Después se llama a `InvokeAsync` con el nombre del complemento y el nombre de la función a la que se quiere llamar. También se pasan el artista, la canción y el género como argumentos.

1. Para ejecutar el código, escriba `dotnet run` en el terminal.

    Debería ver la salida siguiente:

    ```output
    Added 'Danse' to recently played
    ```

    Si abres "Files/RececentlyPlayed.txt", deberías ver la nueva canción agregada a la lista.

### Tarea 2: Proporcionar recomendaciones de canciones personalizadas

En esta tarea, creará una indicación que proporciona recomendaciones personalizadas de canciones al usuario en función de sus canciones reproducidas recientemente. La indicación combina las funciones nativas para generar una recomendación de canción. También puede crear una función a partir de una indicación para que sea reutilizable.

1. En el archivo `MusicLibraryPlugin.cs`, agregue la siguiente función:

    ```c#
    [KernelFunction, Description("Get a list of music available to the user")]
    public static string GetMusicLibrary()
    {
        string dir = Directory.GetCurrentDirectory();
        string content = File.ReadAllText($"Files/MusicLibrary.txt");
        return content;
    }
    ```

    Esta función lee la lista de música disponible de un archivo denominado "MusicLibrary.txt". El archivo contiene una lista con formato JSON de canciones disponibles para el usuario.

1. Actualice el archivo "Program.cs" con el código siguiente:

    ```c#
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    
    string prompt = @"This is a list of music available to the user:
        {{MusicLibraryPlugin.GetMusicLibrary}} 

        This is a list of music the user has recently played:
        {{MusicLibraryPlugin.GetRecentPlays}}

        Based on their recently played music, suggest a song from
        the list to play next";

    var result = await kernel.InvokePromptAsync(prompt);
    Console.WriteLine(result);
    ```

En este código, combinará las funciones nativas con una solicitud semántica. Las funciones nativas pueden recuperar los datos de usuario a los que el modelo de lenguaje de gran tamaño (LLM) no pudo acceder por sí mismo. Además, el LLM puede generar una recomendación de canción basada en la entrada de texto.

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

    var result = await kernel.InvokeAsync(songSuggesterFunction);
    Console.WriteLine(result);
    ```

    En este código, creará una función a partir de una solicitud que sugiere una canción al usuario. A continuación, la agrega a los complementos de kernel. Por último, indique al kernel que ejecute la función.

### Tarea 3: Proporcionar recomendaciones de concierto personalizadas

En esta tarea, creará un complemento que recupera los próximos detalles del concierto. También creará un complemento que pide al LLM que sugiera un concierto basado en las canciones y la ubicación que ha reproducido recientemente el usuario.

1. En la carpeta "Complementos", crea un nuevo archivo denominado "MusicConcertsPlugin.cs".

1. En el archivo "MusicConcertsPlugin", agrega el siguiente código:

    ```c#
    using System.ComponentModel;
    using Microsoft.SemanticKernel;

    public class MusicConcertsPlugin
    {
        [KernelFunction, Description("Get a list of upcoming concerts")]
        public static string GetConcerts()
        {
            string content = File.ReadAllText($"Files/ConcertDates.txt");
            return content;
        }
    }
    ```

    En este código, se crea una función que lee la lista de próximos conciertos de un archivo denominado "ConcertDates.txt". El archivo contiene una lista con formato JSON de los próximos conciertos. Por lo tanto, se debe crear un mensaje para pedir al LLM que sugiera un concierto.

1. En la carpeta "Solicitudes", cree una carpeta denominada "SuggestConcert"

1. Cree un archivo "config.json" en la carpeta "SuggestConcert" con el siguiente contenido:

    ```json
    {
        "schema": 1,
        "type": "completion",
        "description": "Suggest a concert to the user",
        "execution_settings": {
            "default": {
                "max_tokens": 4000,
                "temperature": 0
            }
        },
        "input_variables": [
            {
                "name": "location",
                "description": "The user's location",
                "required": true
            }
        ]
    }
    ```

1. Cree un archivo "skprompt.txt" en la carpeta "SuggestConcert" con el siguiente contenido:

    ```output
    This is a list of the user's recently played songs:
    {{MusicLibraryPlugin.GetRecentPlays}}

    This is a list of upcoming concert details:
    {{MusicConcertsPlugin.GetConcerts}}

    Suggest an upcoming concert based on the user's recently played songs. 
    The user lives in {{$location}}, 
    please recommend a relevant concert that is close to their location.
    ```

    Este mensaje ayuda al LLM a filtrar la entrada del usuario y a recuperar solo el destino del texto. A continuación, invocará al planificador para crear un plan que combine los complementos para lograr el objetivo.

1. Abra el archivo "Program.cs" y actualícelo con el código siguiente:

    ```c#
    var kernel = builder.Build();    
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
    // code omitted for brevity
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);

    string location = "Redmond WA USA";
    var result = await kernel.InvokeAsync<string>(prompts["SuggestConcert"],
        new() {
            { "location", location }
        }
    );
    Console.WriteLine(result);
    ```

1. En el terminal, escriba `dotnet run`.

    Debería ver una salida similar a la siguiente respuesta:

    ```output
    Based on the user's recently played songs and their location in Redmond WA USA, a relevant concert recommendation would be the upcoming concert of Lisa Taylor in Seattle WA, USA on February 22, 2024. Lisa Taylor is an indie-folk artist, and her music genre aligns with the user's recently played songs, such as "Loanh Quanh" by Ly Hoa. Additionally, Seattle is close to Redmond, making it a convenient location for the user to attend the concert.
    ```

    Pruebe a retocar la solicitud y la ubicación para ver qué otros resultados se pueden generar.

## Ejercicio 3: Automatización de sugerencias con el planificador Handlebars

El planificador de Handlebars resulta útil cuando una tarea requiere de varios pasos. Para lograr el objetivo, el planificador selecciona y combina los complementos registrados en el kernel en una serie de pasos. En este ejercicio, generará una plantilla de plan mediante el planificador Handlebars y la usará para automatizar las sugerencias.

**Tiempo de finalización del ejercicio estimado**: 10 minutos

### Tarea 1: Generación de una plantilla de plan

A continuación, reemplace la indicación SuggestConcert y use el planificador Handlebars para realizar la tarea en su lugar. La plantilla de plan se usará para automatizar sugerencias en función de la entrada del usuario.

1. Instale el planificador de Handlebars escribiendo lo siguiente en el terminal:

    `dotnet add package Microsoft.SemanticKernel.Planners.Handlebars --version 1.2.0-preview`

    A continuación, reemplace la indicación SuggestConcert y use el planificador Handlebars para realizar la tarea en su lugar.

1. En el archivo "Program.cs", actualice el código a lo siguiente:

    ```c#
    using Microsoft.SemanticKernel;
    using Microsoft.SemanticKernel.Planning.Handlebars;
    
    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(
        "your-deployment-name",
        "your-endpoint",
        "your-api-key",
        "deployment-model");
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    kernel.ImportPluginFromPromptDirectory("Prompts");

    #pragma warning disable SKEXP0060
    var planner = new HandlebarsPlanner(new HandlebarsPlannerOptions() { AllowLoops = true });

    string location = "Redmond WA USA";
    string goal = @$"Based on the user's recently played music, suggest a 
        concert for the user living in ${location}";

    var plan = await planner.CreatePlanAsync(kernel, goal);
    var result = await plan.InvokeAsync(kernel);

    Console.WriteLine($"{result}");
    ```

    >[!NOTE] Dado que el paquete Handlebars está actualmente en versión preliminar, es posible que tengas que suprimir la advertencia del compilador para ejecutar el código.

1. En el terminal, escriba `dotnet run`.

    Debería ver una respuesta similar a la siguiente salida:

    ```output
    Based on the user's recently played songs, the artist "Mademoiselle" has an upcoming concert in Seattle WA, USA on February 22, 2024, which is close to Redmond WA. Therefore, the recommended concert for the user would be Mademoiselle's concert in Seattle.
    ```

    Luego, cambie el código para generar la plantilla de plan Handlebars.

1. En el archivo "Program.cs", actualice el código a lo siguiente:

    ```c#
    var plan = await planner.CreatePlanAsync(kernel, goal);
    Console.WriteLine("Plan:");
    Console.WriteLine(plan);
    ```

    Ahora puede ver el plan generado. A continuación, modifique el plan para incluir sugerencias de canción o agregar una canción a la lista de reproducción reciente del usuario.

1. Amplíe el código con el siguiente fragmento de código:

    ```c#
    var plan = await planner.CreatePlanAsync(kernel, 
        @$"If add song:
        Add a song to the user's recently played list.
        
        If concert recommendation:
        Based on the user's recently played music, suggest a concert for 
        the user living in a given location.

        If song recommendation:
        Suggest a song from the music library to the user based on their 
        recently played songs.");

    Console.WriteLine("Plan:");
    Console.WriteLine(plan);
    ```

1. Escriba `dotnet run` en el terminal para ver la salida de los planes que ha creado.

    Debería ver una plantilla similar a la siguiente salida:

    ```output
    Plan:
    {{!-- Step 1: Identify Key Values --}}
    {{set "location" location}}
    {{set "addSong" addSong}}
    {{set "concertRecommendation" concertRecommendation}}
    {{set "songRecommendation" concertRecommendation}}

    {{!-- Step 2: Use the Right Helpers --}}
    {{#if addSong}}
        {{set "song" song}}
        {{set "artist" artist}}
        {{set "genre" genre}}
        {{set "songAdded" (MusicLibraryPlugin-AddToRecentlyPlayed artist=artist song=song genre=genre)}}  
        {{json songAdded}}
    {{/if}}

    {{#if concertRecommendation}}
        {{set "concertSuggested" (Prompts-SuggestConcert location=location recentlyPlayedSongs=recentlyPlayedSongs musicLibrary=musicLibrary)}}
        {{json concertSuggested}}
    {{/if}}

    {{#if songRecommendation}}
        {{set "songSuggested" (SuggestSongPlugin-SuggestSong recentlyPlayedSongs=recentlyPlayedSongs musicLibrary=musicLibrary)}}
        {{json songSuggested}}
    {{/if}}

    {{!-- Step 3: Output the Result --}}
    {{json "Goal achieved"}}
    ```

     Observe que la sintaxis es `{{#if ...}}`. Esta sintaxis actúa como una instrucción condicional que el planificador de Handlebars puede usar, similar a un bloque tradicional `if`-`else` en C#. La instrucción `if` debe cerrarse con `{{/if}}`.

    A continuación, usará estas plantillas generadas para crear su propio plan de Handlebars. 

1. En el directorio "Files", crea un nuevo archivo denominado "HandlebarsTemplate.txt" con el siguiente texto:

    ```output
    {{set "addSong" addSong}}
    {{set "concertRecommendation" concertRecommendation}}
    {{set "songRecommendation" songRecommendation}}

    {{#if addSong}}
        {{set "song" song}}
        {{set "artist" artist}}
        {{set "genre" genre}}
        {{set addedSong (MusicLibraryPlugin-AddToRecentlyPlayed artist song genre)}}  
        Output The following content, do not make any modifications:
        {{json addedSong}}     
    {{/if}}

    {{#if concertRecommendation}}
        {{set "location" location}}
        {{set "concert" (Prompts-SuggestConcert location)}}
        Output The following content, do not make any modifications:
        {{json concert}}
    {{/if}}

    {{#if songRecommendation}}
        {{set "recentlyPlayedSongs" (MusicLibraryPlugin-GetRecentPlays)}}
        {{set "musicLibrary" (MusicLibraryPlugin-GetMusicLibrary)}}
        {{set "song" (SuggestSongPlugin-SuggestSong recentlyPlayedSongs musicLibrary)}}
        Output The following content, do not make any modifications:
        {{json song}}
    {{/if}}
    ```

    En esta plantilla, agregará una instrucción al LLM para no realizar ninguna generación de texto. Así, se asegura de que los complementos administran estrictamente la salida. Ahora vamos a probar la plantilla.

### Tarea 2: Uso del plan Handlebars para automatizar sugerencias

En esta tarea, creará una función a partir de la plantilla de plan Handlebars y la usará para automatizar sugerencias basadas en la entrada del usuario.

1. Quite los planes de Handlebars modificando el código existente:

    ```c#
    using Microsoft.SemanticKernel;
    using Microsoft.SemanticKernel.PromptTemplates.Handlebars;

    var builder = Kernel.CreateBuilder();
    builder.AddAzureOpenAIChatCompletion(
        "your-deployment-name",
        "your-endpoint",
        "your-api-key",
        "deployment-model");
    var kernel = builder.Build();
    kernel.ImportPluginFromType<MusicLibraryPlugin>();
    kernel.ImportPluginFromType<MusicConcertsPlugin>();
    kernel.ImportPluginFromPromptDirectory("Prompts");
    
    var songSuggesterFunction = kernel.CreateFunctionFromPrompt(
        promptTemplate: @"Based on the user's recently played music:
        {{$recentlyPlayedSongs}}
        recommend a song to the user from the music library:
        {{$musicLibrary}}",
        functionName: "SuggestSong",
        description: "Suggest a song to the user"
    );

    kernel.Plugins.AddFromFunctions("SuggestSongPlugin", [songSuggesterFunction]);
    ```

1. Agregue código que lea el archivo de plantilla y cree una función:

    ```c#
    string template = File.ReadAllText($"Files/HandlebarsTemplate.txt");

    var handlebarsPromptFunction = kernel.CreateFunctionFromPrompt(
        new() {
            Template = template,
            TemplateFormat = "handlebars"
        }, new HandlebarsPromptTemplateFactory()
    );
    ```

    En este código, se pasa un objeto `Template` al método de kernel `CreateFunctionFromPrompt`junto con `TemplateFormat`. `CreateFunctionFromPrompt` también acepta un tipo `IPromptTemplateFactory` que indica al kernel cómo analizar una plantilla determinada. Puesto que usa una plantilla de Handlebars, se usa el tipo `HandlebarsPromptTemplateFactory`.

    A continuación, vamos a ejecutar la función con algunos argumentos y echemos un vistazo a los resultados.

1. Agregue el siguiente código al archivo `Program.cs`:

    ```c#
    string location = "Redmond WA USA";
    var templateResult = await kernel.InvokeAsync(handlebarsPromptFunction,
        new() {
            { "location", location },
            { "concertRecommendation", true },
            { "songRecommendation", false },
            { "addSong", false },
            { "artist", "" },
            { "song", "" },
            { "genre", "" }
        });

    Console.WriteLine(templateResult);
    ```

1. Escriba `dotnet run` en el terminal para ver la salida de la plantilla del planificador.

    Debería ver una respuesta similar a la siguiente salida:

    ```output
    Based on the user's recently played songs, Ly Hoa seems to be a relevant artist. The closest concert to Redmond WA, USA would be in Portland OR, USA on April 16th, 2024.  
    ```

    La indicación pudo sugerir una canción al usuario en función de la lista de música reproducida recientemente y su ubicación. También puede intentar establecer otras variables en true y ver lo que sucede.

Ahora el código puede realizar diferentes acciones en función de la entrada del usuario. ¡Excelente trabajo!

### Revisar

En este laboratorio, ha creado un agente de inteligencia artificial que puede administrar la biblioteca de música del usuario y proporcionar recomendaciones personalizadas de canciones y conciertos. Use el SDK de kernel semántico para compilar el agente de IA y conectarlo al servicio de modelo de lenguaje grande (LLM). Ha creado complementos personalizados para la biblioteca de música, ha usado el planificador Handlebars para automatizar sugerencias y ha creado una función a partir de la plantilla de plan Handlebars para automatizar las sugerencias en función de la entrada del usuario. Enhorabuena por completar este laboratorio.
