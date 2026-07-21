<div align="center">
  <img src="nullsec-logo.png" alt="Nullsec" width="180" />
  <h1>Nullsec — Ciclo de Detección de Amenazas con Threat Intelligence</h1>
  <p><strong>Proyecto Integrador · SOC / Blue Team</strong></p>
  <img src="https://img.shields.io/badge/ELK%20Stack-8.19.16-005571?style=flat-square&logo=elasticsearch&logoColor=white" />
  <img src="https://img.shields.io/badge/MISP-Threat%20Intel-2b6cb0?style=flat-square" />
  <img src="https://img.shields.io/badge/Sysmon-15.21-00A4EF?style=flat-square" />
  <img src="https://img.shields.io/badge/MITRE%20ATT%26CK-mapeado-FF7B72?style=flat-square" />
  <img src="https://img.shields.io/badge/MTTD-~5--6%20min-3FB950?style=flat-square" />
</div>

---

## Resumen

**Nullsec** es un laboratorio que demuestra un ciclo completo de detección de amenazas de extremo a extremo: un IOC cargado manualmente en **MISP** termina disparando una **alerta real en el SIEM** tras la ejecución controlada del ransomware **Bad Rabbit** sobre un endpoint Windows.

No se trataba de instalar herramientas, sino de demostrar que la inteligencia de amenazas, la ingesta de telemetría y las reglas de detección se integran y funcionan juntas en producción.

> Fue un proyecto grupal con tres módulos. **Mi rol fue el Módulo 2: el núcleo de detección**, el servidor ELK que conectaba MISP (inteligencia) con la VM víctima (telemetría) y correlacionaba ambas fuentes para producir las alertas.

---

## Arquitectura

![Arquitectura del laboratorio](nullsec-architecture.png)

Tres máquinas independientes interconectadas por una **red mesh privada con Tailscale**, cada una a cargo de un módulo:

| Módulo | Rol | Componentes |
|---|---|---|
| **1 — MISP** | Threat Intelligence | IOCs de Bad Rabbit (hashes, IPs C2, dominios), Galaxies MITRE ATT&CK, API para ti_misp |
| **2 — ELK Stack** *(mi módulo)* | **Núcleo de detección (SIEM)** | Elasticsearch · Kibana · Logstash · Fleet Server · ti_misp · 4 reglas KQL |
| **3 — VM Víctima** | Endpoint comprometido | Windows 10 · Elastic Agent · Sysmon · ejecución de BadRabbit.exe |

---

## Mi trabajo — El SIEM (Módulo 2)

- **Stack ELK 8.19.16** completo (Elasticsearch, Kibana, Logstash) desplegado y endurecido.
- **Fleet Server** (puerto 8220) para gestión centralizada de los Elastic Agents.
- **Integración ti_misp v1.43.1** para sincronizar IOCs desde MISP hacia el SIEM. Decisión clave: fijar el `Initial interval` en **2160h (90 días)** en vez del valor por defecto (120h), porque los IOCs de Bad Rabbit se habían cargado 37+ días antes de la integración y de otro modo nunca se hubieran sincronizado.
- **4 reglas de detección KQL** en el Detection Engine de Kibana, ejecutándose cada 5 minutos.

![Reglas de detección](nullsec-reglas-lista.png)

---

## Resultados

Tras la ejecución controlada de Bad Rabbit en la VM víctima, **las 4 reglas dispararon**:

![Alertas disparadas](nullsec-alertas.png)

- **MTTD estimado: 5-6 minutos**
- **484** documentos IOC sincronizados vía ti_misp
- **46** eventos Sysmon capturados durante la ejecución
- Detección validada de extremo a extremo: **PowerShell malicioso · IP de C2 · DNS malicioso · URL maliciosa**
- Técnicas mapeadas a MITRE ATT&CK (T1218 rundll32, T1485, T1562.001, entre otras)

---

## Stack técnico

`Elasticsearch` · `Kibana` · `Logstash` · `Fleet Server` · `Elastic Agent` · `ti_misp` · `MISP` · `Sysmon v15.21` · `Tailscale VPN` · `CAPEv2` · `MITRE ATT&CK` · `KQL`

---

## Aprendizajes

- La detección efectiva no depende de una sola herramienta, sino de la **correlación** entre inteligencia (MISP) y telemetría (Sysmon) a través del SIEM.
- Un detalle de configuración (el intervalo de sincronización) puede ser la diferencia entre detectar o no detectar una amenaza real.
- Diseñar reglas KQL exige entender tanto la técnica ofensiva (cómo actúa el ransomware) como la fuente de datos (qué evento genera cada acción).

---

<div align="center">
  <sub>Parte del portfolio de <a href="https://github.com/juanmalbran">Juan Malbrán · M4LBYTE</a></sub>
</div>
