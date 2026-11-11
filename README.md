# Asistente interno de conocimiento

Bot de Telegram que responde en español sobre documentación aprobada y cita su fuente. Si no encuentra evidencia suficiente, no inventa: indica que no halló una respuesta fiable y remite al mantenedor.

El flujo corre en **n8n Cloud** y guarda documentos y vectores en **Supabase** (Postgres con pgvector). El alcance completo está en [`PRD.md`](PRD.md).

## Estado

Fase de pruebas, bot privado (solo el desarrollador).

- Corpus: contenido público de https://grupoprt.com/ y `Proveedor.pdf` (manual de usuario del sistema de proveedores).
- n8n Cloud en periodo de prueba (14 días, 1.000 ejecuciones); no existe un plan gratuito permanente.
- Fuera de alcance por ahora: WhatsApp, Teams, acceso público por enlace.

## Arquitectura

Tres workflows en n8n Cloud, con ingesta y consulta separadas:

| Workflow | Disparador | Función |
|---|---|---|
| Asistente - Ingesta | Manual | Descarga grupoprt.com, extrae el texto, lo divide en chunks y guarda sus embeddings. |
| Asistente - Ingesta PDF | Formulario web | Recibe un PDF, extrae el texto y guarda sus embeddings con el nombre indicado como fuente. |
| Asistente - Consulta Telegram | Mensaje de Telegram | Un agente busca en la base con la herramienta `Knowledge Base` y responde con la fuente. Si falla, envía un aviso de indisponibilidad. |

```
Telegram → agente (Gemini) → Knowledge Base (Supabase + embeddings OpenAI) → respuesta con fuente
                    └─ error → aviso de servicio no disponible
```

Modelos: chat con Gemini y embeddings `text-embedding-3-small` de OpenAI a 1536 dimensiones, ambos vía los créditos del gateway de n8n.

## Base de datos

Tabla `documents` en Supabase:

| Columna | Tipo |
|---|---|
| `id` | `bigint` (clave primaria) |
| `content` | `text` |
| `metadata` | `jsonb` (`source`, `title`) |
| `embedding` | `vector(1536)` |

Además, la función `match_documents(query_embedding, match_count, filter)` que usa el nodo de vector store. La dimensión debe coincidir con el modelo de embeddings: si se cambia de modelo hay que recrear la columna y reindexar.

## Puesta en marcha

1. Crear el bot con BotFather y guardar su token como credencial de Telegram en n8n.
2. Crear en Supabase la tabla `documents` y la función `match_documents`, y activar RLS en la tabla.
3. Guardar en n8n la credencial de Supabase con la URL del proyecto y la clave secret. Esa clave no debe salir de n8n.
4. Ejecutar la ingesta (manual para la web, formulario para PDF).
5. En el workflow de consulta, restringir el trigger al ID de usuario de Telegram autorizado y publicarlo.

La ingesta no elimina chunks anteriores: repetirla duplica el contenido.
