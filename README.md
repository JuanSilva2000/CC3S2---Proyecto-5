# Informe acerca de el trabajo realizado
Vamos a desarrollar un juego de "Esacape ROOM" donde los jugadores deben resolver una serie de acertijos para escapar de un habitación virtual
El juego consiste en habitaciones secuenciales, donde vamos a ir agregando paso a paso cada funcionalidad

# Infraestructura

## 1: Implementación `ci.yml`
![](img/ci.png)

El archivo `ci.yml` configura el pipeline de integración continua (CI) para el proyecto. Este pipeline se ejecuta automáticamente cada vez que hay un commit en la rama `main`. Está diseñado para correr en una máquina virtual Ubuntu mediante GitHub Actions y realizar una serie de pasos automatizados.

### Explicación:
- **Configuración del evento**: El pipeline se activa con el evento `push` en la rama `main`, asegurando que solo los cambios en esta rama desencadenen la integración continua.
- **Configuración de Node.js**: El entorno se configura para usar la versión 18 de Node.js, lo cual es crucial para garantizar la compatibilidad del código con una versión específica del entorno de ejecución.
- **Instalación de dependencias**: El paso `npm install` se encarga de instalar todas las dependencias definidas en el archivo `package.json`, asegurando que el entorno del pipeline tenga todos los módulos necesarios.
- **Construcción de la imagen Docker**: El pipeline luego construye una imagen Docker basada en el `Dockerfile` presente en el proyecto. Esto garantiza que la aplicación se pueda ejecutar de forma consistente en diferentes entornos.
- **Ejecución de tests**: El paso `npm test` asegura que todos los tests definidos se ejecuten, validando que la aplicación esté funcionando correctamente.
- **Auditoría de seguridad**: `npm audit` revisa las vulnerabilidades en las dependencias, asegurando que el proyecto esté protegido contra posibles riesgos de seguridad.
- **Despliegue del contenedor Docker**: Finalmente, se levanta un contenedor Docker con la aplicación escuchando en el puerto 3000.

## 2: Implementación `Dockerfile`
![](img/Dockerfile.png)

El `Dockerfile` define el entorno de Docker para el proyecto. Aquí se especifican las instrucciones necesarias para construir una imagen Docker que contenga la aplicación.

### Explicación:
- **Base de imagen**: La imagen base es `node:18`, que incluye Node.js en su versión 18. Esto asegura que la aplicación se ejecute en un entorno Node.js de última generación.
- **Directorio de trabajo**: El comando `WORKDIR /app` define el directorio dentro del contenedor donde se ubicará la aplicación.
- **Copia de dependencias**: Se copian los archivos `package*.json` al contenedor, y luego se ejecuta `npm install` para instalar las dependencias de la aplicación.
- **Copia del código fuente**: El comando `COPY . .` copia el código fuente completo desde el directorio del host al contenedor.
- **Exposición del puerto**: `EXPOSE 3000` asegura que el contenedor escuche en el puerto 3000, permitiendo que la aplicación sea accesible.
- **Comando de inicio**: Finalmente, se especifica que la aplicación debe iniciarse ejecutando `node src/app.js` cuando el contenedor esté en funcionamiento.

## 3: Implementación `docker-compose.yml`
![](img/docker-compose.png)

El archivo `docker-compose.yml` se utiliza para orquestar múltiples servicios Docker en un entorno de contenedores, incluyendo la aplicación Node.js, Prometheus para monitoreo, y Grafana para visualización de métricas.

### Explicación:
- **Definición del servicio `app`**: Este servicio construye la imagen de la aplicación utilizando el `Dockerfile` y expone el puerto 3000 (o el que esté definido por la variable de entorno `APP_PORT`). Además, se define la variable de entorno `NODE_ENV` que puede configurarse como `production` o `development` dependiendo del entorno.
- **Servicio `prometheus`**: Prometheus es un sistema de monitoreo que recopila métricas de la aplicación. El volumen monta el archivo `prometheus.yml` desde el sistema de archivos del host para configurar las tareas de monitoreo. El puerto 9090 es expuesto para acceder a la interfaz web de Prometheus.
- **Servicio `grafana`**: Grafana es una herramienta de visualización de métricas que se ejecuta en el puerto 3000 (o 3100 si se usa la variable de entorno `GRAFANA_PORT`). Grafana permite crear paneles interactivos para visualizar los datos recolectados por Prometheus.

## 4: Implementación `prometheus.yml`
![](img/Prometheus.png)

El archivo `prometheus.yml` define las configuraciones para que Prometheus pueda realizar scraping de métricas desde la aplicación Node.js.

### Explicación:
- **Intervalo global de scraping**: El valor `scrape_interval` se define en 15 segundos, lo que significa que Prometheus recopilará métricas cada 15 segundos.
- **Configuración de scraping para la aplicación Node.js**: Se define un job llamado `node-app` con un target estático que apunta a `app:3000`, es decir, el servicio de la aplicación en Docker, que expone sus métricas en el puerto 3000. Prometheus usará esta configuración para recolectar métricas y almacenarlas para visualización y análisis.


  
    
# BACKEND  
  
## 1: Módulo Data  
No se usó niguna base de datos, en vez de eso, para almacenar las habitaciones con sus acertijos, respuesta y pista creamos un objeto en el script `data.js` que los contenia de la siguiente manera:  
![](img/data.png)  
  
Se va a poder interactuar con esos datos con dos funciones, estas están en el script `data_func.js`.   
  
Una de esas funciones se llama `getHabitacionActual()` que básicamente devuelve la tanto la habitación en que se encuentra el jugador y su acertijo respectivo.  

![](img/data_func-1.png)   
  
La otra función `procesarComando()` recibe como parámetro el comando digitado por el usuario desde el frontend y aqui procesamos una respuesta que devolverá la función en base al comando digitado:  
  
![](img/data_func-2.png)  
  
## 2. Módulo src  
Dependencias a tener en cuenta:   
-Usamos `express` para construir nuestra api.  
-Usamos `body-parser` que nos ayuda a procesar el cuerpo de las solicitudes entrantes que tienen formato Json.  
  
Todo eso lo instalamos con
```
npm install expree body-parser
```  
Y en el `package.json` en la sección de dependencies aparece dichas dependencias instaladas:  
```
     "dependencies": {
        "body-parser": "^1.20.3",
        "express": "^4.21.0",
     }
```    
  
El archivo app configuramos los middleware, y lo lanzamos en el puerto 3000, además montamos un conjunto de rutas bajo el prefijo api (ver linea 11 de la imágen):  
  
![](img/src-1.png)  
 
En el archivo `routes.js` es donde se definen los 2 endpoints que utilizarán las funciones definidas del modulo data (`data/data_func.js`) y enviar una respuesta al frontend para que las renderize en el dom y lo muestre al usuario.  
  
![](img/src-2.png)  
  
## 3. Módulo test  
Para los test usamos jest y una dependencia llamada `jest-fetch-mock`, como su nombre lo indica es un mock de fetch que nos ayudará a definir respuestas que debería devolver la api al hacer algun request utilizando el método `mockResponseOnce()`.  
La instalación de esta dependecia se hace mendiante:  
```
    npm install jest-fetch-mock
```
  
1. El primer test es para verificar que la api esta respondiendo al comando `ver pista`, por defecto se comienza en la habitación uno y la pista esperada es "Es una fruta", lo que se hace es verificar que la api devuelva eso  
  
![](img/test-backend-1.png)  
   
2. El segundo test es para verificar que se muestra el mensaje de "Respuesta correcta" cuando se responde bien al acertijo, el fecth se hace con un método POST pues con este se envía datos al servidor (api), en este caso es el comando, como por defecto primero se tiene el primer acertijo entonces al responder con `pera` debería mostrarse el mensaje de respuesta correcta, y eso lo verifica el test:  
  
![](img/test-backend-2.png)
