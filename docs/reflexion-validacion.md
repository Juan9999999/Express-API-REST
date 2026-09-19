**Módulo:** Validación, middleware y manejo centralizado de errores en Express  
**Actividad:** EC1 F3 A4 API de gestión de tareas  

---

### 1. ¿Cuál es la diferencia entre un error de transporte HTTP (validado en middleware) y una regla o invariante de negocio (validada en el servicio)?
**Respuesta:**  
El error de transporte comprueba la estructura sintáctica y superficial de la petición (por ejemplo, que el `Content-Type` sea `application/json`, que el parámetro `id` sea un entero positivo o que el cuerpo contenga los tipos esperados) antes de procesar nada. Por el contrario, una regla o invariante de negocio protege la coherencia del dominio en la capa de servicio, evaluando condiciones lógicas que dependen del estado del sistema (por ejemplo, verificar si una tarea existe antes de intentar completarla o eliminarla).

---

### 2. ¿Por qué se utiliza `res.locals` para almacenar datos normalizados (como `taskId` y `taskTitle`) en lugar de mutar directamente `req.params` o `req.body`?
**Respuesta:**  
Porque `res.locals` está diseñado como el espacio estándar de Express para compartir datos procesados y limpios durante el ciclo de vida de la petición sin alterar los objetos originales. Modificar directamente `req.params` o `req.body` genera mutaciones de estado indeseadas, dificulta el rastreo de datos, rompe las expectativas de tipado de TypeScript y puede provocar efectos secundarios en otros middlewares posteriores.

---

### 3. ¿Qué ventajas ofrece centralizar la respuesta de errores en un único middleware de error frente a manejar bloques `try/catch` con `res.status().json()` en cada controlador?
**Respuesta:**  
Garantiza un contrato de respuesta JSON uniforme y consistente en toda la API (mismo esquema de campos como `error.code`, `error.message`, `error.requestId`), evita la duplicación de código en cada endpoint y previene fallos donde un error no capturado deje colgada la conexión. Además, desacopla la lógica de ejecución del formateo de errores.

---

### 4. ¿Por qué en Express 5 ya no es obligatorio envolver las funciones de los controladores en bloques `try/catch` para capturar errores sincrónicos o promesas rechazadas con `next(error)`?
**Respuesta:**  
Porque Express 5 captura internamente tanto las excepciones síncronas arrojadas (`throw`) como las promesas rechazadas de funciones asíncronas (`async/await`). Al detectar el fallo automáticamente, lo enruta directo al middleware global de manejo de errores sin requerir que el desarrollador invoque `next(error)` de forma manual dentro de un bloque `try/catch`.

---

### 5. ¿Qué diferencia de significado y uso existe en el estándar HTTP entre responder con estado 400 (Bad Request) y 422 (Unprocessable Entity)?
**Respuesta:**  
El código **400 (Bad Request)** indica un error de sintaxis o formato a nivel de transporte donde el servidor no puede parsear o interpretar la solicitud (como un JSON mal formado o un parámetro `:id` que no es numérico). El código **422 (Unprocessable Entity)** señala que la sintaxis y el formato JSON son completamente válidos, pero el contenido de los campos no cumple con las reglas semánticas requeridas (por ejemplo, un campo `title` vacío o que supera los 120 caracteres permitidos).

---

### 6. ¿Por qué es fundamental que la clase `AppError` extienda de la clase base `Error` nativa de JavaScript/TypeScript y qué ventajas aporta tener campos estables como `statusCode`, `code` y `details`?
**Respuesta:**  
Heredar de `Error` asegura que la clase capture la traza de ejecución (`stack trace`) y conserve compatibilidad con `instanceof Error` en TypeScript. Contar con `statusCode`, `code` y `details` permite que el middleware global extraiga el código HTTP exacto, exponga un identificador de error legible y constante para el frontend (sin depender de comparar cadenas de texto de `message`), y entregue retroalimentación puntual sobre fallas en campos específicos.

---

### 7. ¿Cuál es la utilidad técnica de generar un identificador único de solicitud (`requestId` / encabezado `X-Request-Id`) tanto en peticiones exitosas como en respuestas de error?
**Respuesta:**  
Facilita la trazabilidad y la depuración. Permite asociar una respuesta entregada al cliente con los registros de auditoría y logs internos del servidor generados durante esa misma ejecución, permitiendo rastrear problemas en producción de forma inmediata usando ese identificador único.

---

### 8. ¿Por qué es una mala práctica de seguridad exponer el `stack trace` o mensajes de error internos del motor/base de datos al cliente cuando ocurre un error 500 (Internal Server Error)?
**Respuesta:**  
Porque constituye una vulnerabilidad de divulgación de información. Revelar detalles de excepciones internas o trazas de pila expone rutas de archivos del servidor, versiones de dependencias, estructura del código fuente o consultas de base de datos, información que un atacante puede aprovechar para identificar debilidades y planear ataques dirigidos contra el sistema.