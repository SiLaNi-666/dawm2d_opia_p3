# Aplicación Streamlit para el precio de casas en California

Aplicación web que estima el precio medio de las viviendas de una zona de California.
Está formada por dos servicios: una **API hecha con FastAPI** (`app.py`), que carga el modelo
(`modelo_california.pkl`, un árbol de decisión) y hace la predicción, y una **interfaz hecha con
Streamlit** (`streamlit_app.py`), donde el usuario escribe el ingreso medio y el número de habitaciones.
La interfaz completa el resto de variables con valores por defecto, comprueba que están las 8,
las envía en JSON con un `POST /predict` a la API y muestra el resultado en dólares.
Si algo falla (faltan datos, la API devuelve un error o no responde), muestra un mensaje claro.

Autores: Daniel Gracia, David Lopez

## Servicios desplegados en Render

| Servicio | Qué es | URL |
|----------|--------|-----|
| API | FastAPI con el modelo (`app.py`) | <https://dawm2dopia-p3-api.onrender.com> |
| Interfaz | Streamlit (`streamlit_app.py`), que llama a la API | <https://dawm2dopia-p3-web.onrender.com> |

## Cómo ejecutarla en local

```bash
pip install -r requirements.txt
uvicorn app:app --reload                 # terminal 1: la API en http://localhost:8000
streamlit run streamlit_app.py           # terminal 2: la interfaz en http://localhost:8501
```

En la barra lateral de la interfaz, en "URL de la API", hay que poner `http://localhost:8000`.

## Despliegue en Render

Los dos servicios son *Web Service* (plan Free) creados desde el mismo repositorio de GitHub.

- **API:** Build Command `pip install -r requirements.txt` ·
  Start Command `uvicorn app:app --host 0.0.0.0 --port $PORT`
- **Interfaz:** Build Command `pip install -r requirements.txt` ·
  Start Command `streamlit run streamlit_app.py --server.port $PORT --server.address 0.0.0.0 --server.headless true`

La URL de la API que usa la interfaz se cambia en la constante `API_URL` de `streamlit_app.py`
(o, sin tocar el código, en la barra lateral de la aplicación).

## Documentación de la API

URL base: <https://dawm2dopia-p3-api.onrender.com>

### `POST /predict`

Recibe un JSON con las 8 variables de una zona censal (todas numéricas) y devuelve el precio medio
estimado de sus viviendas en `MedHouseVal`, en cientos de miles de dólares (4.88 = 488.000 $).

**Ejemplo de petición**

```json
{
  "MedInc": 8.3, "HouseAge": 29.0, "AveRooms": 6.9, "AveBedrms": 1.05,
  "Population": 1166.0, "AveOccup": 2.8, "Latitude": 34.26, "Longitude": -118.49
}
```

**Ejemplo de respuesta correcta (200)**

```json
{"MedHouseVal": 4.881880465116272}
```

**Errores**

| Código | Cuándo ocurre | Qué muestra la aplicación |
|--------|---------------|---------------------------|
| 422 | Falta un campo o tiene un tipo incorrecto | "La API ha devuelto un error (código 422). Revisa los datos introducidos." y el detalle de la API |
| Sin respuesta | La API no está disponible o tarda demasiado | "No se ha podido conectar con la API. Comprueba que la URL de la barra lateral es correcta y que el servicio está en marcha..." |
