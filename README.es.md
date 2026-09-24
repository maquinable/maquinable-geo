# Maquinable GEO

[English](README.md)

Plugin de Claude Code para GEO (optimización para motores generativos) y SEO técnico. Cuatro skills que trabajan sobre tus artículos y tu sitio, en español y en inglés.

## Skills

| Skill | Qué hace |
|---|---|
| `content-brief` | Investiga una keyword en el SERP (tu idioma y tu mercado) y arma un brief: estructura del top 10, términos obligatorios, longitud objetivo y preguntas reales para el FAQ. Lo guarda en `briefs/<slug>.md`. |
| `geo-optimize` | Le agrega a un artículo terminado una capa pensada para IA: TL;DR al inicio, FAQ de 4 a 6 preguntas al final y JSON-LD FAQPage. En HTML o Markdown. |
| `seo-gate` | Puntúa el artículo sobre 100 antes de publicar, arregla lo que puede y vuelve a puntuar hasta llegar a 85 o a 3 pasadas. Los links internos salen de tu sitemap real. Incluye un control de voz que cuenta y cita los tics de escritura de IA. |
| `seo-audit` | Auditoría de solo lectura de un sitio: sitemap, títulos, meta descriptions, canonical, JSON-LD, H1, alt de imágenes, veredicto de robots.txt para buscadores y crawlers de IA, `/llms.txt`. Devuelve los fixes por severidad. |

Flujo típico: `content-brief` → escribir el artículo → `geo-optimize` → `seo-gate` → publicar.

## Ejemplos de prompts

- "Brief para `software de facturación para autónomos`, mercado España"
- "Optimiza este artículo para IA" (pega el artículo o indica un archivo)
- "Pásale el gate a `borradores/mi-post.md`, keyword `automatizar facturas`, dominio ejemplo.com"
- "Audita ejemplo.com. ¿Los bots de IA pueden leerlo?"

## Configuración opcional

Crea `maquinable-geo.config.md` en la raíz de tu proyecto para que las skills no te pregunten cada vez:

```markdown
# maquinable-geo config
- Dominio: ejemplo.com
- Idioma / mercado: español, España
- Audiencia: dueños de gestorías pequeñas
- Clusters: 1) facturación, 2) nóminas, 3) IA para gestorías
- Voz: directa, concreta, primera persona del plural, sin hype
- CMS: WordPress (sanitiza <script> en el cuerpo del post: sí)
```

## Qué no hace

- No publica nada ni modifica tu sitio. `seo-audit` solo lee.
- No promete posiciones ni citas en IA. Desde 2023, el FAQPage no da rich results en Google salvo en sitios de gobierno y salud. Su valor para GEO es plausible, pero no está probado.
- No calcula un score global de SEO o GEO de tu sitio.
- No inventa URLs, volúmenes de búsqueda ni datos. Lo que no puede verificar lo marca como no verificable.

## Permisos y servicios externos

- Usa las herramientas nativas de Claude: búsqueda web, fetch web y bash (librería estándar de Python para sitemaps y robots.txt).
- No requiere servicios externos ni API keys.
- Opcional, con tus propias keys: PageSpeed Insights (performance y mobile en `seo-audit`), SerpAPI o DataForSEO (datos de SERP más precisos en `content-brief`).
- Opcional: `pip install protego` para verificar robots.txt con un segundo parser (RFC 9309).

## Instalación

```
/plugin marketplace add maquinable/maquinable-geo
/plugin install maquinable-geo@maquinable
```

## Créditos

Las reglas anti-tics de IA de `seo-gate` están adaptadas de [blader/humanizer](https://github.com/blader/humanizer).

## Sobre Maquinable

Hecho por [Maquinable](https://maquinable.com). Si quieres el score de citabilidad en IA de tu sitio y un plan de fixes priorizado, eso es lo que hacemos: [pide el diagnóstico](https://maquinable.com/pedir?ref=claude-plugin).

Soporte: hola@maquinable.com

Privacidad: [PRIVACY.md](PRIVACY.md). El plugin no recolecta datos.

Licencia: MIT
