# Fase 1 — Rebrand visual: color, logo, favicon, página About

**Fecha:** 2026-07-08
**Fase:** 1 de 4 (Rebrand visual → SEO → GEO → Mobile, cada una con su propio spec)
**Estado:** Aprobado para pasar a plan de implementación

## Contexto

El tema actual de Serenity Candle Haus fue construido por una agencia externa
(crédito "SNC Designs" en el footer) usando una paleta crema/beige con acento
dorado (`#6B550E`) hardcodeado en el `<style>` inline de cada sección. Esto no
coincide con la identidad de marca real de la clienta, confirmada por el
email marketing y los archivos de logo que envió: **fucsia `#CB6CE6`, blanco,
negro**. Esta fase recolorea el tema existente sin tocar contenido, textos ni
funcionalidad.

## Objetivos

- Reemplazar el acento dorado `#6B550E` (y el fondo crema) en las ~15
  secciones custom del tema por un sistema de tokens CSS basado en la marca
  real.
- Fondo blanco como color dominante en todas las páginas y secciones.
- Texto principal en negro, con un gris de apoyo para jerarquía secundaria.
- Fucsia de marca como acento puntual (botones pill, precios, hovers,
  badges, iconos) — nunca como fondo masivo.
- Crear la página "Sobre Nosotros", que hoy no existe.
- Agregar favicon (no existe ninguno hoy).
- Reemplazar el logo del header cuando la clienta entregue el archivo.

## No objetivos (explícitamente fuera de esta fase)

- **Efecto de scroll blanco→negro**: se evaluó ubicarlo en `about-banner` y
  luego en `scrolling-gallery`, pero se descartó — la clienta señaló que la
  sección siguiente no podría quedar en negro sólido sin verse forzada, y
  prefiere revisar el resultado del recolor base antes de considerar
  cualquier efecto de este tipo. Si en el futuro se decide agregarlo, es un
  spec aparte.
- SEO, GEO (optimización para agentes de IA) y mejoras de mobile: son las
  fases 2, 3 y 4, cada una con su propio spec una vez cerrada esta.
- Cualquier cambio de contenido/copy fuera de lo que ya existe (excepto la
  copy nueva de la página About, ver abajo).

## Sistema de color (tokens)

| Token | Hex | Uso |
|---|---|---|
| `--color-bg` | `#FFFFFF` | Fondo de página y de todas las secciones (reemplaza el crema `#FDFBF5`/`#F0ECDF`/`#FAFAF8`/`#FDFCF8`) |
| `--color-fg` | `#111111` | Texto principal, títulos |
| `--color-fg-muted` | `#6B6B6B` | Subtítulos, texto secundario, precios tachados (reemplaza `#6D6C6C` y variantes) |
| `--color-border` | `#E5E5E5` | Bordes, separadores, contornos de tarjeta |
| `--color-accent` | `#CB6CE6` | Fucsia de marca: bordes de botón outline, links, iconos activos, badges, focus ring |
| `--color-accent-solid` | `#B125D9` | Relleno de botón primario con texto blanco (variante oscurecida del fucsia base para cumplir contraste AA — ver nota) |
| `--color-accent-hover` | `#BC41DF` | Estado hover |
| `--color-accent-active` | `#9920BC` | Estado :active/pressed |

**Nota de accesibilidad:** el fucsia base `#CB6CE6` con texto blanco encima da
un contraste de 3.08:1, por debajo del mínimo AA (4.5:1) para texto normal.
Por eso el relleno sólido de botones usa `--color-accent-solid` (5.08:1 con
blanco), y el fucsia base se reserva para usos donde el texto es negro
(6.4:1) o donde no hay texto encima (bordes, iconos).

Estos tokens se implementan como variables CSS (probablemente en
`snippets/theme-colors.liquid` incluido desde `theme.liquid`, o directamente
en `:root` dentro de `theme.liquid`) y reemplazan cada hex hardcodeado en los
`<style>` inline de: `header.liquid`, `footer.liquid`, `hero-banner.liquid`,
`product-carousel.liquid`, `featured-product.liquid`, `scrolling-gallery.liquid`,
`about-banner.liquid`, `testimonials-masonry.liquid`, `plp.liquid`,
`list-collections.liquid`, `main-product.liquid`, `cart.liquid`,
`contact-hero.liquid`, `contact-form.liquid`, `favoritos.liquid`,
`comixub.liquid`, `product-recommendations.liquid`.

## Tratamiento por sección

- **Todas las secciones de contenido** (carousels, PLP, PDP, cart, contact,
  favoritos, footer, header): fondo blanco, texto negro/gris, acentos fucsia
  en botones/precios/hovers según la tabla de tokens. Sin excepciones — no
  queda ninguna sección en crema.
- **hero-banner**: mantiene su video/imagen de fondo con overlay oscuro
  existente; solo se ajustan textos/CTA para usar los tokens (ya está en
  blanco/negro, cambia poco).
- **about-banner**: se mantiene tal cual está estructurada — imagen de fondo
  completa con overlay y texto blanco. **No se aplana a un fondo de color
  sólido.** Es la sección de mayor peso visual/fotográfico y se conserva así
  por decisión explícita de la clienta.
- **scrolling-gallery**: se recolorea a fondo blanco fijo (sin efecto de
  scroll de color, ver "No objetivos"). Queda a decisión de la clienta si se
  reactiva en el home (hoy está `disabled: true`) — no se reactiva como
  parte de esta fase salvo que lo pida explícitamente.

## Página "Sobre Nosotros" (nueva)

No existe hoy: el link `/pages/about` en `header.liquid:123` apunta a una
página inexistente. Se crea:

- `templates/page.about.json`
- Sección nueva `sections/about-editorial.liquid` — tratamiento editorial de
  revista (imagen grande + texto largo con jerarquía tipográfica marcada),
  distinto del formato compacto de `about-banner`. Usa los mismos tokens de
  color que el resto del sitio (fondo blanco, texto negro, fucsia como
  acento tipográfico si aplica a algún destacado).
- Copy inicial: se reutiliza el texto existente de la sección `about-banner`
  del home como base. La clienta lo va a ampliar más adelante; el diseño de
  la sección debe soportar párrafos largos sin quedar roto.

## Logo y favicon

- **Pendiente de asset**: la clienta va a enviar el archivo del logo
  definitivo (de las dos variantes vistas en el email: wordmark "serenity."
  en script fucsia, o el emblema "S." con wordmark en negro debajo). Hasta
  que llegue, no se reemplaza el logo del header — se deja como placeholder
  documentado.
- **Favicon**: no existe ninguno configurado en `theme.liquid` ni en
  `settings_schema.json`. Se agrega usando el emblema "S." (mejor legibilidad
  a tamaño diminuto que el wordmark), también pendiente del archivo.
- Una vez recibidos los assets: subir a `assets/`, agregar `<link rel="icon">`
  en `theme.liquid`, y reemplazar la imagen/texto del logo en `header.liquid`.

## Workflow de implementación

Todo el trabajo se hace sobre un **tema de desarrollo de Shopify**
(`shopify theme dev`), nunca directo sobre el tema publicado. La clienta
revisa en la URL de preview en vivo (datos y productos reales) antes de que
se publique nada. Shopify CLI 4.4.0 ya está instalado localmente.

## Dependencias / bloqueos

- Archivo de logo definitivo — pendiente de la clienta.
- Archivo/ícono para favicon — pendiente de la clienta (o se deriva del
  logo una vez recibido).
