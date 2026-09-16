# PRD-001: Sistema Inteligente de Reporte y Prevención de Riesgos Laborales

## Contexto y Problema
En muchas empresas los empleados detectan situaciones peligrosas, pero no saben dónde reportarlas, el reporte demora, se pierden observaciones, no se priorizan los riesgos más críticos, no se tiene una base unificada y no se genera conocimiento para prevenir accidentes.

## Objetivos
Diseñar una aplicación web donde cualquier empleado habilitado pueda generar un reporte de: actos inseguros, condiciones inseguras, casi accidentes, riesgos operativos, problemas de infraestructura, etc. Se requiere que el reporte ingresado se clasifique automáticamente en un tipo y además en base a la información ingresada se calcule el nivel de criticidad.
Los reportes quedan por defecto en estado abierto y después tienen que ser tratados.

## Requerimientos Funcionales
- RF-01: Cualquier usuario autenticado debe poder crear un reporte indicando descripción, ubicación y opcionalmente fotos adjuntas y un tipo sugerido.
- RF-02: El usuario administrador debe poder listar todos los reportes (paginados, con su estado abierto, en progreso, cerrado); el usuario común solo debe ver los suyos.
- RF-03: Al crear un reporte, el sistema debe asignarle una categoría entre: problema infraestructura, riesgo eléctrico, uso de elemento de protección personal, otro (Estas categorías corresponden a la primera versión del sistema y podrán ampliarse en futuras iteraciones.).
- RF-04: Al crear un reporte, el sistema debe asignarle una prioridad entre baja, media, alta, crítica.
- RF-05: Al crearse un nuevo reporte, el sistema generará una notificación simulada o registrará el evento de notificación, a futuro se espera que el sistema notifique por correo electrónico a los usuarios administradores.
- RF-06: El sistema deberá sugerir acciones preventivas en función de la categoría y criticidad del reporte mediante reglas configuradas por el negocio, Alta->requiere intervención inmediata, Crítica-> Detener actividad
- RF-07: El usuario creador podrá editar un reporte mientras permanezca en estado abierto. Una vez que el administrador lo pase a En Progreso o Cerrado ya no podrá modificarlo.
- RF-08: El sistema debe requerir autenticación (usuario + contraseña). Hay dos tipos de usuario, el común (crea y ve solo sus reportes) y el administrador (ve todos, revisa la clasificación automática y gestiona las acciones correctivas)
- RF-09: El sistema debe permitir que un usuario se registre con email + contraseña.
- RF-10: El sistema debe permitir que un usuario modifique su contraseña.
- RF-11: El administrador podrá cambiar el estado de un reporte entre abierto, en progreso y cerrado
- RF-12: El sistema registrará fecha, hora y usuario de creación y cierre de cada reporte.

## Requerimientos No Funcionales
- RNF-01: La clasificación debe responder en < 3 s (p95).
- RNF-02: La clasificación deberá obtener al menos 85% de aciertos sobre un conjunto de casos de prueba definidos por el proyecto.
- RNF-03: La asignación de criticidad deberá obtener al menos 90% de aciertos sobre un conjunto de casos de prueba definidos por el proyecto.
- RNF-04: La API key del modelo no debe estar en el código; se lee de la variable de entorno API_KEY.
- RNF-05: Las contraseñas deben almacenarse con hash seguro (bcrypt/argon2), nunca en texto plano; la sesión expira tras 24 h de inactividad.

## Criterios de Aceptación
- AC-01 (RF-03, RF-04): Dado el reporte "En el sector de carga hay una tapa de desagüe rota que puede provocar una caída", cuando se crea, entonces categoría = problema infraestructura y prioridad = alta.
- AC-02 (RF-01): Dada una descripción vacía, cuando se intenta crear el reporte, entonces el sistema responde HTTP 400 y no lo crea.
- AC-03 (RF-02): Dados más de 20 reportes, cuando se listan, entonces se devuelven paginados de a 20 (parámetros page/size).
- AC-04 (RF-08): Dado un usuario no autenticado, cuando intenta ver la lista de reportes, entonces el sistema responde HTTP 401 y no muestra ningún dato.
- AC-05 (RF-02, RF-08): Dado el usuario A dueño de un reporte y el usuario B autenticado, cuando B intenta ver ese reporte, entonces responde HTTP 403 y no lo muestra. (control de acceso — OWASP #1)
- AC-06 (RF-06): Dado un reporte clasificado como criticidad Crítica, cuando se visualiza el reporte, entonces el sistema muestra la recomendación "Detener actividad y escalar al responsable de seguridad".
- AC-07 (RF-07): Dado un reporte en estado abierto cuyo propietario es el usuario autenticado, cuando modifica la descripción y guarda los cambios, entonces el sistema persiste la nueva información.
- AC-08 (RF-11): Dado un administrador autenticado, cuando cambia el estado de un reporte de Abierto a En Progreso, entonces el nuevo estado queda registrado y visible en futuras consultas.
- AC-09 (RF-12): Dado un reporte cerrado por un administrador, cuando se consulta el detalle del reporte, entonces se muestran fecha, hora y usuario que realizó el cierre.
- AC-10 (RF-12): Dado un reporte recién creado, cuando se consulta el detalle, entonces se muestran la fecha, hora y usuario de creación.

## Fuera de Alcance
CRM completo · multi-canal real (mail/WhatsApp en vivo) · RBAC configurable / más de dos roles · multi-tenant · envío real de mails (el borrador queda para copiar/pegar).
(La autenticación con dos roles (usuario común y administrador) sí forma parte del alcance.: RF-08.)

## Riesgos y Dependencias
- Riesgo: clasificación incorrecta o inconsistente para descripciones ambiguas → mitigación: validación con dataset etiquetado y revisión periódica de casos erróneos.
- Dependencia: API de Claude · base de conocimiento kb.md · SQLite.