---
lab:
  title: 'Laboratorio: Desarrollo de agentes de inteligencia artificial mediante Azure OpenAI y el SDK de Kernel semántico'
  module: 'Module 01: Build your kernel'
---

# Laboratorio: Completar un asistente de viajes con IA
# Manual de laboratorio para alumnos

En este laboratorio, completarás un asistente de viajes con IA mediante el SDK de kernel semántico. Creará un punto de conexión para el servicio de modelo de lenguaje grande (LLM), creará funciones de kernel semántico y usará la funcionalidad de llamada automática de funciones del SDK de kernel semántico para enrutar la intención del usuario a los complementos adecuados, incluidos algunos complementos creados previamente que se han proporcionado. También proporcionará contexto al LLM mediante el historial de conversaciones y permitirá al usuario continuar con la conversación.

## Escenario de laboratorio

Usted es desarrollador de una agencia de viajes que se especializa en la creación de experiencias de viaje personalizadas para sus clientes. Se te ha encargado crear un agente de viajes con IA que ayude a los clientes a obtener más información sobre los destinos y las actividades de sus viajes. El asistente de viajes con IA debe ser capaz de convertir importes en divisas, sugerir destinos y actividades, proporcionar frases útiles en diferentes idiomas y traducir frases. El asistente de viajes con IA también debe ser capaz de proporcionar respuestas contextualmente relevantes a las solicitudes del usuario mediante el historial de conversaciones.

## Objetivos

Al completar este laboratorio, realizará lo siguiente:

* Crear un punto de conexión para el servicio de modelo de lenguaje grande (LLM)
* Compilar un objeto Kernel semántico
* Ejecutar indicaciones mediante el SDK de kernel semántico
* Crear funciones y complementos semánticos del kernel
* Uso de la funcionalidad de llamada automática de funciones del SDK de kernel semántico

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
   
     `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/02/Lab-02-Starter.zip`

1. Descarga el archivo ZIP haciendo clic en el botón <kbd>...</kbd> situado en la parte superior derecha de la página, o pulsa <kbd>Ctrl</kbd>+<kbd>Mayús</kbd>+<kbd>S</kbd>.

1. Extraiga el contenido del archivo ZIP en una ubicación fácil de encontrar y recordar, como una carpeta en el escritorio.

1. Abra Visual Studio Code y seleccione **Archivo** > **Abrir carpeta**.

1. Vaya a la carpeta **Starter** que extrajo antes y elija **Seleccionar carpeta**.

1. Abra el archivo **Program.cs** en el editor de código.

> [!NOTE]
> Si se te pide que confíes en la carpeta, selecciona **Yes, I trust the authors**

## Ejercicio 1: Creación de un complemento con el SDK de kernel semántico

En este ejercicio, creará un punto de conexión para el servicio de modelo de lenguaje grande (LLM). El SDK de kernel semántico usa este punto de conexión para conectarse a LLM y ejecutar solicitudes. El SDK de kernel semántico admite los LLM HuggingFace, OpenAI y Azure Open AI. En este ejemplo, se usa Azure Open AI.

**Tiempo de finalización del ejercicio estimado**: 10 minutos

### Tarea 1: Crear un recurso de Azure OpenAI

1. Vaya a [https://portal.azure.com](https://portal.azure.com).

1. Cree un nuevo recurso de Azure OpenAI con la configuración predeterminada.

    > [!NOTE]
    > Si ya tiene un recurso de Azure OpenAI, puede omitir este paso.

1. Tras crear el nuevo recurso, seleccione **Ir al recurso**.

1. En la página **Información general**, selecciona **Ir al portal de Azure Foundry**.

1. Selecciona **Crear nueva implementación** y después **desde modelos base**.

1. En la lista de modelos, selecciona **gpt-35-turbo-16k**.

1. Seleccione **Confirmar**

1. Escribe un nombre para la implementación y deja las opciones predeterminadas.

1. Cuando se complete la implementación, vuelve a tu recurso de Azure OpenAI en Azure Portal.

1. En **Administración de recursos**, vaya a **Claves y puntos de conexión**.

    Usará los datos en la siguiente tarea para compilar el kernel. Recuerde mantener sus claves privadas y seguras.

### Tarea 2: Crear un complemento nativo

En esta tarea, crearás un complemento de función nativa que puede convertir una cantidad de una divisa base a una divisa de destino.

1. Vuelva a su proyecto de Visual Studio Code.

1. Abre el archivo **appsettings.json** y actualiza los valores con tu Id. de modelo, punto de conexión y clave de API de los servicios de Azure OpenAI.

    ```json
    {
        "modelId": "gpt-35-turbo-16k",
        "endpoint": "",
        "apiKey": ""
    }
    ```

1. Ve al archivo denominado **CurrencyConverterPlugin.cs** en la carpeta **Plugins**

1. En el archivo **CurrencyConverterPlugin.cs**, agrega el código siguiente en el comentario **Create a kernel function that gets the exchange rate**:

    ```c#
    // Create a kernel function that gets the exchange rate
    [KernelFunction("convert_currency")]
    [Description("Converts an amount from one currency to another, for example USD to EUR")]
    public static decimal ConvertCurrency(decimal amount, string fromCurrency, string toCurrency)
    {
        decimal exchangeRate = GetExchangeRate(fromCurrency, toCurrency);
        return amount * exchangeRate;
    }
    ```

    En este código, usas el decorador **KernelFunction** para declarar tu función nativa. También usas el decorador **Descripción** para agregar una descripción de lo que hace la función. A continuación, agrega alguna lógica para convertir una cantidad determinada de una divisa a otra.

1. Abra el archivo **Program.cs**.

1. Importa el complemento de conversión de divisas en el comentario **Add plugins to the kernel**:

    ```c#
    // Add plugins to the kernel
    kernel.ImportPluginFromType<CurrencyConverterPlugin>();
    ```

    A continuación, vamos a probar el complemento.

1. Haz clic con el botón derecho en el archivo **Program.cs** y haz clic en "Abrir en terminal integrado"

1. En el terminal, escriba `dotnet run`. 

    Escribe una solicitud de aviso para convertir la divisa, por ejemplo, "¿Cuánto es 10 USD en Hong Kong?"

    Deberías ver un resultado parecido a los siguiente:

    ```output
    Assistant: 10 USD is equivalent to 77.70 Hong Kong dollars (HKD).
    ```

## Ejercicio 2: Creación de un aviso de Handlebars

En este ejercicio, crearás una función a partir de un aviso de Handlebars. La función pedirá al LLM que cree un itinerario de viaje para el usuario. Comencemos.

**Tiempo de finalización del ejercicio estimado**: 10 minutos

### Tarea 1: Crear una función a partir de un aviso de Handlebars

1. Agrega la siguiente directiva `using` al archivo **Program.cs**:

    `using Microsoft.SemanticKernel.PromptTemplates.Handlebars;`

1. Agrega el código siguiente en el comentario **Create a handlebars prompt**:

    ```c#
    // Create a handlebars prompt
    string hbprompt = """
        <message role="system">Instructions: Before providing the user with a travel itinerary, ask how many days their trip is</message>
        <message role="user">I'm going to {{city}}. Can you create an itinerary for me?</message>
        <message role="assistant">Sure, how many days is your trip?</message>
        <message role="user">{{input}}</message>
        <message role="assistant">
        """;
    ```

    En este código, crearás una solicitud de pocas etapas con el formato de plantilla de Handlebars. La solicitud guiará al modelo para recuperar más información del usuario antes de crear un itinerario de viaje.

1. Agrega el código siguiente en el comentario **Create the prompt template config using handlebars format**:

    ```c#
    // Create the prompt template config using handlebars format
    var templateFactory = new HandlebarsPromptTemplateFactory();
    var promptTemplateConfig = new PromptTemplateConfig()
    {
        Template = hbprompt,
        TemplateFormat = "handlebars",
        Name = "GetItinerary",
    };
    ```

    En este código, crearás una configuración de plantilla de Handlebars a partir de la solicitud. Puedes usarlo para crear una función de complemento.

1. Agrega el código siguiente en el comentario **Create a plugin function from the prompt**: 

    ```c#
    // Create a plugin function from the prompt
    var promptFunction = kernel.CreateFunctionFromPrompt(promptTemplateConfig, templateFactory);
    var itineraryPlugin = kernel.CreatePluginFromFunctions("TravelItinerary", [promptFunction]);
    kernel.Plugins.Add(itineraryPlugin);
    ```

    Este código crea una función de complemento para la solicitud y la agrega al kernel. Ahora estás preparado para invocar tu función.

1. Escribe `dotnet run` en el terminal para ejecutar el código.

    Prueba la siguiente entrada para solicitar al LLM un itinerario.

    ```output
    Assistant: How may I help you?
    User: I'm going to Hong Kong, can you create an itinerary for me?
    Assistant: Sure! How many days will you be staying in Hong Kong?
    User: 10
    Assistant: Great! Here's a 10-day itinerary for your trip to Hong Kong:
    ...
    ```

    ¡Ahora ya has creado las bases de un asistente de viajes de inteligencia artificial! Vamos a usar indicaciones y complementos para agregar más características

1.  Agrega el complemento de reserva de vuelos en el comentario **Add plugins to the kernel**:

    ```c#
    // Add plugins to the kernel
    kernel.ImportPluginFromType<CurrencyConverterPlugin>();
    kernel.ImportPluginFromType<FlightBookingPlugin>();
    ```

    Este complemento simula las reservas de vuelos con el archivo **flights.json** con detalles ficticios. A continuación, agrega algunas indicaciones adicionales del sistema al asistente.

1.  Agrega el código siguiente en el comentario **Add system messages to the chat**:

    ```c#
    // Add system messages to the chat
    history.AddSystemMessage("The current date is 01/10/2025");
    history.AddSystemMessage("You are a helpful travel assistant.");
    history.AddSystemMessage("Before providing destination recommendations, ask the user about their budget.");
    ```

    Estas indicaciones ayudarán a crear una experiencia de usuario fluida y a simular el complemento de reserva de vuelos. Ya estás listo para probar el código.

1. Escriba `dotnet run` en el terminal.

    Prueba a escribir algunas de las indicaciones siguientes:

    ```output
    1. Can you give me some destination recommendations for Europe?
    2. I want to go to Barcelona, can you create an itinerary for me?
    3. How many Euros is 100 USD?
    4. Can you book me a flight to Barcelona?
    ```

    Prueba otras entradas y ve cómo responde el asistente para viajes.

## Ejercicio 3: Requerimiento del consentimiento del usuario para acciones

En este ejercicio, agregarás una función de invocación de filtro que solicitará la aprobación del usuario antes de permitir que el asistente reserve un vuelo en su nombre. Comencemos.

### Tarea 1: Crear un filtro de invocación de función

1. Crea un nuevo archivo denominado **PermissionFilter.cs**.

1. En el nuevo archivo, escribe el código siguiente:

    ```c#
    #pragma warning disable SKEXP0001 
    using Microsoft.SemanticKernel;
    
    public class PermissionFilter : IFunctionInvocationFilter
    {
        public async Task OnFunctionInvocationAsync(FunctionInvocationContext context, Func<FunctionInvocationContext, Task> next)
        {
            // Check the plugin and function names
            
            await next(context);
        }
    }
    ```

    >[!NOTE] 
    > En la versión 1.30.0 del SDK de kernel semántico, los filtros de función están sujetos a cambios y requieren una supresión de advertencia. 

    En este código, implementarás la interfaz `IFunctionInvocationFilter`. Siempre se llama al método `OnFunctionInvocationAsync` cuando se invoca una función desde un asistente de IA.

1. Agrega el código siguiente para detectar cuándo se invoca la función `book_flight`:

    ```c#
    // Check the plugin and function names
    if ((context.Function.PluginName == "FlightBookingPlugin" && context.Function.Name == "book_flight"))
    {
        // Request user approval

        // Proceed if approved
    }
    ```

    Este código usa `FunctionInvocationContext` para determinar qué complemento y función se invocaron.

1. Agrega la siguiente lógica para solicitar el permiso del usuario para reservar el vuelo:

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

1. Ve al archivo **Program.cs**.

1. Agrega el código siguiente en el comentario **Add filters to the kernel**:

    ```c#
    // Add filters to the kernel
    kernel.FunctionInvocationFilters.Add(new PermissionFilter());
    ```

1. Escriba `dotnet run` en el terminal.

    Escribe una indicación para reservar un vuelo. Debería ver una respuesta similar a la siguiente:

    ```output
    User: Find me a flight to Tokyo on the 19
    Assistant: I found a flight to Tokyo on the 19th of January. The flight is with Air Japan and the price is $1200.
    User: Y
    System Message: The assistant requires an approval to complete this operation. Do you approve (Y/N)
    User: N
    Assistant: I'm sorry, but I am unable to book the flight for you.
    ```

    El asistente debe requerir la aprobación del usuario antes de continuar con las reservas.

### Revisar

En este laboratorio, creaste un punto de conexión para el servicio de modelo de lenguaje grande (LLM), creaste un objeto de kernel semántico y ejecutaste indicaciones mediante el SDK de kernel semántico. También has creado complementos y has aprovechado los mensajes del sistema para guiar al modelo. Enhorabuena por completar este laboratorio.
