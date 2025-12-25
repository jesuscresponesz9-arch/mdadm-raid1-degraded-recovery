# Informe Técnico: Ciclo de Vida de un Arreglo RAID-1 con mdadm

## Objetivo
Validar operativamente el comportamiento de un arreglo RAID-1 ante el fallo de un componente y documentar el proceso de reconstrucción hasta restablecer un estado óptimo de redundancia.

---

## 1. Escenario Inicial
Se configuró un arreglo RAID-1 bajo la utilidad `mdadm`, identificado como `/dev/md0`. El arreglo se sustenta sobre dispositivos de bloque loopback para propósitos de simulación.

* **Dispositivos:** `/dev/loop10`, `/dev/loop11`
* **Capacidad unitaria:** 1022 MiB
* **Estado inicial:** `clean`
* **Sincronización:** Ambos miembros en estado `active sync`.



---

## 2. Procedimiento Operativo

### 2.1 Construcción del Arreglo
La creación se realizó mediante el comando `mdadm --create`, estableciendo un superbloque versión 1.2. 

**Evidencia de inspección (`mdadm --detail`):**
* **Active Devices:** 2
* **Working Devices:** 2
* **Estado:** `clean`

### 2.2 Simulación de Fallo
Se forzó un estado de error en `/dev/loop10` mediante el parámetro `--fail`. El sistema registró el cambio de estado de manera inmediata:

* **Estado reportado:** `clean, degraded`
* **Identificación de unidad:** `/dev/loop10` marcado como `faulty`.
* **Topología:** `RaidDevice 0` marcado como `removed`.
* **Disponibilidad:** El arreglo mantuvo la operatividad de lectura/escritura a través de `/dev/loop11`.



### 2.3 Proceso de Recuperación y Resync
La restauración de la redundancia se ejecutó siguiendo el flujo de mantenimiento estándar:
1.  **Eliminación:** Extracción lógica del dispositivo fallido con `--remove`.
2.  **Incorporación:** Inserción de un nuevo dispositivo (o el mismo saneado) mediante `--add`.
3.  **Resync:** Inicio automático de la operación de reconstrucción de datos bit a bit hacia el nuevo miembro.



---

## 3. Estado Final
Tras completar el ciclo de reconstrucción, el sistema reportó la normalización del servicio:

* **Estado final:** `clean`
* **Active Devices:** 2
* **Failed Devices:** 0
* **Membresía:** `/dev/loop10` y `/dev/loop11` en estado `active sync`.
* **Contador de eventos:** 41 (reflejando la trazabilidad de la reconstrucción).

---

## 4. Conclusión Técnica
La evidencia demuestra que la implementación de RAID-1 vía `mdadm` garantiza la **continuidad operativa** ante el fallo físico de un nodo. El proceso de recuperación es eficiente y permite restaurar la tolerancia a fallos mediante la operación de `resync` sin degradación permanente del servicio.

---

## 5. Limitaciones y Advertencias
El RAID-1 proporciona redundancia a nivel de hardware/bloque, pero **no sustituye una estrategia de backups**. No ofrece protección frente a:
* Eliminación accidental de archivos.
* Corrupción lógica del sistema de ficheros.
* Ataques de Ransomware.

*Se recomienda el uso de este esquema en conjunto con políticas de respaldo independientes y cifrado de datos.*
