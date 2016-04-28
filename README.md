<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Manual](#manual)
  - [Objetivo](#objetivo)
  - [Filosofía](#filosof%C3%ADa)
  - [Entrega contínua](#entrega-cont%C3%ADnua)
  - [Operaciones](#operaciones)
  - [Seguridad](#seguridad)
  - [Código heredado](#código-heredado)
  - [Arquitectura](#arquitectura)
  - [Dependencias](#dependencias)
  - [Guías de estilo](#gu%C3%ADas-de-estilo)
  - [Documentación](#documentaci%C3%B3n)
  - [Tests](#tests)
  - [Control de calidad](#control-de-calidad)
  - [Workflow](#workflow)
  - [Diseño / Experiencia de usuario](#dise%C3%B1o--experiencia-de-usuario)
  - [Código de conducta](#c%C3%B3digo-de-conducta)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Manual

Este es un documento vivo que describe workflows, herramientas y otras yerbas en torno a nuestra forma de trabajar desde el desarrollo hasta la implementación continua.

La idea es documentar el estado y evolución de nuestra metodología como referencia interna del equipo y al mismo tiempo compartir nuestras estrategias con todo aquel que quiera aprovecharlas, discutirlas, etc.

Elijo este formato en lugar de utilizar un wiki por dos motivos:
+ Permite apreciar la evolución histórica del documento.
+ Resulta conveniente y oportuno para ejercitar nuestro workflow basado en github.

## Objetivo

Nos proponemos adoptar mejores prácticas, procesos, cultura y metodología tomados del movimiento open source para aplicarlos al desarrollo de software en el sector público.

De manera similar a como el movimiento [inner source](http://paypal.github.io/InnerSourceCommons/index.html) refrezca las prácticas en las empresas privadas en posde  aumentar su eficiencia y calidad, nos valemos de estos recursos para ganar en economía, agilidad y calidad en el estado.

Algunas de las ventajas que ofrece este enfoque son:
* La unificación de criterios y mejores prácticas entre equipos dispersos en una multiplicidad de dependencias.
* La transparencia en cuanto a avances, aprovechamiento de recursos y calidad.
* El intercambio de aportes entre equipos aprovechando una capacidad instalada diversa y dispersa. Por ejemplo, quién mejor se desenvuelve en el ámbito de la experiencia de usuario puede realizar aportes a todas las aplicaciones, mejorando notablemente las interfaces al tiempo que son puestas en valor su experiencia y habilidades.
* Una mejora progresiva de la calidad mediante la utilización de una serie de herramientas que permiten su cuantificación.

## Filosofía

> “Escribe programas que hagan una cosa y la hagan bien, que trabajen en armonía con 
> otros y que manejen flujos de texto, pues esta es una interfaz universal.” – *Doug Mcllroy*

## Entrega contínua

## Operaciones

La consigna es "automatizarlo todo". 

Expresar la infraestructura como código para ganar en reproducibilidad e integrar su evolución a nuestro flujo de trabajo cotidiano.

Cultivar la estrategia chat ops, tanto como herramienta de automatización como para mantener a todo el equipo en una misma página.

Nuestro equipo de operaciones no debería hacer prácticamente nada sobre la infraestructura en si misma. Su trabajo consiste en diseñarla y programar a Juanito (nuestro chat bot basado en hubot) para que haga el trabajo duro por nosotros.

El diseño actual cuenta con dos servidores proxmox sobre los que corren una serie de máquinas virtuales mayoritariamente linux.

Cada máquina virtual alberga el stack completo de una aplicación, desde el sistema operativo hasta el servidor web.

Esto permitió ganar en reproducibilidad y una mayor flexibilidad a la hora de gestionar los recursos.

La dificultad que implica configurar un cluster capaz de correr este tipo de arquitectura, obstaculizó el avance en terminos de escalado, redundancia y alta disponibilidad.

De presentarse hoy una falla de harware en alguno de los servidores, el equipo de operaciones se encuentra en condiciones de reestablecer los servicios en pocas horas.

Se dispone, también, de algunos servidores más, actualmente en desuso.

En esta nueva etapa, la propuesta consiste en migrar progresivamente hacia la implememtación de un cluster coreOS de cuatro servidores mas reverse proxy mas control plane.

Nos encontramos ejercitando y familiarizandonos con las herramientas de gestión coreOS en entornos de desarrollo basados en Vagrant al tiempo que desarrollamos los módulos necesarios para la automátización con Juanito.

Resulta crítico para el real aprovechamiento de este diseño, el que el equipo de desarrollo acompañe los avances realizando las modificaciones necesarias para que las aplicaciones cumplan con requerimientos de aplicaciones cloud native (principalmente que los procesos sean stateless).![Datacenter](https://cdn.rawgit.com/MinEduTDF/manual/master/datacenter.mmd.png)

## Seguridad

### Https

Toda comunicación entre los usuarios y nuestros servicios es encriptada mediante la utilización de certificados ssl de alta calidad.

Para gestionarlos de manera automatizada ante [letsencrypt](https://letsencrypt.org) utilizamos [docker-letsencrypt-proxy-companion]( https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion/blob/master/README.md).

Esto nos permite contar con certificados gratuitos, de calidad y que se renuevan automáticamente evitando las incómodas y frustrantes advertencias sobre certificados de seguridad caducos.

## Código heredado

Durante los últimos años se han desarrollado alrededor de quince aplicaciones destinadas a resolver diversos aspectos de la gestión ministerial.

Estas constituyen un capital invaluable en términos de análisis, feedback de usuarios y experiencia en producción.

Con el propósito de capitalizar y poner en valor estos esfuerzos, emprendemos un proceso de modernización del software a desarrollarse en varias etapas.

### Puesta a punto para liberación.

* Mover datos de conección a base de datos a variables de entorno.
  * Disminuye el riesgo de publicar credenciales accidentalmente.
  * Conforma con aplicaciones de doce factores.
  * Simplifica la configuración para el desarrollo en entornos virtualizados.
* Implementación del driver PDO.
  * El driver mysql ha sido depreciado.
  * PDO abstrae la base de datos volviendo trivial un cambio de motor en caso de ser necesario.
  * Simplifica notablemente la tarea de sanitizar la entrada de usuario.
* Sanitización de la entrada de usuario.
  * Previene ataques de inyección sql.
  * Previene ataques xxs.
* Abstraer la conección a base de datos.
  * Evita la duplicación de código.
  * Mejora la performance de la aplicación al evitar establecer una nueva conección por ruta.
* Hashing de passwords.
  * Incrementa la privacidad de los usuarios.
  * Previene el filtrado intencional o no de credenciales.

### Sustentabilidad

* Implementar composer para gestión de dependencias.
  * Hace más liviano el repositorio.
  * Evita la duplicación de código.
  * Viabiliza la actualización.
  * Permite la gestión programática de dependencias.
* Implementar php code sniffer y normas de estilo.
  * Mejora la legibilidad del código.
  * Permite evitar errores comunes.
  * Viabiliza la colaboración.
* Implementar migraciones de bases de datos.
  * Permite mantener la base de datos en control de versiones.
  * Facilita la tarea de modificar schemas en producción.
  * Mejora notablemente la experiencia en la colaboración al contar el desarrollador con la base de datos exacta para cada versión del software.
* Dockerizar.
  * Agiliza la colaboración.
  * Mejora la reproducibilidad.
  * Evita sorpresas ante la puesta en producción.
  * Permite integrar a nuestro pipeline automatizado de entrega continua.

### Separación de responsabilidades

* Refactorización en MVC.
  * Separa responsabilidades.
  * Ordena el código.
  * Facilita la colaboración.


## Arquitectura

Actualmente se presentan dos tipos de aplicaciones monolíticas bién diferenciados:

* Aplicaciones programadas desde cero, sin utilizar librerías ni patrones de diseño como MVC, ORM, etc.
* Aplicaciones basadas en framework MVC más plugins.

![Monolitos](https://cdn.rawgit.com/MinEduTDF/manual/master/monolitos.mmd.png)
*El usuario interactua con una diversidad de interfaces.*

El objetivo es desarrollar una migración progresiva hacia una arquitectura de microservicios fuertemente modularizada. Que se apoye sobre una gestión automatizada de dependencias y que presente al usuario una única interface que integra de manera transparente todos los servicios.

Esto favorece la manejabilidad de los proyectos al dividirlos en módulos pequeños, bién documentados y testeados y agiliza las operaciones al tiempo que optimiza la utilización de recursos materiales.

![Microservicios](https://cdn.rawgit.com/MinEduTDF/manual/master/microservicios.mmd.png)
*Todos los servicios se integran en una única interface simplificando la tarea del usuario.*

## Dependencias

Todas las dependencias serán manejadas mediante la utilización de gestores locales por repositorio (npm, composer).

Esto facilita su actualización, evita la duplicación de código, disminuye el tamaño de los repositorios y permite correr distintas versiones en un mismo sistema.

Otra ventaja es que mediante el uso programático podemos ensayar una actualización que sólo será puesta en producción de resultar exitosa sin mediar intervención humana.


## Guías de estilo

## Documentación

## Tests

## Control de calidad

Una de las prácticas más potentes florecida en el movimiento open source es la exhaustiva utilización de herramientas automatizadas para el control de calidad.

Su implementación nos permite brindar garantías en cuanto a estabilidad, seguridad, legibilidad del código, etc. al tiempo que crecemos como desarrolladores aprendiendo de un feedback objetivo.

En este sentido, el plan de trabajo consiste en:
* Implementar estas herramientas en todos nuestros desarrollos.
* Elevar progresivamente las exigencias en cuanto a complegidad del código, normas de estilo, cobertura de tests, etc.

La idea es permitir durante el tiempo que resulte necesario para apropiarse de las prácticas, la apertura de pull requests que incluyan código pobremente documentado y testeado e incluso la puesta en producción de código con baja puntuación de standares de calidad.

Este período permite continuar con el desarrollo como se venía dando pero enriquecido por el feedback de las herramientas y la tutoría de otros miembros del equipo a travez de aportes en el proceso de pull request.

De esta manera se favorece una familiarización con el proceso sin obstaculizar el trabajo cotidiano permitiendo al desarrollador capitalizar algunos de los beneficios de estas prácticas manteniendo su frustración al mínimo posible.

## Workflow

* 1º- Le pedimos a Juanito que nos prepare un pipeline de entrega contínua para alguno de los siguientes tipos de proyectos:
    
  * Módulo javascript.
  * Aplicación javascript.
  * Módulo php.
  * Aplicación php.
  * Módulo hubot.

* 2º- Unos segundos después, Juanito nos provee la url del nuevo repositorio junto con instrucciones para empezar a trabajar.

* 3º- Clonamos el repositorio.

* 4º- Instalamos dependencias.

* 5º- Creamos un branch de nombre descriptivo.

* 6º- Codeamos actualizando tests y documentación para reflejar los cambios.

* 7º- Ejecutamos git push originnombredelbranch.

* 8º- Se ejecutan una serie de validaciones automáticas.

* 9º- Se crea pull request.

* 10º- Se valida el ódigo en el [servidor de integracion](https://travis.ci).

* 11º- Si todo anduvo bién, Juanito avisa a los responsables de realizar la revisión de pares.

* 12º- Una vez aprobada la pull request se publica una nueva versión, respetando versionado semántico, que se pone en producción automáticamente.

* 13º- Juanito avisa que se ha publicado una nueva versión y felicita al autor.

## Diseño / Experiencia de usuario

## Código de conducta

En función de perseguir la conformación de una comunidad segura e inclusiva, todos nuestros repositorios adherirán como código de conducta al [contributor covenant](http://contributor-covenant.org/version/1/4/es/code_of_conduct.md).
