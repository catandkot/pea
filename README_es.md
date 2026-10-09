<div align="center">

[한국어](README.md) · [简体中文](README_zh.md) · [繁體中文](README_zh-TW.md) · [日本語](README_ja.md) · [Türkçe](README_tr.md) · [Русский](README_ru.md) · [Tiếng Việt](README_vi.md) · [ไทย](README_th.md) · [Deutsch](README_de.md) · **Español** · [Français](README_fr.md) · [Українська](README_uk.md) · [Polski](README_pl.md) · [Português (Brasil)](README_pt-BR.md) · [العربية](README_ar.md) · [فارسی](README_fa.md)

# 🫛 PEA

### Agente de granja inteligente — lee, decide y actúa sobre los datos del sitio de cultivo.

**Los sensores ya están hablando. PEA hace que su mensaje se entienda.**

[![license](https://img.shields.io/badge/license-MIT-f4c430?style=flat-square)](LICENSE)
[![status](https://img.shields.io/badge/status-concept-5c4ee5?style=flat-square)](#estado-del-proyecto)

[Inicio rápido](#inicio-rápido) · [Qué hace](#qué-hace) · [Cómo funciona](#cómo-funciona) · [Principios de seguridad](#principios-de-seguridad) · [Hoja de ruta](#hoja-de-ruta)

</div>

---

PEA (la arveja) es un **agente de granja inteligente** para invernaderos y cultivos en campo abierto. Reúne lecturas de sensores como la humedad del suelo, la temperatura y la humedad relativa, la radiación solar, el CO2 y el EC/pH del drenaje para explicar *lo que está ocurriendo ahora mismo*, y controla actuadores como el riego, la ventilación, el sombreo o la iluminación suplementaria **dentro de límites de seguridad**.

Cada decisión viene acompañada de su fundamento. PEA distingue y reporta por separado los **valores observados**, los **valores inferidos** y los **valores que aún se desconocen**. «El suelo está seco» es un dato del historial de sensores; «por la tarde hay un riesgo alto de marchitez» es una inferencia; y «si la válvula de riego se abrió de verdad» sigue siendo desconocido hasta que el caudalímetro lo confirme.

El trabajo se queda dentro de la granja. Los únicos datos que salen al exterior son los que el usuario elige de forma explícita.

## Inicio rápido

> **Los comandos y nombres de paquete que aparecen a continuación todavía no son definitivos.** Por ahora no hay ningún paquete `pea` publicado en npm. Cuando se publique, actualizaremos este documento con los comandos reales.

```bash
# 1) install (package not published yet)
npx pea-agents setup

# 2) readiness check — gateway, sensors and actuators
pea doctor

# 3) ask about today's greenhouse
pea analyze ./farm.toml
```

Una vez conectado al agente, basta con preguntar cosas así.

```text
El riego de la mañana en el invernadero 3 fue un 12% superior al objetivo. ¿A qué se debe?
¿Cómo tengo que cambiar el calendario de riego de esta tarde para ajustar la tasa de drenaje al 25%?
Muéstrame cuántas horas duró la humedad foliar tomando como referencia la temperatura mínima de anoche.
```

## Qué hace

| Área | Lo que hace PEA | Datos que utiliza |
| --- | --- | --- |
| Entorno de cultivo | Tendencias acumuladas de temperatura y humedad, VPD y radiación solar dentro del invernadero, con indicación de los intervalos anómalos | Sensores de temperatura y humedad, piranómetro |
| Riego y solución nutritiva | Cálculo del volumen de riego a partir de la radiación acumulada y alerta por desviación respecto al objetivo de tasa de drenaje | Sensores de humedad y peso del suelo, EC/pH del drenaje |
| Ventilación y climatización | Control para mantener el VPD objetivo y propuesta de niveles de apertura de ventanas y ventiladores | Retroalimentación de apertura de ventanas, meteorología exterior |
| Alertas de plagas y enfermedades | Cálculo del riesgo de enfermedades fúngicas a partir de la duración de la humedad foliar y la combinación de temperatura y humedad | Historial de temperatura y humedad, sensor de humedad foliar |
| Diario de cultivo | Registro automático del historial de riego, entorno y labores, con apoyo para introducir muestreos de crecimiento | Registro de labores, fotografías de campo |
| Predicción de cosecha | Estimación de la fecha de cosecha según la temperatura acumulada y comparación entre ciclos de cultivo | Historial de temperatura, parámetros de variedad |

Los elementos y umbrales de la tabla son **valores que deben recalibrarse para cada cultivo, variedad y región**. PEA solo propone valores por defecto; no los da por sentados como el valor de referencia de esa granja.

## Cómo funciona

Cuando un agente consulta a PEA mediante MCP, PEA reúne los datos en la pasarela y devuelve el resultado junto con su fundamento. Con ese resultado, el agente explica la situación o propone un plan de control. El mismo flujo de trabajo se usa igual desde la CLI en la terminal.

```text
Agente → MCP → PEA → Pasarela → Sensores
                   ← observaciones + inferencias + incógnitas
```

- **Local primero** — la recogida y el juicio se procesan dentro de la granja.
- **Fundamento primero** — junto a cada conclusión se guardan el valor original, la hora y la fuente.
- **Lo desconocido como desconocido** — los tramos sin sensor o con la comunicación interrumpida no se rellenan con estimaciones: se declaran como tales.
- **Resultados idénticos** — la CLI y MCP comparten el mismo flujo de trabajo y el mismo contrato de resultados.

## Integraciones compatibles

| Integración | Modo | Estado |
| --- | --- | --- |
| Pasarela MQTT | Recogida de temas de sensores mediante suscripción al bróker | Previsto |
| Modbus RTU / TCP | Sondeo de registros del controlador | Previsto |
| LoRaWAN | Sensores de campo abierto a través de la pasarela | Previsto |
| Datos meteorológicos | API meteorológica regional, temperatura, humedad y precipitación exteriores | Previsto |
| Sistemas de gestión de cultivo existentes | Fusión del historial mediante importación de archivos (CSV/JSON) | Previsto |

## Principios de seguridad

Mover un actuador es algo irreversible. Por eso PEA respeta lo siguiente.

1. **Aprobación humana por defecto** — las acciones reales como el riego, la ventilación o la iluminación suplementaria primero imprimen un plan y requieren aprobación.
2. **Límites máximos estrictos** — los topes configurados en la granja (volumen máximo de riego por evento, rango de temperatura permitido, etc.) no pueden superarse por decisión del agente.
3. **A prueba de fallos** — ante una interrupción de las comunicaciones o un corte de energía, el sistema pasa al lado seguro (detener el riego, abrir la ventilación, etc.).
4. **La pulverización no se automatiza** — PEA no aplica fitosanitarios ni fertilizantes: solo propone el momento y el fundamento.
5. **Registro completo** — cada control deja constancia del objetivo, la hora, el fundamento y el resultado de la ejecución.

## Hoja de ruta

- **M0 — Concepto y documentación** *(actual)*: definición del producto, principios de seguridad y diseño del contrato de resultados
- **M1 — Recogida**: integración con la pasarela, almacenamiento del historial de sensores y consulta de tendencias ambientales
- **M2 — Asesoramiento**: asesor de riego y ventilación, riesgo de plagas y enfermedades y automatización del diario de cultivo
- **M3 — Control con aprobación**: control automático bajo aprobación humana y a prueba de fallos
- **M4 — Operación**: predicción de cosecha, comparación entre varias granjas e informes por ciclo de cultivo

## Estado del proyecto

Este documento describe el **concepto de producto de PEA**. El código incluido en el checkout actual es una herramienta de ingeniería inversa (`rea-agents`) y todavía no implementa las funciones de recogida y control descritas arriba. Los nombres de los comandos, la lista de integraciones y los umbrales se fijarán junto con la implementación. Los README traducidos siguen basándose en la documentación anterior, por lo que los iremos ajustando de forma progresiva al nuevo nombre de marca.

## Contribuir

Las incidencias y los pull requests son bienvenidos. Resultan especialmente útiles los valores observados en campo, los casos de fallo y los valores de calibración por cultivo.

## Licencia

[MIT](LICENSE)
