# Lab 10: (IaC) Monitoreo
   
En este laboratorio exploraremos monitoreo con herramientas disponibles, contenedores Docker para recolectar, almacenar y visualizar métricas y logs de una aplicación distribuida (Frontend y Backend).

---

## Requisitos Previos e Instalación

1. **Docker Desktop:** Descargar e instalar desde [Docker Desktop](https://www.docker.com/products/docker-desktop/).
   
2. **Entorno Git:** Vincular el repositorio local con GitHub:
   ```bash
   # Inicializar el repositorio si es necesario
   git init

   # Validar conexiones remotas existentes
   git remote -v

   # Desconectar repositorios previos si aplica
   git remote remove origin

   # Validar credenciales locales
   git config user.name
   git config user.email

   # Vincular con el nuevo repositorio remoto
   git remote add origin <URL_DE_TU_REPOSITORIO>


## Aplicaciones
```bash
docker compose up -d --build
```

## Servicios y URLs
| Servicio       | URL                         | Notas                                  |
|----------------|-----------------------------|----------------------------------------|
| Frontend       | http://localhost:8080       | Hello World + botones de tráfico/carga |
| Backend (API)  | http://localhost:3001       | `/api/hello`, `/metrics`, `/load`      |
| Grafana        | http://localhost:3000       | admin / admin                          |
| Prometheus     | http://localhost:9090       | datasource ya provisionado             |
| Loki           | http://localhost:3100       | datasource ya provisionado             |
| Alloy (UI)     | http://localhost:12345      | estado del recolector de logs          |
| cAdvisor       | http://localhost:8081       | métricas por contenedor                |
| node-exporter  | http://localhost:9100/metrics | métricas del host                    |

## Configuraciones
- **Datasources** Prometheus y Loki (provisionados automáticamente).
- Logs etiquetados por Alloy con `tier=application` o `tier=infrastructure`.

## Actividad
- El **dashboard** (paneles de CPU + logs de app e infra).
- La **alarma** de CPU > 50%.

## Reset
```bash
docker compose down -v (borra también dashboards/alarmas creados)

PREGUNTAS

- 1. ¿Por qué necesitamos Loki además de Prometheus si ya tenemos /metrics?
Prometheus está diseñado y optimizado de manera única para almacenar métricas numéricas organizadas como series temporales (números, contadores, tasas). No procesa texto estructurado ni mensajes individuales de error.

Loki se necesita de manera complementaria porque es un motor especializado en guardar logs generados por los contenedores. Las métricas nos dicen cuándo ocurrió algo fuera de lo normal, pero los logs de Loki nos permiten investigar el contexto textual del error para saber exactamente por qué falló el código.

- 2. ¿Qué ventaja aporta que las fuentes de datos de Grafana estén aprovisionadas como código y no creadas a mano?

Repetibilidad y Automatización: Permite desplegar el stack completo en cualquier servidor, entorno (Desarrollo, QA, Producción) o máquina local sin necesidad de configurar nada manualmente a través de clics en la interfaz web.

Consistencia: Elimina los errores humanos derivados de escribir mal direcciones IP, nombres de URLs o puertos al conectar las fuentes.

Control de Versiones: Los cambios en las configuraciones quedan registrados en el historial de Git, facilitando auditorías y restauraciones rápidas ante fallas (rollbacks).

- 3. El panel "CPU contenedor" y el panel "CPU host" pueden mostrar valores muy distintos. ¿Por qué? ¿Cuál usarías para alertar sobre una aplicación concreta?

El panel de CPU del Host mide el uso total del microprocesador de la máquina física o virtual completa, el cual distribuye su potencia entre el sistema operativo, procesos externos y docenas de otros contenedores en ejecución.

El panel de CPU contenedor mide exclusivamente la porción de recursos asignada y consumida de forma aislada por el contenedor específico donde corre el binario o proceso de Node.js (en este caso, el backend).

Para alertar sobre el rendimiento de una aplicación concreta, se debe usar siempre el CPU del contenedor. Si usáramos el CPU del Host, un proceso pesado interno del servidor dispararía falsos positivos, afectando el diagnóstico correcto de nuestro software.

- 4. ¿Qué diferencia hay entre el evaluation interval y el pending period de una alarma?

Evaluation Interval: Es la frecuencia exacta de tiempo con la que Grafana ejecuta la consulta de PromQL hacia la base de datos para revisar si el valor medido cruzó el umbral definido (por ejemplo, evaluar la regla cada 10s).

Pending Period: Es el tiempo de seguridad que la alarma debe mantenerse violando el umbral establecido antes de declararse oficialmente en estado crítico (Firing). Su finalidad es absorber picos de rendimiento normales y transitorios para evitar alertas ruidosas y falsas alarmas instantáneas.


