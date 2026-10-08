# Sitio de prueba — medir la profundidad del crawler de Meta

Sitio estático de 4 páginas para determinar hasta qué nivel llega el rastreo de
`agent_config/websites` de Meta Business Agent.

## Estructura

```
index.html          Nivel 1  — landing
├── historia.html   Nivel 2A — enlazada desde la home
├── sucursales.html Nivel 2B — enlazada desde la home
    └── cocina.html Nivel 3  — enlazada SOLO desde sucursales.html
```

`cocina.html` no está enlazada desde la home. Es el sondeo: si el agente conoce
su contenido, el crawler siguió enlaces dos saltos desde la URL raíz.

## Canarios

Cada página tiene un código único y un dato que no existe en ninguna otra parte
del conocimiento del agente (ni en business info, ni en las FAQs, ni en el PDF
de la carta).

| Nivel | Página | Código | Dato exclusivo |
|---|---|---|---|
| 1 | index.html | `FOGON-NIVEL-1-PIMIENTA` | La salsa secreta se llama **Fogón Bravo** |
| 2A | historia.html | `FOGON-NIVEL-2A-LAUREL` | El primer cliente fue **Don Aníbal Quiroga** |
| 2B | sucursales.html | `FOGON-NIVEL-2B-ROMERO` | La sucursal de **Chacras de Coria**, Viamonte 330 |
| 3 | cocina.html | `FOGON-NIVEL-3-AZAFRAN` | La masa madre se llama **Dolores** |

## Cómo usarlo

1. Publicar la carpeta en un host accesible desde internet (dominio propio,
   GitHub Pages, Netlify, lo que sea). Tiene que ser **público**: el crawler de
   Meta no pasa por autenticación.
2. Cargar **solo la URL raíz** en `agent_config/websites`. No cargar las
   páginas internas: el objetivo es ver si las encuentra solo.
3. Esperar a que `crawl_status` deje de estar en progreso y anotar
   `pages_crawled`.
4. Preguntarle al agente por cada dato exclusivo, de arriba hacia abajo.

## Preguntas de verificación

| Preguntale | Si contesta, llegó a |
|---|---|
| ¿Cómo se llama la salsa secreta de la casa? | Nivel 1 |
| ¿Quién fue el primer cliente de El Fogón? | Nivel 2A |
| ¿Tienen sucursal en Chacras de Coria? | Nivel 2B |
| ¿Cómo se llama la masa madre? | Nivel 3 |

El `pages_crawled` que devuelve la API debería coincidir con la cantidad de
niveles que el agente efectivamente conoce. Si no coinciden —por ejemplo,
`pages_crawled: 4` pero el agente no sabe lo de Dolores— el problema no es el
rastreo sino la indexación o el recorte de contexto, y es un hallazgo distinto.

## Variables a dejar fijas

- **Sin `sitemap.xml`** a propósito. Si el crawler no sigue enlaces, agregar un
  sitemap después es el segundo experimento y aísla la variable.
- **`robots.txt` permisivo**, para descartar que un bloqueo explique un
  resultado pobre.
- Nada de JavaScript: todo el contenido está en el HTML servido. Si más adelante
  querés saber si el crawler ejecuta JS, esa es otra prueba aparte.
