# Taller: matriz inicial de riesgos de sistemas — Kike Parking

**Materia:** Auditoría de Sistemas (ET0114) · Sesión 7
**Docente:** Juan Duque
**Integrantes:**
- Samuel Orozco Gómez
- Juan José López Monsalve

---

## 1. Contexto actualizado del caso

Kike Parking es una aplicación que conecta a dueños de parqueaderos privados con conductores en Medellín. En esta entrega analizamos el servidor y la base de datos donde se guarda la información, el proceso de reservas y pagos con Wompi o Mercado Pago, la infraestructura en la nube y el proceso de publicar parqueaderos. Conservamos los marcos del primer taller (COBIT, ISO/IEC 27001, ITIL y PCI-DSS) para decidir qué controles esperaríamos encontrar. Lo que más nos preocupa es proteger los datos personales de los usuarios (Ley 1581 de 2012), que la app esté disponible y que los pagos sean seguros.

## 2. Supuestos

No tenemos acceso a una operación real, así que lo siguiente son suposiciones, no hechos comprobados.

1. La app permite registrarse, iniciar sesión, ver, consultar y crear parqueaderos, y ver la lista de usuarios. Suponemos que, al salir a producción, estas funciones seguirían sin pedir identificación ni permisos especiales en todos los casos.
2. La app funcionaría en un solo servidor en la nube, con una sola base de datos.
3. Todavía no hay copias de seguridad automáticas que se hayan probado, ni vigilancia del servicio, ni un compromiso formal de disponibilidad.
4. Los pagos se confirmarán con avisos que envía la pasarela de pagos, y la app no guardará números de tarjeta.
5. Se guardan datos personales como nombre, correo, placa y documento de identidad.
6. El inicio de sesión no limita los intentos fallidos ni exige reglas mínimas para las contraseñas.
7. La parte de la app que recibe los avisos de la pasarela no verifica que el aviso sea auténtico ni consulta a la pasarela si el pago fue realmente aprobado antes de confirmar la reserva.
8. Quien publica un parqueadero no tiene que demostrar que es el dueño o que está autorizado para arrendarlo.

## 3. Activos y procesos analizados

| ID | Activo / proceso | Tipo |
|----|------------------|------|
| A1 | Base de datos con la información de usuarios y reservas | Información |
| A2 | Servidor de la app (donde se inicia sesión y se consulta la lista de usuarios) | Aplicación |
| A3 | Proceso de reservas y pagos (Wompi / Mercado Pago) | Proceso |
| A4 | Servidor y servicios en la nube | Infraestructura |
| A5 | Proceso de publicar parqueaderos | Proceso |

## 4. Escala usada

**Probabilidad:** de 1 (muy raro que pase) a 5 (muy probable que pase).
**Impacto:** de 1 (daño pequeño y fácil de arreglar) a 5 (daño grave a la operación, a datos importantes o al cumplimiento de la ley).
**Nivel de riesgo = Probabilidad × Impacto.**

Los valores se justifican con los supuestos de la sección 2 y deberían ajustarse si se obtiene evidencia real del equipo de TI.

## 5. Resumen de la matriz (ordenada por nivel de riesgo)

| # | Activo | Riesgo (resumen) | P | I | Nivel |
|---|--------|------------------|---|---|-------|
| R1 | A2 | Que cualquiera pueda ver los datos de todos los usuarios | 4 | 5 | **20** |
| R2 | A2 | Que alguien adivine contraseñas y entre a cuentas ajenas | 4 | 4 | **16** |
| R3 | A1 | Que se pierdan los datos por no tener respaldos probados | 3 | 5 | **15** |
| R4 | A3 | Que se confirmen reservas sin que se haya pagado de verdad | 3 | 4 | **12** |
| R5 | A4 | Que la app quede fuera de servicio sin que el equipo lo note | 3 | 4 | **12** |
| R6 | A5 | Que se publiquen parqueaderos falsos o que no son del publicante | 3 | 3 | **9** |

---

## 6. Detalle de cada riesgo

### R1 — Que cualquiera pueda ver los datos de todos los usuarios

| Campo | Contenido |
|-------|-----------|
| **Activo / proceso** | A2 Servidor de la app (y A1 base de datos de usuarios) |
| **Amenaza** | Una persona externa, o un usuario cualquiera, que descubre cómo pedir la lista de usuarios |
| **Vulnerabilidad** | La lista de usuarios se puede consultar sin iniciar sesión ni tener permisos de administrador, y muestra todos los datos de cada persona (supuestos 1 y 5) |
| **Riesgo** | Un tercero podría descargar la lista completa de usuarios con sus correos, documentos y placas. Esto sería un incumplimiento de la Ley 1581, podría traer sanciones y haría que los usuarios pierdan la confianza |
| **Probabilidad** | **4 (Probable):** es fácil de encontrar y no se necesitan conocimientos avanzados |
| **Impacto** | **5:** quedan expuestos datos personales de todos los usuarios y se incumple la ley |
| **Nivel** | **20** |
| **Control actual** | No hay evidencia de que se pida identificación ni permisos para ver esa lista (supuesto 1) |
| **Control propuesto** | (1) Que solo un administrador que haya iniciado sesión pueda ver la lista (preventivo). (2) Que la lista no muestre datos innecesarios como la contraseña o el documento, y que se muestre por partes y no toda de una vez (preventivo). (3) Hacer una prueba que confirme que, sin iniciar sesión, la app rechaza la consulta (detectivo) |
| **Marco relacionado** | ISO/IEC 27001 (control de accesos) y Ley 1581 de 2012 |
| **Justificación de prioridad** | Es el riesgo más alto porque es muy fácil de aprovechar y tendría las peores consecuencias legales y de reputación |

### R2 — Que alguien adivine contraseñas y entre a cuentas ajenas

| Campo | Contenido |
|-------|-----------|
| **Activo / proceso** | A2 Servidor de la app (inicio de sesión) |
| **Amenaza** | Una persona o programa que prueba muchas contraseñas, o usa contraseñas filtradas de otros sitios, hasta lograr entrar |
| **Vulnerabilidad** | No hay un límite de intentos fallidos al iniciar sesión ni reglas mínimas para las contraseñas (supuesto 6) |
| **Riesgo** | Un atacante podría entrar a cuentas de conductores o dueños, hacer reservas a nombre de ellos y ver sus datos personales |
| **Probabilidad** | **4 (Probable):** este tipo de ataque es muy común en apps donde cualquiera puede registrarse |
| **Impacto** | **4:** afecta los datos y las reservas de varios usuarios, pero no de toda la base |
| **Nivel** | **16** |
| **Control actual** | Las contraseñas se guardan cifradas con bcrypt y las sesiones usan JWT, lo que reduce el daño si la base se filtra. No hay evidencia de límite de intentos ni de reglas para las contraseñas (supuesto 6). Evidencia que revisaríamos: configuración del inicio de sesión y registros de intentos fallidos |
| **Control propuesto** | (1) Bloquear el inicio de sesión durante 15 minutos después de 5 intentos fallidos seguidos por cuenta o por dirección de origen (preventivo). (2) Exigir contraseñas de mínimo 8 caracteres que no sean de uso común (preventivo). (3) Registrar los intentos fallidos y revisar cada semana las cuentas con muchos intentos (detectivo). Se verifica con una prueba que, en el sexto intento fallido, la app rechace el acceso |
| **Marco relacionado** | ISO/IEC 27001 (política de seguridad: gestión de contraseñas y control de accesos) |
| **Justificación de prioridad** | Es el segundo riesgo porque abre la puerta a los datos de varios usuarios y es un ataque común y barato de intentar. Queda por debajo de R1 porque requiere más intentos y afecta cuentas individuales y no a toda la base |

### R3 — Que se pierdan los datos por no tener respaldos probados

| Campo | Contenido |
|-------|-----------|
| **Activo / proceso** | A1 Base de datos con la información de usuarios y reservas |
| **Amenaza** | Una falla del proveedor de la base de datos, un borrado accidental o un ataque que elimine o cifre la información |
| **Vulnerabilidad** | No hay copias de seguridad automáticas que se hayan probado ni un plan de recuperación (supuesto 3) |
| **Riesgo** | Podrían perderse usuarios, reservas y parqueaderos publicados, y la app quedaría sin funcionar hasta reconstruir los datos, sin forma de saber qué reservas y pagos existían |
| **Probabilidad** | **3 (Posible):** el servicio funcionaría con un solo servidor y una sola base de datos, sin respaldos verificados (supuestos 2 y 3) |
| **Impacto** | **5:** se pierden datos críticos y se afecta toda la operación y las obligaciones con los usuarios |
| **Nivel** | **15** |
| **Control actual** | No hay evidencia de respaldos automáticos ni de una prueba de restauración (supuesto 3) |
| **Control propuesto** | (1) Respaldo automático diario de la base de datos, con 30 días de retención y guardado fuera del servidor principal (preventivo). (2) Prueba de restauración cada trimestre en un entorno aparte, con un acta que deje constancia del resultado (detectivo). (3) Definir por escrito cuánta información se podría perder como máximo (por ejemplo, 24 horas) y quién ejecuta la recuperación (correctivo) |
| **Marco relacionado** | ISO/IEC 27001 (continuidad del negocio) y COBIT (responsable de base de datos definido en el mapa de procesos) |
| **Justificación de prioridad** | La pérdida de datos no se puede deshacer y hoy no hay forma de comprobar que se puedan recuperar. Queda por debajo de R1 y R2 porque su probabilidad es menor |

### R4 — Que se confirmen reservas sin que se haya pagado de verdad

| Campo | Contenido |
|-------|-----------|
| **Activo / proceso** | A3 Proceso de reservas y pagos |
| **Amenaza** | Un usuario malintencionado que envía un aviso de pago falso o repite uno antiguo para que la app crea que pagó |
| **Vulnerabilidad** | La app confirma la reserva con el aviso que recibe, sin comprobar que viene de la pasarela ni consultar si el pago fue aprobado (supuesto 7) |
| **Riesgo** | Podrían confirmarse reservas sin pago real. Los propietarios perderían ingresos y la plataforma tendría diferencias en sus cuentas y reclamos de usuarios |
| **Probabilidad** | **3 (Posible):** requiere conocer cómo la app recibe los avisos, pero no es difícil si no hay ninguna verificación |
| **Impacto** | **4:** afecta los ingresos y la confianza de propietarios y conductores, aunque no expone datos personales |
| **Nivel** | **12** |
| **Control actual** | No tenemos la documentación de la integración con la pasarela (es una de las evidencias que pedimos en el primer taller), así que asumimos que no se verifica el aviso (supuesto 7). La app no guarda números de tarjeta (supuesto 4), lo que reduce el alcance de PCI-DSS |
| **Control propuesto** | (1) Verificar la firma de cada aviso con el secreto que entrega la pasarela y rechazar los que no coincidan (preventivo). (2) Antes de confirmar la reserva, consultar a la pasarela el estado de la transacción y confirmar solo si está aprobada y el monto coincide (preventivo). (3) Conciliación diaria entre reservas confirmadas y transacciones aprobadas, con un reporte de diferencias revisado por el responsable de pagos (detectivo) |
| **Marco relacionado** | PCI-DSS (gestión segura de pagos y registros de transacciones) y COBIT (supervisión del proceso) |
| **Justificación de prioridad** | Causa pérdida económica directa, pero es menos probable que R1 a R3 y no expone datos personales |

### R5 — Que la app quede fuera de servicio sin que el equipo lo note

| Campo | Contenido |
|-------|-----------|
| **Activo / proceso** | A4 Servidor y servicios en la nube |
| **Amenaza** | Una falla del proveedor de nube o del servidor, o un aumento fuerte de usuarios al mismo tiempo |
| **Vulnerabilidad** | Todo funciona en un solo servidor, sin vigilancia del servicio ni compromiso formal de disponibilidad (supuestos 2 y 3) |
| **Riesgo** | La app podría quedar caída durante horas sin que el equipo se entere a tiempo, y los conductores no podrían reservar ni los propietarios recibir pagos |
| **Probabilidad** | **3 (Posible):** un solo servidor sin respaldo de servicio puede caerse, y cualquier proveedor tiene fallas ocasionales |
| **Impacto** | **4:** se interrumpe un servicio importante (reservas y pagos), aunque no hay pérdida ni exposición de datos |
| **Nivel** | **12** |
| **Control actual** | No hay evidencia de monitoreo ni de un acuerdo de nivel de servicio (supuesto 3) |
| **Control propuesto** | (1) Un monitoreo externo que consulte la app cada minuto y envíe una alerta al responsable de soporte cuando no responda (detectivo). (2) Definir un acuerdo interno de nivel de servicio con disponibilidad mensual del 99% y respuesta menor a 3 segundos, como propusimos en el primer taller, con un reporte mensual (detectivo). (3) Un procedimiento escrito para restablecer el servicio, con responsables y tiempo máximo de atención (correctivo) |
| **Marco relacionado** | ITIL (disponibilidad, gestión de incidentes y niveles de servicio) y COBIT |
| **Justificación de prioridad** | Detiene el negocio mientras dure la caída, pero es recuperable y no compromete datos, por eso queda por debajo de los riesgos que exponen o pierden información |

### R6 — Que se publiquen parqueaderos falsos o que no son del publicante

| Campo | Contenido |
|-------|-----------|
| **Activo / proceso** | A5 Proceso de publicar parqueaderos |
| **Amenaza** | Una persona que publica un parqueadero inexistente, o que es de otro, para cobrar reservas |
| **Vulnerabilidad** | Para publicar no se verifica que la persona sea la dueña o esté autorizada (supuesto 8) |
| **Riesgo** | Un conductor podría reservar y pagar un parqueadero que no existe o que no puede usar, lo que generaría reclamos, devoluciones y pérdida de confianza en la plataforma |
| **Probabilidad** | **3 (Posible):** cualquiera puede registrarse, aunque el fraude exige que alguien reserve y pague |
| **Impacto** | **3:** afecta a los conductores que caigan en la publicación, no a toda la operación |
| **Nivel** | **9** |
| **Control actual** | No hay evidencia de un proceso de verificación de propietarios (supuesto 8) |
| **Control propuesto** | (1) Pedir a cada propietario un documento de identidad y un soporte de que puede arrendar el espacio (por ejemplo, certificado de tradición o autorización del administrador), y activar la publicación solo después de que un responsable la revise (preventivo). (2) Un botón para reportar publicaciones sospechosas, con baja en máximo 24 horas mientras se revisa (correctivo). (3) Registrar quién publicó, cuándo y quién aprobó cada parqueadero (detectivo) |
| **Marco relacionado** | COBIT (procesos y responsables definidos) e ISO/IEC 27001 (control de accesos y registros) |
| **Justificación de prioridad** | Tiene el nivel más bajo porque el daño queda limitado a casos individuales, pero afecta la reputación del servicio |