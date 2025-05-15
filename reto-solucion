# SOLUCIÓN

## Supuestos Clave para la Propuesta
- Plataforma eCommerce con volumen medio-alto de transacciones, que requiere alta escalabilidad, baja latencia y alta disponibilidad.
- Infraestructura principal en AWS, integrada con sistemas on-premise existentes.
- Arquitectura basada en microservicios desacoplados para facilitar mantenimiento y escalabilidad.
- Manejo de datos estructurados y semiestructurados con diferentes necesidades de persistencia.
- Uso de un motor para orquestar procesos de negocio BPNM.
- Desarrollo ágil con despliegues automatizados para asegurar calidad y rapidez.
- Experiencia de usuario coherente entre web y móvil.

## Diseño de Arquitectura (Diagramas y Descripción):

### Consideraciones pera cada Microservicio - BACKEND

Se utilizará los siguientes patrones: 

Hexagonal Architecture, para desacoplar la lógica de negocio de tecnologías externas, facilitando pruebas, mantenibilidad y evolución del servicio. Hexagonal sobre Clean nos permite una implementación mas agil y menos estricta, ademas de tener en cuenta que tendremos un servicio de orquestación y la logica de flujo compleja ira sobre dicha herramienta.

CQRS: Para mejorar el rendimiento y la escalabilidad en e-commerce cuando se eleve el nivel de transaccionalidad, separando modelos de lectura y escritura.

Patrones de Resiliencia (Circuit Breaker, Retry, Timeout, Fallback): Para prevenir el colapso en cascada de microservicios dependientes. Especialmente críticos en flujos de checkout, pago o inventario. (Ver patrones de Orquestación y resilencia)

### Diagrama de Arquitectura 

```mermaid
graph TD
    subgraph Frontend
        A1[Angular Web App]
        A2[Flutter Mobile App]
    end

    subgraph API_Gateway
        G[API Gateway AWS API Gateway]
    end

    subgraph Microservicios
        MS1[Servicio Usuarios Spring Boot]
        MS2[Servicio Pedidos Spring Boot]
        MS3[Servicio Inventario Spring Boot]
        MS4[Servicio Pagos Spring Boot]
        MS5[Servicio BPMN Camunda]
    end

    subgraph Bases_de_Datos
        DB1[Oracle DB]
        DB2[PostgreSQL DB]
        DB3[MongoDB]
        Cache[Redis Cache]
    end

    subgraph Mensajería
        MQ[AWS SNS SQS]
    end

    subgraph Infraestructura_AWS
        EKS[AWS EKS Kubernetes]
        ECR[AWS ECR Docker Registry]
        CI_CD[CI/CD Pipeline CodePipeline CodeBuild]
    end

    %% Relaciones Frontend <-> Gateway
    A1 -->|REST/HTTP| G
    A2 -->|REST/HTTP| G

    %% Gateway -> Microservicios
    G --> MS1
    G --> MS2
    G --> MS3
    G --> MS4
    G --> MS5

    %% Microservicios <-> Bases de datos
    MS1 --> DB1
    MS2 --> DB2
    MS3 --> DB3
    MS4 --> DB2
    MS1 --> Cache
    MS2 --> Cache

    %% Microservicios <-> Mensajería
    MS2 -->|Evento pedido creado| MQ
    MS3 -->|Evento inventario actualizado| MQ
    MS4 -->|Evento pago procesado| MQ
    MS5 -->|Orquestación procesos| MQ

    MQ --> MS1
    MQ --> MS2
    MQ --> MS3
    MQ --> MS4
    MQ --> MS5

    %% Infraestructura
    MS1 --> EKS
    MS2 --> EKS
    MS3 --> EKS
    MS4 --> EKS
    MS5 --> EKS
    EKS --> ECR
    CI_CD --> ECR
    CI_CD --> EKS

```

### Comunicaciones

Sincrónica: El frontend (Angular y Flutter) se comunica con los microservicios a través del API Gateway, que enruta peticiones REST con autenticación y autorización centralizada en el mismo Gateway.

Asincrónica: Se usa AWS SQS para eventos de negocio que requieren desacople temporal o que se ejecutan en segundo plano (ejemplo: orden creada, stock actualizado, pago confirmado). Esto mejora escalabilidad y resiliencia.

### Persistencia de Datos
- Oracle: Alta consistencia y transaccionalidad para datos críticos (usuarios, pagos).
- PostgreSQL: Para datos transaccionales secundarios (pedidos), considerando buena escalabilidad horizontal.
- MongoDB: Para datos no relacionales, como catálogo de productos con esquemas flexibles y rápido acceso (lectura/escritura).
- Redis: Cache en memoria para acelerar consultas frecuentes, gestionar sesiones, contadores, carrito de compras.


## Patrones de Orquestación y Resiliencia:

- Orquestación con BPMN (Camunda): Para manejar flujos de negocio complejos (pedidos, pagos, inventarios) donde garantizamos control del flujo y manejo de errores.

- Orquestación con SAGA - Coreografía : Para tareas simples, eventos asíncronos y notificaciones

- Circuit breakers (Resilience4j): Para la comunicaciónes directas entre microservicios.

- Otros patrones a considerar:
    - Retry  
    - Timeout  
    - Fallback  
    - Rate Limiting  
    - Load Balancer  
    - Health Checks  
    - Caching


## Infraestructura y CI/CD:

### Diagrama de Infraestructura

```mermaid
---
config:
  layout: elk
---
flowchart TD
 subgraph subGraph0["EKS Cluster - Contenedores"]
        A1["Microservicios Java - Spring Boot"]
        A2["Camunda BPMN Engine"]
        A3["Spring Cloud Gateway"]
        A4["Vault - Gestión de secretos"]
  end
 subgraph subGraph1["Servicios SaaS y AWS Gestionados"]
        B1["Amazon RDS - PostgreSQL"]
        B2["Amazon RDS - Oracle"]
        B3["MongoDB Atlas - NoSQL"]
        B4["Amazon ElastiCache - Redis"]
        B5["Amazon MSK - Kafka"]
        B6["AWS ALB - Load Balancer"]
  end
    B6 --> A3
    A3 --> A1
    A1 --> B1 & B2 & B3 & B4 & B5 & A2 & A4

```

#### Integración híbrida (AWS - On-premise):
VPN/Direct Connect asegura baja latencia y seguridad en comunicación con sistemas existentes, importante para migración y coexistencia.

#### Flujo CI

```mermaid 
flowchart TD
  A[Commit de Codigo] --> B[Build y Analisis Estatico con SonarQube]
  B --> C{Calidad Aprobada?}

  C -- No --> D[Notificar errores a desarrolladores]
  C -- Si --> E[Build Artefactos]

  E --> F{Es Backend Frontend o Mobile?}

  F -- Backend --> G[Compilar y empaquetar JAR WAR]
  F -- Frontend --> H[Build Angular JS CSS HTML]
  F -- Mobile --> I[Build Flutter APK IPA]

  G --> J[Push a JFrog Artifactory]
  H --> J
  I --> J

  J --> K[Escaneo de seguridad en JFrog]
  K --> L{Vulnerabilidades?}

  L -- Si --> M[Alerta y Bloqueo Pipeline]
  L -- No --> N[Ejecutar pruebas unitarias]

  N --> O{Tests pasaron?}

  O -- No --> P[Reportar fallos a QA y Dev]
  O -- Si --> Q[Pipeline aprobado para despliegue]

```

### Estrategia de Deployment 
Se adopta **Rolling Update** como estrategia principal en Amazon EKS, ya que permite la actualización progresiva de pods sin generar interrupciones en el servicio y lograr el mayor uptime.

Para cambios críticos, se considera **Blue/Green Deployment**, habilitando la creación de entornos paralelos y la conmutación controlada mediante AWS Application Load Balancer (ALB), permitiendo validación previa antes de redirigir tráfico.

Ambas estrategias se respaldan con **Readiness Probes**, que aseguran que solo los pods listos reciban tráfico, y con mecanismos de rollback automático en caso de fallos.

El **monitoreo del despliegue** se realiza con **Amazon CloudWatch**, centralizando métricas y logs para una supervisión continua y efectiva del estado del sistema.

#### Flujo CD

```mermaid 
flowchart TD
  A[CodePipeline: Build y Push Imagen Docker a JFrog Artifactory] --> B[CodePipeline: Actualizar Deployment en EKS]

  B --> C[Kubernetes: Rolling Update]
  C --> D[Reemplazo progresivo de Pods - uno a uno]
  D --> E{Pods Pasan Readiness Probes?}

  E -- Sí --> F[Tráfico dirigido a nuevos Pods]
  E -- No --> G[Kubernetes: Rollback automático a version anterior]

  F --> H[Monitoreo continuo con Amazon CloudWatch]

  %% Alternativa para cambios críticos
  B --> I[CodePipeline: Blue/Green Deployment]
  I --> J[Desplegar nuevo entorno paralelo]
  J --> K[Pruebas y validacion]
  K --> L{Validacion exitosa?}
  L -- Sí --> M[Switch ALB a nuevo entorno]
  L -- No --> N[Rollback y eliminar nuevo entorno]

```

## Plan de Pruebas y Calidad:
- QA Automatizado: Usamos Cypress, para frontend web, en lugar de Selenium por su integración nativa y rapidez, y Appium para pruebas móviles Flutter.

- QA Manual (basado en ISTQB): Incluye pruebas exploratorias, basadas en casos, usabilidad, regresión y ad-hoc para cubrir escenarios complejos y experiencia de usuario no automatizable, con trazabilidad en Jira.

- Unitarias: Estas pruebas automatizadas se ejecutan en batch tras cada build dentro del pipeline CI.


## Desarrollo Frontend y Mobile:

### Integración con Microservicios 

- Angular y Flutter consumirán APIs RESTful de los microservicios backend usando JSON y autenticación con tokens JWT.

- Se implementará desarrollo por contrato usando OpenAPI (Swagger) para definir los contratos API antes del desarrollo, asegurando alineación entre backend y frontend. Además, se usarán mock APIs generadas a partir de OpenAPI para que frontend pueda desarrollar y probar en paralelo con backend.

### Coherencia de interfaces

- Para asegurar coherencia visual y de experiencia, se definirá un design system común usando Figma, que documente colores, tipografías, iconos y patrones UX.


## Implementación del BPMN Engine:
- 	Seleccionar un BPMN engine (por ejemplo, Camunda o Activiti) e integrarlo en la arquitectura.
- 	Diseñar un proceso de negocio clave usando BPMN y describir cómo se implementará y ejecutará.
- 	Explicar cómo se monitorizarán y optimizarán los procesos de negocio.

- Se selecciona Camunda por que es una solucion escalable y robusta, ademas, ofrece interfaz grafica para diseño y monitoreo de flujos, integracion con microservicios, integración con AWS SQS .

- El siguiente flujo tiene los siguientes pasos: 

    - Orquesta los microservicios: Validación, Cobro y Envío.
    - Usa un Event en la tarea de cobro para simular fallo (p.ej., timeout pago).
    - En caso de fallo, ejecuta un rollback para cancelar el pedido.
    - El rollback se puede implementar como llamada a un microservicio compensatorio que revierta acciones previas.

![Mi Imagen](data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz4KPCEtLSBjcmVhdGVkIHdpdGggYnBtbi1qcyAvIGh0dHA6Ly9icG1uLmlvIC0tPgo8IURPQ1RZUEUgc3ZnIFBVQkxJQyAiLS8vVzNDLy9EVEQgU1ZHIDEuMS8vRU4iICJodHRwOi8vd3d3LnczLm9yZy9HcmFwaGljcy9TVkcvMS4xL0RURC9zdmcxMS5kdGQiPgo8c3ZnIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiIHdpZHRoPSI1NjAiIGhlaWdodD0iMTgwIiB2aWV3Qm94PSIxMTYgNzUgNTYwIDE4MCIgdmVyc2lvbj0iMS4xIj48ZGVmcz48cGF0dGVybiBpZD0iZGpzLWdyaWQtcGF0dGVybi0zMzA0NTMiIHdpZHRoPSIxMCIgaGVpZ2h0PSIxMCIgcGF0dGVyblVuaXRzPSJ1c2VyU3BhY2VPblVzZSI+PGNpcmNsZSBjeD0iMC41IiBjeT0iMC41IiByPSIwLjUiIHN0eWxlPSJmaWxsOiByZ2IoMjA0LCAyMDQsIDIwNCk7Ii8+PC9wYXR0ZXJuPjwvZGVmcz48ZyBjbGFzcz0iZGpzLWdyb3VwIj48ZyBjbGFzcz0iZGpzLWVsZW1lbnQgZGpzLXNoYXBlIiBkYXRhLWVsZW1lbnQtaWQ9IlZhbGlkYXRlT3JkZXIiIHN0eWxlPSJkaXNwbGF5OiBibG9jazsiIHRyYW5zZm9ybT0ibWF0cml4KDEgMCAwIDEgMjIwIDgwKSI+PGcgY2xhc3M9ImRqcy12aXN1YWwiPjxyZWN0IHg9IjAiIHk9IjAiIHdpZHRoPSIxMDAiIGhlaWdodD0iNjAiIHJ4PSIxMCIgcnk9IjEwIiBzdHlsZT0ic3Ryb2tlLWxpbmVjYXA6IHJvdW5kOyBzdHJva2UtbGluZWpvaW46IHJvdW5kOyBzdHJva2U6IHJnYigzNCwgMzYsIDQyKTsgc3Ryb2tlLXdpZHRoOiAycHg7IGZpbGw6IHdoaXRlOyBmaWxsLW9wYWNpdHk6IDAuOTU7Ii8+PHRleHQgbGluZUhlaWdodD0iMS4yIiBjbGFzcz0iZGpzLWxhYmVsIiBzdHlsZT0iZm9udC1mYW1pbHk6IEFyaWFsLCBzYW5zLXNlcmlmOyBmb250LXNpemU6IDEycHg7IGZvbnQtd2VpZ2h0OiBub3JtYWw7IGZpbGw6IHJnYigzNCwgMzYsIDQyKTsiPjx0c3BhbiB4PSIxMS42NzY3NTc4MTI1IiB5PSIzMy42Ij5WYWxpZGF0ZSBPcmRlcjwvdHNwYW4+PC90ZXh0PjwvZz48cmVjdCBjbGFzcz0iZGpzLWhpdCBkanMtaGl0LWFsbCIgeD0iMCIgeT0iMCIgd2lkdGg9IjEwMCIgaGVpZ2h0PSI2MCIgc3R5bGU9ImZpbGw6IG5vbmU7IHN0cm9rZS1vcGFjaXR5OiAwOyBzdHJva2U6IHdoaXRlOyBzdHJva2Utd2lkdGg6IDE1cHg7Ii8+PHJlY3QgeD0iLTUiIHk9Ii01IiByeD0iMTQiIHdpZHRoPSIxMTAiIGhlaWdodD0iNzAiIGNsYXNzPSJkanMtb3V0bGluZSIgc3R5bGU9ImZpbGw6IG5vbmU7Ii8+PC9nPjwvZz48ZyBjbGFzcz0iZGpzLWdyb3VwIj48ZyBjbGFzcz0iZGpzLWVsZW1lbnQgZGpzLXNoYXBlIiBkYXRhLWVsZW1lbnQtaWQ9IkNoYXJnZVBheW1lbnQiIHN0eWxlPSJkaXNwbGF5OiBibG9jazsiIHRyYW5zZm9ybT0ibWF0cml4KDEgMCAwIDEgMzUwIDgwKSI+PGcgY2xhc3M9ImRqcy12aXN1YWwiPjxyZWN0IHg9IjAiIHk9IjAiIHdpZHRoPSIxMDAiIGhlaWdodD0iNjAiIHJ4PSIxMCIgcnk9IjEwIiBzdHlsZT0ic3Ryb2tlLWxpbmVjYXA6IHJvdW5kOyBzdHJva2UtbGluZWpvaW46IHJvdW5kOyBzdHJva2U6IHJnYigzNCwgMzYsIDQyKTsgc3Ryb2tlLXdpZHRoOiAycHg7IGZpbGw6IHdoaXRlOyBmaWxsLW9wYWNpdHk6IDAuOTU7Ii8+PHRleHQgbGluZUhlaWdodD0iMS4yIiBjbGFzcz0iZGpzLWxhYmVsIiBzdHlsZT0iZm9udC1mYW1pbHk6IEFyaWFsLCBzYW5zLXNlcmlmOyBmb250LXNpemU6IDEycHg7IGZvbnQtd2VpZ2h0OiBub3JtYWw7IGZpbGw6IHJnYigzNCwgMzYsIDQyKTsiPjx0c3BhbiB4PSIzMC4zMjAzMTI1IiB5PSIyNi40Ij5DaGFyZ2UgPC90c3Bhbj48dHNwYW4geD0iMjYuMzIwMzEyNSIgeT0iNDAuOCI+UGF5bWVudDwvdHNwYW4+PC90ZXh0PjwvZz48cmVjdCBjbGFzcz0iZGpzLWhpdCBkanMtaGl0LWFsbCIgeD0iMCIgeT0iMCIgd2lkdGg9IjEwMCIgaGVpZ2h0PSI2MCIgc3R5bGU9ImZpbGw6IG5vbmU7IHN0cm9rZS1vcGFjaXR5OiAwOyBzdHJva2U6IHdoaXRlOyBzdHJva2Utd2lkdGg6IDE1cHg7Ii8+PHJlY3QgeD0iLTUiIHk9Ii01IiByeD0iMTQiIHdpZHRoPSIxMTAiIGhlaWdodD0iNzAiIGNsYXNzPSJkanMtb3V0bGluZSIgc3R5bGU9ImZpbGw6IG5vbmU7Ii8+PC9nPjwvZz48ZyBjbGFzcz0iZGpzLWdyb3VwIj48ZyBjbGFzcz0iZGpzLWVsZW1lbnQgZGpzLXNoYXBlIiBkYXRhLWVsZW1lbnQtaWQ9IlNoaXBPcmRlciIgc3R5bGU9ImRpc3BsYXk6IGJsb2NrOyIgdHJhbnNmb3JtPSJtYXRyaXgoMSAwIDAgMSA0ODAgODApIj48ZyBjbGFzcz0iZGpzLXZpc3VhbCI+PHJlY3QgeD0iMCIgeT0iMCIgd2lkdGg9IjEwMCIgaGVpZ2h0PSI2MCIgcng9IjEwIiByeT0iMTAiIHN0eWxlPSJzdHJva2UtbGluZWNhcDogcm91bmQ7IHN0cm9rZS1saW5lam9pbjogcm91bmQ7IHN0cm9rZTogcmdiKDM0LCAzNiwgNDIpOyBzdHJva2Utd2lkdGg6IDJweDsgZmlsbDogd2hpdGU7IGZpbGwtb3BhY2l0eTogMC45NTsiLz48dGV4dCBsaW5lSGVpZ2h0PSIxLjIiIGNsYXNzPSJkanMtbGFiZWwiIHN0eWxlPSJmb250LWZhbWlseTogQXJpYWwsIHNhbnMtc2VyaWY7IGZvbnQtc2l6ZTogMTJweDsgZm9udC13ZWlnaHQ6IG5vcm1hbDsgZmlsbDogcmdiKDM0LCAzNiwgNDIpOyI+PHRzcGFuIHg9IjIwLjkwNTI3MzQzNzUiIHk9IjMzLjYiPlNoaXAgT3JkZXI8L3RzcGFuPjwvdGV4dD48L2c+PHJlY3QgY2xhc3M9ImRqcy1oaXQgZGpzLWhpdC1hbGwiIHg9IjAiIHk9IjAiIHdpZHRoPSIxMDAiIGhlaWdodD0iNjAiIHN0eWxlPSJmaWxsOiBub25lOyBzdHJva2Utb3BhY2l0eTogMDsgc3Ryb2tlOiB3aGl0ZTsgc3Ryb2tlLXdpZHRoOiAxNXB4OyIvPjxyZWN0IHg9Ii01IiB5PSItNSIgcng9IjE0IiB3aWR0aD0iMTEwIiBoZWlnaHQ9IjcwIiBjbGFzcz0iZGpzLW91dGxpbmUiIHN0eWxlPSJmaWxsOiBub25lOyIvPjwvZz48L2c+PGcgY2xhc3M9ImRqcy1ncm91cCI+PGcgY2xhc3M9ImRqcy1lbGVtZW50IGRqcy1zaGFwZSIgZGF0YS1lbGVtZW50LWlkPSJFbmRFdmVudCIgc3R5bGU9ImRpc3BsYXk6IGJsb2NrOyIgdHJhbnNmb3JtPSJtYXRyaXgoMSAwIDAgMSA2MTAgMTAwKSI+PGcgY2xhc3M9ImRqcy12aXN1YWwiPjxjaXJjbGUgY3g9IjE4IiBjeT0iMTgiIHI9IjE4IiBzdHlsZT0ic3Ryb2tlLWxpbmVjYXA6IHJvdW5kOyBzdHJva2UtbGluZWpvaW46IHJvdW5kOyBzdHJva2U6IHJnYigzNCwgMzYsIDQyKTsgc3Ryb2tlLXdpZHRoOiA0cHg7IGZpbGw6IHdoaXRlOyBmaWxsLW9wYWNpdHk6IDAuOTU7Ii8+PC9nPjxyZWN0IGNsYXNzPSJkanMtaGl0IGRqcy1oaXQtYWxsIiB4PSIwIiB5PSIwIiB3aWR0aD0iMzYiIGhlaWdodD0iMzYiIHN0eWxlPSJmaWxsOiBub25lOyBzdHJva2Utb3BhY2l0eTogMDsgc3Ryb2tlOiB3aGl0ZTsgc3Ryb2tlLXdpZHRoOiAxNXB4OyIvPjxjaXJjbGUgY3g9IjE4IiBjeT0iMTgiIHI9IjI0IiBjbGFzcz0iZGpzLW91dGxpbmUiIHN0eWxlPSJmaWxsOiBub25lOyIvPjwvZz48L2c+PGcgY2xhc3M9ImRqcy1ncm91cCI+PGcgY2xhc3M9ImRqcy1lbGVtZW50IGRqcy1zaGFwZSIgZGF0YS1lbGVtZW50LWlkPSJFbmRFdmVudF9sYWJlbCIgc3R5bGU9ImRpc3BsYXk6IGJsb2NrOyIgdHJhbnNmb3JtPSJtYXRyaXgoMSAwIDAgMSA1ODYgMTM2KSI+PGcgY2xhc3M9ImRqcy12aXN1YWwiPjx0ZXh0IGxpbmVIZWlnaHQ9IjEuMiIgY2xhc3M9ImRqcy1sYWJlbCIgc3R5bGU9ImZvbnQtZmFtaWx5OiBBcmlhbCwgc2Fucy1zZXJpZjsgZm9udC1zaXplOiAxMXB4OyBmb250LXdlaWdodDogbm9ybWFsOyBmaWxsOiByZ2IoMzQsIDM2LCA0Mik7Ij48dHNwYW4geD0iMCIgeT0iOS44OTk5OTk5OTk5OTk5OTkiPk9yZGVyIENvbXBsZXRlZDwvdHNwYW4+PC90ZXh0PjwvZz48cmVjdCBjbGFzcz0iZGpzLWhpdCBkanMtaGl0LWFsbCIgeD0iMCIgeT0iMCIgd2lkdGg9Ijg1IiBoZWlnaHQ9IjE0IiBzdHlsZT0iZmlsbDogbm9uZTsgc3Ryb2tlLW9wYWNpdHk6IDA7IHN0cm9rZTogd2hpdGU7IHN0cm9rZS13aWR0aDogMTVweDsiLz48cmVjdCB4PSItNSIgeT0iLTUiIHJ4PSI0IiB3aWR0aD0iOTUiIGhlaWdodD0iMjQiIGNsYXNzPSJkanMtb3V0bGluZSIgc3R5bGU9ImZpbGw6IG5vbmU7Ii8+PC9nPjwvZz48ZyBjbGFzcz0iZGpzLWdyb3VwIj48ZyBjbGFzcz0iZGpzLWVsZW1lbnQgZGpzLXNoYXBlIiBkYXRhLWVsZW1lbnQtaWQ9IlJvbGxiYWNrT3JkZXIiIHN0eWxlPSJkaXNwbGF5OiBibG9jazsiIHRyYW5zZm9ybT0ibWF0cml4KDEgMCAwIDEgNDIwIDE5MCkiPjxnIGNsYXNzPSJkanMtdmlzdWFsIj48cmVjdCB4PSIwIiB5PSIwIiB3aWR0aD0iMTIwIiBoZWlnaHQ9IjYwIiByeD0iMTAiIHJ5PSIxMCIgc3R5bGU9InN0cm9rZS1saW5lY2FwOiByb3VuZDsgc3Ryb2tlLWxpbmVqb2luOiByb3VuZDsgc3Ryb2tlOiByZ2IoMzQsIDM2LCA0Mik7IHN0cm9rZS13aWR0aDogMnB4OyBmaWxsOiB3aGl0ZTsgZmlsbC1vcGFjaXR5OiAwLjk1OyIvPjx0ZXh0IGxpbmVIZWlnaHQ9IjEuMiIgY2xhc3M9ImRqcy1sYWJlbCIgc3R5bGU9ImZvbnQtZmFtaWx5OiBBcmlhbCwgc2Fucy1zZXJpZjsgZm9udC1zaXplOiAxMnB4OyBmb250LXdlaWdodDogbm9ybWFsOyBmaWxsOiByZ2IoMzQsIDM2LCA0Mik7Ij48dHNwYW4geD0iMTkuOTA0Mjk2ODc1IiB5PSIzMy42Ij5Sb2xsYmFjayBPcmRlcjwvdHNwYW4+PC90ZXh0PjwvZz48cmVjdCBjbGFzcz0iZGpzLWhpdCBkanMtaGl0LWFsbCIgeD0iMCIgeT0iMCIgd2lkdGg9IjEyMCIgaGVpZ2h0PSI2MCIgc3R5bGU9ImZpbGw6IG5vbmU7IHN0cm9rZS1vcGFjaXR5OiAwOyBzdHJva2U6IHdoaXRlOyBzdHJva2Utd2lkdGg6IDE1cHg7Ii8+PHJlY3QgeD0iLTUiIHk9Ii01IiByeD0iMTQiIHdpZHRoPSIxMzAiIGhlaWdodD0iNzAiIGNsYXNzPSJkanMtb3V0bGluZSIgc3R5bGU9ImZpbGw6IG5vbmU7Ii8+PC9nPjwvZz48ZyBjbGFzcz0iZGpzLWdyb3VwIj48ZyBjbGFzcz0iZGpzLWVsZW1lbnQgZGpzLXNoYXBlIiBkYXRhLWVsZW1lbnQtaWQ9IlJvbGxiYWNrRW5kRXZlbnQiIHN0eWxlPSJkaXNwbGF5OiBibG9jazsiIHRyYW5zZm9ybT0ibWF0cml4KDEgMCAwIDEgNTgyIDIwMikiPjxnIGNsYXNzPSJkanMtdmlzdWFsIj48Y2lyY2xlIGN4PSIxOCIgY3k9IjE4IiByPSIxOCIgc3R5bGU9InN0cm9rZS1saW5lY2FwOiByb3VuZDsgc3Ryb2tlLWxpbmVqb2luOiByb3VuZDsgc3Ryb2tlOiByZ2IoMzQsIDM2LCA0Mik7IHN0cm9rZS13aWR0aDogNHB4OyBmaWxsOiB3aGl0ZTsgZmlsbC1vcGFjaXR5OiAwLjk1OyIvPjwvZz48cmVjdCBjbGFzcz0iZGpzLWhpdCBkanMtaGl0LWFsbCIgeD0iMCIgeT0iMCIgd2lkdGg9IjM2IiBoZWlnaHQ9IjM2IiBzdHlsZT0iZmlsbDogbm9uZTsgc3Ryb2tlLW9wYWNpdHk6IDA7IHN0cm9rZTogd2hpdGU7IHN0cm9rZS13aWR0aDogMTVweDsiLz48Y2lyY2xlIGN4PSIxOCIgY3k9IjE4IiByPSIyNCIgY2xhc3M9ImRqcy1vdXRsaW5lIiBzdHlsZT0iZmlsbDogbm9uZTsiLz48L2c+PC9nPjxnIGNsYXNzPSJkanMtZ3JvdXAiPjxnIGNsYXNzPSJkanMtZWxlbWVudCBkanMtc2hhcGUiIGRhdGEtZWxlbWVudC1pZD0iU3RhcnRFdmVudCIgc3R5bGU9ImRpc3BsYXk6IGJsb2NrOyIgdHJhbnNmb3JtPSJtYXRyaXgoMSAwIDAgMSAxNDIgOTIpIj48ZyBjbGFzcz0iZGpzLXZpc3VhbCI+PGNpcmNsZSBjeD0iMTgiIGN5PSIxOCIgcj0iMTgiIHN0eWxlPSJzdHJva2UtbGluZWNhcDogcm91bmQ7IHN0cm9rZS1saW5lam9pbjogcm91bmQ7IHN0cm9rZTogcmdiKDM0LCAzNiwgNDIpOyBzdHJva2Utd2lkdGg6IDJweDsgZmlsbDogd2hpdGU7IGZpbGwtb3BhY2l0eTogMC45NTsiLz48L2c+PHJlY3QgY2xhc3M9ImRqcy1oaXQgZGpzLWhpdC1hbGwiIHg9IjAiIHk9IjAiIHdpZHRoPSIzNiIgaGVpZ2h0PSIzNiIgc3R5bGU9ImZpbGw6IG5vbmU7IHN0cm9rZS1vcGFjaXR5OiAwOyBzdHJva2U6IHdoaXRlOyBzdHJva2Utd2lkdGg6IDE1cHg7Ii8+PGNpcmNsZSBjeD0iMTgiIGN5PSIxOCIgcj0iMjMiIGNsYXNzPSJkanMtb3V0bGluZSIgc3R5bGU9ImZpbGw6IG5vbmU7Ii8+PC9nPjwvZz48ZyBjbGFzcz0iZGpzLWdyb3VwIj48ZyBjbGFzcz0iZGpzLWVsZW1lbnQgZGpzLXNoYXBlIiBkYXRhLWVsZW1lbnQtaWQ9IkV2ZW50XzBsOGNla2MiIHN0eWxlPSJkaXNwbGF5OiBibG9jazsiIHRyYW5zZm9ybT0ibWF0cml4KDEgMCAwIDEgMzYyIDEyMikiPjxnIGNsYXNzPSJkanMtdmlzdWFsIj48Y2lyY2xlIGN4PSIxOCIgY3k9IjE4IiByPSIxOCIgc3R5bGU9InN0cm9rZS1saW5lY2FwOiByb3VuZDsgc3Ryb2tlLWxpbmVqb2luOiByb3VuZDsgc3Ryb2tlOiByZ2IoMzQsIDM2LCA0Mik7IHN0cm9rZS13aWR0aDogMS41cHg7IGZpbGw6IHdoaXRlOyBmaWxsLW9wYWNpdHk6IDE7Ii8+PGNpcmNsZSBjeD0iMTgiIGN5PSIxOCIgcj0iMTUiIHN0eWxlPSJzdHJva2UtbGluZWNhcDogcm91bmQ7IHN0cm9rZS1saW5lam9pbjogcm91bmQ7IHN0cm9rZTogcmdiKDM0LCAzNiwgNDIpOyBzdHJva2Utd2lkdGg6IDEuNXB4OyBmaWxsOiBub25lOyBmaWxsLW9wYWNpdHk6IDE7Ii8+PHBhdGggZD0ibSA3LjIsMjUuOTkxOTk5OTk5OTk5OTk3IDAuMDkzNTAwMDAwMDAwMDAwMDEsLTAuMDI1MzAwMDAwMDAwMDAwMDAzIDcuMzM5MiwtOS42MTA3MDAwMDAwMDAwMDEgNy42NjcwMDAwMDAwMDAwMDEsOC45NjYxIDQuNzAwMywtMTguMjIwNCAtNS44NzA3LDExLjY1MDEgLTcuMjk5NjAwMDAwMDAwMDAxLC05LjU4NTQwMDAwMDAwMDAwMiB6IiBzdHlsZT0iZmlsbDogd2hpdGU7IHN0cm9rZS1saW5lY2FwOiByb3VuZDsgc3Ryb2tlLWxpbmVqb2luOiByb3VuZDsgc3Ryb2tlOiByZ2IoMzQsIDM2LCA0Mik7IHN0cm9rZS13aWR0aDogMXB4OyIvPjwvZz48cmVjdCBjbGFzcz0iZGpzLWhpdCBkanMtaGl0LWFsbCIgeD0iMCIgeT0iMCIgd2lkdGg9IjM2IiBoZWlnaHQ9IjM2IiBzdHlsZT0iZmlsbDogbm9uZTsgc3Ryb2tlLW9wYWNpdHk6IDA7IHN0cm9rZTogd2hpdGU7IHN0cm9rZS13aWR0aDogMTVweDsiLz48Y2lyY2xlIGN4PSIxOCIgY3k9IjE4IiByPSIyMyIgY2xhc3M9ImRqcy1vdXRsaW5lIiBzdHlsZT0iZmlsbDogbm9uZTsiLz48L2c+PC9nPjxnIGNsYXNzPSJkanMtZ3JvdXAiPjxnIGNsYXNzPSJkanMtZWxlbWVudCBkanMtY29ubmVjdGlvbiIgZGF0YS1lbGVtZW50LWlkPSJGbG93MSIgc3R5bGU9ImRpc3BsYXk6IGJsb2NrOyI+PGcgY2xhc3M9ImRqcy12aXN1YWwiPjxkZWZzPjxtYXJrZXIgaWQ9Im1hcmtlci03d204a2pybGRzaXAxaHNxaTdjcnY2c2VoIiB2aWV3Qm94PSIwIDAgMjAgMjAiIHJlZlg9IjExIiByZWZZPSIxMCIgbWFya2VyV2lkdGg9IjEwIiBtYXJrZXJIZWlnaHQ9IjEwIiBvcmllbnQ9ImF1dG8iPjxwYXRoIGQ9Ik0gMSA1IEwgMTEgMTAgTCAxIDE1IFoiIHN0eWxlPSJzdHJva2UtbGluZWNhcDogcm91bmQ7IHN0cm9rZS1saW5lam9pbjogcm91bmQ7IHN0cm9rZTogcmdiKDM0LCAzNiwgNDIpOyBzdHJva2Utd2lkdGg6IDFweDsgZmlsbDogcmdiKDM0LCAzNiwgNDIpOyIvPjwvbWFya2VyPjwvZGVmcz48cGF0aCBkYXRhLWNvcm5lci1yYWRpdXM9IjUiIHN0eWxlPSJmaWxsOiBub25lOyBzdHJva2UtbGluZWNhcDogcm91bmQ7IHN0cm9rZS1saW5lam9pbjogcm91bmQ7IHN0cm9rZTogcmdiKDM0LCAzNiwgNDIpOyBzdHJva2Utd2lkdGg6IDJweDsgbWFya2VyLWVuZDogdXJsKCcjbWFya2VyLTd3bThranJsZHNpcDFoc3FpN2NydjZzZWgnKTsiIGQ9Ik0xNzgsMTE0TDE5NCwxMTRDMTk2LjUsMTE0LDE5OSwxMTEuNSwxOTksMTA5TDE5OSwxMDNDMTk5LDEwMC41LDIwMS41LDk4LDIwNCw5OEwyMjAsOTgiLz48L2c+PHBhdGggZD0iTTE3OCwxMTRMMTk5LDExNEwxOTksOThMMjIwLDk4IiBjbGFzcz0iZGpzLWhpdCBkanMtaGl0LXN0cm9rZSIgc3R5bGU9ImZpbGw6IG5vbmU7IHN0cm9rZS1vcGFjaXR5OiAwOyBzdHJva2U6IHdoaXRlOyBzdHJva2Utd2lkdGg6IDE1cHg7Ii8+PHJlY3QgeD0iMTczIiB5PSI5MyIgcng9IjQiIHdpZHRoPSI1MiIgaGVpZ2h0PSIyNiIgY2xhc3M9ImRqcy1vdXRsaW5lIiBzdHlsZT0iZmlsbDogbm9uZTsiLz48L2c+PC9nPjxnIGNsYXNzPSJkanMtZ3JvdXAiPjxnIGNsYXNzPSJkanMtZWxlbWVudCBkanMtY29ubmVjdGlvbiIgZGF0YS1lbGVtZW50LWlkPSJGbG93MiIgc3R5bGU9ImRpc3BsYXk6IGJsb2NrOyI+PGcgY2xhc3M9ImRqcy12aXN1YWwiPjxkZWZzPjxtYXJrZXIgaWQ9Im1hcmtlci0xY3pwOTFmbjkxYjc4dnZ5dnA0NWZwYnJ3IiB2aWV3Qm94PSIwIDAgMjAgMjAiIHJlZlg9IjExIiByZWZZPSIxMCIgbWFya2VyV2lkdGg9IjEwIiBtYXJrZXJIZWlnaHQ9IjEwIiBvcmllbnQ9ImF1dG8iPjxwYXRoIGQ9Ik0gMSA1IEwgMTEgMTAgTCAxIDE1IFoiIHN0eWxlPSJzdHJva2UtbGluZWNhcDogcm91bmQ7IHN0cm9rZS1saW5lam9pbjogcm91bmQ7IHN0cm9rZTogcmdiKDM0LCAzNiwgNDIpOyBzdHJva2Utd2lkdGg6IDFweDsgZmlsbDogcmdiKDM0LCAzNiwgNDIpOyIvPjwvbWFya2VyPjwvZGVmcz48cGF0aCBkYXRhLWNvcm5lci1yYWRpdXM9IjUiIHN0eWxlPSJmaWxsOiBub25lOyBzdHJva2UtbGluZWNhcDogcm91bmQ7IHN0cm9rZS1saW5lam9pbjogcm91bmQ7IHN0cm9rZTogcmdiKDM0LCAzNiwgNDIpOyBzdHJva2Utd2lkdGg6IDJweDsgbWFya2VyLWVuZDogdXJsKCcjbWFya2VyLTFjenA5MWZuOTFiNzh2dnl2cDQ1ZnBicncnKTsiIGQ9Ik0zMjAsMTEwTDM1MCwxMTAiLz48L2c+PHBhdGggZD0iTTMyMCwxMTBMMzUwLDExMCIgY2xhc3M9ImRqcy1oaXQgZGpzLWhpdC1zdHJva2UiIHN0eWxlPSJmaWxsOiBub25lOyBzdHJva2Utb3BhY2l0eTogMDsgc3Ryb2tlOiB3aGl0ZTsgc3Ryb2tlLXdpZHRoOiAxNXB4OyIvPjxyZWN0IHg9IjMxNSIgeT0iMTA1IiByeD0iNCIgd2lkdGg9IjQwIiBoZWlnaHQ9IjEwIiBjbGFzcz0iZGpzLW91dGxpbmUiIHN0eWxlPSJmaWxsOiBub25lOyIvPjwvZz48L2c+PGcgY2xhc3M9ImRqcy1ncm91cCI+PGcgY2xhc3M9ImRqcy1lbGVtZW50IGRqcy1jb25uZWN0aW9uIiBkYXRhLWVsZW1lbnQtaWQ9IkZsb3czIiBzdHlsZT0iZGlzcGxheTogYmxvY2s7Ij48ZyBjbGFzcz0iZGpzLXZpc3VhbCI+PGRlZnM+PG1hcmtlciBpZD0ibWFya2VyLTkzYTdycDE2bTBlOGljMGp0a2Nhcjd4YmkiIHZpZXdCb3g9IjAgMCAyMCAyMCIgcmVmWD0iMTEiIHJlZlk9IjEwIiBtYXJrZXJXaWR0aD0iMTAiIG1hcmtlckhlaWdodD0iMTAiIG9yaWVudD0iYXV0byI+PHBhdGggZD0iTSAxIDUgTCAxMSAxMCBMIDEgMTUgWiIgc3R5bGU9InN0cm9rZS1saW5lY2FwOiByb3VuZDsgc3Ryb2tlLWxpbmVqb2luOiByb3VuZDsgc3Ryb2tlOiByZ2IoMzQsIDM2LCA0Mik7IHN0cm9rZS13aWR0aDogMXB4OyBmaWxsOiByZ2IoMzQsIDM2LCA0Mik7Ii8+PC9tYXJrZXI+PC9kZWZzPjxwYXRoIGRhdGEtY29ybmVyLXJhZGl1cz0iNSIgc3R5bGU9ImZpbGw6IG5vbmU7IHN0cm9rZS1saW5lY2FwOiByb3VuZDsgc3Ryb2tlLWxpbmVqb2luOiByb3VuZDsgc3Ryb2tlOiByZ2IoMzQsIDM2LCA0Mik7IHN0cm9rZS13aWR0aDogMnB4OyBtYXJrZXItZW5kOiB1cmwoJyNtYXJrZXItOTNhN3JwMTZtMGU4aWMwanRrY2FyN3hiaScpOyIgZD0iTTQ1MCwxMTBMNDgwLDExMCIvPjwvZz48cGF0aCBkPSJNNDUwLDExMEw0ODAsMTEwIiBjbGFzcz0iZGpzLWhpdCBkanMtaGl0LXN0cm9rZSIgc3R5bGU9ImZpbGw6IG5vbmU7IHN0cm9rZS1vcGFjaXR5OiAwOyBzdHJva2U6IHdoaXRlOyBzdHJva2Utd2lkdGg6IDE1cHg7Ii8+PHJlY3QgeD0iNDQ1IiB5PSIxMDUiIHJ4PSI0IiB3aWR0aD0iNDAiIGhlaWdodD0iMTAiIGNsYXNzPSJkanMtb3V0bGluZSIgc3R5bGU9ImZpbGw6IG5vbmU7Ii8+PC9nPjwvZz48ZyBjbGFzcz0iZGpzLWdyb3VwIj48ZyBjbGFzcz0iZGpzLWVsZW1lbnQgZGpzLWNvbm5lY3Rpb24iIGRhdGEtZWxlbWVudC1pZD0iRmxvdzQiIHN0eWxlPSJkaXNwbGF5OiBibG9jazsiPjxnIGNsYXNzPSJkanMtdmlzdWFsIj48ZGVmcz48bWFya2VyIGlkPSJtYXJrZXItMXl4aHVybGlwZ3ByMnVlMDl1eGRrZ3I0YiIgdmlld0JveD0iMCAwIDIwIDIwIiByZWZYPSIxMSIgcmVmWT0iMTAiIG1hcmtlcldpZHRoPSIxMCIgbWFya2VySGVpZ2h0PSIxMCIgb3JpZW50PSJhdXRvIj48cGF0aCBkPSJNIDEgNSBMIDExIDEwIEwgMSAxNSBaIiBzdHlsZT0ic3Ryb2tlLWxpbmVjYXA6IHJvdW5kOyBzdHJva2UtbGluZWpvaW46IHJvdW5kOyBzdHJva2U6IHJnYigzNCwgMzYsIDQyKTsgc3Ryb2tlLXdpZHRoOiAxcHg7IGZpbGw6IHJnYigzNCwgMzYsIDQyKTsiLz48L21hcmtlcj48L2RlZnM+PHBhdGggZGF0YS1jb3JuZXItcmFkaXVzPSI1IiBzdHlsZT0iZmlsbDogbm9uZTsgc3Ryb2tlLWxpbmVjYXA6IHJvdW5kOyBzdHJva2UtbGluZWpvaW46IHJvdW5kOyBzdHJva2U6IHJnYigzNCwgMzYsIDQyKTsgc3Ryb2tlLXdpZHRoOiAycHg7IG1hcmtlci1lbmQ6IHVybCgnI21hcmtlci0xeXhodXJsaXBncHIydWUwOXV4ZGtncjRiJyk7IiBkPSJNNTgwLDExOEw2MTAsMTE4Ii8+PC9nPjxwYXRoIGQ9Ik01ODAsMTE4TDYxMCwxMTgiIGNsYXNzPSJkanMtaGl0IGRqcy1oaXQtc3Ryb2tlIiBzdHlsZT0iZmlsbDogbm9uZTsgc3Ryb2tlLW9wYWNpdHk6IDA7IHN0cm9rZTogd2hpdGU7IHN0cm9rZS13aWR0aDogMTVweDsiLz48cmVjdCB4PSI1NzUiIHk9IjExMyIgcng9IjQiIHdpZHRoPSI0MCIgaGVpZ2h0PSIxMCIgY2xhc3M9ImRqcy1vdXRsaW5lIiBzdHlsZT0iZmlsbDogbm9uZTsiLz48L2c+PC9nPjxnIGNsYXNzPSJkanMtZ3JvdXAiPjxnIGNsYXNzPSJkanMtZWxlbWVudCBkanMtY29ubmVjdGlvbiIgZGF0YS1lbGVtZW50LWlkPSJGbG93Um9sbGJhY2tFbmQiIHN0eWxlPSJkaXNwbGF5OiBibG9jazsiPjxnIGNsYXNzPSJkanMtdmlzdWFsIj48ZGVmcz48bWFya2VyIGlkPSJtYXJrZXItZXA4bW9oMm12YzRkMWsyYWJ6ZTZ6emowdiIgdmlld0JveD0iMCAwIDIwIDIwIiByZWZYPSIxMSIgcmVmWT0iMTAiIG1hcmtlcldpZHRoPSIxMCIgbWFya2VySGVpZ2h0PSIxMCIgb3JpZW50PSJhdXRvIj48cGF0aCBkPSJNIDEgNSBMIDExIDEwIEwgMSAxNSBaIiBzdHlsZT0ic3Ryb2tlLWxpbmVjYXA6IHJvdW5kOyBzdHJva2UtbGluZWpvaW46IHJvdW5kOyBzdHJva2U6IHJnYigzNCwgMzYsIDQyKTsgc3Ryb2tlLXdpZHRoOiAxcHg7IGZpbGw6IHJnYigzNCwgMzYsIDQyKTsiLz48L21hcmtlcj48L2RlZnM+PHBhdGggZGF0YS1jb3JuZXItcmFkaXVzPSI1IiBzdHlsZT0iZmlsbDogbm9uZTsgc3Ryb2tlLWxpbmVjYXA6IHJvdW5kOyBzdHJva2UtbGluZWpvaW46IHJvdW5kOyBzdHJva2U6IHJnYigzNCwgMzYsIDQyKTsgc3Ryb2tlLXdpZHRoOiAycHg7IG1hcmtlci1lbmQ6IHVybCgnI21hcmtlci1lcDhtb2gybXZjNGQxazJhYnplNnp6ajB2Jyk7IiBkPSJNNTQwLDIyMEw1ODIsMjIwIi8+PC9nPjxwYXRoIGQ9Ik01NDAsMjIwTDU4MiwyMjAiIGNsYXNzPSJkanMtaGl0IGRqcy1oaXQtc3Ryb2tlIiBzdHlsZT0iZmlsbDogbm9uZTsgc3Ryb2tlLW9wYWNpdHk6IDA7IHN0cm9rZTogd2hpdGU7IHN0cm9rZS13aWR0aDogMTVweDsiLz48cmVjdCB4PSI1MzUiIHk9IjIxNSIgcng9IjQiIHdpZHRoPSI1MiIgaGVpZ2h0PSIxMCIgY2xhc3M9ImRqcy1vdXRsaW5lIiBzdHlsZT0iZmlsbDogbm9uZTsiLz48L2c+PC9nPjxnIGNsYXNzPSJkanMtZ3JvdXAiPjxnIGNsYXNzPSJkanMtZWxlbWVudCBkanMtY29ubmVjdGlvbiIgZGF0YS1lbGVtZW50LWlkPSJGbG93XzB0cWU1NTYiIHN0eWxlPSJkaXNwbGF5OiBibG9jazsiPjxnIGNsYXNzPSJkanMtdmlzdWFsIj48ZGVmcz48bWFya2VyIGlkPSJtYXJrZXItMHljaTVkdzRmMnlzdGVxdmZhNTZzOTh0MyIgdmlld0JveD0iMCAwIDIwIDIwIiByZWZYPSIxMSIgcmVmWT0iMTAiIG1hcmtlcldpZHRoPSIxMCIgbWFya2VySGVpZ2h0PSIxMCIgb3JpZW50PSJhdXRvIj48cGF0aCBkPSJNIDEgNSBMIDExIDEwIEwgMSAxNSBaIiBzdHlsZT0ic3Ryb2tlLWxpbmVjYXA6IHJvdW5kOyBzdHJva2UtbGluZWpvaW46IHJvdW5kOyBzdHJva2U6IHJnYigzNCwgMzYsIDQyKTsgc3Ryb2tlLXdpZHRoOiAxcHg7IGZpbGw6IHJnYigzNCwgMzYsIDQyKTsiLz48L21hcmtlcj48L2RlZnM+PHBhdGggZGF0YS1jb3JuZXItcmFkaXVzPSI1IiBzdHlsZT0iZmlsbDogbm9uZTsgc3Ryb2tlLWxpbmVjYXA6IHJvdW5kOyBzdHJva2UtbGluZWpvaW46IHJvdW5kOyBzdHJva2U6IHJnYigzNCwgMzYsIDQyKTsgc3Ryb2tlLXdpZHRoOiAycHg7IG1hcmtlci1lbmQ6IHVybCgnI21hcmtlci0weWNpNWR3NGYyeXN0ZXF2ZmE1NnM5OHQzJyk7IiBkPSJNMzgwLDE1OEwzODAsMjE1QzM4MCwyMTcuNSwzODIuNSwyMjAsMzg1LDIyMEw0MjAsMjIwIi8+PC9nPjxwYXRoIGQ9Ik0zODAsMTU4TDM4MCwyMjBMNDIwLDIyMCIgY2xhc3M9ImRqcy1oaXQgZGpzLWhpdC1zdHJva2UiIHN0eWxlPSJmaWxsOiBub25lOyBzdHJva2Utb3BhY2l0eTogMDsgc3Ryb2tlOiB3aGl0ZTsgc3Ryb2tlLXdpZHRoOiAxNXB4OyIvPjxyZWN0IHg9IjM3NSIgeT0iMTUzIiByeD0iNCIgd2lkdGg9IjUwIiBoZWlnaHQ9IjcyIiBjbGFzcz0iZGpzLW91dGxpbmUiIHN0eWxlPSJmaWxsOiBub25lOyIvPjwvZz48L2c+PGcgY2xhc3M9ImRqcy1ncm91cCI+PGcgY2xhc3M9ImRqcy1lbGVtZW50IGRqcy1zaGFwZSIgZGF0YS1lbGVtZW50LWlkPSJSb2xsYmFja0VuZEV2ZW50X2xhYmVsIiBzdHlsZT0iZGlzcGxheTogYmxvY2s7IiB0cmFuc2Zvcm09Im1hdHJpeCgxIDAgMCAxIDU1OSAxNzgpIj48ZyBjbGFzcz0iZGpzLXZpc3VhbCI+PHRleHQgbGluZUhlaWdodD0iMS4yIiBjbGFzcz0iZGpzLWxhYmVsIiBzdHlsZT0iZm9udC1mYW1pbHk6IEFyaWFsLCBzYW5zLXNlcmlmOyBmb250LXNpemU6IDExcHg7IGZvbnQtd2VpZ2h0OiBub3JtYWw7IGZpbGw6IHJnYigzNCwgMzYsIDQyKTsiPjx0c3BhbiB4PSIwIiB5PSI5Ljg5OTk5OTk5OTk5OTk5OSI+T3JkZXIgQ2FuY2VsbGVkPC90c3Bhbj48L3RleHQ+PC9nPjxyZWN0IGNsYXNzPSJkanMtaGl0IGRqcy1oaXQtYWxsIiB4PSIwIiB5PSIwIiB3aWR0aD0iODEiIGhlaWdodD0iMTQiIHN0eWxlPSJmaWxsOiBub25lOyBzdHJva2Utb3BhY2l0eTogMDsgc3Ryb2tlOiB3aGl0ZTsgc3Ryb2tlLXdpZHRoOiAxNXB4OyIvPjxyZWN0IHg9Ii01IiB5PSItNSIgcng9IjQiIHdpZHRoPSI5MSIgaGVpZ2h0PSIyNCIgY2xhc3M9ImRqcy1vdXRsaW5lIiBzdHlsZT0iZmlsbDogbm9uZTsiLz48L2c+PC9nPjxnIGNsYXNzPSJkanMtZ3JvdXAiPjxnIGNsYXNzPSJkanMtZWxlbWVudCBkanMtc2hhcGUiIGRhdGEtZWxlbWVudC1pZD0iU3RhcnRFdmVudF9sYWJlbCIgc3R5bGU9ImRpc3BsYXk6IGJsb2NrOyIgdHJhbnNmb3JtPSJtYXRyaXgoMSAwIDAgMSAxMjEgMTI4KSI+PGcgY2xhc3M9ImRqcy12aXN1YWwiPjx0ZXh0IGxpbmVIZWlnaHQ9IjEuMiIgY2xhc3M9ImRqcy1sYWJlbCIgc3R5bGU9ImZvbnQtZmFtaWx5OiBBcmlhbCwgc2Fucy1zZXJpZjsgZm9udC1zaXplOiAxMXB4OyBmb250LXdlaWdodDogbm9ybWFsOyBmaWxsOiByZ2IoMzQsIDM2LCA0Mik7Ij48dHNwYW4geD0iMCIgeT0iOS44OTk5OTk5OTk5OTk5OTkiPk9yZGVyIFJlY2VpdmVkPC90c3Bhbj48L3RleHQ+PC9nPjxyZWN0IGNsYXNzPSJkanMtaGl0IGRqcy1oaXQtYWxsIiB4PSIwIiB5PSIwIiB3aWR0aD0iNzgiIGhlaWdodD0iMTQiIHN0eWxlPSJmaWxsOiBub25lOyBzdHJva2Utb3BhY2l0eTogMDsgc3Ryb2tlOiB3aGl0ZTsgc3Ryb2tlLXdpZHRoOiAxNXB4OyIvPjxyZWN0IHg9Ii01IiB5PSItNSIgcng9IjQiIHdpZHRoPSI4OCIgaGVpZ2h0PSIyNCIgY2xhc3M9ImRqcy1vdXRsaW5lIiBzdHlsZT0iZmlsbDogbm9uZTsiLz48L2c+PC9nPjwvc3ZnPg==)

- El monitoreo se integra con AWS CloudWatch para seguimiento y alertas de fallos o cuellos de botella.

## Buenas Prácticas:

Stack tecnológico:

- Backend: Java (Spring Boot, Spring Cloud, Feign, Maven, Camunda, JUnit, Mockito)
- Frontend web: Angular + TypeScript, Cypress
- Frontend móvil: Flutter
- Bases de datos: Oracle, PostgreSQL, MongoDB, Redis
- Contenedores y orquestación: Docker, AWS EKS (con gestión de secrets)
- CI/CD: AWS Pipelines, JFrog Artifactory
- Distribución: CDN
- Calidad: SonarQube (con Quality Gates), SonarLint
- Resiliencia: Resilience4j
- Monitoreo: OpenTelemetry, AWS CloudWatch
- Seguridad: OWASP, JFrog Vulnerabilities


Buenas Prácticas para el Equipo de Desarrollo

- Uso de **Scrum** para gestión ágil de proyectos, con ciclos cortos.
- Aplicar principios de **Clean Code**, **SOLID**, **KISS**, **DRY** para mantener código legible, mantenible y escalable.
- Implementación de proceso de **DevSecOpS**.
- Implementar **Trunk-Based Development** para integración temprana y continua.
- Uso de herramientas de análisis estático y control de calidad: **SonarLint**, **SonarQube**, y escaneo de vulnerabilidades con **JFrog Xray**.
- Documentación y revisión constante del código para fomentar el conocimiento compartido y la mejora continua, mediante PRs de doble revisores.
- Seguir estándares de seguridad recomendados por **OWASP** para proteger la aplicación.
