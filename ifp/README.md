# IFP — Guías de Estudio Pareto

Guías de preparación de exámenes para módulos IFP aplicando el **Método Pareto**:
identificar el 20% del contenido que genera el 80% de los puntos.

## Metodología

**Regla fundamental:** lo que salió en el examen de ejemplo no repetirá exactamente.
El profesor preguntará la variante lógica o el tema adyacente del mismo grupo.

Cada módulo incluye:
- `guia-pareto-estudio.md` — qué estudiar y por qué (temas adyacentes al ejemplo)
- `simulacro-examen.md` — preguntas + corrección del profesor con palabras clave y trampas típicas

## Módulos disponibles

| Módulo | Guía | Simulacro |
|--------|------|-----------|
| [Seguridad y Alta Disponibilidad](seg-alta-disp/) | ✓ | ✓ |
| [Hacking Ético](haking-etico/) | ✓ | ✓ |
| Seguridad en Redes y Servicios | Pendiente | Pendiente |
| Sostenibilidad Aplicada | Pendiente | Pendiente |

## Cómo usar

1. Lee `guia-pareto-estudio.md` del módulo
2. Sin mirar las respuestas, responde las preguntas del `simulacro-examen.md`
3. Compara con la corrección del profesor y anota tu puntuación
4. Repasa solo los temas donde no salieron las palabras clave

## Generar una guía para un nuevo módulo

Con Claude Code, coloca los materiales del profesor en `docs/` y ejecuta:

```
/pareto-estudio ./docs
```
