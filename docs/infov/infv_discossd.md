---
title: "¿Y esto tiene discos SSD?"
date: "2019-05-27"
Copyright: "&copy; 2019-2023 Antonio Hernan"
License: "CC BY-SA 4.0"
categories: 
  - "informaticaviejuna"
---

La muerte reciente de Eduardo Punset me ha traído a la cabeza cuando en mi empresa gestionábamos uno de sus portales de ayuda. El portal iba, como solemos decir coloquialmente, **como el ojete**, y vino la típica reunión Comerciales-Desarrollo-Sistemas para **tirarnos los trastos a la cabeza y localizar al culpable.**

 

![](images/Selección_440.png)

Aparecieron **frases célebres, desgastadas por el uso**...

"_En desarrollo el portal vuela_..." "_En mi entorno local del portátil va como un tiro_..." "_¿Y si le metemos más memoria al servidor? ¿le ponemos más CPU?_" "_¿Tienen todos los parches?_"

Típicas todas ellas de personas con ninguna o muy poca experiencia que **apenas aciertan a copiar y pegar código encontrado en [StackOverFlow](https://stackoverflow.com/)**.

Y entonces soltaron esta, que me **hizo saltar del sitio**:

"¿Pero ese servidor **tiene los discos SSD no**?", claro que si majadero, este y todos los de ese CPD, valen más sus cabinas de discos que tus dos riñones vendidos al mercado negro...

Efectivamente, era **un problema de codificación** (again), que en desarrollo o en local con apenas 10 entradas funciona todo como un avión, pero que **en producción con algunas decenas de miles de entradas** ese bucle que haces trayéndote toda la información sin filtrar mata al portal al segundo usuario.

Por supuesto que la solución no sirvió para nada, **meses después** con otro portal similar los **mismos problemas**, el código era un copia/pega y seguían ahí las deficiencias de siempre, no supieron ni reutilizar el código.

Pero esta vez la reunión la empecé yo y dejé claro que los discos eran SSD, y devolví el revés preguntando primero si su aplicación estaba preparada para soportar tanta velocidad en disco.
