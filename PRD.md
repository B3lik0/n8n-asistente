# PRD: Asistente interno de conocimiento

**Estado:** borrador con alcance inicial confirmado  
**Objetivo:** ayudar al equipo a encontrar respuestas fiables sobre procedimientos, políticas, promociones y capacitación desde un canal de mensajería.

## Resumen

Construir un asistente que responda en español a un equipo inicial de unas 20 personas usando documentación aprobada y citando sus fuentes. El MVP será un bot de Telegram al que podrá escribir cualquiera que encuentre o reciba su enlace; por eso el corpus inicial se limita al PDF `Proveedor.pdf`, sujeto a confirmar que no contiene información sensible. n8n Cloud coordinará el flujo y Supabase almacenará documentos y vectores. Las pruebas usarán el trial de n8n Cloud (14 días, tope de 1.000 ejecuciones; no existe un plan gratuito permanente).

## Ajustes vigentes para la fase de pruebas

Prevalecen sobre cualquier texto contrario más abajo.

- **Acceso:** el bot será privado, solo para el desarrollador, durante las pruebas. La apertura por enlace queda para una fase posterior.
- **Corpus de prueba:** contenido público de https://grupoprt.com/ (no sensible). `Proveedor.pdf` (manual de usuario del sistema de proveedores, no sensible) se indexa mediante el workflow "Asistente - Ingesta PDF".
- **Ejecución:** solo n8n Cloud (trial). No hay sandbox local: las credenciales solo pueden guardarse en Cloud, por lo que todo se construye y prueba allí.
- **Logs:** mínimos, solo lo necesario para depurar pruebas.
- **Expectativa:** el sitio tiene poco contenido; se espera que el bot se abstenga a menudo.

La gratuidad del canal y del plan de n8n no implica que todo el servicio sea gratuito: el modelo de lenguaje, la generación de embeddings y Supabase pueden tener costes o límites propios. No se habilitarán cargos automáticos; al alcanzar una cuota, el bot avisará que el servicio está temporalmente limitado.

## Usuarios y necesidad

| Usuario | Necesidad |
|---|---|
| Integrante del equipo | Resolver dudas operativas sin buscar manualmente en varios documentos. |
| Mantenedor | Aprobar y actualizar el PDF/contenido; revisar dudas no resueltas. |
| Administrador | Compartir el enlace del bot y revisar errores y uso. |

## Alcance del MVP

- Bot de Telegram cuyo enlace se compartirá con el equipo; no habrá lista de IDs autorizados en el MVP.
- Respuestas en español sobre procedimientos, políticas, promociones y capacitación.
- Corpus inicial: un PDF de información interna que no sea sensible.
- Búsqueda semántica sobre documentos aprobados en Supabase (Postgres/pgvector).
- Respuestas con nombre del documento y sección o enlace disponible como fuente.
- Si no hay evidencia suficiente, el bot debe decir que no encontró una respuesta fiable y dirigir al usuario a un contacto o proceso de escalamiento.
- Ingesta inicial controlada de documentos; excluir o actualizar contenido obsoleto.
- Registro mínimo para diagnóstico y medición, sin almacenar más datos personales de los necesarios.
- Límites de longitud y frecuencia para controlar consumo y abuso, dado que cualquier persona con el enlace podrá probar el bot.
- Ejecución en n8n Cloud (trial para pruebas; el MVP requerirá un plan de pago o self-hosting).

## Fuera de alcance inicial

- WhatsApp: requiere definir el acceso a WhatsApp Business Platform y un proveedor/canal compatible.
- Teams: requiere tenant, permisos y credenciales disponibles.
- Responder en nombre de la empresa a clientes externos.
- Ejecutar acciones que cambien registros, autoricen descuentos o sustituyan aprobaciones humanas.
- Indexar automáticamente todo el correo, chats o carpetas sin selección y aprobación.
- Garantizar respuestas sin coste: los planes gratuitos y sus cuotas pueden cambiar.

## Flujo principal

1. El administrador comparte el enlace del bot con el equipo.
2. El mantenedor carga `Proveedor.pdf` y confirma su categoría, vigencia y que es apto para compartir a través de un bot accesible por enlace.
3. El sistema extrae y divide el texto, genera embeddings y guarda fragmentos con metadatos en Supabase.
4. El integrante hace una pregunta en Telegram.
5. El flujo busca fragmentos pertinentes, aplica umbral de relevancia y genera una respuesta basada en ellos.
6. El bot responde con las fuentes o, cuando la documentación no contiene evidencia suficiente, explica que no encontró una respuesta fiable y ofrece contactar al mantenedor en `desarrolladortim@grupoprt.com`.
7. El usuario puede indicar si la respuesta fue útil o reportar un problema.

## Requisitos funcionales

| ID | Requisito |
|---|---|
| F1 | Cualquier persona que encuentre o reciba el enlace puede consultar el bot; no se le debe presentar como autenticado o restringido a las 20 personas iniciales. |
| F2 | Las respuestas deben basarse en contenido recuperado y mostrar fuentes identificables. |
| F3 | El asistente no debe inventar políticas, precios, promociones ni pasos operativos cuando falte evidencia. |
| F4 | El responsable debe poder añadir, reemplazar y retirar documentos sin reconstruir todo el sistema. |
| F5 | Cada documento debe conservar al menos título, categoría, fecha de vigencia, propietario y versión o fecha de actualización. |
| F6 | El sistema debe permitir reportar respuestas incorrectas y revisar los fragmentos utilizados. |
| F7 | Debe haber límites configurables de uso y una forma visible de escalar dudas no resueltas. |

## Requisitos no funcionales

- **Privacidad:** no indexar datos personales, secretos, credenciales ni documentos restringidos sin una decisión explícita y controles adecuados.
- **Seguridad:** guardar tokens en las credenciales de n8n Cloud, proteger Supabase y aplicar límites de uso. No incluir claves en workflows exportados, documentos ni mensajes.
- **Acceso a datos:** las personas consultan a través del bot; las credenciales de Supabase quedan solo en el servicio n8n. No se distribuye acceso directo a la base vectorial.
- **Acceso al bot:** el enlace no autentica a sus usuarios. Solo se cargará contenido aprobado no sensible; aplicar límites de uso y revisar el acceso si el uso crece.
- **Trazabilidad:** conservar información suficiente para diagnosticar respuestas, con retención limitada y evitando guardar el texto completo de conversaciones por defecto.
- **Disponibilidad:** el MVP puede ser de mejor esfuerzo; se debe comunicar cuando n8n, Supabase o el proveedor de IA no esté disponible o alcance su cuota.
- **Control de costes:** límites por usuario, topes de tamaño y seguimiento de consumo; no depender de una cuota gratuita como garantía.
- **Mantenibilidad:** separar ingestión de documentos, recuperación y respuesta para poder cambiar canal o proveedor.

## Criterios de aceptación

1. Cada mensaje de texto válido recibe una respuesta del bot: una respuesta sustentada por el PDF con su fuente, un aviso de que no encontró evidencia suficiente o un mensaje claro de indisponibilidad/cuota agotada.
2. Una pregunta fuera de la documentación nunca recibe una política inventada; el bot indica que no encontró respuesta y ofrece el correo del mantenedor.
3. Las personas que llegan al bot solo pueden consultar el corpus aprobado; el bot no expone credenciales ni acceso directo a Supabase.
4. Al reemplazar o retirar un documento, sus fragmentos anteriores dejan de ser elegibles para respuestas.
5. Una falla de Telegram, n8n, Supabase o del modelo se registra como error y no se presenta como respuesta exitosa.
6. El equipo puede consultar el uso y detectar errores sin exponer claves ni credenciales.

## Propuesta de entrega por fases

1. **Validación:** confirmar que `Proveedor.pdf` es el documento correcto y apto para consultas mediante un bot accesible por enlace.
2. **Prueba técnica:** bot de Telegram accesible por enlace, `Proveedor.pdf` y flujo en n8n Cloud gratuito.
3. **MVP:** ingestión repetible, respuestas con fuentes, límites de uso y registro de errores.
4. **Operación:** revisar cuotas, copias de seguridad, retención y responsable de contenido en los planes gratuitos seleccionados.
5. **Canales adicionales:** evaluar Teams y WhatsApp cuando estén disponibles las credenciales y se conozcan sus costes y requisitos.

## Decisiones aún abiertas

| Decisión | Propuesta inicial | Dato necesario |
|---|---|---|
| Canal de MVP | Telegram, acceso por enlace | Confirmado: cualquiera que encuentre o reciba el enlace podrá usar el bot; no se recopilarán IDs para una lista permitida. |
| Ejecución | n8n Cloud (trial) | Confirmado: trial de n8n Cloud para pruebas. Docker local eliminado. Plan para el MVP pendiente (pago o self-hosting). |
| Base vectorial | Supabase existente | Confirmado: solo n8n tendrá credenciales; el equipo consultará por el bot. |
| Modelo y embeddings | Priorizar opciones gratuitas | Pendiente elegir proveedor. Si se alcanza una cuota, el bot informará que está temporalmente limitado; no habrá cargos automáticos. |
| Contenido | `Proveedor.pdf` | Confirmado como PDF inicial. Falta confirmar que no contiene información sensible, quién aprueba cambios y cada cuánto se actualiza. |
| Seguridad | Bot accesible por enlace, corpus no sensible | Confirmado: no se restringirá por ID de usuario. No incluir información personal o confidencial en esta prueba. |
| Escalamiento | Correo del mantenedor | `desarrolladortim@grupoprt.com`. |
| Éxito del piloto | El bot responde | Confirmado: todos los mensajes válidos reciben una respuesta basada en el PDF o un aviso de que no hay evidencia suficiente, con fuente o contacto de escalamiento cuando aplique. |

## Riesgos y mitigaciones

- **Respuestas incorrectas:** usar recuperación con fuentes, umbral de relevancia, respuesta de abstención y pruebas con preguntas reales.
- **Información caducada:** asignar propietario y vigencia a cada documento, con revisión y retirada.
- **Costes inesperados:** medir tokens y embeddings, limitar uso y establecer un tope presupuestario antes del piloto.
- **Exposición de datos:** empezar con material no sensible, asumir que el enlace del bot puede compartirse fuera del equipo y evitar publicar n8n local sin endurecimiento.
- **Dependencia de un plan gratuito:** confirmar límites actuales antes de elegir alojamiento y diseñar el flujo para cambiar de proveedor.
- **Uso por terceros:** cualquier persona con el enlace puede consumir la cuota y consultar el corpus. No incluir información sensible, aplicar límites y dejar de compartir el enlace si se detecta abuso.

## Pendientes para implementar

1. Confirmar que `Proveedor.pdf` puede consultarse por un bot cuyo enlace podrá compartir cualquier persona.
2. Crear el bot en Telegram con BotFather y configurar el token como credencial en n8n Cloud; no compartir el token en el PRD ni en el chat.
3. Confirmar acceso al proyecto Supabase y preparar credenciales limitadas para n8n.
4. Elegir el proveedor de modelo y embeddings. La cuenta gratuita de n8n no incluye necesariamente esos servicios; si una cuota se agota, el asistente debe dejar de responder con IA y evitar cargos automáticos.
5. Definir qué hacer cuando el plan gratuito de n8n Cloud alcance límites de ejecuciones o retención.
6. Acordar quién aprueba las actualizaciones de `Proveedor.pdf`.
