# Plan de auditoría · Seguridad del canal digital de la Clínica Santa Lucía

**SI-084 · Auditoría de Sistemas** · Examen práctico de Unidad II · CONFIDENCIAL — USO ACADÉMICO

| | |
|---|---|
| **Apellidos y nombres** | David Anampa |
| **Código de estudiante** | 2022074268 |
| **URL del repositorio** | `https://github.com/David-Anampa/U2.S11.EXAMEN_UNIDAD02_PR-CTICO_ANAMPA.git` |
| **Fecha** | 2026-10-28 |

## 1. Resultados del reconocimiento · OWASP WSTG 4.2

| Prueba | Identificador WSTG | Lo que observé, con cifras | Declaración del proveedor que contrasta | ¿Coincide? | Evidencia |
|---|---|---|---|---|---|
| E1 | WSTG-INFO-04 | Se encontraron 3 servicios activos: portal en :8080 (WordPress 5.8.1, no 6.6), entorno de pruebas en :8081 (no declarado, con banner "ENTORNO DE PRUEBAS · WebSur S.A.C.") y API de citas en :8088 (no declarada, expone 3 endpoints: /v1/citas, /v1/pacientes, /v1/resultados). Solo 1 fue declarado; 2 son desconocidos para la clínica. | D1 | No | `evidencias/E1_aplicaciones.txt` |
| E2 | WSTG-INFO-02 | El portal responde con Server: nginx/1.20.2 y X-Powered-By: PHP/7.4.3. La versión de PHP alcanzó fin de vida el 28/11/2022, sin parches de seguridad desde hace casi 4 años. La API expone X-Powered-By: Express. Las versiones expuestas en cabeceras facilitan la búsqueda de CVEs específicos. | D2 | No | `evidencias/E2_servidor.txt` |
| E3 | WSTG-INFO-03 | El robots.txt revela 4 rutas sensibles: /admin/, /respaldos/, /resultados/ y /wp-content/plugins/wp-file-manager/. El sitemap.xml lista 4 páginas públicas, todas con URL HTTP (no HTTPS). El archivo /.well-known/ no existe (HTTP 404). El robots.txt opera como un mapa para atacantes, señalando las rutas que el proveedor quiso ocultar de buscadores. | — | — | `evidencias/E3_metarchivos.txt` |
| E4 | WSTG-INFO-05 | Se encontraron 5 comentarios HTML con datos internos en 3 páginas: correo del proveedor (soporte@websur.test), usuario admin_pruebas pendiente de eliminación, IP interna del sistema de laboratorio (10.20.0.15:8090/lis/resultados), parámetro de consulta ver.php?orden=NNNNN (IDOR potencial), y confirmación de que el entorno de pruebas usa copia de la BD de producción restaurada el 12/09/2026. | D8 | No | `evidencias/E4_contenido.txt` |
| E5 | WSTG-INFO-08 | Se identificaron 5 componentes: WordPress 5.8.1 (sep 2021, declarado 6.6), jQuery 3.4.1 (may 2019, CVE-2020-11022 y CVE-2020-11023 XSS), Contact Form 7 v5.3.1 (desactualizado), WP File Manager v6.8 (CVE-2020-25213, ejecución remota de código, CVSS 9.8), tema MedicalPro v2.1.0. De 5 componentes, 4 están desactualizados; 1 tiene vulnerabilidad RCE crítica. | D3 | No | `evidencias/E5_componentes.txt` |
| E6 | WSTG-INFO-06 | Se identificaron 4 formularios con 17 campos. El de citas (7 campos + 1 oculto + 1 checkbox) usa method="GET": DNI, nombre, teléfono, síntomas y fecha de nacimiento viajan en la URL. El checkbox "acepto" no tiene texto de consentimiento. El campo oculto medico_id=17 es manipulable (IDOR). El de postulaciones acepta subida de archivos a wp-admin/admin-ajax.php. La contraseña de resultados es la fecha de nacimiento (ddmmaaaa). | D7 | No | `evidencias/E6_entradas.txt` |
| E7 | WSTG-CONF-04 · WSTG-CONF-05 | El panel /admin/ (HTTP 200) está accesible públicamente con formulario de login, sin restricción por IP. El directorio /respaldos/ (HTTP 200) tiene autoindex habilitado y lista 4 archivos: 2 dumps SQL (portal_2025-09-30.sql.gz, portal_2026-03-31.sql.gz), 1 exportación CSV con cabeceras nombres,dni,celular,especialidad,motivo_consulta, y 1 wp-config.php.bak. Los respaldos están dentro del servidor web, no fuera. | D5 · D6 | No | `evidencias/E7_expuestos.txt` |
| E8 | WSTG-CONF-07 · WSTG-CONF-12 · WSTG-SESS-02 | 0 de 7 cabeceras de seguridad están presentes (sin HSTS, X-Content-Type-Options, X-Frame-Options, CSP, Referrer-Policy, Permissions-Policy, X-XSS-Protection). La cookie PHPSESSID=8f3a1c9e2b7d4a60e5 tiene valor estático (compartida por todos los usuarios), carece de Secure, HttpOnly y SameSite. La API tiene Access-Control-Allow-Origin: * (CORS abierto). Se exponen Server: nginx/1.20.2 y X-Powered-By: PHP/7.4.3. | D4 | No | `evidencias/E8_cabeceras.txt` |

## 2. Materia de control · Sección III

Seguridad de la información del canal digital de la Clínica Santa Lucía S.A.C. durante el periodo del 01 de enero al 30 de septiembre de 2026. El reconocimiento reveló que las ocho declaraciones del proveedor WebSur S.A.C. son falsas, lo que evidencia un riesgo material de exposición de datos de salud de más de 12 400 pacientes. El incidente del 08/09/2026, en el que una paciente visualizó el hemograma de otra persona, confirma que las deficiencias no son teóricas sino que ya produjeron una brecha de confidencialidad de datos sensibles.

## 3. Alcance · Sección IV

| Bloque | Contenido |
|---|---|
| Incluye | **Portal institucional** (portal.clinica-santalucia.test / 127.0.0.1:8080): reserva de citas (3 850 citas/mes), resultados de laboratorio (12 400 pacientes con cuenta), postulaciones (214 con CV en 2026) y panel de administración. **Entorno de pruebas** (127.0.0.1:8081): réplica no declarada con copia de base de datos de producción. **API de citas** (127.0.0.1:8088): servicio no declarado con endpoints /v1/citas, /v1/pacientes, /v1/resultados. **Directorio de respaldos** (/respaldos/): 2 dumps SQL, 1 CSV de citas y 1 wp-config.php.bak. |
| Periodo auditado | Del 01/01/2026 al 30/09/2026, conforme a la solicitud de la Gerencia General. Este periodo incluye el incidente del 08/09/2026. |
| Excluye expresamente | La red interna de la clínica y los sistemas detrás del firewall (fuera del canal digital). Los servidores de correo electrónico institucional. Direcciones fuera de 127.0.0.1 (fuera del alcance autorizado). Las aplicaciones móviles, si las hubiera, por no estar dentro del canal digital declarado. |
| Limitaciones inherentes | La auditoría proporcionará seguridad razonable, no absoluta. El reconocimiento se limitó a peticiones GET y HEAD a páginas públicas, conforme al alcance autorizado; no se ejecutaron pruebas activas de penetración. No se verificó el código fuente del servidor por no tener acceso al repositorio del proveedor. La configuración interna de la base de datos y del servidor de aplicaciones no es observable externamente. |

## 4. Objetivos · Sección V

**Objetivo general.** Determinar si el canal digital de la Clínica Santa Lucía S.A.C. protege adecuadamente la confidencialidad, integridad y disponibilidad de los datos de salud de sus pacientes, conforme a la OWASP Web Security Testing Guide 4.2, el OWASP Application Security Verification Standard 4.0.3 y la NTP-ISO/IEC 27001:2022.

| N.º | Objetivo específico | Pregunta de auditoría |
|---|---|---|
| OE1 | Determinar si la configuración del servidor web y de sus componentes de software impide la explotación de vulnerabilidades conocidas, conforme a WSTG-INFO-08, ASVS 4.0.3 V14.2 y NTP-ISO/IEC 27001:2022 A.8.8. | ¿Están actualizados y libres de vulnerabilidades conocidas los componentes del canal digital (WordPress, plugins, librerías y servidor web)? |
| OE2 | Determinar si los controles de autenticación y gestión de sesiones del módulo de resultados de laboratorio impiden el acceso no autorizado a datos de salud de terceros, conforme a WSTG-ATHN-01, ASVS 4.0.3 V2.1 y la Ley 29733, art. 13.5. | ¿Evitan los controles del módulo de resultados que un paciente acceda a los datos de salud de otro paciente, como ocurrió en el incidente del 08/09/2026? |
| OE3 | Determinar si la gestión de archivos expuestos, respaldos y entornos no declarados protege los datos personales y de salud contra accesos no autorizados, conforme a WSTG-CONF-04, ASVS 4.0.3 V8.3 y NTP-ISO/IEC 27001:2022 A.8.10. | ¿Están protegidos contra acceso público los respaldos de base de datos, los archivos de configuración y los entornos no declarados del canal digital? |

## 5. Enfoque, materialidad y riesgo · Sección VI

| Elemento | Decisión | Sustento en lo que observó o en la solicitud |
|---|---|---|
| Enfoque | Sustantivo | Las ocho declaraciones del proveedor resultaron falsas en el reconocimiento (E1 a E8). No existen controles de seguridad verificables sobre los cuales depositar confianza: no hay cabeceras de seguridad (E8), los componentes están desactualizados con vulnerabilidades críticas (E5), los respaldos están expuestos (E7) y el panel de administración es público (E7). Un enfoque de confianza en controles no es viable porque no hay controles operativos que probar. La comisión realizará pruebas directas sobre los datos y la configuración. |
| Materialidad cuantitativa | S/ 186 000 (1 % de los ingresos anuales 2025 de S/ 18 600 000) | Base declarada en la solicitud de auditoría. Se consideran materiales las deficiencias cuyo impacto económico potencial (sanciones SUSALUD, multas por Ley 29733, pérdida de pacientes) supere este umbral. |
| Materialidad cualitativa | Cualquier exposición de datos de salud es material sin importar el monto | Los datos de salud son de categoría especial según el art. 13.6 de la Ley 29733. Una sola brecha (como el incidente del 08/09/2026) compromete la confianza de los 12 400 pacientes y expone a la clínica a sanciones de la Autoridad Nacional de Protección de Datos Personales y de SUSALUD. |
| Riesgo inherente | Alto | La clínica maneja datos de salud de 12 400 pacientes, opera bajo supervisión de SUSALUD y la Ley 29733, y ya sufrió un incidente de brecha de datos. El sector salud es un objetivo frecuente de ataques. El volumen de datos sensibles (3 850 citas/mes, 214 CVs con DNI) amplifica el impacto de cualquier exposición. |
| Riesgo de control | Alto | El reconocimiento demostró que los controles son inexistentes o ineficaces: 0 de 7 cabeceras de seguridad configuradas (E8), componentes con vulnerabilidad RCE activa (E5), respaldos públicos (E7), panel admin sin restricción (E7), cookie de sesión estática compartida (E8). Las 8 declaraciones del proveedor son falsas, lo que indica ausencia de supervisión por parte de la clínica (solo 3 personas en sistemas). |
| Riesgo de detección | Bajo | El enfoque sustantivo con pruebas directas sobre la réplica autorizada, combinado con herramientas de la OWASP WSTG 4.2, reduce la probabilidad de no detectar deficiencias materiales. El reconocimiento ya identificó vulnerabilidades críticas en la fase de planificación. |

## 6. Procedimientos · Sección VII

La comisión ejecutará seis procedimientos de auditoría sobre la réplica autorizada del canal digital durante la Unidad III. Los procedimientos P-01 y P-02 cubren el objetivo específico OE1 (componentes y configuración), P-03 y P-04 cubren OE2 (autenticación y acceso a resultados, incluyendo el incidente del 08/09/2026), y P-05 y P-06 cubren OE3 (archivos expuestos y entornos no declarados).

### P-01 — Verificación de versiones y vulnerabilidades conocidas en componentes

| Columna | Contenido |
|---|---|
| Objetivo específico | OE1 |
| Origen en el reconocimiento | E5 · Se identificaron 4 de 5 componentes desactualizados, incluyendo WP File Manager 6.8 con CVE-2020-25213 (RCE, CVSS 9.8) |
| Riesgo | Un atacante no autenticado podría ejecutar código arbitrario en el servidor explotando el plugin vulnerable, comprometiendo todos los datos de pacientes |
| Criterio | WSTG-CRYP-01 · OWASP ASVS 4.0.3 V14.2.1 · NTP-ISO/IEC 27001:2022 A.8.8 |
| Tipo de prueba | Sustantiva — verificación directa de versiones y CVEs |
| Población y muestra | Los 5 componentes identificados en E5 (WordPress, jQuery, Contact Form 7, WP File Manager, MedicalPro). Muestra: censo completo |
| Procedimiento | 1. Acceder a http://127.0.0.1:8080/ y obtener el código fuente HTML. 2. Extraer la versión de cada componente desde las etiquetas meta, script y link. 3. Consultar el catálogo NVD (https://nvd.nist.gov/) y WPScan para cada componente y versión. 4. Registrar cada CVE encontrado con su CVSS, descripción y si existe parche. 5. Verificar si el exploit público de CVE-2020-25213 es aplicable a la versión 6.8 de WP File Manager. 6. Documentar la brecha entre la versión instalada y la versión declarada por el proveedor. |
| Evidencia a obtener | Captura del código fuente con las versiones. Lista de CVEs por componente con CVSS. Comparación entre versión declarada y versión real |
| Definición de excepción | Se considera excepción toda versión de componente que tenga un CVE publicado con CVSS ≥ 7.0 sin parche aplicado, o toda versión que haya alcanzado fin de vida (EOL) |
| Horas estimadas | 3 |

### P-02 — Evaluación de cabeceras de seguridad HTTP y configuración TLS

| Columna | Contenido |
|---|---|
| Objetivo específico | OE1 |
| Origen en el reconocimiento | E8 · 0 de 7 cabeceras de seguridad presentes; cookie PHPSESSID estática sin Secure, HttpOnly ni SameSite; CORS abierto en la API |
| Riesgo | La ausencia de cabeceras de seguridad permite ataques de clickjacking, XSS, MIME sniffing y secuestro de sesión. La cookie estática permite suplantación de identidad |
| Criterio | WSTG-CONF-07 · WSTG-CONF-12 · WSTG-SESS-02 · OWASP ASVS 4.0.3 V3.4.1, V3.4.2, V3.4.3, V14.4.3 · NTP-ISO/IEC 27001:2022 A.8.24 |
| Tipo de prueba | Sustantiva — captura y análisis de cabeceras HTTP |
| Población y muestra | Los 3 servicios activos (portal :8080, pruebas :8081, API :8088). Muestra: 5 páginas por servicio (raíz, citas, resultados, admin, respaldos) |
| Procedimiento | 1. Realizar peticiones HEAD a cada URL de la muestra con curl -I. 2. Registrar todas las cabeceras de respuesta, incluyendo Set-Cookie. 3. Verificar la presencia y valor de cada cabecera de seguridad: HSTS, X-Content-Type-Options, X-Frame-Options, CSP, Referrer-Policy, Permissions-Policy, X-XSS-Protection. 4. Analizar los atributos de cada cookie (Secure, HttpOnly, SameSite, path, domain). 5. Verificar si el valor de PHPSESSID cambia entre peticiones (sesión dinámica vs. estática). 6. Verificar la política CORS de la API (Access-Control-Allow-Origin). |
| Evidencia a obtener | Volcado completo de cabeceras por servicio y página. Tabla resumen de cabeceras presentes/ausentes. Análisis de atributos de cookies |
| Definición de excepción | Se considera excepción la ausencia de cualquiera de las 7 cabeceras de seguridad evaluadas, o la ausencia de cualquiera de los 3 atributos obligatorios de cookies de sesión (Secure, HttpOnly, SameSite) |
| Horas estimadas | 2 |

### P-03 — Prueba de controles de autenticación del módulo de resultados de laboratorio

| Columna | Contenido |
|---|---|
| Objetivo específico | OE2 |
| Origen en el reconocimiento | E6 · La contraseña inicial es la fecha de nacimiento (ddmmaaaa), predecible. E4 · Los resultados se muestran en ver.php?orden=NNNNN (identificador secuencial). Este procedimiento responde directamente al incidente del 08/09/2026 |
| Riesgo | Un atacante podría acceder a los resultados de laboratorio (datos de salud) de cualquier paciente adivinando la contraseña (fecha de nacimiento) o iterando el parámetro de orden (IDOR), reproduciendo el incidente del 08/09/2026 |
| Criterio | WSTG-ATHN-01 · WSTG-ATHN-07 · OWASP ASVS 4.0.3 V2.1.1, V2.1.4 · NTP-ISO/IEC 27001:2022 A.8.5 · Ley 29733, art. 13.5 y 13.6 |
| Tipo de prueba | Sustantiva — prueba directa de los mecanismos de autenticación |
| Población y muestra | Módulo de resultados (/resultados/ver.php). Población: 12 400 pacientes con cuenta. Muestra: 10 intentos controlados con credenciales de prueba generadas para la réplica |
| Procedimiento | 1. Acceder a http://127.0.0.1:8080/resultados/ y documentar el formulario de autenticación. 2. Verificar si el sistema limita los intentos fallidos de autenticación (lockout, CAPTCHA, retardo). 3. Generar 5 combinaciones de DNI y fecha de nacimiento como contraseña; intentar autenticación con cada una. 4. Si se obtiene acceso, verificar si el parámetro orden en ver.php?orden=NNNNN permite acceder a resultados de otros pacientes modificando el valor (IDOR). 5. Documentar si el sistema exige cambio de contraseña en el primer acceso. 6. Verificar la existencia de política de complejidad de contraseñas y de mecanismo seguro de recuperación. |
| Evidencia a obtener | Captura del formulario de login. Registro de intentos de autenticación y respuestas. Prueba de enumeración del parámetro orden. Verificación de controles antifuerza bruta |
| Definición de excepción | Se considera excepción si el sistema permite más de 5 intentos fallidos de autenticación sin bloqueo, si la contraseña predeterminada (fecha de nacimiento) no requiere cambio obligatorio, o si modificar el parámetro orden permite acceder a resultados de otro paciente |
| Horas estimadas | 4 |

### P-04 — Verificación del consentimiento informado en formularios con datos de salud

| Columna | Contenido |
|---|---|
| Objetivo específico | OE2 |
| Origen en el reconocimiento | E6 · El formulario de citas recopila datos de salud (motivo_consulta/síntomas) y el checkbox "acepto" no tiene texto. El de postulaciones recopila DNI y CV sin consentimiento |
| Riesgo | La clínica recopila datos de salud sin consentimiento informado válido, lo que constituye una infracción a la Ley 29733 susceptible de sanción por la Autoridad Nacional de Protección de Datos Personales |
| Criterio | WSTG-INFO-06 · OWASP ASVS 4.0.3 V1.8.1 · NTP-ISO/IEC 27001:2022 A.5.34 · Ley 29733, art. 5, 13.5 y 13.6 |
| Tipo de prueba | Sustantiva — revisión documental y funcional |
| Población y muestra | Los 4 formularios del portal. Muestra: censo completo |
| Procedimiento | 1. Acceder a cada formulario público del portal (/citas/, /resultados/, /postula/, /admin/). 2. Documentar si existe checkbox de consentimiento y su texto asociado. 3. Verificar si el texto referencia la finalidad del tratamiento, el responsable, los derechos ARCO y la política de privacidad. 4. Verificar si los formularios con datos de salud (citas) exigen consentimiento expreso para datos sensibles, conforme al art. 13.6 de la Ley 29733. 5. Verificar si el formulario de citas usa GET (datos en URL) o POST. 6. Documentar las implicaciones de que los datos de salud viajen en la URL (logs, historial, caché). |
| Evidencia a obtener | Captura de cada formulario con su código fuente. Tabla de cumplimiento de requisitos de consentimiento por formulario. Captura del método HTTP de cada formulario |
| Definición de excepción | Se considera excepción todo formulario que recopile datos personales o de salud sin texto de consentimiento informado que incluya finalidad, responsable y derechos ARCO, o todo formulario que transmita datos de salud por GET |
| Horas estimadas | 2 |

### P-05 — Verificación de acceso a respaldos y archivos de configuración expuestos

| Columna | Contenido |
|---|---|
| Objetivo específico | OE3 |
| Origen en el reconocimiento | E7 · /respaldos/ accesible con autoindex on, listando dumps SQL, CSV de citas y wp-config.php.bak. E3 · robots.txt revela la ruta |
| Riesgo | Un atacante podría descargar los dumps de la base de datos con datos de los 12 400 pacientes, obtener credenciales del wp-config.php.bak, o acceder a la exportación CSV con DNI, teléfonos y síntomas de pacientes |
| Criterio | WSTG-CONF-04 · WSTG-CONF-05 · OWASP ASVS 4.0.3 V8.3.4, V12.5.1 · NTP-ISO/IEC 27001:2022 A.8.10, A.8.4 |
| Tipo de prueba | Sustantiva — verificación directa de accesibilidad |
| Población y muestra | Directorio /respaldos/ (4 archivos observados en E7). Rutas sensibles de robots.txt (/admin/, /wp-content/plugins/wp-file-manager/). Muestra: todos los archivos listados + rutas mencionadas en metarchivos |
| Procedimiento | 1. Acceder a http://127.0.0.1:8080/respaldos/ y documentar el listado de directorio. 2. Verificar si cada archivo listado es descargable (realizar petición HEAD, no descargar contenido completo). 3. Verificar los permisos y cabeceras de cada archivo (Content-Type, Content-Disposition). 4. Acceder a /wp-content/plugins/wp-file-manager/ y documentar si es accesible. 5. Verificar si existe algún mecanismo de autenticación o autorización para acceder al directorio de respaldos. 6. Documentar las fechas de los respaldos y evaluar la política de retención. |
| Evidencia a obtener | Captura del listado de /respaldos/. Cabeceras HEAD de cada archivo. Verificación de accesibilidad de cada ruta sensible. Análisis de la política de respaldos |
| Definición de excepción | Se considera excepción todo archivo de respaldo, dump de base de datos o archivo de configuración accesible sin autenticación desde una URL pública, o todo directorio con autoindex habilitado que contenga archivos sensibles |
| Horas estimadas | 2 |

### P-06 — Evaluación de entornos y servicios no declarados

| Columna | Contenido |
|---|---|
| Objetivo específico | OE3 |
| Origen en el reconocimiento | E1 · Entorno de pruebas en :8081 y API en :8088 no declarados por el proveedor. E4 · El entorno de pruebas contiene copia de BD de producción (12/09/2026) con datos reales de pacientes |
| Riesgo | Los entornos no declarados amplían la superficie de ataque sin supervisión. El entorno de pruebas con datos de producción expone información de salud real. La API con CORS abierto y endpoints de pacientes podría permitir consultas directas a datos sensibles |
| Criterio | WSTG-CONF-11 · OWASP ASVS 4.0.3 V1.14.5, V14.3.2 · NTP-ISO/IEC 27001:2022 A.8.25, A.8.31 |
| Tipo de prueba | Sustantiva — verificación directa de existencia y configuración |
| Población y muestra | Servicios en 127.0.0.1, puertos 8080-8090. Muestra: censo completo (10 puertos escaneados, 3 activos encontrados) |
| Procedimiento | 1. Realizar peticiones GET al recurso raíz (/) de cada puerto en el rango 8080-8090 y documentar las respuestas. 2. Para el entorno de pruebas (:8081): documentar el banner, verificar si comparte credenciales con producción, verificar la presencia de datos reales en los comentarios HTML. 3. Para la API (:8088): documentar el índice de endpoints, verificar la política CORS (Access-Control-Allow-Origin), verificar si la documentación Swagger (/swagger/) es accesible. 4. Verificar si existe segregación de redes o controles de acceso entre los entornos. 5. Comparar la configuración de cada entorno (cabeceras, cookies, componentes) con la del portal de producción. 6. Documentar los riesgos de mantener datos de producción en el entorno de pruebas. |
| Evidencia a obtener | Respuesta HTTP de cada puerto escaneado. Código fuente HTML del entorno de pruebas. Índice JSON de la API. Comparación de configuración entre entornos |
| Definición de excepción | Se considera excepción todo entorno o servicio accesible externamente que no haya sido declarado en el inventario del proveedor, o todo entorno de pruebas que contenga datos reales de producción |
| Horas estimadas | 3 |

## 7. Anexo A · Matriz de riesgos

| N.º | Riesgo | Causa, con su evidencia | Impacto | Nivel | Control existente | Control esperado |
|---|---|---|---|---|---|---|
| R1 | Ejecución remota de código en el servidor web por explotación de WP File Manager 6.8 (CVE-2020-25213, CVSS 9.8) | WordPress y sus plugins no se actualizan desde 2021 (E5). La versión instalada es vulnerable a un exploit público que permite subir y ejecutar archivos PHP sin autenticación | Compromiso total del servidor: acceso a la base de datos con datos de salud de 12 400 pacientes, modificación de contenido, propagación lateral a sistemas internos | Crítico | Ninguno. El proveedor declaró actualizaciones mensuales (D3), pero la versión instalada tiene más de 5 años | Gestión de parches con actualizaciones en las primeras 72 horas para CVEs críticos. Monitoreo continuo de vulnerabilidades (ASVS V14.2, A.8.8) |
| R2 | Acceso no autorizado a resultados de laboratorio de otros pacientes (brecha de confidencialidad de datos de salud) | Contraseña predeterminada predecible (fecha de nacimiento), identificador secuencial en ver.php?orden=NNNNN susceptible de IDOR (E6, E4). Incidente del 08/09/2026 confirma la materialización del riesgo | Violación de la Ley 29733 (datos sensibles de salud), sanción de SUSALUD y de la Autoridad Nacional de Protección de Datos, daño reputacional, pérdida de pacientes | Crítico | Ninguno verificable. WebSur catalogó el incidente como «error puntual de caché» sin informe ni corrección | Autenticación robusta (ASVS V2.1), controles de acceso a nivel de registro (IDOR), auditoría de accesos, forzar cambio de contraseña inicial (A.8.5) |
| R3 | Exposición de respaldos de base de datos y archivo de configuración accesibles públicamente | Directorio /respaldos/ con autoindex habilitado en nginx, sin autenticación (E7). Contiene dumps SQL y wp-config.php.bak con potenciales credenciales | Descarga masiva de datos personales y de salud, obtención de credenciales de la BD, escalación de privilegios | Alto | Ninguno. Los respaldos están dentro del servidor web, accesibles por HTTP. El proveedor declaró que están «fuera del servidor web» (D6, falsa) | Almacenamiento de respaldos fuera del webroot, con cifrado y acceso restringido. Eliminación de archivos .bak (ASVS V8.3, A.8.10) |
| R4 | Secuestro de sesión por cookie estática compartida y ausencia de protección de transporte | Cookie PHPSESSID con valor fijo inyectado por nginx, sin Secure, HttpOnly ni SameSite (E8). Sin HSTS ni cabeceras de seguridad. Todas las comunicaciones en HTTP plano | Suplantación de identidad de cualquier usuario (session fixation), interceptación de credenciales y datos de salud en tránsito | Alto | Ninguno. Las cabeceras de seguridad no están configuradas (0 de 7). El proveedor declaró HTTPS y cabeceras configuradas (D4, falsa) | Implementación de HTTPS con HSTS. Generación dinámica de cookies de sesión con Secure, HttpOnly y SameSite. Cabeceras de seguridad completas (ASVS V3.4, V14.4, A.8.24) |
| R5 | Incumplimiento de la Ley 29733 por recopilación de datos de salud sin consentimiento informado válido | El formulario de citas recopila datos de salud (síntomas) con checkbox «acepto» sin texto legal (E6). Usa method="GET", exponiendo datos en la URL. El formulario de postulaciones no tiene consentimiento | Sanción por la Autoridad Nacional de Protección de Datos Personales, demandas de pacientes, orden de cese de operaciones del canal digital | Alto | Ninguno. El checkbox existe pero no tiene texto asociado, no referencia política de privacidad ni informa sobre finalidad, responsable o derechos ARCO | Consentimiento informado con texto completo conforme al art. 13.5 de la Ley 29733. Formularios con method="POST". Política de privacidad accesible (ASVS V1.8, A.5.34) |
