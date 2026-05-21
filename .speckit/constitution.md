# Constitución de Arquitectura y Desarrollo Especificado por Contratos (SDD)

## 1. Propósito

Este documento establece la constitución técnica, arquitectónica y de gobernanza para el ecosistema **Cinelogía**, compuesto por una arquitectura orientada a microservicios distribuida en múltiples repositorios. Su finalidad es definir principios obligatorios, restricciones de diseño, estándares operativos y prácticas de **Spec-Driven Development (SDD)** para garantizar coherencia, mantenibilidad, escalabilidad, seguridad y trazabilidad en la evolución del sistema.

La constitución es normativa. Toda decisión de arquitectura, implementación, integración, despliegue y evolución funcional debe alinearse con este documento o justificar explícitamente su excepción mediante un registro de decisión arquitectónica.

## 2. Alcance del sistema

La constitución aplica al sistema compuesto por los siguientes repositorios:

- `jorge90c/17.-Cinelogia-Client`
- `jorge90c/17.-Cinelogia-Gateway`
- `jorge90c/17.-Cinelogia-Security`
- `jorge90c/17.-Cinelogia---APIRest`
- `jorge90c/17.-Cinelogia-Eureka`
- `jorge90c/TFM-Documentaci-n` como repositorio de documentación, gobierno técnico y especificaciones transversales.

## 3. Visión arquitectónica

Cinelogía adopta una arquitectura de microservicios con separación explícita de responsabilidades:

- **Client**: interfaz de usuario basada en Thymeleaf, Bootstrap y componentes de presentación.
- **Gateway**: punto único de entrada, enrutamiento centralizado, políticas transversales y control de acceso perimetral.
- **Security**: microservicio especializado en capacidades de autenticación, autorización y/o gestión de identidad y seguridad.
- **API Rest**: microservicio de negocio y exposición de funcionalidades del dominio mediante APIs REST.
- **Eureka**: registro y descubrimiento dinámico de servicios.
- **TFM-Documentaci-n**: repositorio fuente de verdad documental para especificaciones, lineamientos y decisiones arquitectónicas.

La arquitectura debe favorecer bajo acoplamiento, alta cohesión, versionado explícito de contratos, autonomía operativa y evolución incremental segura.

## 4. Principios constitucionales

### 4.1 La especificación precede a la implementación
Ningún cambio funcional relevante debe implementarse sin una especificación previa, explícita y revisable. La especificación define el comportamiento esperado, reglas de negocio, contratos, restricciones, escenarios de error y criterios de aceptación.

### 4.2 Los contratos son la fuente de verdad entre servicios
Toda integración entre componentes debe estar gobernada por contratos versionados, verificables y trazables. Las APIs, eventos, DTOs compartidos y reglas de intercambio no pueden depender de supuestos implícitos.

### 4.3 Cada microservicio es dueño de su responsabilidad
Cada servicio debe poseer una responsabilidad funcional clara, límites definidos y mínima dependencia de detalles internos de otros servicios.

### 4.4 La persistencia es privada por servicio
Los servicios con base de datos son propietarios exclusivos de su esquema y modelo persistente. Ningún otro servicio debe acceder directamente a la base de datos de otro servicio.

### 4.5 La seguridad es transversal y verificable
La seguridad no es opcional ni periférica. Debe estar presente en autenticación, autorización, transporte, validación de entradas, gestión de secretos, auditoría y tratamiento de errores.

### 4.6 Todo cambio debe ser observable
Toda capacidad crítica del sistema debe generar evidencia operativa mediante logs, métricas, trazas y estados de salud.

### 4.7 La compatibilidad hacia atrás es una responsabilidad explícita
Los cambios en contratos públicos deben diseñarse para evitar rupturas abruptas. Si una ruptura es inevitable, debe versionarse y comunicarse formalmente.

### 4.8 Las decisiones arquitectónicas deben quedar registradas
Toda excepción, desviación o cambio estructural relevante debe documentarse mediante ADRs (Architecture Decision Records).

## 5. Enfoque de Spec-Driven Development (SDD)

En Cinelogía, el ciclo de trabajo obligatorio para nuevas capacidades y cambios significativos será:

1. **Definición del problema**: necesidad, contexto, objetivo de negocio y alcance.
2. **Especificación funcional**: comportamiento esperado, actores, flujos, validaciones, errores y criterios de aceptación.
3. **Definición de contrato**: endpoints, payloads, códigos de respuesta, restricciones, autenticación y versionado.
4. **Diseño técnico**: componentes impactados, dependencias, persistencia, riesgos, observabilidad y estrategia de pruebas.
5. **Implementación**: desarrollo alineado a la especificación aprobada.
6. **Verificación**: pruebas unitarias, integración, contrato, seguridad y aceptación.
7. **Trazabilidad**: relación entre especificación, commits, pull requests, despliegues y evidencias.

Ningún desarrollo se considerará completo si no puede trazarse desde una especificación aprobada hasta una evidencia verificable de cumplimiento.

## 6. Estructura mínima de una especificación

Toda especificación en este ecosistema debe incluir como mínimo:

- Título y propósito
- Contexto y problema que resuelve
- Alcance y fuera de alcance
- Actores o consumidores
- Reglas de negocio
- Casos de uso o flujo principal y alternos
- Contratos de entrada y salida
- Manejo de errores
- Requisitos de seguridad
- Impacto en datos y persistencia
- Consideraciones de observabilidad
- Criterios de aceptación verificables
- Riesgos, supuestos y dependencias

## 7. Gobierno de microservicios

### 7.1 Responsabilidades
Cada microservicio debe tener:

- propósito claramente documentado;
- fronteras funcionales explícitas;
- contrato público identificable;
- configuración externa y no hardcodeada;
- estrategia de salud y monitoreo;
- pruebas automatizadas acordes al riesgo.

### 7.2 Acoplamiento
Se prohíbe:

- compartir tablas entre microservicios;
- acoplar lógica de negocio al gateway;
- depender de clases internas de otro repositorio como contrato implícito;
- duplicar reglas críticas sin estrategia de consistencia definida.

### 7.3 Descubrimiento de servicios
El descubrimiento mediante Eureka debe utilizarse para localización dinámica entre servicios, pero no sustituye la definición formal de contratos ni el versionado de APIs.

## 8. Reglas por componente

### 8.1 Client (`17.-Cinelogia-Client`)

- Debe enfocarse en experiencia de usuario, composición de vistas y consumo controlado de servicios.
- No debe contener lógica de negocio crítica que deba residir en backend.
- Debe validar entradas a nivel de UX sin reemplazar validación de servidor.
- Debe manejar errores de integración de forma comprensible para el usuario.
- Debe desacoplar la presentación de detalles internos de los microservicios.

### 8.2 Gateway (`17.-Cinelogia-Gateway`)

- Es el punto de entrada unificado del sistema.
- Debe centralizar concerns transversales: routing, filtros comunes, seguridad perimetral, trazabilidad y políticas técnicas.
- No debe contener lógica de negocio de dominio.
- Debe soportar versionado y trazabilidad de peticiones.
- Debe propagar de forma segura los contextos de autenticación y correlación.

### 8.3 Security (`17.-Cinelogia-Security`)

- Es responsable de capacidades de seguridad del dominio asignado.
- Toda decisión de autenticación o autorización debe estar documentada como contrato y regla verificable.
- Debe persistir únicamente los datos de seguridad que le pertenecen.
- Debe aplicar cifrado, hashing y manejo seguro de credenciales conforme a estándares actuales.
- Debe auditar eventos sensibles de seguridad.

### 8.4 API Rest (`17.-Cinelogia---APIRest`)

- Debe encapsular la lógica de negocio propia del dominio expuesto.
- Sus APIs deben ser consistentes, documentadas y versionables.
- Debe aislar controladores, servicios, repositorios y DTOs con responsabilidades claras.
- No debe exponer entidades persistentes directamente como contrato externo.
- Debe validar reglas de negocio en backend de forma determinística.

### 8.5 Eureka (`17.-Cinelogia-Eureka`)

- Debe limitarse a funciones de registro y descubrimiento.
- No debe asumir responsabilidades funcionales del dominio.
- Debe operar como infraestructura liviana, estable y observable.

### 8.6 Documentación (`TFM-Documentaci-n`)

- Es la fuente documental oficial del sistema.
- Debe contener constitución, especificaciones, ADRs, diagramas, catálogos de contratos y convenciones.
- Toda documentación debe mantenerse sincronizada con la evolución efectiva del sistema.

## 9. Persistencia y reglas de datos

Los microservicios `17.-Cinelogia-Security` y `17.-Cinelogia---APIRest` poseen persistencia sobre **MySQL**. Para ellos rigen las siguientes normas obligatorias:

### 9.1 Propiedad del dato
Cada servicio es dueño exclusivo de su modelo de datos, su esquema y su evolución. Está prohibido el acceso cruzado directo a tablas de otro microservicio.

### 9.2 Diseño del esquema
- El esquema debe reflejar el dominio del servicio y no necesidades de otros servicios.
- Las claves primarias, foráneas internas e índices deben responder a integridad y rendimiento propios del servicio.
- Toda convención de nombres debe ser consistente y documentada.

### 9.3 Evolución del esquema
- Todo cambio estructural en base de datos debe versionarse mediante migraciones controladas.
- Las migraciones deben ser repetibles, trazables y reversibles cuando sea viable.
- No se permiten cambios manuales en producción sin registro formal.

### 9.4 Integridad y consistencia
- Las validaciones de integridad deben combinar controles de aplicación y restricciones de base de datos cuando corresponda.
- Las transacciones deben definirse con el menor alcance posible.
- Se debe evitar la lógica de negocio compleja oculta en la base de datos salvo justificación explícita.

### 9.5 Seguridad de datos
- Las credenciales de conexión nunca deben residir hardcodeadas en el código fuente.
- Los datos sensibles deben protegerse con cifrado, hashing o enmascaramiento según corresponda.
- El acceso a datos debe seguir el principio de mínimo privilegio.

### 9.6 Rendimiento
- Toda consulta crítica debe ser medible y optimizable.
- Debe evitarse el N+1, cargas excesivas y consultas no indexadas en rutas críticas.
- Los DTOs y proyecciones deben utilizarse cuando reduzcan acoplamiento y costo de transferencia.

## 10. Contratos de APIs

### 10.1 Principios
Toda API REST debe:

- estar documentada;
- estar versionada cuando sea pública o consumida externamente;
- definir claramente formatos de request/response;
- devolver códigos HTTP coherentes;
- exponer errores estructurados y accionables;
- declarar requisitos de autenticación y autorización.

### 10.2 Reglas de compatibilidad
- Se prioriza compatibilidad hacia atrás en cambios no mayores.
- La eliminación o renombrado de campos públicos requiere estrategia de transición.
- Las rupturas de contrato requieren nueva versión o plan de deprecación explícito.

### 10.3 DTOs y entidades
- Los DTOs son contratos externos; las entidades son estructuras internas.
- No se deben exponer entidades JPA directamente fuera del límite del servicio.
- Los mapeos deben ser explícitos y controlados.

## 11. Seguridad

La seguridad es obligatoria en todo el ecosistema y debe observar:

- autenticación robusta y centralizada según el diseño vigente;
- autorización basada en roles, permisos o políticas verificables;
- validación estricta de entrada;
- uso de HTTPS/TLS en entornos integrados y productivos;
- protección de secretos mediante variables de entorno o gestores seguros;
- registros de auditoría para operaciones sensibles;
- principio de mínimo privilegio en servicios, bases de datos y despliegues.

Todo incidente o excepción de seguridad debe tratarse como hallazgo de alto impacto y documentarse.

## 12. Observabilidad

Todo servicio debe aspirar a exponer como mínimo:

- logs estructurados y contextualizados;
- identificadores de correlación entre peticiones;
- métricas de salud, latencia, errores y throughput;
- endpoints de health check cuando aplique;
- trazabilidad suficiente para diagnosticar fallos distribuidos.

Los logs no deben filtrar secretos, credenciales ni datos personales sensibles.

## 13. Gestión de configuración

- Toda configuración debe externalizarse por ambiente.
- No se permite hardcodear endpoints, credenciales ni parámetros sensibles.
- Debe existir separación clara entre configuración local, integración, pruebas y producción.
- Las diferencias de ambiente no deben alterar contratos funcionales sin documentación explícita.

## 14. Calidad y pruebas

Todo cambio relevante debe definir su estrategia de pruebas:

- **Pruebas unitarias** para reglas de negocio y componentes aislados.
- **Pruebas de integración** para persistencia, configuración y colaboración entre capas.
- **Pruebas de contrato** para APIs consumidas o expuestas.
- **Pruebas de seguridad** para autenticación, autorización y manejo de entradas.
- **Pruebas end-to-end** para flujos críticos del sistema cuando aplique.

Los criterios de aceptación de la especificación deben mapearse a pruebas verificables.

## 15. Documentación y trazabilidad

Toda iniciativa relevante debe poder vincularse con:

- especificación funcional/técnica;
- ADR si hubo decisión arquitectónica;
- pull request o commit asociado;
- evidencia de prueba;
- impacto sobre contratos, seguridad o datos.

Si una funcionalidad no está documentada, se considera incompleta desde la perspectiva de gobierno técnico.

## 16. Convenciones de cambio

Todo cambio deberá clasificarse en una de estas categorías:

- **Cambio compatible**: no rompe contrato ni comportamiento esperado del consumidor.
- **Cambio evolutivo**: amplía capacidades sin eliminar compatibilidad actual.
- **Cambio disruptivo**: rompe contratos o modifica comportamiento esencial; requiere versionado, comunicación y plan de transición.

## 17. ADRs y excepciones

Toda excepción a esta constitución debe:

1. quedar registrada en un ADR;
2. explicar la razón técnica o de negocio;
3. evaluar riesgos y compensaciones;
4. definir duración temporal o condición de revisión.

La ausencia de ADR en una excepción arquitectónica se considera incumplimiento.

## 18. Plantilla mínima de cumplimiento SDD

Toda nueva especificación del sistema debe responder, como mínimo:

1. ¿Qué problema resuelve?
2. ¿Qué servicio es responsable?
3. ¿Cuál es el contrato esperado?
4. ¿Qué reglas de negocio aplica?
5. ¿Qué datos lee o persiste?
6. ¿Qué riesgos de seguridad implica?
7. ¿Cómo se observará en operación?
8. ¿Cómo se validará mediante pruebas?
9. ¿Rompe compatibilidad?
10. ¿Requiere ADR?

## 19. Cumplimiento

Esta constitución tiene carácter rector para el ecosistema Cinelogía. Su cumplimiento es obligatorio para el diseño, implementación y evolución de los microservicios y componentes asociados. Cualquier desviación deberá justificarse explícitamente y quedar documentada.

## 20. Vigencia

La presente constitución entra en vigor desde su publicación en el repositorio documental oficial y debe revisarse ante cambios arquitectónicos relevantes, incorporación de nuevos servicios, modificaciones sustanciales de seguridad o evolución del modelo de integración entre microservicios.
