# Laboratorio 7: Desarrollo Serverless con AWS Lambda, DynamoDB y API Gateway

En este laboratorio exploraremos el desarrollo de aplicaciones serverless utilizando AWS Lambda, DynamoDB y API Gateway. Desarrollaremos una aplicación simple para registrar naves espaciales y sus pilotos, implementando operaciones básicas de lectura y escritura en una base de datos NoSQL. Este laboratorio te permitirá entender los fundamentos de la arquitectura serverless y cómo construir APIs RESTful escalables sin gestionar infraestructura.

## Marco Teórico

### ¿Qué es la computación serverless?

La computación serverless es un modelo de ejecución en la nube donde el proveedor de servicios administra automáticamente la infraestructura subyacente. Los desarrolladores solo se enfocan en escribir código, mientras que el proveedor maneja el aprovisionamiento, escalamiento y mantenimiento de los servidores.

### Servicios AWS para arquitectura serverless

#### AWS Lambda

AWS Lambda es un servicio de computación que ejecuta código sin necesidad de aprovisionar o administrar servidores. Ejecuta tu código solo cuando es necesario y escala automáticamente desde unas pocas solicitudes por día hasta miles por segundo.

**Características principales:**

-   Ejecución de código sin servidor
-   Escalamiento automático
-   Pago por uso (solo pagas por el tiempo de ejecución)
-   Soporte para múltiples lenguajes (Python, Node.js, Java, etc.)
-   Integración nativa con otros servicios AWS

**Modelo de precios:**

-   Número de solicitudes
-   Duración de ejecución (GB-segundo)
-   Capa gratuita: 1 millón de solicitudes gratuitas por mes

#### DynamoDB

DynamoDB es una base de datos NoSQL totalmente administrada que proporciona un rendimiento rápido y predecible con escalabilidad automática.

**Características principales:**

-   Base de datos NoSQL completamente administrada
-   Rendimiento de milisegundos de un dígito
-   Escalabilidad automática
-   Cifrado en reposo
-   Respaldos automáticos

**Conceptos clave:**

-   **Tabla**: Colección de elementos
-   **Elemento (Item)**: Registro individual en la tabla
-   **Atributo**: Campo de datos en un elemento
-   **Clave de partición**: Atributo que determina la partición física donde se almacenan los datos y con la que se identifican los elementos

#### API Gateway

API Gateway es un servicio completamente administrado que facilita a los desarrolladores crear, publicar, mantener, monitorear y proteger APIs a cualquier escala. Tiene diferentes modos para poder generar APIs como REST API, HTTP API y WebSocket API.

**Características principales:**

-   Creación de APIs HTTP y WebSocket
-   Autenticación y autorización
-   Límites de velocidad
-   Integración directa con Lambda

#### CloudWatch

Amazon CloudWatch es un servicio de monitoreo y observabilidad para recursos y aplicaciones en AWS. Proporciona datos y perspectivas procesables para monitorear aplicaciones, responder a cambios en el rendimiento del sistema y optimizar la utilización de recursos.

**Características principales:**

-   Monitoreo de métricas y logs
-   Alarmas y notificaciones
-   Dashboards personalizados
-   Integración con otros servicios AWS

## Pasos iniciales

### Requisitos previos

Antes de comenzar, asegúrate de tener:

1. Cuenta de AWS
2. Navegador web
3. Conocimientos básicos de Python
4. Acceso a la consola de administración de AWS

## 1. Acceso a AWS y configuración inicial

### Paso 1.1: Inicio de sesión en AWS

1. Ve a [https://console.aws.amazon.com](https://console.aws.amazon.com)
2. Inicia sesión con tu cuenta
3. Una vez en la consola, verifica que estés en la región **sa-east-1** (Sudamérica - São Paulo)
    - Si no estás en sa-east-1, haz clic en el selector de región en la esquina superior derecha
    - Selecciona "South America (São Paulo) sa-east-1"

### Paso 1.2: Consideraciones de costos

**⚠️ Importante sobre costos:**

-   AWS Lambda: Capa gratuita incluye 1 millón de solicitudes gratuitas por mes
-   DynamoDB: Capa gratuita incluye 25 GB de almacenamiento
-   API Gateway: Capa gratuita incluye 1 millón de llamadas API por mes
-   CloudWatch: Capa gratuita incluye 10 métricas personalizadas y 5 GB de logs por mes

Para este laboratorio, si sigues las instrucciones y eliminas los recursos al final, el costo debería estar completamente dentro de la capa gratuita.

### Paso 1.2: Familiarización con los servicios

1. En la consola de AWS, agrega los siguientes servicios como favoritos:

    - **Lambda**
    - **DynamoDB**
    - **API Gateway**
    - **CloudWatch**

2. Explora brevemente cada servicio para familiarizarte con la interfaz

## 2. Creación de tabla DynamoDB

### Paso 2.1: Crear la tabla para naves espaciales

1. En la consola de AWS, busca y accede a **DynamoDB**
2. En el menú del lado izquierdo (puede estar colapsado) selecciona "Tables" y haz clic en "Create table"
3. Configura los siguientes parámetros:
    - **Table name**: `spaceship_registry`
    - **Partition key**: `spaceship_id` (String)
    - **Sort key**: Dejar vacío
4. En **Settings**, selecciona "Customize settings"
5. **Table class**: DynamoDB Standard
6. **Capacity mode**: On-demand
7. Deja todas las demás configuraciones por omisión
8. Haz clic en "Create table"
9. Espera a que la tabla esté en estado "Active" (puede tomar 1-2 minutos)

### Paso 2.2: Explorar la tabla creada

1. Haz clic en el nombre de tu tabla `spaceship_registry`
2. Explora las pestañas disponibles:
    - **Overview**: Información general y métricas
    - **Items**: Visualización de los elementos (inicialmente vacía)
    - **Metrics**: Métricas de rendimiento
    - **Monitoring**: Monitoreo y alertas
3. Anota el **Table ARN** que aparece en la pestaña "Overview" - lo necesitarás más adelante

## 3. Creación de función Lambda

### Paso 3.1: Crear la función Lambda para GET

1. En la consola de AWS, busca y accede a **Lambda**
2. En el menú izquierdo (puede estar colapsado) selecciona "Functions" y haz clic en "Create function"
3. Configura los siguientes parámetros:
    - **Author from scratch**: Seleccionado
    - **Function name**: `spaceship-get`
    - **Runtime**: Python 3.13
    - **Architecture**: x86_64
4. En **Permissions**, expande "Change default execution role" y selecciona "Create a new role with basic Lambda permissions"
5. Expande **Additional configurations** y habilita "Function URL". En authentication type selecciona "None"
6. Haz clic en "Create function"

### Paso 3.2: Configurar permisos para DynamoDB (función GET)

1. Una vez creada la función, ve a la pestaña "Configuration"
2. En el panel izquierdo, selecciona "Permissions"
3. Haz clic en el enlace del "Role name" (será algo como `spaceship-get-role-xyz`)
4. Se abrirá la consola de IAM (Identity and Access Management)
5. Haz clic en "Add permissions" → "Attach policies"
6. Busca y selecciona `AmazonDynamoDBFullAccess`
7. Haz clic en "Add permissions"

### Paso 3.3: Implementar el código de la función GET

1. Regresa a tu función Lambda `spaceship-get`
2. Ve a la pestaña "Code"
3. En el editor de código integrado de Lambda, borra el contenido por omisión del archivo `lambda_function.py`
4. Reemplaza todo el código con el siguiente:

```python
import json
import boto3

def lambda_handler(event, context):
    # Conectar a DynamoDB
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('spaceship_registry')

    # Obtener el ID de la nave si se proporciona
    spaceship_id = event.get('pathParameters', {}).get('id') if event.get('pathParameters') else None

    try:
        if spaceship_id:
            # Obtener una nave específica
            response = table.get_item(Key={'spaceship_id': spaceship_id})
            if 'Item' in response:
                return {
                    'statusCode': 200,
                    'body': json.dumps(dict(response['Item']))
                }
            else:
                return {
                    'statusCode': 404,
                    'body': json.dumps({'error': 'Nave no encontrada'})
                }
        else:
            # Obtener todas las naves
            response = table.scan()
            items = response.get('Items', [])
            return {
                'statusCode': 200,
                'body': json.dumps([dict(item) for item in items])
            }
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps({'error': 'Error interno del servidor'})
        }
```

6. Analiza el código para entender su funcionamiento. Las palabras clave son:
    - `boto3.resource('dynamodb')`: Conexión a DynamoDB
    - `table.get_item(...)`: Obtener un ítem específico
    - `table.scan()`: Obtener todos los ítems
    - Manejo de errores y respuestas HTTP
7. Haz clic en **"Deploy"** para guardar la función GET
8. Espera a que aparezca el mensaje "Changes deployed"

### Paso 3.4: Crear la segunda función Lambda para POST

1. Ve de nuevo a la consola de **Lambda** y crea una función llamada `spaceship-post` con las mismas características que la anterior
2. Otorga permisos de `AmazonDynamoDBFullAccess` a esta función siguiendo los mismos pasos que en el Paso 3.2
3. Reemplaza el código con el siguiente:

```python
import json
import boto3

def lambda_handler(event, context):
    # Conectar a DynamoDB
    dynamodb = boto3.resource('dynamodb')
    table = dynamodb.Table('spaceship_registry')

    try:
        # Obtener datos del body
        body = json.loads(event['body'])

        # Validar campos requeridos
        if not all(field in body for field in ['spaceship_id', 'model', 'pilot']):
            return {
                'statusCode': 400,
                'body': json.dumps({'error': 'Campos requeridos: spaceship_id, model, pilot'})
            }

        # Crear item para DynamoDB (solo los campos especificados)
        item = {
            'spaceship_id': body['spaceship_id'],
            'model': body['model'],
            'pilot': body['pilot']
        }

        # Guardar en DynamoDB
        table.put_item(Item=item)

        return {
            'statusCode': 201,
            'body': json.dumps({'message': 'Nave registrada exitosamente', 'spaceship': item})
        }

    except json.JSONDecodeError:
        return {
            'statusCode': 400,
            'body': json.dumps({'error': 'JSON inválido'})
        }
    except Exception as e:
        print(f"Error: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps({'error': 'Error interno del servidor'})
        }
```

4. Analiza el código para entender su funcionamiento. Las palabras clave son:
    - `boto3.resource('dynamodb')`: Conexión a DynamoDB
    - `table.put_item(...)`: Insertar un ítem
    - Validación de campos y manejo de errores
5. Haz clic en **"Deploy"** para guardar la función POST
6. Espera a que aparezca el mensaje "Changes deployed"

### Paso 3.7: Probar las funciones Lambda

#### Probar función POST:

1. Ve a la función `spaceship-post`
2. En el "Code source", en el menú izquierdo, haz clic en "Test" y luego en "Create new test event"
3. Debemos crear un nuevo test que llamaremos `test-post-spaceship`
4. Reemplaza el JSON por omisión con:

```json
{
    "body": "{\"spaceship_id\": \"MILLENNIUM-FALCON\", \"model\": \"YT-1300\", \"pilot\": \"Han Solo\"}"
}
```

5. Haz clic en "Save" y luego "Invoke"
6. Verifica que la respuesta tenga status code 201
7. Vuelve a DynamoDB, en el menú izquierdo selecciona "Explore items", selecciona tu tabla `spaceship_registry` y verifica que la nave que registraste aparezca en la lista

#### Probar función GET:

1. Ve a la función `spaceship-get` y crea un evento `test-get-all-spaceships` con un JSON del evento:

```json
{
    "pathParameters": null
}
```

2. Guarda y ejecuta el test
3. Verifica que la respuesta muestre la nave que registraste

## 4. Probar desde Postman directamente

Si nos vamos a la configuración de cualquiera de las funciones lambda, en el menú izquierdo podemos ver la opción "Function URL". Ahí podemos copiar la URL pública de la función y probarla directamente desde Postman. En el caso de la función `spaceship-get` incluso la podemos probar directamente en el browser.

El caso de `spaceship-post` es similar, pero debemos hacer una petición POST con un body en formato JSON.

## 5. Creación y configuración de API Gateway

### Paso 5.1: Crear la HTTP API

1. En la consola de AWS, busca y accede a **API Gateway**
2. Expande el menú izquierdo, haz clic en "APIs" y luego en "Create API"
3. En **HTTP API**, haz clic en "Build" y llama a la API `spaceship-registry-api`
4. Al apretar "Next", en la configuración de rutas no ingreses nada y haz clic en "Next"
5. En la lista de "Define stages", deja el stage por omisión `$default` y haz clic en "Next"
6. La última ventana es el resumen. Revisa que todo esté correcto y haz clic en "Create"

### Paso 5.2: Configurar integraciones

Una vez creada la API, debemos agregar las integraciones con nuestras funciones Lambda. Nos vamos al menú izquierdo y seleccionamos Routes:

1. Hacemos click en "Create"
2. Seleccionamos método "GET", path `/spaceships` y hacemos click en "Create"
3. Repetimos el proceso para crear otra ruta:
    - Método: GET
    - Path: `/spaceships/{id}`
    - Hacemos click en "Create"
4. Finalmente, creamos la ruta para POST:
    - Método: POST
    - Path: `/spaceships`
    - Hacemos click en "Create"

El arbol de rutas debería verse así:

```
/spaceships
 ├── GET
 ├── POST
 └── /{id}
      └── GET
```

### Paso 5.3: Configurar integraciones con Lambda

1. Selecciona la ruta `/spaceships` con método GET
2. En la sección "Integration", haz clic en "Attach integration"
3. Selecciona "Create and attach an integration"
4. Configura los siguientes parámetros:
    - **Integration type**: Lambda function
    - **Lambda function**: `spaceship-get`
5. Haz clic en "Create"
6. Repite el proceso para la ruta `/spaceships/{id}` con método GET, usando la misma función `spaceship-get`
7. Finalmente, repite el proceso para la ruta `/spaceships` con método POST, usando la función `spaceship-post`

### Paso 5.4: Probar la API con browser

1. En el menú izquierdo, selecciona "Stages"
2. Selecciona el stage `$default`
3. Copia la "Invoke URL" que aparece en la parte superior
4. Abre una nueva pestaña en tu navegador y pega la URL seguida de `/spaceships`, debería verse de la forma `https://TU-API-ID.execute-api.us-east-1.amazonaws.com/spaceships`
5. Deberías recibir la misma respuesta que al probar la función Lambda directamente, obteniendo la nave "MILLENNIUM-FALCON"

### Paso 5.5: Probar la API con Postman

Abrimos Postman para hacer pruebas más completas:

1. Vamos al sitio [https://www.postman.com/](https://www.postman.com/)
2. Iniciamos sesión o creamos una cuenta gratuita
3. En el menú superior seleccionamos "Workspaces" y luego "My Workspace"
4. Hacemos click en "Create Collection" y la llamamos `Spaceship Registry API`
5. Haciendo click en "New" sobre la colección creada, seleccionamos "HTTP" y creamos los requests que se indican a continuación
6. Al terminar de configurar cada request, hacemos click en "Save" para guardarlo en la colección

#### GET /spaceships

-   **Name**: Get all spaceships
    -   El nombre se encuentra en la parte superior de la nueva ventana. Su valor por omisión es "Untitled Request"
-   **Method**: GET
-   **URL**: `https://TU-API-ID.execute-api.us-east-1.amazonaws.com/spaceships`

#### GET /spaceships/{id}

-   **Name**: Get spaceship by ID
-   **Method**: GET
-   **URL**: `https://TU-API-ID.execute-api.us-east-1.amazonaws.com/spaceships/MILLENNIUM-FALCON`

#### POST /spaceships

-   **Name**: Create Spaceship
-   **Method**: POST
-   **URL**: `https://TU-API-ID.execute-api.us-east-1.amazonaws.com/spaceships`
-   **Body** (raw JSON):

```json
{
    "spaceship_id": "X-WING-RED5",
    "model": "T-65 X-wing",
    "pilot": "Luke Skywalker"
}
```

## 6. Inspección de logs en CloudWatch

### Paso 6.1: Acceder a los logs de Lambda

1. En AWS, ve a la consola de **CloudWatch**
2. En el panel izquierdo, haz clic en "Logs" y luego "Log groups"
3. Busca los log groups de tus funciones: `/aws/lambda/spaceship-get` y `/aws/lambda/spaceship-post`
4. Haz clic en el log group para acceder

### Paso 6.2: Explorar los log streams

1. Verás múltiples "log streams"
2. Haz clic en el log stream más reciente
3. Observa los diferentes tipos de logs:
    - **START RequestId**: Inicio de una invocación
    - **END RequestId**: Final de una invocación
    - **REPORT RequestId**: Reporte de métricas (duración, memoria usada, etc.)
    - Tus logs personalizados (los `print()` en tu código)

### Paso 6.3: Analizar métricas en los logs

En cada entrada **REPORT**, observa:

```
REPORT RequestId: 12345678-1234-1234-1234-123456789012
Duration: 156.93 ms	Billed Duration: 157 ms	Memory Size: 128 MB	Max Memory Used: 67 MB	Init Duration: 234.12 ms
```

-   **Duration**: Tiempo real de ejecución
-   **Billed Duration**: Tiempo facturado (redondeado)
-   **Memory Size**: Memoria asignada a la función
-   **Max Memory Used**: Memoria máxima utilizada
-   **Init Duration**: Tiempo de cold start (solo aparece en cold starts)

### Paso 6.4: Cold Start vs Warm Start en Lambda

Una función lambda tiene dos tipos de arranques:

- **Cold Start**: Ocurre cuando Lambda debe inicializar un nuevo contenedor para ejecutar tu función. Incluye:
    - Descarga del código
    - Inicialización del runtime
    - Ejecución del código de inicialización
    - **Tiempo adicional**: 100ms - varios segundos
- **Warm Start**: Ocurre cuando Lambda reutiliza un contenedor existente:
    - Solo ejecuta el código de la función
    - **Tiempo adicional**: Mínimo (pocos ms)

Para observar la diferencia:

1. Ejecuta tus funciones varias veces seguidas
2. Examina los logs y nota:
    - **Cold start**: Tendrá `Init Duration` en el REPORT
    - **Warm start**: NO tendrá `Init Duration`
3. Compara los tiempos de Duration entre cold y warm starts

### Paso 6.5: Explorar métricas automáticas

1. En CloudWatch, ve a "Metrics" → "All metrics"
2. Busca "Lambda" para ver métricas automáticas de tus funciones
3. Explora métricas como:
    - **Invocations**: Número de invocaciones
    - **Duration**: Tiempo promedio de ejecución
    - **Errors**: Número de errores
    - **Throttles**: Invocaciones limitadas

## 7. Inspección de datos en DynamoDB

### Paso 7.1: Explorar elementos en la tabla

1. Ve a la consola de **DynamoDB**
2. Haz clic en "Tables" → "spaceship_registry"
3. Ve a la pestaña "Explore table items"
4. Observa las naves espaciales que has registrado
5. Experimenta con:
    - **Scan**: Ver todos los elementos
    - **Query**: Buscar elementos específicos (por spaceship_id)

### Paso 7.2: Agregar elementos manualmente

1. Haz clic en "Create item"
2. Agrega una nueva nave con los siguientes datos:
    - **spaceship_id**: `STAR-DESTROYER-ISD1`
    - **model**: `Imperial I-class Star Destroyer`
    - **pilot**: `Darth Vader`
3. Haz clic en "Create item"
4. Verifica que aparezca en la lista

### Paso 7.3: Editar elementos existentes

1. Selecciona un elemento existente
2. Haz clic en "Actions" → "Edit item"
3. Modifica algún campo (por ejemplo, cambia el piloto)
4. Haz clic en "Save changes"
5. Verifica el cambio en la tabla

## 8. Monitoreo y métricas

### Paso 8.1: Explorar métricas de DynamoDB

1. Ve a tu tabla DynamoDB (menú izquierdo → Tables → spaceship_registry)
2. Haz clic en la pestaña "Monitor"
3. Observa las métricas:
    - **Put latency**: Latencia de escritura
    - **Get latency**: Latencia de lectura
    - **Successful write requests**: Escrituras exitosas
    - **Scan latency**: Latencia de escaneo

### Paso 8.2: Explorar métricas de Lambda

1. Ve a alguna de tus funciones Lambda
2. Haz clic en la pestaña "Monitor"
3. Observa las métricas disponibles:
    - **Invocations**: Número de invocaciones
    - **Duration**: Tiempo promedio de ejecución
    - **Error count and success rate**: Errores y tasa de éxito
    - **Throttles**: Limitaciones de concurrencia

## 9. Experimentación

### Paso 9.1: Generar múltiples invocaciones

1. Ejecuta rápidamente varias pruebas de tu API (5-10 veces) usando Postman
2. Ve a CloudWatch y observa cómo las invocaciones posteriores son más rápidas (warm starts)
3. Observa los tiempos de respuesta y revisa los logs en CloudWatch para ver las diferencias en memoria y tiempo de ejecución.

### Paso 9.2: Prints

1. Agrega `print()` statements en tus funciones Lambda para registrar información adicional (por ejemplo, datos recibidos, pasos del proceso)
2. Despliega los cambios y vuelve a invocar las funciones
3. Revisa los logs en CloudWatch para ver los mensajes impresos y entender mejor el flujo de ejecución

## 10. Limpieza de recursos

### Paso 10.1: Eliminar recursos en orden

Para evitar cargos innecesarios, elimina los recursos en el siguiente orden:

#### Eliminar API Gateway

1. Ve a API Gateway y anda a "APIs"
2. Selecciona tu API "spaceship-registry-api"
3. Haz clic en "Delete"
4. Confirma escribiendo el nombre de la API

#### Eliminar funciones Lambda

1. Ve a Lambda y selecciona "Functions"
2. Selecciona tus funciones
3. Haz clic en "Actions" → "Delete"

#### Eliminar tabla DynamoDB

1. Ve a DynamoDB
2. Selecciona tu tabla "spaceship_registry"
3. Haz clic en "Delete"
4. Confirma dejando las opciones por omisión

#### Eliminar rol de IAM

1. Busca IAM en la consola de AWS, y en el menu izquierdo selecciona "Roles"
2. Busca los roles que se crearon para tu función Lambda (deberían tener la forma `spaceship-get-role-xyz` y `spaceship-post-role-xyz`)
3. Selecciónalo y haz clic en "Delete"
4. Confirma la eliminación

## Conclusión

En este laboratorio has aprendido los conceptos fundamentales de la arquitectura serverless utilizando AWS Lambda, DynamoDB y API Gateway. Has implementado una aplicación simple que puede registrar y consultar naves espaciales, incluyendo:

-   Desarrollo de funciones Lambda separadas con Python 3.13
-   Operaciones básicas de lectura y escritura en DynamoDB
-   Creación de APIs con API Gateway
-   Análisis de cold start vs warm start usando CloudWatch
-   Monitoreo básico de aplicaciones serverless
-   Uso de Postman para pruebas de API

Estos son los fundamentos para construir aplicaciones serverless más complejas y escalables. A medida que avances, podrás explorar funcionalidades adicionales.
