# API de notas con Prometheus y Grafana

API en FastAPI para crear, listar, editar y eliminar notas. Los datos se
guardan en un volumen Docker y la aplicación expone métricas para Prometheus,
visualizadas mediante un dashboard de Grafana.

## Funcionalidades

- Interfaz web CRUD en `http://localhost:8000`.
- Documentación OpenAPI en `http://localhost:8000/docs`.
- Persistencia de notas en el volumen `notas-data`.
- Métricas Prometheus en `http://localhost:8000/metrics`.
- Tests unitarios con pytest.
- Pipeline multibranch mediante `Jenkinsfile`.
- Manifiestos de Kubernetes con Deployment, ConfigMap y PVC.

## Métricas

- `notes_total`: cantidad actual de notas almacenadas.
- `notes_created_total`: notas creadas desde que arrancó el proceso.
- `http_request_duration_seconds`: histograma de duración y llamadas por
  endpoint, método y código HTTP.

Prometheus consulta `notas-api:8000/metrics` cada 15 segundos según
[`monitoring/prometheus.yml`](monitoring/prometheus.yml).

## Inicio rápido con los contenedores existentes

Con Docker Desktop iniciado:

```powershell
docker start notas-api-metrics prometheus grafana
docker ps
```

Servicios:

- API: `http://localhost:8000`
- Prometheus: `http://localhost:9090`
- Grafana: `http://localhost:3000`

Para detenerlos:

```powershell
docker stop notas-api-metrics prometheus grafana
```

## Instalación desde cero

Crear la red y los volúmenes:

```powershell
docker network create notas-monitoring
docker volume create notas-data
docker volume create prometheus-data
docker volume create grafana-data
```

Construir y ejecutar la API:

```powershell
docker build -t notas-api:crud .
docker run -d --name notas-api-metrics --network notas-monitoring --network-alias notas-api -p 8000:8000 -e NOTES_FILE=/data/notes.json -v notas-data:/data --restart unless-stopped notas-api:crud
```

Ejecutar Prometheus y Grafana:

```powershell
docker run -d --name prometheus --network notas-monitoring -p 9090:9090 -v "${PWD}\monitoring\prometheus.yml:/etc/prometheus/prometheus.yml:ro" -v prometheus-data:/prometheus --restart unless-stopped prom/prometheus
docker run -d --name grafana --network notas-monitoring -p 3000:3000 -v grafana-data:/var/lib/grafana --restart unless-stopped grafana/grafana
```

## Configuración de Grafana

1. Ingresar a `http://localhost:3000`.
2. Agregar una fuente de datos Prometheus con URL
   `http://prometheus:9090`.
3. Importar [`monitoring/grafana-dashboard.json`](monitoring/grafana-dashboard.json).
4. Seleccionar la fuente Prometheus cuando Grafana lo solicite.

El dashboard **API de Notas - Observabilidad** incluye cantidades de notas,
tráfico, errores, latencia p95, llamadas por endpoint, códigos HTTP y evolución
temporal.

## Tests

```powershell
python -m pip install -r requirements-dev.txt
python -m pytest
```
