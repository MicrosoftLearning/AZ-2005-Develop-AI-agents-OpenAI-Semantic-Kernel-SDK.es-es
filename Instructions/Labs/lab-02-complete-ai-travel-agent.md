---
lab:
  title: 'Laboratorio: Desarrollo de agentes de inteligencia artificial mediante Azure OpenAI y el SDK de Kernel semántico'
  module: 'Module 01: Build your kernel'
---

# Laboratorio: Completar un agente de viajes de IA
# Manual de laboratorio para alumnos

En este laboratorio, completará un agente de viajes de IA mediante el SDK de kernel semántico. Creará un punto de conexión para el servicio de modelo de lenguaje grande (LLM), creará funciones de kernel semántico y usará la funcionalidad de llamada automática de funciones del SDK de kernel semántico para enrutar la intención del usuario a los complementos adecuados, incluidos algunos complementos creados previamente que se han proporcionado. También proporcionará contexto al LLM mediante el historial de conversaciones y permitirá al usuario continuar con la conversación.

## Escenario de laboratorio

Usted es desarrollador de una agencia de viajes que se especializa en la creación de experiencias de viaje personalizadas para sus clientes. Se le ha encargado crear un agente de viajes de IA que ayude a los clientes a obtener más información sobre los destinos y las actividades del viaje. El agente de viajes de IA debe ser capaz de convertir importes de divisas, sugerir destinos y actividades, proporcionar frases útiles en diferentes idiomas y traducir. El agente de viajes de IA también debe ser capaz de proporcionar respuestas contextualmente relevantes a las solicitudes del usuario mediante el historial de conversaciones.

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

* [Visual Studio Code.](https://code.visualstudio.com)
* [El SDK de .NET 7.0 más reciente.](https://dotnet.microsoft.com/download/dotnet/7.0)
* La [extensión de C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp) para Visual Studio Code.

### Preparación del entorno de desarrollo

Para estos ejercicios, hay disponible un proyecto de inicio para su uso. Siga estos pasos para configurar el proyecto de inicio:

> [!IMPORTANT]
> Debe tener instalado .NET Framework 8.0, así como las extensiones de VS Code para C# y el Administrador de paquetes NuGet.

1. Descargue el archivo ZIP ubicado en `https://github.com/MicrosoftLearning/AZ-2005-Develop-AI-agents-OpenAI-Semantic-Kernel-SDK/blob/master/Allfiles/Labs/02/Lab-02-Starter.zip`.

1. Extraiga el contenido del archivo ZIP en una ubicación fácil de encontrar y recordar, como una carpeta en el escritorio.

1. Abra Visual Studio Code y seleccione **Archivo** > **Abrir carpeta**.

1. Vaya a la carpeta **Starter** que extrajo antes y elija **Seleccionar carpeta**.

1. Abra el archivo **Program.cs** en el editor de código.

## Ejercicio 1: Creación de un complemento con el SDK de kernel semántico

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

1. En **Seleccionar un modelo**, seleccione **gpt-35-turbo-16k**.

    Uso de la versión predeterminada del modelo

1. Escriba un nombre para la implementación

1. Cuando se complete la implementación, vuelva al recurso de Azure OpenAI.

1. En **Administración de recursos**, vaya a **Claves y puntos de conexión**.

    Usará estos valores en la siguiente tarea para compilar el kernel. Recuerde mantener sus claves privadas y seguras.

1. Abra el archivo **Program.cs** en Visual Studio Code.

1. Actualice las siguientes variables con el nombre de implementación de Azure Open AI Services, la clave de API y el punto de conexión

    ```csharp
    string yourDeploymentName = "";
    string yourEndpoint = "";
    string yourKey = "";
    ```

    > [!NOTE]
    > El modelo de implementación debe ser "gpt-35-turbo-16k" para que funcionen algunas de las características del SDK de Semantic Kernel.

### Tarea 2: Creación de una función nativa

En esta tarea, creará una función nativa que puede convertir una cantidad de una divisa base a una divisa de destino.

1. Cree un nuevo archivo denominado `CurrencyConverter.cs` en la carpeta **Plugins/ConvertCurrency**

1. En el archivo `CurrencyConverter.cs`, agregue el código siguiente para crear una función de complemento:

    ```c#
    using AITravelAgent;

    class CurrencyConverter
    {
        [KernelFunction, 
        Description("Convert an amount from one currency to another")]
        public static string ConvertAmount()
        {
            var currencyDictionary = Currency.Currencies;
        }
    }
    ```

    En este código, se usa el decorador para declarar la función nativa `KernelFunction`. También se usa el decorador `Description` para agregar una descripción de lo que hace la función. Puede usar `Currency.Currencies` para obtener un diccionario de divisas y sus tipos de cambio. A continuación, agregue alguna lógica para convertir una cantidad determinada de una divisa a otra.

1. Modifique la función `ConvertAmount` con el código siguiente:

    ```c#
    [KernelFunction, Description("Convert an amount from one currency to another")]
    public static string ConvertAmount(
        [Description("The target currency code")] string targetCurrencyCode, 
        [Description("The amount to convert")] string amount, 
        [Description("The starting currency code")] string baseCurrencyCode)
    {
        var currencyDictionary = Currency.Currencies;
        
        Currency targetCurrency = currencyDictionary[targetCurrencyCode];
        Currency baseCurrency = currencyDictionary[baseCurrencyCode];

        if (targetCurrency == null)
        {
            return targetCurrencyCode + " was not found";
        }
        else if (baseCurrency == null)
        {
            return baseCurrencyCode + " was not found";
        }
        else
        {
            double amountInUSD = Double.Parse(amount) * baseCurrency.USDPerUnit;
            double result = amountInUSD * targetCurrency.UnitsPerUSD;
            return @$"${amount} {baseCurrencyCode} is approximately 
                {result.ToString("C")} in {targetCurrency.Name}s ({targetCurrencyCode})";
        }
    }
    ```

    En este código, se usa el diccionario `Currency.Currencies` para obtener el objeto `Currency` de las divisas base y de destino. A continuación, use el objeto `Currency` para convertir la cantidad de la divisa base a la divisa de destino. Por último, se devuelve una cadena con la cantidad convertida. A continuación, vamos a probar el complemento.

1. En el archivo `Program.cs`, importe e invoque la nueva función del complemento con el código siguiente:

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    kernel.ImportPluginFromType<ConversationSummaryPlugin>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    var result = await kernel.InvokeAsync("CurrencyConverter", 
        "ConvertAmount", 
        new() {
            {"targetCurrencyCode", "USD"}, 
            {"amount", "52000"}, 
            {"baseCurrencyCode", "VND"}
        }
    );

    Console.WriteLine(result);
    ```

    En este código, usará el método para importar el complemento `ImportPluginFromType`. Después, use el método `InvokeAsync` para invocar la función del complemento. El método `InvokeAsync` toma el nombre del complemento, el nombre de la función y un diccionario de parámetros. Por último, imprima el resultado en la consola. A continuación, ejecute el código para asegurarse de que funciona.

1. En el terminal, escriba `dotnet run`. Debería ver la siguiente salida:

    ```output
    $52000 VND is approximately $2.13 in US Dollars (USD)
    ```

    Ahora que el complemento funciona correctamente, vamos a crear un mensaje de lenguaje natural que pueda detectar las divisas y la cantidad que el usuario quiere convertir.

### Tarea 3: Análisis de la entrada del usuario con una indicación

En esta tarea, creará un mensaje que analiza la entrada del usuario para identificar la divisa de destino, la divisa base y la cantidad que se va a convertir.

1. Cree una nueva carpeta denominada `GetTargetCurrencies` en la carpeta **Prompts**

1. En la carpeta `GetTargetCurrencies`, cree un nuevo archivo denominado `config.json`

1. Escriba el texto siguiente en el archivo `config.json`:

    ```output
    {
        "schema": 1,
        "type": "completion",
        "description": "Identify the target currency, base currency, and amount to convert",
        "execution_settings": {
            "default": {
                "max_tokens": 800,
                "temperature": 0
            }
        },
        "input_variables": [
            {
                "name": "input",
                "description": "Text describing some currency amount to convert",
                "required": true
            }
        ]
    }
    ```

1. En la carpeta `GetTargetCurrencies`, cree un nuevo archivo denominado `skprompt.txt`

1. Pegue el texto siguiente en el archivo `skprompt.txt`:

    ```html
    <message role="system">Identify the target currency, base currency, and 
    amount from the user's input in the format target|base|amount</message>

    For example: 

    <message role="user">How much in GBP is 750.000 VND?</message>
    <message role="assistant">GBP|VND|750000</message>

    <message role="user">How much is 60 USD in New Zealand Dollars?</message>
    <message role="assistant">NZD|USD|60</message>

    <message role="user">How many Korean Won is 33,000 yen?</message>
    <message role="assistant">KRW|JPY|33000</message>

    <message role="user">{{$input}}</message>
    <message role="assistant">target|base|amount</message>
    ```

### Tarea 4: Compruebe su trabajo

En esta tarea, ejecute su aplicación y verifique que su código funciona correctamente. 

1. Pruebe la nueva solicitud actualizando el archivo `Program.cs` con el código siguiente:

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    var result = await kernel.InvokeAsync(prompts["GetTargetCurrencies"],
        new() {
            {"input", "How many Australian Dollars is 140,000 Korean Won worth?"}
        }
    );

    Console.WriteLine(result);
    ```

1. Escriba `dotnet run` en el terminal. Debería ver la salida siguiente:

    ```output
    AUD|KRW|140000
    ```

    > [!NOTE]
    > Si el código no genera la salida esperada, puede revisar el código en la carpeta **Solution**. Quizá tenga que cambiar la indicación en el archivo `skprompt.txt` para generar resultados más exactos.

Ahora tiene un complemento que puede convertir una cantidad de una divisa a otra y una solicitud que se puede usar para analizar la entrada del usuario en un formato que la función `ConvertAmount` puede usar. Esto permitirá a los usuarios convertir fácilmente importes de divisa mediante el agente de viajes de IA.

## Ejercicio 2: Automatización de la selección de complementos en función de la intención del usuario

En este ejercicio, detectará la intención del usuario y enrutará la conversación hacia los complementos que desee. Puede usar un complemento proporcionado para recuperar la intención del usuario. Comencemos.

**Tiempo de finalización del ejercicio estimado**: 10 minutos

### Tarea 1: Uso del complemento GetIntent

1. Actualice el archivo `Program.cs` con el código siguiente:

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    Console.WriteLine("What would you like to do?");
    var input = Console.ReadLine();

    var intent = await kernel.InvokeAsync<string>(
        prompts["GetIntent"], 
        new() {{ "input",  input }}
    );

    ```

    En este código, usará la solicitud de `GetIntent` para detectar la intención del usuario. Después almacenará la intención en una variable llamada `intent`. A continuación, enruta la intención a su complemento `CurrencyConverter`.

1. Agregue el siguiente código al archivo `Program.cs`:

    ```c#
    switch (intent) {
        case "ConvertCurrency": 
            var currencyText = await kernel.InvokeAsync<string>(
                prompts["GetTargetCurrencies"], 
                new() {{ "input",  input }}
            );
            var currencyInfo = currencyText!.Split("|");
            var result = await kernel.InvokeAsync("CurrencyConverter", 
                "ConvertAmount", 
                new() {
                    {"targetCurrencyCode", currencyInfo[0]}, 
                    {"baseCurrencyCode", currencyInfo[1]},
                    {"amount", currencyInfo[2]}, 
                }
            );
            Console.WriteLine(result);
            break;
        default:
            Console.WriteLine("Other intent detected");
            break;
    }
    ```

    El complemento `GetIntent` devuelve los siguientes valores: ConvertCurrency, SuggestDestinations, SuggestActivities, Translate, HelpfulPhrases, Unknown. Usa una instrucción `switch` para enrutar la intención del usuario al complemento apropiado. 
    
    Si la intención del usuario es convertir moneda, usa la solicitud de `GetTargetCurrencies` para recuperar la información de la moneda. Después usará el complemento `CurrencyConverter` para convertir el importe.

    A continuación, agregará algunos casos para controlar las demás intenciones. Por ahora, vamos a usar la capacidad de llamada de función automática del SDK de Semantic Kernel para enrutar la intención a los complementos disponibles.

1. Cree la configuración de llamada de función automática añadiendo el siguiente código a su archivo `Program.cs`:

    ```c#
    kernel.ImportPluginFromType<CurrencyConverter>();
    var prompts = kernel.ImportPluginFromPromptDirectory("Prompts");

    OpenAIPromptExecutionSettings settings = new()
    {
        ToolCallBehavior = ToolCallBehavior.AutoInvokeKernelFunctions
    };

    Console.WriteLine("What would you like to do?");
    var input = Console.ReadLine();
    var intent = await kernel.InvokeAsync<string>(
        prompts["GetIntent"], 
        new() {{ "input",  input }}
    );
    ```

    A continuación, agregue casos a las instrucciones del conmutador para las otras intenciones.

1. Actualice el archivo `Program.cs` con el código siguiente:

    ```c#
    switch (intent) {
        case "ConvertCurrency": 
            // ...Code you entered previously...
            break;
        case "SuggestDestinations":
        case "SuggestActivities":
        case "HelpfulPhrases":
        case "Translate":
            var autoInvokeResult = await kernel.InvokePromptAsync(input!, new(settings));
            Console.WriteLine(autoInvokeResult);
            break;
        default:
            Console.WriteLine("Other intent detected");
            break;
    }
    ```

    En este código, usa la configuración de `AutoInvokeKernelFunctions` para llamar automáticamente a las funciones y solicitudes que están referenciadas en su kernel. Si la intención del usuario es convertir divisas, el complemento `CurrencyConverter` realiza su tarea. 
    
    Si la intención del usuario es obtener sugerencias de destinos o actividades, traducir una frase u obtener frases útiles en un idioma, la configuración de `AutoInvokeKernelFunctions` llama automáticamente a los complementos existentes que se incluyeron en el código del proyecto.

    También puede agregar código para ejecutar la entrada del usuario como una solicitud al modelo de lenguaje de gran tamaño (LLM) si no entra en ninguno de estos casos de intención.

1. Actualice el caso predeterminado con el código siguiente:

    ```c#
    default:
        Console.WriteLine("Sure, I can help with that.");
        var otherIntentResult = await kernel.InvokePromptAsync(input!, new(settings));
        Console.WriteLine(otherIntentResult);
        break;
    ```

    Ahora, si el usuario tiene una intención diferente, el LLM puede controlar la solicitud del usuario. ¡Pruébelo!

### Tarea 2: Compruebe su trabajo

En esta tarea, ejecute su aplicación y verifique que su código funciona correctamente. 

1. Escriba `dotnet run` en el terminal. Cuando se le solicite, escriba un texto similar a la siguiente solicitud:

    ```output
    What would you like to do?
    How many TTD is 50 Qatari Riyals?    
    ```

1. Debería ver una salida similar a la siguiente respuesta:

    ```output
    $50 QAR is approximately $93.10 in Trinidadian Dollars (TTD)
    ```

1. Escriba `dotnet run` en el terminal. Cuando se le solicite, escriba un texto similar a la siguiente solicitud:

    ```output
    What would you like to do?
    I want to go somewhere that has lots of warm sunny beaches and delicious, spicy food!
    ```

1. Debería ver una salida similar a la siguiente respuesta:

    ```output
    Based on your preferences for warm sunny beaches and delicious, spicy food, I have a few destination recommendations for you:

    1. Thailand: Known for its stunning beaches, Thailand offers a perfect combination of relaxation and adventure. You can visit popular beach destinations like Phuket, Krabi, or Koh Samui, where you'll find crystal-clear waters and white sandy shores. Thai cuisine is famous for its spiciness, so you'll have plenty of mouthwatering options to try, such as Tom Yum soup, Pad Thai, and Green Curry.

    2. Mexico: Mexico is renowned for its beautiful coastal regions and vibrant culture. You can explore destinations like Cancun, Playa del Carmen, or Tulum, which boast stunning beaches along the Caribbean Sea. Mexican cuisine is rich in flavors and spices, offering a wide variety of dishes like tacos, enchiladas, and mole sauces that will satisfy your craving for spicy food.

    ...

    These destinations offer a perfect blend of warm sunny beaches and delicious, spicy food, ensuring a memorable trip for you. Let me know if you need any further assistance or if you have any specific preferences for your trip!
    ```

1. Escriba `dotnet run` en el terminal. Cuando se le solicite, escriba un texto similar a la siguiente solicitud:

    ```output
    What would you like to do?
    Can you give me a recipe for chicken satay?

1. You should see a response similar to the following response:

    ```output
    Sure, I can help with that.
    Certainly! Here's a recipe for chicken satay:

    ...
    ```

    La intención debería estar enfocada a su caso predeterminado y el LLM debería controlar la solicitud de una receta de pollo satay.

    > [!NOTE]
    > Si el código no genera la salida esperada, puede revisar el código en la carpeta **Solución**.

A continuación, modifiquemos la lógica de enrutamiento para proporcionar un historial de conversaciones a determinados complementos. Proporcionar el historial permite a los complementos recuperar respuestas más relevantes desde el punto de vista contextual a las solicitudes del usuario.

### Tarea 3: Enrutamiento completo del complemento

En este ejercicio, usará el historial de conversaciones para proporcionar contexto al modelo de lenguaje de gran tamaño (LLM). También ajustará el código para que permita al usuario continuar la conversación, al igual que un bot de chat real. Comencemos.

1. Modifique el código para que use un bucle `do`-`while` para aceptar la entrada del usuario:

    ```c#
    string input;

    do 
    {
        Console.WriteLine("What would you like to do?");
        input = Console.ReadLine();

        // ...
    }
    while (!string.IsNullOrWhiteSpace(input));
    ```

    Ahora puede mantener la conversación en marcha hasta que el usuario escriba una línea en blanco.

1. Capture detalles sobre el viaje del usuario modificando el caso `SuggestDestinations`:

    ```c#
    case "SuggestDestinations":
        chatHistory.AppendLine("User:" + input);
        var recommendations = await kernel.InvokePromptAsync(input!);
        Console.WriteLine(recommendations);
        break;
    ```

1. Use los detalles del viaje en el caso `SuggestActivities` con el código siguiente:

    ```c#
     case "SuggestActivities":
        var chatSummary = await kernel.InvokeAsync(
            "ConversationSummaryPlugin", 
            "SummarizeConversation", 
            new() {{ "input", chatHistory.ToString() }});
        break;
    ```

    En este código, usará la función integrada `SummarizeConversation` para resumir el chat con el usuario. A continuación, vamos a usar el resumen para sugerir actividades en el destino.

1. Extienda el caso `SuggestActivities` con el código siguiente:

    ```c#
    var activities = await kernel.InvokePromptAsync(
        input,
        new () {
            {"input", input},
            {"history", chatSummary},
            {"ToolCallBehavior", ToolCallBehavior.AutoInvokeKernelFunctions}
    });

    chatHistory.AppendLine("User:" + input);
    chatHistory.AppendLine("Assistant:" + activities.ToString());
    
    Console.WriteLine(activities);
    break;
    ```

    En este código, agregará `input` y `chatSummary` como argumentos de kernel. Después, el kernel invoca la solicitud y la enruta al complemento `SuggestActivities`. Además, anexará la entrada del usuario y la respuesta del asistente al historial de chat, y se mostrarán los resultados. Después, debe agregar la variable `chatSummary` al complemento `SuggestActivities`.

1. Vaya a **Prompts/SuggestActivities/config.json** y abra el archivo en Visual Studio Code

1. En `input_variables`, agregue una variable para el historial de chat:

    ```json
    "input_variables": [
      {
          "name": "history",
          "description": "Some background information about the user",
          "required": false
      },
      {
          "name": "destination",
          "description": "The destination a user wants to visit",
          "required": true
      }
  ]
  ```

1. Vaya a **Prompts/SuggestActivities/skprompt.txt** y abra el archivo

1. Agregue un mensaje para usar el historial de chats:

    ```html 
    You are an experienced travel agent. 
    You are helpful, creative, and very friendly. 
    Consider the traveler's background: {{$history}}
    ```

    Deje el resto del mensaje tal como está. Ahora el complemento usa el historial de chat para proporcionar contexto al LLM.

### Tarea 4: Compruebe su trabajo

En esta tarea, ejecutará la aplicación y comprobará que el código funciona correctamente.

1. Compare los casos de modificador actualizados con el código siguiente:

    ```c#
    case "SuggestDestinations":
            chatHistory.AppendLine("User:" + input);
            var recommendations = await kernel.InvokePromptAsync(input!);
            Console.WriteLine(recommendations);
            break;
    case "SuggestActivities":

        var chatSummary = await kernel.InvokeAsync(
            "ConversationSummaryPlugin", 
            "SummarizeConversation", 
            new() {{ "input", chatHistory.ToString() }});

        var activities = await kernel.InvokePromptAsync(
            input!,
            new () {
                {"input", input},
                {"history", chatSummary},
                {"ToolCallBehavior", ToolCallBehavior.AutoInvokeKernelFunctions}
        });

        chatHistory.AppendLine("User:" + input);
        chatHistory.AppendLine("Assistant:" + activities.ToString());
        
        Console.WriteLine(activities);
        break;
    ```

1. Escriba `dotnet run` en el terminal. Cuando se le solicite, escriba texto similar al siguiente:

    ```output
    What would you like to do?
    How much is 60 USD in new zealand dollars?
    ```

1. Debe recibir un resultado similar al siguiente:

    ```output
    $60 USD is approximately $97.88 in New Zealand Dollars (NZD)
    What would you like to do?
    ```

1. Escriba una solicitud para obtener sugerencias de destinos con algunas indicaciones de contexto, por ejemplo:

    ```output
    What would you like to do?
    I'm planning an anniversary trip with my spouse, but they are currently using a wheelchair and accessibility is a must. What are some destinations that would be romantic for us?
    ```

1. Debe recibir algunos resultados con recomendaciones de destinos accesibles.

1. Escriba una solicitud para sugerencias de actividades, por ejemplo:

    ```output
    What would you like to do?
    What are some things to do in Barcelona?
    ```

1. Debe recibir recomendaciones que se ajusten al contexto anterior, por ejemplo, actividades accesibles en Barcelona similares a las siguientes:

    ```output
    1. Visit the iconic Sagrada Família: This breathtaking basilica is an iconic symbol of Barcelona's architecture and is known for its unique design by Antoni Gaudí.

    2. Explore Park Güell: Another masterpiece by Gaudí, this park offers stunning panoramic views of the city, intricate mosaic work, and whimsical architectural elements.

    3. Visit the Picasso Museum: Explore the extensive collection of artworks by the iconic painter Pablo Picasso, showcasing his different periods and styles.
    ```

    > [!NOTE]
    > Si el código no genera la salida esperada, puede revisar el código en la carpeta **Solución**.

Puede seguir probando la aplicación con diferentes indicaciones e indicaciones de contexto. Excelente trabajo. Ha proporcionado correctamente indicaciones de contexto al LLM y ha ajustado el código para permitir que el usuario continúe la conversación.

### Revisar

En este laboratorio, ha creado un punto de conexión para el servicio de modelo de lenguaje grande (LLM), ha creado un objeto kernel semántico, ha ejecutado indicaciones mediante el SDK de kernel semántico, ha creado funciones y complementos de kernel semántico, y ha usado la funcionalidad de llamada automática del SDK de kernel semántico para enfocar la intención del usuario a los complementos adecuados. También ha proporcionado contexto al LLM mediante el historial de conversaciones y permitido al usuario continuar con la conversación. Enhorabuena por completar este laboratorio.