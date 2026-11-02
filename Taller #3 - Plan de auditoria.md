# Plan de auditoría propuesto: control de acceso a los datos de usuarios en Kike Parking

**Materia:** Auditoría de Sistemas (ET0114)  
**Docente:** Juan Duque  
**Integrantes:** Samuel Orozco Gómez y Juan José López Monsalve  
**Entrega final:** domingo 1 de noviembre de 2026, hasta las 11:59 p. m. (hora de Colombia)

> **Nota:** Este documento presenta una propuesta de planificación. La auditoría no se ha ejecutado; por tanto, las evidencias mencionadas son las que se solicitarían u obtendrían y no resultados ya alcanzados.

## 1. Contexto y riesgo seleccionado

Kike Parking es una aplicación planteada para conectar a propietarios de parqueaderos privados con conductores que buscan estacionamiento en Medellín. La plataforma contempla usuarios, parqueaderos, reservas y pagos electrónicos, por lo que manejaría información personal como nombres, correos, placas y documentos de identidad. Es el proyecto final del propio equipo, y su preocupación central en esta auditoría es que los datos personales de los usuarios no queden expuestos.

Se conserva el caso y la matriz de riesgos del taller anterior. El riesgo seleccionado es **R1: que cualquiera pueda ver los datos de todos los usuarios**.

| Elemento de la matriz | Descripción |
|---|---|
| Activo o proceso | A2: servidor de la aplicación, relacionado con A1: base de datos de usuarios |
| Amenaza | Una persona externa o un usuario cualquiera descubre cómo solicitar la lista de usuarios |
| Vulnerabilidad supuesta | La lista podría consultarse sin iniciar sesión ni tener permisos de administrador, y mostraría todos los datos de cada persona |
| Consecuencia posible | Exposición de correos, documentos y placas; incumplimiento de la Ley 1581 de 2012, posibles sanciones y pérdida de confianza de los usuarios |
| Probabilidad | 4 (probable): es fácil de encontrar y no se necesitan conocimientos avanzados |
| Impacto | 5 (grave): quedan expuestos datos personales de todos los usuarios y se incumple la ley |
| Nivel de riesgo | 20/25 |
| Control actual | No hay evidencia de que se pida identificación o permisos para ver esa lista. La matriz sí registra, en R2, que las sesiones usan JWT y que las contraseñas se guardan cifradas con bcrypt |
| Controles propuestos en la matriz | (1) Que solo un administrador con sesión iniciada pueda ver la lista; (2) que la lista no muestre datos innecesarios y se entregue por partes; (3) una prueba que confirme que, sin iniciar sesión, la consulta se rechaza |

Estos valores y justificaciones se mantienen como fueron asignados en la matriz. La vulnerabilidad todavía es un supuesto: no contamos con pruebas que confirmen que la consulta esté abierta.

**Por qué auditar R1.** Además de tener el puntaje más alto, R1 afecta datos personales de todos los usuarios a la vez, lo que lo convierte en un asunto de cumplimiento legal (Ley 1581 de 2012) y no solo técnico. También es un riesgo que se puede revisar de forma concreta: el equipo tiene acceso al código del backend, de modo que el control (autenticación, permisos y campos devueltos) puede contrastarse con el código, la configuración y pruebas controladas, sin depender únicamente de lo que alguien afirme.

**Marcos de referencia.** En el primer trabajo se seleccionaron COBIT, ISO/IEC 27001, ITIL y PCI-DSS. Para este plan se usan ISO/IEC 27001 (control de accesos) y la Ley 1581 de 2012 (tratamiento de datos personales), que son los marcos que la matriz relaciona con R1. Se agrega COBIT como ajuste respecto a la matriz, únicamente para revisar quién responde por el control y cómo se supervisa. ITIL y PCI-DSS no son criterios aquí, porque no se auditará la gestión de incidentes ni el procesamiento de pagos.

## 2. Objetivo de auditoría

**Determinar si las funciones de Kike Parking que consultan datos de usuarios exigen autenticación y aplican permisos según el rol de cada persona, y si muestran únicamente los datos necesarios para su función.**

Para responder este objetivo se propone revisar el código y la configuración del backend y ejecutar pruebas controladas de acceso. Como apoyo, se revisará si existen registros que permitan reconstruir esas consultas. El objetivo no presupone que los controles funcionen o fallen; eso solo podría determinarse al ejecutar la auditoría.

## 3. Alcance

La revisión se concentrará en las funciones que permiten consultar una lista o información de usuarios.

**Se incluye:**
- El backend de Kike Parking y las rutas que devuelven datos de usuarios.
- El inicio de sesión, la validación de la sesión (JWT) y la validación de permisos de esas rutas.
- Las consultas a la base de datos relacionadas con esos datos.
- Los campos que aparecen en las respuestas y los registros disponibles sobre los accesos.

**Población y muestra:** la población serán las rutas, funciones y consultas que puedan devolver datos de usuarios, junto con los roles que tengan acceso. Primero se elaborará un inventario de rutas a partir del código. Se propone revisar todas las rutas que devuelvan la lista general de usuarios y, para las demás, priorizar las que expongan más información o tengan permisos administrativos. No se fija una cantidad porque el inventario aún no se ha levantado.

**Periodo propuesto:** del 1 de agosto al 30 de octubre de 2026. Se propone porque cubre los tres meses previos a este plan y permite revisar las versiones más recientes del código y sus cambios, usando el historial de versiones del repositorio si existe. Si el equipo confirma otras fechas de desarrollo o la aplicación no estuvo operativa en ese intervalo, se ajustaría el periodo y se aclararía que una versión de pruebas no demuestra cómo funciona producción.

**Responsables y custodios de la evidencia:** como el sistema es el proyecto final del propio equipo, quienes explican el proceso y custodian el código, la configuración y la base de datos son los mismos integrantes. Falta confirmar qué integrante desarrolló cada módulo y si existe alguien más con rol de administrador, de base de datos o de seguridad; una persona puede cumplir más de una función.

**Se excluye:** la revisión de pagos, respaldos, disponibilidad de la nube, verificación de propietarios de parqueaderos y una auditoría general de toda la infraestructura. Estos temas no son necesarios para responder el objetivo de R1. Por eso, las conclusiones solo se referirían al control de acceso y a la exposición de datos dentro del alcance definido, no a la seguridad completa de la aplicación.

**Entorno y datos de prueba:** las pruebas se harían en un entorno local o de pruebas, con cuentas de ensayo y datos ficticios. No se descargarían datos personales reales ni se realizarían pruebas destructivas. Si fuera necesario probar en producción, se haría solo con límites acordados y por escrito entre los integrantes.

## 4. Criterios de evaluación

Los criterios son las condiciones con las que se compararía la evidencia. No se cuenta con políticas internas escritas de Kike Parking, así que cada criterio se marca como pendiente de confirmar. Si la política o la matriz de permisos no existe, el equipo propone como criterio provisional los controles planteados en la matriz para R1, identificados como **propuesta del equipo** y no como un requisito de un marco.

| Control | Condición esperada | Fuente y estado |
|---|---|---|
| Autenticación | Las solicitudes sin autorización no deben obtener datos protegidos. | Política de control de acceso de Kike Parking, por confirmar mediante revisión de la documentación existente. Si no existe, se usa como criterio provisional el control propuesto (1) de R1 (propuesta del equipo). ISO/IEC 27001 se usa como referencia general de control de accesos, sin atribuirle requisitos técnicos no verificados. |
| Permisos por rol | Cada persona debe acceder solo a las funciones y datos permitidos para su rol. | Matriz de roles y permisos, por confirmar si existe. COBIT sirve como referencia para revisar responsabilidades y supervisión. |
| Datos mostrados | La respuesta debe incluir solo los campos necesarios para la función. | Definición de campos por rol, por confirmar; si no existe, se toma el control propuesto (2) de R1 (propuesta del equipo). La Ley 1581 de 2012 es referencia pertinente para el tratamiento de datos personales. |
| Registros de acceso | Si existen registros, deben ayudar a revisar la actividad relacionada con las consultas. | Procedimiento de registro y monitoreo, por confirmar. No se da por hecho que exista. |

Antes de las pruebas se reunirían los documentos aplicables y se anotaría su nombre, versión, fecha y responsable. No se inventarán requisitos internos ni se afirmará que un marco establece plazos o configuraciones que no se hayan verificado.

## 5. Programa de trabajo

Se proponen tres procedimientos complementarios: revisar cómo debería funcionar el control, probarlo y comprobar qué registros existen para respaldar la revisión.

### Procedimiento 1. Revisar el diseño del control

**Qué se hará:** elaborar el inventario de rutas del backend y reunir la descripción del sistema y, si existen, la política de acceso y la matriz de roles. Luego se revisará en el código cómo se valida la identidad (verificación del token de sesión) y cómo se deciden los permisos, y se comprobará que esa validación ocurre en el servidor y no solo en la interfaz. Se compararán el código y la configuración con las consultas a la base de datos y con lo que los integrantes declaren sobre el funcionamiento.

**Población o muestra:** las rutas que devuelvan la lista general de usuarios y las funciones que el inventario identifique como capaces de mostrar datos personales.

**Evidencia esperada:** inventario de rutas, política y matriz de permisos si existen, fragmentos relevantes del código y configuración, y notas de la explicación del equipo respaldadas por el código.

**Posibles excepciones:** que no haya permisos definidos, que una ruta no compruebe la identidad, que no se verifique el token o el rol en el servidor, que los permisos se validen solo en la interfaz, o que la configuración no coincida con lo declarado.

### Procedimiento 2. Probar el acceso con distintos perfiles

**Qué se hará:** en el entorno acordado, se probarán las rutas seleccionadas en tres situaciones: solicitud sin iniciar sesión, usuario autenticado sin permisos administrativos y administrador autorizado. Se compararán las respuestas con la matriz de permisos (o con el criterio provisional) y se revisará si aparecen datos que no sean necesarios.

**Población o muestra:** las rutas seleccionadas a partir del inventario, priorizando las de mayor exposición potencial. La cantidad de pruebas se definirá cuando el inventario esté completo.

**Evidencia esperada:** plan de pruebas, solicitudes y respuestas, fecha y entorno de cada prueba, versión del código probada y una tabla con los resultados por ruta y perfil. Cualquier dato personal que aparezca accidentalmente se enmascarará.

**Posibles excepciones:** que una solicitud sin autenticar obtenga datos protegidos, que un usuario común acceda a funciones administrativas o que la respuesta muestre información personal innecesaria. Si una prueba no puede realizarse, también se documentará.

### Procedimiento 3. Revisar los registros disponibles (apoyo)

Este procedimiento no busca por sí solo concluir sobre el control: sirve para corroborar las pruebas del Procedimiento 2 y para evaluar si existe el control detectivo que la matriz plantea para R1.

**Qué se hará:** revisar si la aplicación genera registros de acceso o de errores para las rutas seleccionadas durante el periodo definido. Se revisará qué información guardan y, cuando sea posible, se compararán los eventos con las pruebas ejecutadas. También se revisará quién puede consultar esos registros y cómo se protegen.

**Población o muestra:** los registros disponibles del periodo. Si el volumen es manejable, se revisarán los eventos relacionados con las pruebas; si es alto, se elegirá una muestra que incluya diferentes perfiles y resultados, dejando constancia del método.

**Evidencia esperada:** exportación de registros con datos sensibles enmascarados, descripción de los campos registrados y comparación entre eventos y pruebas.

**Posibles excepciones:** que no existan registros pertinentes, que no permitan identificar el resultado de una solicitud, que haya diferencias con las pruebas o que almacenen datos personales innecesarios.

### Matriz de trazabilidad

| ID del riesgo | Control | Criterio y fuente | Procedimiento | Población o muestra | Evidencia esperada |
|---|---|---|---|---|---|
| R1 | Autenticación | Las consultas protegidas deben exigir autenticación; política por confirmar, criterio provisional: control propuesto (1) de R1. | 1 y 2 | Rutas que devuelvan datos de usuarios; prueba sin autenticar. | Código/configuración y respuesta de prueba. |
| R1 | Permisos por rol | Solo los roles autorizados pueden consultar las funciones correspondientes; matriz de roles por confirmar. | 1 y 2 | Rutas seleccionadas y perfiles disponibles. | Validación de permisos en el código y respuestas por perfil. |
| R1 | Limitación de datos | La respuesta debe incluir solo los campos necesarios; definición interna por confirmar, criterio provisional: control propuesto (2) de R1. | 1 y 2 | Respuestas de las rutas seleccionadas. | Definición de campos autorizados y respuestas comparadas. |
| R1 | Trazabilidad | Los registros existentes deben permitir revisar la actividad pertinente; procedimiento por confirmar. | 3 | Registros disponibles y eventos de prueba. | Exportación de registros y comparación con las pruebas. |

La explicación del equipo sirve para comprender el proceso, pero no bastaría por sí sola para demostrar que un control funciona. Por eso se contrastaría con el código, la configuración, las pruebas y los registros cuando estén disponibles.

## 6. Organización de la ejecución

El siguiente cronograma es una estimación para una futura ejecución. Las duraciones son estimadas, no actividades ya realizadas.

| Fase | Actividades | Responsable | Duración estimada |
|---|---|---|---|
| Preparación | Confirmar alcance, criterios, entorno de pruebas y perfiles de prueba. | Ambos integrantes | 1 día |
| Recolección de información | Reunir políticas y documentación existentes, levantar el inventario de rutas y localizar los registros. | Juan José López Monsalve | 1–2 días |
| Revisión del diseño | Revisar el código, la configuración y las consultas (Procedimiento 1). | Samuel Orozco Gómez | 2 días |
| Pruebas y registros | Probar los tres perfiles y revisar los registros disponibles (Procedimientos 2 y 3). | Ambos integrantes | 2 días |
| Cierre | Completar la matriz de trazabilidad y documentar limitaciones. | Ambos integrantes | 1 día |

**Qué se podría ejecutar realmente y qué quedaría como planificación.**

| Parte del plan | Estado | Razón |
|---|---|---|
| Procedimiento 1 (revisión del diseño en el código) | Ejecutable | El equipo tiene acceso al código del backend. |
| Procedimiento 2 (pruebas por perfil) | Ejecutable | Puede hacerse en un entorno local o de pruebas con datos ficticios. Sus resultados no se extenderían a producción. |
| Procedimiento 3 (registros) | Depende de que la aplicación genere registros | Si no existen, se documenta como limitación. |
| Confirmación de políticas y matriz de permisos | Planificación | Es probable que no estén formalizadas; si no existen, se usa el criterio provisional. |

**Dependencias:** disponer de un entorno de pruebas funcional con cuentas de ensayo para cada perfil, que el código revisado corresponda a la versión del periodo, y que la aplicación genere registros para poder aplicar el Procedimiento 3.

**Recursos necesarios:** acceso al repositorio del backend, documentación técnica disponible, entorno de pruebas, cuentas de ensayo, registros disponibles y una herramienta para enviar solicitudes al servidor.

**Distribución del equipo:** Samuel Orozco Gómez coordinaría la revisión del diseño y organizaría los criterios y evidencias. Juan José López Monsalve organizaría la recolección de documentos, el inventario de rutas y la tabla de rutas y perfiles de prueba. Ambos revisarían la trazabilidad entre el riesgo, el objetivo, los criterios y la evidencia.

## 7. Supuestos, información pendiente y limitaciones

**Hechos respaldados por los trabajos anteriores y por el equipo:**
- El caso elegido es Kike Parking, una plataforma planteada para conectar propietarios de parqueaderos y conductores en Medellín (primer trabajo).
- La matriz previa identifica R1 como el riesgo de mayor nivel, con probabilidad 4, impacto 5 y nivel 20.
- La matriz registra que las sesiones usan JWT y que las contraseñas se guardan cifradas con bcrypt (R2 de la matriz).
- En el primer trabajo se seleccionaron COBIT, ISO/IEC 27001, ITIL y PCI-DSS.
- La posible falta de autenticación y permisos se planteó como un supuesto, no como resultado de una prueba.
- El equipo tiene acceso al código del backend (declaración del equipo).

**Supuestos de planificación:**
- El backend tiene funciones que consultan datos de usuarios; se confirmaría al levantar el inventario.
- Es posible identificar perfiles de acceso (sin sesión, usuario común y administrador) y contar con un entorno de pruebas.
- El periodo del 1 de agosto al 30 de octubre de 2026 tiene versiones y, ojalá, registros disponibles.

**Información por reunir o confirmar:**
- Política de control de acceso y de tratamiento de datos personales, si existen.
- Matriz de roles y permisos, si existe.
- Inventario de rutas que devuelven datos de usuarios.
- Definición de los campos que cada rol puede consultar.
- Registros de acceso y errores disponibles.
- Qué integrante desarrolló cada módulo y si hay otras personas con roles de administración o seguridad.

**Propuesta del equipo:** el objetivo, el alcance, el periodo, los criterios provisionales basados en los controles propuestos de R1 y los tres procedimientos.

**Limitaciones:**
- El equipo audita su propio proyecto, por lo que no hay independencia respecto al sistema revisado. Esto debe tenerse presente al interpretar cualquier resultado.
- No contamos con evidencia de una operación real. Por eso no podemos afirmar que la vulnerabilidad exista, que los controles sean eficaces o que se haya expuesto información antes de ejecutar la revisión.
- Si no se obtiene una política o matriz de permisos, el criterio queda pendiente de confirmar y se aplica el criterio provisional.
- Si no hay registros, solo podrá indicarse que no se contó con evidencia de trazabilidad; eso no demuestra que no hayan ocurrido accesos indebidos.
- Los resultados de pruebas en un entorno de desarrollo o pruebas no se extenderían automáticamente a producción.

## 8. Fuentes y referencia a la matriz

1. Equipo de trabajo. *Actividad: Marcos de Gobierno y Gestión de TI para Kike Parking* (primer trabajo).
2. Equipo de trabajo. *Taller: matriz inicial de riesgos de sistemas — Kike Parking* (Taller #2).
3. Congreso de Colombia. Ley 1581 de 2012, sobre protección de datos personales (texto disponible en el sitio oficial de la Secretaría del Senado).
4. ISO/IEC 27001, como marco general de referencia para la gestión de la seguridad de la información. La edición y los requisitos pertinentes deberán confirmarse antes de ejecutar la auditoría.
5. ISACA. COBIT, como referencia para el gobierno y la gestión de la información y la tecnología.

La matriz de riesgos vigente está en un archivo de Excel en Drive: **[https://docs.google.com/spreadsheets/d/1uPGmBmRTmc8f4-XerBRpHBH9qElMHSMP/edit?usp=drive_link&ouid=111623729669916694762&rtpof=true&sd=true]**. Contiene los seis riesgos (R1 a R6) con sus niveles, controles actuales y propuestos, además de los activos y los supuestos. Es la misma matriz del Taller #2 (*“Taller #2 - Auditoria.md”*), presentada en Excel. La referencia utilizada es **R1: “Que cualquiera pueda ver los datos de todos los usuarios”**. Este plan desarrolla ese riesgo sin reemplazar la matriz original.
