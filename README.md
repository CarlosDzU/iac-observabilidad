# iac-observabilidad

## 📝 Descripción General
Este repositorio contiene la implementación como código de un stack de observabilidad completo. Incluye aplicaciones de prueba (Frontend y Backend) instrumentadas para emitir métricas y logs. El stack recopila, procesa y visualiza estos datos a través de:
- **Prometheus:** Recolección de métricas (del host, contenedores y aplicaciones).
- **Loki & Alloy:** Recolección, enrutamiento y almacenamiento de logs de aplicación e infraestructura.
- **Grafana:** Visualización centralizada mediante dashboards y gestión de alarmas.

---

## 🚀 Guía de Implementación

### Levantar el Entorno
Para construir las imágenes locales y levantar todos los servicios en segundo plano, sitúate en la raíz del proyecto y ejecuta:
```bash
docker compose up -d --build
```

### Limpieza del Entorno
Al finalizar las pruebas, para detener los contenedores y **eliminar todos los volúmenes** (borrando datos guardados de Grafana y Prometheus):
```bash
docker compose down -v
```

### Comandos Útiles para Resolución de Errores
Si algún servicio falla o necesitas diagnosticar problemas, estos comandos te serán útiles:
- **Ver el estado de los contenedores:** `docker compose ps`
- **Ver los logs de un servicio en tiempo real:** `docker compose logs -f <nombre_servicio>` (ej. `docker compose logs -f grafana`)
- **Detener el stack sin borrar los datos:** `docker compose down`

---

## ⚠️ Modificaciones Respecto a la Guía Original

Durante la implementación en el laboratorio, se realizaron dos cambios clave en las consultas PromQL respecto a las indicaciones originales, adaptándolas al entorno real:

1. **Panel 8.1 (CPU contenedor backend):**
   - **Consulta original:** `sum(rate(container_cpu_usage_seconds_total{name="lab-backend"}[1m])) * 100`
   - **Consulta implementada:** `rate(backend_process_cpu_seconds_total{job="backend"}[1m]) * 100`
   - **Explicación:** En el código del backend (`server.js`), la librería de Prometheus fue configurada para forzar el prefijo `backend_` a todas las métricas nativas por defecto. Debido a esto y a ciertas limitaciones de visibilidad de cAdvisor, se optó por utilizar la métrica directa del proceso expuesta por la propia aplicación en lugar de la del contenedor.

2. **Alarma de CPU:**
   - **Consulta original:** Apuntaba al CPU del contenedor.
   - **Consulta implementada:** `100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[1m])) * 100)`
   - **Explicación:** Se modificó la condición de la alarma para que se dispare basándose en la métrica del CPU total del Host físico en lugar del contenedor aislado.

---
