Definición de Arquitectura de Software
FACILE — Sistema de Gestión Hotelera (PMS)

1. Selección y justificación del patrón arquitectónico
FACILE implementa una arquitectura Cliente-Servidor con un backend organizado bajo el patrón de Arquitectura en Capas (Layered Architecture). Esta combinación no es arbitraria: responde directamente al problema que el sistema busca resolver.
El hotel operaba con información dispersa en múltiples herramientas sin sincronización entre sí. Para resolverlo, se necesitaba un sistema con una única fuente de verdad a la que múltiples usuarios (recepción, housekeeping, contabilidad, gerencia) pudieran acceder simultáneamente desde distintos dispositivos. El modelo cliente-servidor garantiza exactamente eso: un servidor central actúa como árbitro único de la información, mientras que los clientes solo presentan datos y capturan acciones.

1.1 Arquitectura en capas del backend
Dentro del servidor, el sistema se divide en cuatro capas con responsabilidades bien delimitadas. La regla fundamental es que cada capa solo puede comunicarse con la inmediatamente inferior, lo que evita dependencias cruzadas y hace el sistema predecible:

Capa Controller
Es el punto de entrada del sistema. Recibe cada petición HTTP proveniente del frontend, valida que tenga el formato correcto y la delega a la capa de negocio. No toma decisiones sobre el hotel; solo gestiona la comunicación. Ejemplo: ReservaController expone el endpoint POST /api/reservas y recibe los datos de una nueva reserva.
Capa Service
Aquí reside toda la inteligencia del sistema hotelero. Es la única capa que conoce las reglas del negocio: cómo calcular el precio total de una estadía (precio base + servicios consumidos + IVA 19% − descuento), qué validaciones se deben cumplir antes de confirmar un check-in, o cómo actualizar el estado de una unidad al momento del check-out. Si las reglas del hotel cambian, solo se modifica esta capa.
Capa Repository
Gestiona el acceso y la persistencia en la base de datos. Traduce las operaciones de negocio en consultas SQL contra MySQL. La capa Service no conoce nada de SQL ni de tablas; simplemente le pide al Repository "dame las reservas activas de hoy" y recibe objetos listos para usar.
Capa Entity / DTO
Define cómo se modelan y transfieren los datos. Las entities representan las tablas de la base de datos (por ejemplo, la entidad Reserva corresponde a la tabla reserva). Los DTOs (Data Transfer Objects) son objetos más simples que transfieren solo la información necesaria entre capas o hacia el frontend, sin exponer la estructura interna de la base de datos.

1.2 Justificación de la separación de responsabilidades
La razón de fondo de este patrón es que cada capa tiene una sola razón para cambiar. Si en el futuro se migra la base de datos, solo se toca el Repository. Si se agrega una nueva regla de precios, solo se modifica el Service. Si se rediseña la API, solo cambia el Controller. Ninguna modificación en una capa obliga a reescribir las demás, lo que hace el sistema mantenible, testeable y escalable de forma independiente.

2. Modelo de comunicación y seguridad
2.1 API REST desacoplada
La comunicación entre el frontend y el backend se realiza mediante una API REST desacoplada. El cliente web no accede directamente a la base de datos ni conoce la lógica interna del servidor: únicamente realiza peticiones HTTP a endpoints definidos y recibe respuestas en formato JSON. Los cuatro métodos HTTP utilizados tienen una semántica precisa:

Método	Función	Ejemplo en FACILE
GET	Consultar información	GET /api/reservas/{id}
POST	Crear un nuevo recurso	POST /api/reservas
PUT	Actualizar un recurso existente	PUT /api/reservas/{id}
DELETE	Eliminar o cancelar un recurso	DELETE /api/reservas/{id}

Este diseño desacoplado ofrece una ventaja adicional de cara al futuro: cualquier cliente nuevo (una aplicación móvil, un portal de reservas para huéspedes) podría consumir la misma API sin necesidad de modificar el backend.

2.2 Seguridad: Spring Security + JWT
La seguridad se implementa con Spring Security + JWT (JSON Web Token), adoptando un modelo stateless (sin estado). El servidor no almacena sesiones: cuando un usuario inicia sesión con su número de cédula y contraseña, recibe un token firmado que deberá incluir en cada petición posterior. El servidor valida ese token de forma independiente en cada llamada y extrae el rol del usuario para determinar qué operaciones tiene permitidas.
Este modelo es el adecuado para FACILE porque permite que múltiples instancias del servidor operen en paralelo sin compartir estado de sesión, y garantiza que cada área del hotel solo acceda a los módulos que le corresponden según su rol asignado (Administrador, Recepción, Housekeeping, Contabilidad, Gerencia).

3. Definición del stack tecnológico
La selección de tecnologías responde a dos criterios principales: dominio previo del equipo para reducir el riesgo técnico del proyecto, y compatibilidad entre las capas del sistema para garantizar una integración estable.

Capa	Tecnologías	Justificación
Frontend	HTML, CSS, Visual Studio Code	Estándares web universales con amplia compatibilidad en navegadores. VS Code ofrece un entorno de desarrollo ligero, con extensiones de productividad y soporte nativo para HTML y CSS sin requerir configuración adicional.
Backend	Java, Spring Boot, IntelliJ IDEA	Spring Boot reduce la configuración del ecosistema Java y provee de fábrica: inyección de dependencias, gestión de transacciones (@Transactional), seguridad integrada y servidor embebido. El equipo domina Java, lo que reduce el riesgo técnico del proyecto.
Base de datos	MySQL	Familiaridad del equipo con el motor, lo que permite un aprendizaje rápido y una gestión eficiente de la información. MySQL es ampliamente documentado y tiene integración directa con Spring Boot mediante JPA/Hibernate.
Infraestructura / IDE	Visual Studio Code, IntelliJ IDEA, MySQL Workbench	VS Code para desarrollo frontend, IntelliJ IDEA como IDE principal para Java y Spring Boot, y MySQL Workbench para gestión y visualización de la base de datos. Conjunto de herramientas estándar en el mercado laboral.

3.1 Herramientas complementarias
•	Control de versiones — Git: Permite al equipo de cinco integrantes trabajar en ramas paralelas sin conflictos, revertir cambios ante errores y mantener un historial completo de la evolución del proyecto.
•	Pruebas de API — Postman: Permite probar cada endpoint REST de forma aislada antes de conectarlo al frontend, verificando respuestas, códigos HTTP y comportamiento ante datos inválidos.
•	Calidad de código — ESLint + Prettier: ESLint analiza el código JavaScript del frontend en busca de errores y malas prácticas; Prettier aplica formato consistente de forma automática, reduciendo la fricción en revisiones de código entre integrantes del equipo.

3.2 Flujo de integración entre capas
El flujo de una petición completa dentro del sistema sigue el siguiente recorrido:
•	El usuario realiza una acción en el navegador (HTML + CSS). El cliente envía una petición HTTP con el token JWT incluido en el encabezado.
•	Spring Security intercepta la petición, valida el token y verifica los permisos del rol antes de permitir el acceso al Controller.
•	El Controller recibe la petición, extrae los parámetros y llama al método correspondiente en la capa Service.
•	El Service aplica las reglas de negocio, ejecuta las validaciones necesarias y delega la persistencia al Repository.
•	El Repository construye la consulta SQL, la ejecuta contra MySQL y devuelve los datos como objetos Entity.
•	El Service transforma los datos en un DTO y lo devuelve al Controller, que lo serializa en JSON y responde al cliente.

