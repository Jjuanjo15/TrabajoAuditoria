# Actividad: Marcos de Gobierno y Gestión de TI para Kike Parking

## 1. Organización

Nosotros decidimos elegir nuestro proyecto final para hacer la auditoría, su nombre es **Kike Parking**.

## 2. Contexto, Sistemas de información y riesgos

Es una plataforma digital colaborativa que conecta propietarios de espacios de parqueo privados con conductores que necesitan estacionamiento en Medellín. La aplicación maneja datos personales de usuarios, información de vehículos, reservas, pagos electrónicos y geolocalización. Sus principales riesgos son el manejo de la seguridad de la información, protección de datos personales, disponibilidad del servicio, procesamiento de pagos y cumplimiento normativo de la Ley 1581 de 2012.

## 3. Marcos que vamos a usar

1. COBIT
2. ISO/IEC 27001
3. ITIL
4. PCI-DSS

## 4. Justificación

### COBIT

Con el uso de COBIT podemos definir cómo se gobernará la tecnología, quién será responsable de cada proceso y organizar toda la gestión tecnológica del proyecto.

La aplicación depende de varios servicios como:

- Infraestructura en AWS/Azure.
- Google Maps.
- Pasarelas de pago.
- Bases de datos con información sensible.
- Procesos de soporte y operación.

Todo eso necesita ser administrado. COBIT permite alinear la tecnología con los objetivos del negocio mientras se identifican riesgos y se definen procesos de control y supervisión.

### ISO/IEC 27001

Se requiere proteger la información.

Kike Parking almacena:

- Datos personales.
- Correos electrónicos.
- Placas de vehículos.
- Documentos de identidad.
- Información de pagos.

Tenemos la necesidad de proteger datos sensibles y cumplir con la Ley 1581 de 2012, por esta razón ISO 27001 es fundamental para:

- Gestión de riesgos.
- Control de accesos.
- Cifrado.
- Gestión de incidentes.
- Continuidad del negocio.

### ITIL

La aplicación debe garantizar la calidad del servicio.

Kike Parking requiere:

- Disponibilidad continua.
- Atención de incidentes.
- Soporte técnico.
- Gestión de cambios.
- Gestión de capacidad.

Esto es importante porque la aplicación debe funcionar en tiempo real y soportar picos de usuarios.

ITIL ayuda a:

- Reducir caídas.
- Mejorar tiempos de respuesta.
- Gestionar incidentes.
- Mantener niveles de servicio.

### PCI-DSS

El sistema utiliza pagos electrónicos mediante Wompi o Mercado Pago.

Por ello es necesario aplicar controles PCI-DSS para:

- Protección de datos de tarjetas.
- Cifrado de transacciones.
- Gestión segura de pagos.

## 5. Evidencias que solicitaría al equipo de TI

### COBIT

#### Evidencia 1

Mapa de procesos de TI y organigrama con responsables.

Ejemplo:

- Administrador del sistema.
- Responsable de base de datos.
- Responsable de seguridad.
- Responsable de soporte.

#### Evidencia 2

Matriz de riesgos tecnológicos.

Ejemplo:

- Caída de AWS.
- Fraude en pagos.
- Robo de información.
- Fallas de Google Maps API.

### ISO/IEC 27001

#### Evidencia 1

Política de Seguridad de la Información aprobada.

Debe incluir:

- Control de accesos.
- Gestión de contraseñas.
- Manejo de incidentes.

#### Evidencia 2

Registro de análisis y tratamiento de riesgos.

Ejemplos:

- Riesgo de fuga de datos.
- Riesgo de acceso no autorizado.
- Riesgo de pérdida de información.

### ITIL

#### Evidencia 1

Registro de incidentes y solicitudes de soporte.

Ejemplos:

- Errores de reserva.
- Problemas de inicio de sesión.
- Fallos en pagos.

#### Evidencia 2

Acuerdos de Nivel de Servicio (SLA).

Ejemplos:

- Disponibilidad del sistema del 99%.
- Tiempo de respuesta menor a 3 segundos.
- Tiempo máximo de atención de incidentes.

### PCI-DSS

#### Evidencia 1

Documentación de integración con Wompi o Mercado Pago.

Debe demostrar:

- Uso de HTTPS.
- Tokenización de pagos.
- No almacenamiento de números de tarjeta.

#### Evidencia 2

Registros de auditoría y monitoreo de transacciones.

Ejemplos:

- Intentos de fraude.
- Pagos rechazados.
- Accesos administrativos al sistema.