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

## 💡 Preguntas Teóricas

**1. ¿Por qué necesitamos Loki además de Prometheus si ya tenemos `/metrics`?**
Porque sirven para diferentes propósitos. Por un lado, Prometheus solo guarda números, porcentajes, contadores, sirve en general para analizar datos en grandes cantidades a lo largo del tiempo, así puedes ver cuando ocurrió un problema, pero no te da muchos detalles del mismo. Por otro lado, Loki almacena logs, lo que te permite visualizar los mensajes de error, los json, líneas de texto entre otros, así puedes saber exactamente cuál fue el lugar de origen o el mensaje de lo que ocasionó estos errores, por eso ambos se usan en conjunto.

**2. ¿Qué ventaja aporta que las fuentes de datos de Grafana estén aprovisionadas como código y no creadas a mano?**
Aporta reproducibilidad, automatización y prevención de errores. Al estar definidas en un archivo (en este caso el datasources.yml), cualquier persona con acceso al repositorio puede clonarlo, hacer `docker compose up`, y Grafana ya tendrá Prometheus y Loki conectados de manera automática. Si se hiciera a mano, cada vez que se despliegue el entorno, o si se borran los contenedores por accidente, alguien tendría que entrar a la interfaz, recordar las URLs, configurar los accesos y guardarlos, lo cual es lento y propenso a errores.

**3. El panel "CPU contenedor" y el panel "CPU host" pueden mostrar valores muy distintos. ¿Por qué? ¿Cuál usarías para alertar sobre una aplicación concreta?**
Muestran valores distintos porque tienen alcances diferentes. El "CPU host" mide el porcentaje de uso de toda la máquina física o virtual, sumando todo lo que ocurre en ella. El "CPU contenedor" mide únicamente la porción de recursos que está consumiendo esa aplicación, en este caso el backend. Por ejemplo, el backend podría estar al 100% de su capacidad bloqueado por un ciclo infinito (como el del caso que simulamos), pero si el servidor tiene 8 núcleos, el host total apenas registrará un 12% de uso. Para alertar sobre una aplicación concreta usarías el "CPU contenedor", porque te interesa saber si esa aplicación está sufriendo, independientemente de si al servidor físico le sobra capacidad o no.

**4. ¿Qué diferencia hay entre el evaluation interval y el pending period de una alarma?**
El *evaluation interval* es la frecuencia con la que Grafana "despierta", ejecuta la consulta (query) y revisa si se superó el límite, mientras que el *pending period* es el tiempo que la condición debe mantenerse superada de forma continua antes de que la alarma realmente cambie a estado Firing y envíe el correo/notificación, lo que sirve como un amortiguador para evitar falsas alarmas provocadas por picos repentinos y muy cortos de milisegundos que se resuelven solos.