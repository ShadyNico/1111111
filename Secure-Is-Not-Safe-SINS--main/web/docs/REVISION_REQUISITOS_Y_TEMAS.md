# Revisión de cumplimiento: requisitos del proyecto y temas del curso

## 1) Conclusión rápida

**No se cumplen todos los temas de la imagen al 100%.**
El proyecto sí implementa correctamente el núcleo de **BD distribuida con SQL Server + Linked Server + app cliente-servidor** entre dos nodos (host Fedora y remoto Windows), pero hay temas que están **parciales o no evidenciados** (sobre todo los de **Base de Datos en la Nube** y la operación **UPDATE** en flujo de aplicación).

---

## 2) Revisión de requisitos del proyecto (enunciado)

### Requisito A: dos equipos en red (host + remoto)
- **Estado:** ✅ Cumplido.
- **Evidencia:** arquitectura con nodo central Fedora y nodo sensor Windows; ambos con SQL Server y comunicación por red/Linked Server.

### Requisito B: servidor anfitrión con aplicación principal + al menos una tabla
- **Estado:** ✅ Cumplido.
- **Evidencia:** la app web (Apache/PHP) está en el nodo central; tablas locales `Forense_Logs` y `Audit_Transacciones` en `CentralSIEM`.

### Requisito C: servidor remoto con el resto de tablas
- **Estado:** ✅ Cumplido.
- **Evidencia:** tabla `Live_Alerts` creada en `SensorDB` (Windows).

### Requisito D: operaciones INSERT, SELECT, UPDATE y DELETE sobre datos distribuidos
- **Estado:** ⚠️ Parcial.
- **Qué sí está:**
  - `INSERT` (simulación de ataque en remoto y auditorías/local).
  - `SELECT` (consultas de alertas en remoto y logs locales).
  - `DELETE` (limpieza y archivado de alertas remotas).
- **Qué falta evidenciar en la app:**
  - **`UPDATE`** explícito en APIs principales (`get_alerts.php` / `actions.php`) no aparece implementado como operación funcional.

### Requisito E: SQL Server en ambos equipos
- **Estado:** ✅ Cumplido.
- **Evidencia:** scripts separados para Fedora y Windows con SQL Server 2022.

### Requisito F: Linked Server correctamente configurado
- **Estado:** ✅ Cumplido.
- **Evidencia:** script de creación de `SENSOR_REMOTO` con `sp_addlinkedserver` + prueba de consulta remota.

### Requisito G: diseño distribuido con tablas relacionadas entre servidores
- **Estado:** ✅ Cumplido (modelo funcional por fragmentación/flujo).
- **Evidencia:** `Live_Alerts` (remoto) + `Forense_Logs`/`Audit_Transacciones` (host), relacionadas por flujo de archivado (`Original_AlertID`).

### Requisito H: aplicación consumiendo datos desde ambos servidores
- **Estado:** ✅ Cumplido.
- **Evidencia:** `get_alerts.php` consulta remoto (`[SENSOR_REMOTO]...Live_Alerts`) y local (`Forense_Logs`), frontend consume API vía fetch.

### Requisito I: verificación de consultas distribuidas desde la app
- **Estado:** ✅ Cumplido.
- **Evidencia:** API + scripts SQL/documentación de prueba de consultas al Linked Server.

---

## 3) Revisión de temas de la imagen (clasificación, diseño, arquitectura, nube)

## Clasificación

1. **Homogéneas (autónomas, no autónomas)**
   - **Estado:** ⚠️ Parcial/No objetivo principal.
   - **Comentario:** el proyecto está enfocado a arquitectura federada heterogénea (Fedora + Windows), no a clasificar/implementar formalmente variantes homogéneas.

2. **Heterogéneas (federadas, múltiples)**
   - **Estado:** ✅ Cumplido.

## Diseño

3. **Particionamiento y fragmentación**
   - **Estado:** ✅ Cumplido.
   - **Comentario:** separación funcional de datos activos vs históricos entre nodos.

4. **Replicación, ventajas y desventajas**
   - **Estado:** ✅ Cumplido (implementación) / ⚠️ Parcial (análisis formal de trade-offs).
   - **Comentario:** hay replicación tipo copy-delete en `archive`; no se ve una matriz formal extensa de pros/contras en código.

5. **Consultas centralizadas / distribuidas**
   - **Estado:** ✅ Cumplido.

6. **Control de la concurrencia (transacciones, deadlock)**
   - **Estado:** ✅ Cumplido (transacción + control optimista).
   - **Comentario:** hay `BEGIN DISTRIBUTED TRANSACTION` y verificación por timestamp.

7. **Almacenamiento y procesamiento distribuido**
   - **Estado:** ✅ Cumplido.

8. **Fallas y recuperación**
   - **Estado:** ✅ Cumplido.
   - **Comentario:** rollback transaccional + manejo de errores.

9. **Seguridades y consolidación**
   - **Estado:** ✅ Cumplido (nivel aplicación básico-intermedio).
   - **Comentario:** prepared statements, CORS controlado y auditoría.

## Arquitectura

10. **Cliente-servidor**
    - **Estado:** ✅ Cumplido.

11. **Punto a punto**
    - **Estado:** ⚠️ Parcial.
    - **Comentario:** la topología real es principalmente centralizada host→remoto por Linked Server, no una malla P2P completa.

12. **Multipunto y multi-database**
    - **Estado:** ✅ Cumplido en multi-base; ⚠️ limitado a 2 nodos (no multipunto amplio).

## Base de Datos en la Nube

13. **Plataformas como servicio**
    - **Estado:** ❌ No evidenciado.

14. **Base de Datos como servicio**
    - **Estado:** ❌ No evidenciado.

15. **Seguridad en la nube**
    - **Estado:** ❌ No evidenciado.

---

## 4) Hallazgos técnicos importantes

1. El flujo `archive` ya está ajustado para ejecutarse en el **host** con transacción distribuida explícita.
2. El proyecto cumple sólidamente el bloque de BD distribuida on-premise (SQL Server + Linked Server).
3. El principal gap para requisitos funcionales es **exponer una operación UPDATE** en la app/API.
4. Los temas de nube de la imagen no forman parte de una implementación visible en este repositorio.

---

## 5) Qué recomendar para cerrar al 100%

1. Agregar endpoint `update_alert_status` (o ampliar `actions.php`) con `UPDATE` en:
   - `Forense_Logs.Estado_Procesamiento` (host), o
   - `Live_Alerts.Estado_Alerta` (remoto vía linked server).
2. Documentar caso de uso y prueba de `UPDATE` (captura + query antes/después).
3. Si el curso exige bloque “Nube”, añadir anexo teórico/práctico mínimo (aunque sea comparativo) para PaaS/DBaaS y seguridad cloud.

---

## 6) Evidencias revisadas (archivos)

- `README.md`
- `sql/setup_fedora.sql`
- `sql/setup_windows.sql`
- `sql/setup_linked_server.sql`
- `web/sql/setup_distributed_database.sql`
- `web/api/get_alerts.php`
- `web/api/actions.php`
- `web/assets/main.js`
- `web/docs/CODIGO_ACTUALIZADO_ACTIONS.md`
