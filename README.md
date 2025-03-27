# Prueba de concepto: Personalización de la salida CSS de PatternFly con SASS

Prueba de concepto de cómo implementar el sistema de diseño de patternfly a las necesidades específicas de Iberia.

Basado en el ADR de arquitectura de referencia [002-CSS-libraries](https://backstage.iberia.com/docs/default/Component/solution-architects.front-end-architectural-reference/ADRS/002-ADR-CSS-styles/)

## Introducción

Esta prueba de concepto tiene como propósito demostrar cómo adaptar la salida de CSS de un _design system_ existente (PatternFly) a las necesidades específicas de un proyecto (en este caso, Iberia). En lugar de utilizar los estilos predeterminados de PatternFly tal cual, se muestra cómo modificar y generar un conjunto de hojas de estilo personalizado mediante SASS. El objetivo es conservar la estructura y componentes de PatternFly, pero aplicando la paleta de colores, tipografías y espaciados propios de Iberia, todo ello sin alterar directamente el código fuente de PatternFly.

En resumen, el proyecto ilustra cómo tomar PatternFly como base y, a través de configuraciones SASS personalizadas (diseño de tokens/variables), obtener un CSS final adaptado a la identidad visual deseada. Esto permite beneficiarse de la consistencia y robustez de un sistema de diseño probado, a la vez que se aplica un _branding_ específico.

## ¿Qué es PatternFly y cómo se usa en este proyecto?

PatternFly es un sistema de diseño de código abierto mantenido por Red Hat, que provee una colección de componentes de interfaz de usuario, estilos CSS y guías de diseño para aplicaciones web empresariales. Incluye estilos base (reset/normalize de CSS), componentes (botones, formularios, tarjetas, etc.), diseños (_layouts_) y utilidades (clases auxiliares para espaciados, alineación, colores de fondo, etc.). PatternFly está pensado para asegurar consistencia visual y buenas prácticas de UX en distintas aplicaciones, de forma similar a como lo hacen otros frameworks (como Bootstrap), pero enfocado en necesidades de interfaces empresariales.

En este proyecto, PatternFly (versión 6.x) se incorpora como una dependencia de Node, y sus estilos se incluyen a nivel de código fuente SASS. En lugar de simplemente enlazar los CSS compilados de PatternFly, el proyecto importa los archivos SASS de PatternFly desde `node_modules`. Esto permite _interceptar_ y personalizar ciertos valores (como colores o tamaños) antes de compilar. Específicamente, la prueba de concepto utiliza los estilos base, las clases de layout y las utilidades de PatternFly. Al integrarlos mediante SASS, se consigue que el CSS generado mantenga las clases y estructura de PatternFly, pero con las adaptaciones visuales definidas para Iberia.

## Integración de SASS personalizado para modificar la salida CSS

La personalización se logra mediante la definición de **design tokens** propios y la reconfiguración de algunas importaciones de PatternFly. PatternFly utiliza un sistema de _tokens_ (variables de diseño expresadas como propiedades CSS custom, p. ej. `--pf-t--global--Color--100`) para definir colores, espaciados, tipografías y otros valores de estilo de forma semántica. En la configuración estándar, PatternFly trae sus propios valores predeterminados para estos tokens. En este proyecto, se reemplazan esos valores por unos personalizados, manteniendo los mismos nombres de token para asegurar la compatibilidad.

En la práctica, esto se logra creando archivos SASS con los tokens deseados (colores corporativos, tamaños de fuente específicos, etc.) y **usándolos en lugar de** los tokens de PatternFly. Por ejemplo, en `src/base.scss` (que construye los estilos globales base), la importación de las variables por defecto de PatternFly se comenta o evita, y se importa en su lugar el módulo de _tokens_ personalizado del proyecto. En código SASS, esto se ve así:

```scss
// Importación original de variables de PatternFly (omitida):
// @use "../node_modules/@patternfly/patternfly/base/patternfly-variables";

// Importación de tokens personalizados del proyecto:
@use "./tokens";
```

El módulo `./tokens` (definido en la carpeta `src/tokens`) contiene la definición de todos los tokens adaptados. Al compilar, este módulo inyecta en el CSS resultante las variables CSS con los nuevos valores. Con esta técnica, cuando posteriormente se incluyen los estilos de PatternFly (por ejemplo las utilidades o _layouts_), dichos estilos utilizarán las variables CSS ya establecidas por nuestros tokens personalizados. En otras palabras, las clases de PatternFly terminan aplicando los colores y medidas de Iberia simplemente porque las variables `--pf-...` correspondientes ahora contienen esos valores.

Además, el sistema de tokens incorpora soporte para temas (claro/oscuro) y orientaciones (LTR/RTL) de forma automática. En el archivo `src/tokens/_index.scss`, por ejemplo, se incluyen los tokens para el tema por defecto (claro) bajo el selector `:root` y los tokens del tema oscuro bajo el selector `:root:where(.pf-v6-theme-dark)`. Esto significa que, si una aplicación agrega la clase `.pf-v6-theme-dark` al elemento raíz (como indica la documentación de PatternFly), automáticamente se aplicarán las variables de color/diseño del tema oscuro definido en nuestros tokens. De igual manera, se utiliza un mixin `pf-v6-rtl` de PatternFly para incluir cualquier ajuste necesario para soportar layouts de derecha a izquierda si hiciera falta. Todo este mecanismo aprovecha las utilidades internas de PatternFly (`pf-v6-tokens`, `pf-v6-set-inverse`, etc.), pero alimentándolas con nuestros valores.

En resumen, la integración SASS personalizada consiste en **definir nuestros propios valores de tokens y asegurarnos de que PatternFly use esos valores** al generar el CSS y valorar qué debería de ser incluido y que quedaría fuera com en el caos de LTR/RTL ya que nuestro sistema de diseño no contempla componentes RTL. No se modifica directamente el código de PatternFly; en su lugar, se configura la compilación para que el _output_ final refleje la identidad visual personalizada.

## Estructura del proyecto

El proyecto está organizado de manera clara para separar las fuentes SASS, los tokens y la salida compilada. A continuación se describen las carpetas y archivos clave:

- **`package.json`**: Define las dependencias y comandos de construcción. Aquí se declara la dependencia de PatternFly (`@patternfly/patternfly`) y de Sass. También se configuran _scripts_ de NPM para compilar los archivos SASS en CSS (por ejemplo, `npm run css` que compila todos los archivos necesarios).
- **`src/`**: Carpeta de código fuente SASS personalizado.
  - `base.scss`: Hoja SASS principal de estilos base. Importa los _resets_ y normalizaciones de PatternFly (`reset`, `normalize`, etc.), así como estilos comunes. En este archivo se incorpora también el módulo de tokens personalizados (`@use "./tokens"`), de modo que al compilar se generen las variables CSS globales con valores adaptados. El resultado compilado es `dist/base.css`, que contiene los estilos globales (reset CSS, normalización) y la definición de todas las variables CSS de diseño para el tema de Iberia.
  - `layout.scss`: Hoja SASS para los _layouts_. Importa el índice de _layouts_ de PatternFly (`@patternfly/patternfly/layouts/index`), lo que incluye clases utilitarias de disposición (por ejemplo, clases como `.pf-v6-l-grid`, `.pf-v6-l-bullseye`, etc. para diseño de rejilla, alineación central, etc.). Gracias a la previa definición de tokens en `base.scss`, las propiedades CSS dentro de estas clases (márgenes, padding, colores de fondo si aplica) ya apuntan a las variables custom definidas. Se compila en `dist/layout.css`.
  - `utilities.scss`: Hoja SASS para utilidades. Similar a `layout.scss`, importa el índice de utilidades de PatternFly (`@patternfly/patternfly/utilities/index`). Esto genera clases utilitarias prefijadas (por ejemplo, `.pf-v6-u-mx-lg` para margen horizontal grande, `.pf-v6-u-background-color-100` para cierto color de fondo, etc.), nuevamente utilizando las variables globales definidas en nuestros tokens. Se compila en `dist/utilities.css`.
  - `tokens/`: Carpeta que contiene los archivos de tokens de diseño personalizados. Estos archivos definen los valores específicos que se quieren usar en lugar de los predeterminados de PatternFly.
    - **`_index.scss`**: Archivo SASS de entrada para los tokens. Este archivo reúne e importa los distintos grupos de tokens (paleta, tema por defecto, tema oscuro, locales) y los aplica en los selectores adecuados. Incluye, por ejemplo, la inserción de tokens en `:root` (tema claro) y en `.pf-v6-theme-dark` (tema oscuro). Es el módulo que `base.scss` usa para traer todos los tokens.
    - **`ib-tokens-palette.scss`**: Contiene la **paleta de colores base** definida para Iberia. Aquí se definen variables CSS para colores fundamentales (ejemplo: primarios, secundarios, grises, etc.) con el prefijo de PatternFly (`--pf-t--color--...`). Estas actúan como colores de base que luego los tokens semánticos usarán.
    - **`ib-tokens-default.scss`**: Define los **tokens del tema por defecto** (tema claro) de forma semántica. Por ejemplo, puede mapear `--pf-t--global--BackgroundColor--100` a un valor de la paleta (p. ej. un gris claro), o `--pf-t--global--BorderColor--default` a otro token base. Aquí se establecen tamaños de fuente, espesores de borde, espaciados globales, etc., para la versión estándar de la interfaz.
    - **`ib-tokens-dark.scss`**: Contiene los **tokens del tema oscuro** correspondientes. Generalmente redefinen una subset de tokens para cuando la interfaz está en modo oscuro (por ejemplo, valores de fondo y texto invertidos, colores con más contraste, etc.). Estos tokens se aplican sólo bajo la clase `.pf-v6-theme-dark` en el CSS final, permitiendo cambiar de tema dinámicamente.
    - **`ib-tokens-local.scss`**: Incluye tokens **locales o adicionales** específicos de Iberia que no vengan en PatternFly por defecto. Por ejemplo, en este archivo se ajustaron tamaños de fuentes para encabezados (`heading`) adaptándolos a las necesidades locales. Son personalizaciones extras que extienden el sistema de tokens de PatternFly allí donde se necesite algo específico. Sus valores también se incorporan en `:root` junto con los tokens por defecto.
- **`dist/`**: Carpeta de salida con los archivos CSS generados después de compilar.

  - `base.css`: CSS resultante de `base.scss`, contiene los _resets_ globales de navegador, normalizaciones, y la definición de **todas las variables CSS** (`--pf-t-...`) para el tema claro de Iberia, junto con las definiciones del tema oscuro (dentro de `.pf-v6-theme-dark`) listas para usarse.

  - `layout.css`: CSS resultante de `layout.scss`, con todas las clases de _layout_ de PatternFly (grillas, flex, alineadores, etc.), pero apuntando a los valores de diseño personalizados mediante las variables globales ya definidas en `base.css`.

  - `utilities.css`: CSS resultante de `utilities.scss`, con todas las clases utilitarias (espaciado, visibilidad, color de fondo, texto, etc.), igualmente utilizando los tokens custom en su interior.

En conjunto, esta estructura permite aislar claramente la lógica de _theming_ (en `tokens/`) de los estilos utilitarios y base. Los archivos SASS se mantienen pequeños y enfocados, y la salida CSS está separada por ámbito (global vs. utilidades vs. layouts), facilitando su consumo selectivo si fuera necesario.

## Pasos para instalar y ejecutar la prueba de concepto

Para probar esta prueba de concepto en un entorno local, siga estos pasos:

1. **Obtener el código**: clone el repositorio/proyecto que contiene estos archivos
2. **Instalar dependencias**: Asegúrese de tener Node.js y NPM instalados. En la raíz del proyecto, ejecute el comando `npm install`. Esto descargará las dependencias declaradas en `package.json`, incluyendo PatternFly y el compilador Sass.
3. **Compilar los estilos**: Ejecute el comando de compilación de CSS definido en los scripts de NPM. Puede usar `npm run css` para generar todos los CSS de una vez. Este comando internamente ejecuta Sass para compilar `src/base.scss`, `src/layout.scss` y `src/utilities.scss`, produciendo los archivos correspondientes en `dist/`. Alternativamente, puede ejecutar cada script individual (`npm run css:base`, `npm run css:layout`, `npm run css:utilities`) si solo quiere compilar una parte específica.
4. **Verificar la salida**: Una vez finalizada la compilación, verifique la carpeta `dist/`. Deberían existir los archivos `base.css`, `layout.css` y `utilities.css`. Puede abrir estos archivos para inspeccionar que contienen las clases de PatternFly con las variables y valores adaptados. Por ejemplo, en `base.css` verá las variables CSS `--pf-t-...` definidas con los colores y medidas especificados para Iberia.
5. **Usar los estilos**: Para probar los estilos en una página, puede incluir las hojas CSS generadas en un HTML de prueba. Por ejemplo, agregue `<link rel="stylesheet" href="dist/base.css">` (y los otros según necesidad) en el `<head>` de un HTML. Luego puede usar clases de PatternFly en el HTML (por ejemplo, una estructura con clases de layout o utilidades) y comprobar que refleja los estilos personalizados. Recuerde que si quiere probar el tema oscuro, debería añadir la clase `pf-v6-theme-dark` a un contenedor apropiado (por ejemplo `<html class="pf-v6-theme-dark">`) y observar los cambios de estilo.

_(Nota:_ Esta prueba de concepto no es una aplicación ejecutable como tal, sino una librería de estilos. Por lo tanto, "ejecutarla" se refiere a compilar los CSS y luego emplearlos en un contexto web para validar que los estilos funcionan según lo esperado).

## Comentarios sobre la viabilidad y extensibilidad del enfoque

El enfoque demostrado es **técnicamente viable** y se alinea con las recomendaciones modernas para personalizar sistemas de diseño. Al apoyarse en los tokens de PatternFly, se asegura que la gran mayoría de las personalizaciones se realizan de forma declarativa y centralizada (a través de variables CSS), en lugar de tener que sobrescribir montones de reglas CSS manualmente. Esto mejora el mantenimiento: si en el futuro se requiere cambiar un color corporativo o un tamaño de fuente global, bastaría con actualizar el valor correspondiente en los archivos de tokens y recompilar, propagándose el cambio a todos los estilos dependientes.

Una **ventaja importante** de este método es que aprovecha completamente las actualizaciones y mejoras del sistema PatternFly sin _forkear_ su código. Por ejemplo, si aparece una nueva versión de PatternFly con nuevos componentes o parches, se podría actualizar la dependencia y recompilar, y mientras los nombres de tokens se mantengan consistentes, la apariencia personalizada persistirá.

También hay que considerar los **límites y precauciones**: dado que la personalización depende en gran medida de las variables CSS de PatternFly, es importante asegurarse de que todos los aspectos que se quieren cambiar estén efectivamente tokenizados. PatternFly 6 ha llevado muchos valores a tokens semánticos, pero si algo puntual no lo estuviera, podría requerir agregar CSS adicional manualmente. No obstante, esas situaciones serían la excepción. El enfoque basado en tokens cubre colores, tipografías, espaciados, bordes y demás con granularidad suficiente para la mayoría de los casos de branding.

En cuanto a rendimiento y organización, dividir la salida en varios archivos CSS (base, layout, utilities, etc.) ofrece flexibilidad a los equipos consumidores en Iberia: podrían incluir solo lo que necesiten.

Finalmente, esta prueba de concepto demuestra una **buena práctica de arquitectura CSS**: reutilizar en lo posible un sistema consolidado (PatternFly) y extenderlo mediante configuraciones, en lugar de reinventar todo el CSS. La viabilidad queda patente en que, con relativamente pocos archivos SASS, se obtuvo un conjunto de estilos coherentes con la identidad de Iberia. La extensibilidad está garantizada por la propia naturaleza escalable de los tokens y la estructura modular. En conclusión, el enfoque es sólido para implementar un sistema de diseño personalizado apoyándose en PatternFly, facilitando consistencia visual y mantenibilidad a largo plazo.
