# Informe de auditoría y plan de acción: control de acceso a los datos de usuarios en Kike Parking

**Materia:** Auditoría de Sistemas (ET0114) · Taller final
**Docente:** Juan Duque

## 1. Identificación

| Campo | Contenido |
|---|---|
| Integrantes | Samuel Orozco Gómez y Juan José López Monsalve |
| Caso | Kike Parking, app que conecta propietarios de parqueaderos con conductores en Medellín (proyecto final del equipo) |
| Sistema | Backend `kikeparking-api` (Node.js, Express, PostgreSQL) |
| Riesgo | R1: que cualquiera pueda ver los datos de todos los usuarios (nivel 20) |
| Fecha de las pruebas | 26 de noviembre de 2026 |
| Tipo de ejercicio | **Mixto:** código y pruebas **reales**; datos **ficticios** en un entorno **local**. No se probó producción |

## 2. Resumen ejecutivo

Auditamos R1, el riesgo más alto de la matriz, ejecutando los tres procedimientos del plan: revisión del código, pruebas de acceso con distintos perfiles y revisión de registros. La vulnerabilidad, que en el plan era un supuesto, **quedó confirmada**: la ruta `GET /users` entrega el nombre, el correo, el rol y la foto de todos los usuarios a cualquier solicitud, incluso sin iniciar sesión. Se documentan tres hallazgos y una mejora. La acción principal propuesta es retirar la ruta, que la app no usa, y repetir las pruebas para comprobar el cierre.

## 3. Objetivo y alcance

**Objetivo (sin cambios respecto al plan).** Determinar si las funciones que consultan datos de usuarios exigen autenticación, aplican permisos por rol y muestran solo los datos necesarios.

**Alcance.** De las 37 rutas del backend, 10 devuelven datos de usuarios (inventario en el Anexo A). Se revisó el código de las 10 y se probó con solicitudes la única que lista a todos los usuarios, `GET /users`.

**Periodo.** El plan fijó del 1 de agosto al 30 de octubre de 2026. La ruta se creó el 15 de octubre (commit `e4435ad`). Revisamos el código de la última versión del periodo (`7a69abb`, 28 de octubre) y de la versión actual (`e73faef`, 12 de noviembre). **Ajuste:** las pruebas se hicieron sobre la versión actual, porque es la vigente; el código de la ruta es igual en ambas.

**Exclusiones.** Pagos, respaldos, disponibilidad, verificación de propietarios, el servidor de producción (Render) y la base de datos real (Neon).

## 4. Criterios y metodología

No existen una política de acceso ni una matriz de roles escritas: los repositorios no tienen documentos de ese tipo. Por eso, como preveía el plan, usamos **criterios provisionales** basados en los controles propuestos para R1 en la matriz (**propuesta del equipo**, no requisitos de un marco).

| ID | Control | Condición esperada | Fuente |
|---|---|---|---|
| C1 | Autenticación | Sin un token válido no se obtiene la lista | Control propuesto 1 de R1. Referencia: ISO/IEC 27001 |
| C2 | Permisos por rol | Solo un administrador puede ver la lista | Control propuesto 1 de R1. Referencia: COBIT |
| C3 | Limitación de datos | La lista no muestra datos innecesarios y se entrega por partes | Control propuesto 2 de R1. Referencia: Ley 1581 de 2012 |
| C4 | Trazabilidad | Si hay registros, deben permitir revisar las consultas | Plan, procedimiento 3 (condicional) |

**Procedimientos:**
1. **Diseño:** revisión del código de las 10 rutas (middlewares, servicio, consulta y campos devueltos) en las dos versiones.
2. **Pruebas:** solicitudes con `curl` a `GET /users` con cuatro perfiles (sin token, conductor, propietario y token inventado) y tres solicitudes de contraste. La base local tenía 5 usuarios ficticios. El perfil **administrador no se pudo probar** porque el sistema solo tiene los roles `conductor` y `propietario`.
3. **Registros:** revisión de la configuración del logger y de la salida del servidor durante las pruebas.

El papel de trabajo, con los pasos para repetir la prueba, está en el **Anexo A**.

## 5. Resultados y hallazgos

| ID | Solicitud | Perfil | Esperado | Obtenido | ¿Excepción? |
|---|---|---|---|---|---|
| P01 | `GET /users` | Sin token | 401 | **200**, lista de 5 usuarios con correo | **Sí** |
| P02 | `GET /users` | Conductor | 403 | **200**, misma lista | **Sí** |
| P03 | `GET /users` | Propietario | 403 | **200**, misma lista | **Sí** |
| P04 | `GET /users` | Token inventado | 401 | **200**, misma lista | **Sí** |
| P05 | `GET /users/me/rating` | Sin token | 401 | 401 | No |
| P06 | `GET /bookings/received` | Conductor | 403 | 403 | No |
| P07 | `GET /parking-spots/1` | Sin token | Sin datos innecesarios | 200, nombre y foto del propietario, sin correo | No |

Las pruebas de contraste muestran que la autenticación y los roles **sí funcionan donde se aplican**. Las otras 9 rutas revisadas no muestran el correo de terceros. Ninguna respuesta incluye la contraseña. Ninguna de las 21 solicitudes quedó registrada en la salida del servidor (Anexo B).

### H-01. La lista de usuarios se entrega sin autenticación

1. **Criterio:** C1.
2. **Condición:** en `src/routes/v1/users.route.ts`, la ruta `router.get("/", ...)` no usa `requireAuth` en ninguna de las dos versiones revisadas. P01 y P04 obtuvieron la lista completa.
3. **Causa:** pendiente de confirmar. Según el equipo, la ruta se creó en desarrollo y su retiro quedó aplazado; no hay un documento que lo respalde.
4. **Efecto:** *posible*: cualquiera con la URL podría descargar el nombre y el correo de todos los usuarios, en contra de la Ley 1581 de 2012. *Observado*: solo en local con datos ficticios; no demuestra que alguien la haya consultado en producción.
5. **Recomendación:** retirar la ruta; la app móvil no la llama. Si se necesita, protegerla con `requireAuth` y un rol de administrador.

### H-02. Cualquier rol puede ver la lista y no hay rol de administrador

1. **Criterio:** C2.
2. **Condición:** la ruta no usa `requireRole`. P02 y P03 obtuvieron la lista. El sistema no tiene un rol de administrador ni una matriz de roles escrita.
3. **Causa:** pendiente de confirmar.
4. **Efecto:** *posible*: aun con autenticación, cualquier persona que se registre vería los datos de todos.
5. **Recomendación:** documentar una matriz de roles por ruta y, si la lista es necesaria, crear un rol de administrador que no se pueda elegir al registrarse.

### H-03. La lista expone el correo de todos y se entrega completa

1. **Criterio:** C3.
2. **Condición:** cada usuario se devuelve con `id`, `name`, `email`, `role` y `avatarUrl`. La consulta (`findMany`) no tiene límite ni paginación: P01 trajo los 5 usuarios de una vez.
3. **Causa:** pendiente de confirmar. La ruta usa el mismo formato que el perfil propio, que sí necesita el correo.
4. **Efecto:** *posible*: con una sola solicitud se obtendrían todos los correos registrados. **Corrección a la matriz:** la base de datos no guarda documento de identidad ni placa (supuesto 5), así que el impacto real se limita a nombre, correo, rol y foto.
5. **Recomendación:** si la ruta se mantiene, quitar el correo de los listados y paginar los resultados. Revisar el impacto de R1 en la matriz.

### M-01. Mejora: no hay registros de acceso

No es un hallazgo porque C4 era condicional. La aplicación no registra las solicitudes: el logger solo anota el arranque, los errores y los pagos simulados. Sin registros no se puede saber si la ruta de H-01 fue consultada por terceros.

## 6. Conclusión y limitaciones

**Conclusión.** Dentro del alcance, la evidencia muestra que `GET /users` **no aplica** los controles de autenticación, permisos por rol ni limitación de datos definidos para R1. El supuesto de la matriz pasa a ser un hecho comprobado en el código. Las demás rutas revisadas sí exigen sesión cuando corresponde y no exponen el correo de terceros.

**Límites de la conclusión:**
- No permite afirmar que hubo accesos indebidos ni fuga de datos, porque no hay registros y no se revisó producción.
- Las pruebas fueron en local con 5 usuarios ficticios; no se probó el servidor de producción.
- No hay independencia: el equipo audita su propio proyecto.
- Los criterios son provisionales, porque no existen políticas aprobadas.

## 7. Plan de acción

Responsables y plazos **propuestos**: no se han acordado ni implementado.

| ID | Acción propuesta | Responsable propuesto | Prioridad y justificación | Plazo propuesto | Evidencia de cierre |
|---|---|---|---|---|---|
| H-01 | Retirar `GET /users` o, si se mantiene, agregar `requireAuth` | Desarrollador del backend | **Alta:** nivel 20, se reproduce sin conocimientos técnicos y afecta a todos los usuarios | 1 semana | Pull Request aprobado y repetición de P01 y P04 con respuesta 404 o 401 |
| H-02 | Documentar la matriz de roles y, si se necesita la lista, crear un rol de administrador y aplicar `requireRole` | Líder técnico | **Media:** si se retira la ruta el riesgo inmediato desaparece, pero la matriz sirve para revisar las demás | 30 días | Matriz publicada en el repositorio y repetición de P02 y P03 con respuesta 403 |
| H-03 | Quitar el correo de los listados, paginar los resultados y actualizar el impacto de R1 en la matriz | Desarrollador del backend y equipo auditor | **Media:** reduce la exposición si falla un control preventivo | 2 semanas | Respuesta sin `email` y con límite de registros; matriz actualizada |
| M-01 (mejora) | Registrar fecha, ruta, código de respuesta e id del usuario de cada solicitud, sin datos personales | Desarrollador del backend | **Media:** es el control detectivo que falta | 30 días | Salida del servidor con las solicitudes de una nueva ejecución de P01 a P07 |

## 8. Referencias

1. *Actividad: Marcos de Gobierno y Gestión de TI para Kike Parking* (primer trabajo).
2. *Taller #2: matriz inicial de riesgos* (`Taller #2 - Auditoria.md`), fila **R1**. Matriz en Excel: [Drive](https://docs.google.com/spreadsheets/d/1uPGmBmRTmc8f4-XerBRpHBH9qElMHSMP/edit?usp=drive_link&ouid=111623729669916694762&rtpof=true&sd=true).
3. *Plan de auditoría* (`Taller #3 - Plan de auditoria.md`).
4. Backend: <https://github.com/005Sorozg/kikeparking-api> · App: <https://github.com/005Sorozg/kikeparking_p>.
5. Ley 1581 de 2012 (protección de datos personales) · ISO/IEC 27001 (referencia general de control de accesos) · ISACA, COBIT.

---

### Anexo A. Papel de trabajo PT-R1-01

| Campo | Contenido |
|---|---|
| Objetivo | Comprobar si `GET /users` exige autenticación, limita por rol y devuelve solo datos necesarios |
| Criterio | C1, C2 y C3 (sección 4), criterio provisional |
| Población | 10 rutas que devuelven datos de usuarios (tabla abajo), código del 15 de octubre al 12 de noviembre de 2026 |
| Muestra | Código: 10 de 10 rutas. Pruebas: `GET /users` con 4 perfiles y 3 solicitudes de contraste. Selección dirigida a la ruta de mayor exposición |
| Entorno | Versión `e73faef` en local con PostgreSQL. 5 usuarios **ficticios** |
| Resultado | 4 de 4 solicitudes a `GET /users` con excepción; 3 de 3 de contraste sin excepción |
| Limitaciones | Sin perfil administrador; sin pruebas en producción; sin registros de acceso |

**Inventario de rutas con datos de usuarios:**

| Ruta | Datos | De quién |
|---|---|---|
| `GET /users` | id, nombre, correo, rol, foto | **Todos los usuarios** |
| `POST /users/register`, `POST /users/login`, `PATCH /users/me`, `POST` y `DELETE /users/me/avatar` | id, nombre, correo, rol, foto | El propio usuario |
| `GET /parking-spots/:id` | nombre, foto, calificación | El propietario |
| `GET /parking-spots/:id/reviews` | nombre, foto | Autores de reseñas |
| `GET /bookings/received`, `GET /bookings/mine` | nombre, calificación | Conductores |
| `GET /conversations` | nombre, foto | La otra persona del chat |

**Pasos para repetir la prueba:**
1. Clonar `kikeparking-api` y ubicar la versión: `git checkout e73faef`.
2. Ver la historia de la ruta: `git log --follow -- src/routes/v1/users.route.ts`. Comparar la ruta en `7a69abb` y `e73faef` con `git show`.
3. Configurar `.env` con una base **de pruebas**, ejecutar las migraciones, arrancar con `npm run dev` y crear usuarios ficticios con `POST /users/register`.
4. Ejecutar las solicitudes y comparar con la tabla de la sección 5:

```bash
B=http://localhost:3000/kikeparking-api/v1
curl -i $B/users                                               # P01
curl -i -H "Authorization: Bearer <token conductor>" $B/users  # P02
curl -i -H "Authorization: Bearer <token propietario>" $B/users # P03
curl -i -H "Authorization: Bearer token-falso" $B/users        # P04
curl -i $B/users/me/rating                                     # P05
curl -i -H "Authorization: Bearer <token conductor>" $B/bookings/received  # P06
curl -i $B/parking-spots/1                                     # P07
```

5. Revisar la salida del servidor durante las solicitudes (C4).

### Anexo B. Evidencia (correos enmascarados)

**P01, HTTP 200** (P02 a P04 idénticas):

| id | name | email | role |
|---|---|---|---|
| 1 | Pedro | p\*\*\*@x.co | propietario |
| 2 | Ana | a\*\*\*@x.co | conductor |
| 3 | Laura Ficticia | l\*\*\*@ejemplo.test | conductor |
| 4 | Carlos Ficticio | c\*\*\*@ejemplo.test | conductor |
| 5 | Marta Ficticia | m\*\*\*@ejemplo.test | propietario |

**P05:** HTTP 401 `AUTH_TOKEN_MISSING` · **P06:** HTTP 403 `AUTH_FORBIDDEN_ROLE` · **P07:** el propietario se devuelve sin `email`.

**Código, `users.route.ts` (`e73faef`):**

```ts
router.get("/", async (req, res) => { ... });                       // línea 22: sin requireAuth
router.get("/me/rating", requireAuth, async (req, res) => { ... }); // línea 29: con requireAuth
```

**Salida completa del servidor durante las 21 solicitudes:** solo los tres mensajes de arranque (versión, entorno y puerto); ninguna solicitud registrada.