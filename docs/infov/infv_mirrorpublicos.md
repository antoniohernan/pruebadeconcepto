---
title: "Mirror para todos los públicos"
date: "2019-05-29"
categories: 
  - "informaticaviejuna"
---

Tengo unos conocidos podcaster que emplean una expresión para designar a los usuarios de a pié, esos a los que la tecnología supera por momentos, los denominan "**la población civil**".

Todos tenemos amigos que pertenecen a este grupo, **suelen tener equipos** (teléfonos, tabletas, portátiles, televisiones y ahora con los coches eléctricos incluso coches...) **a los que no  sacan todo su partido** pero oye!! dan bien el pego.

Por lo general tienen un concepto muy bueno de si mismo (**superlativo** en algunos casos) y hablan de sus cuñados con muchas chanzas, siendo ellos los peores.

Después de esta breve intro para poneros en situación os cuento la historia de uno de estos integrantes de la población civil, un chico que **se empeñaba en abrazar la tecnología a base de dinero pero la tecnología era escurridiza en su caso**.

Aquí que me pregunta un día, bastante apenado, el por qué de **las luces rojas de una NAS**, con discos supermegachulos WD Green no se que más.

Hay que reconocer que **las NAS tuvieron su momento** y que años atrás (sin Netflix, HBO, y otro muchos) eran para **descargar y descagar sin parar**, sin tener ni que pensar en montar tu propio servidor con todos esos servicios corriendo, pero eso lo dejamos para los que tienen ganas.

Que me pierdo... resulta que este buen chaval había comprado **una NAS de dos bahías para el**, y otra para casa de sus padres, y **3 discos iguales** (nótese la **falta de paridad**)

Su plan era poner la NAS de su casa a funcionar como un martillo pilón, bajando contenido día y noche. La configuración de los discos era **RAID 1 ([mirror](https://es.wikipedia.org/wiki/RAID#RAID_1_(espejo))).**

Y aquí fue donde aparezco yo a tener que explicar su poltergeist, lo que el quería hacer es, una vez que tenía **descargado bastante material**, **sacar uno de los dos discos de su NAS**, tirar para casa de sus padres y **pinchar allí el disco**, que la **NAS y su RAID 1 hiciese su magia** para que los **discos se quedase iguales** y una vez finalizado volver para casa, esperando que mientras tanto su NAS en casa había seguido llenando el disco que seguía vivo como si nada.

Ni que decir que esto es un despropósito, al sacar el disco de la primera NAS, el raid 1 se queda en alarma, le falta un disco en su configuración, y seguirá asi hasta que le pongas el disco y le digas que se reconstruya.

![](images/xScreen_Shot_2018-05-04_at_8_43_17_AM.jpg.pagespeed.gpjpjwpjwsjsrjrprwricpmd.ic_.vtdD56cB1Q.jpg)

Lo mismo en la NAS de destino, al sacar el disco aquella NAS se queda en alarma hasta que volvemos a reconstruirlo todo.

Cunclusión... compra otro disco y monta los dos Raid1.. que **te vas a pasar la vida reparando el raid**.
