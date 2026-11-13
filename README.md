# Librería Honey01 — reserva temporal

Proyecto Flask de Librería Honey adaptado a la **Actividad 1: Carrito con reserva temporal**.

Se conserva la estructura, las categorías, los PDFs en GridFS, las cuentas, el panel de administración, la mercancía y los estilos existentes. La reserva temporal se aplica a los artículos de **Merch** (libros físicos y mercancía), porque son productos que tienen unidades disponibles en inventario. Los libros digitales PDF continúan siendo de lectura gratuita.

## Qué hace la reserva

- Al pulsar **Reservar durante 5 minutos**, la aplicación descuenta temporalmente una unidad del stock disponible.
- La reserva se guarda en MongoDB Atlas, colección `reservas_temporales`.
- Se guarda el artículo, el identificador del navegador, el usuario registrado cuando inició sesión, el nombre del comprador, la fecha de creación y la fecha de vencimiento.
- El carrito muestra un contador regresivo.
- Si el comprador cancela, la unidad vuelve al stock.
- Si pasan 5 minutos, la reserva vence y la unidad vuelve al stock la próxima vez que la aplicación consulte o libere reservas.
- Si el comprador inicia sesión y confirma antes de que termine el tiempo, la reserva cambia a `comprado` y se registra el pedido de demostración.
- La operación de reserva descuenta stock con una actualización condicional en MongoDB. Si ya no hay unidades disponibles, la API no crea otra reserva.

## Archivos principales

- `app.py`: conexión a MongoDB, reservas, vencimiento, cancelación y confirmación.
- `templates/merch.html`: botón para reservar artículos.
- `templates/carrito.html`: carrito y contador de 5 minutos.
- `templates/checkout.html`: confirmación de compra en modo demostración.
- `static/estilos.css`: se conservan los estilos del proyecto original.

## Configuración local

1. Abre la carpeta `libreria_honey01` en Visual Studio Code.
2. Abre una terminal en esa carpeta.
3. Instala las dependencias:

```bash
pip install -r requirements.txt
```

4. Configura las variables de entorno. En PowerShell, para la terminal actual:

```powershell
$env:MONGO_URI = "TU_CADENA_DE_MONGODB_ATLAS"
$env:SECRET_KEY = "una-clave-larga-y-aleatoria"
$env:ADMIN_EMAIL = "tu-correo-administrador"
$env:ADMIN_PASSWORD = "una-contrasena-de-administrador"
$env:RESERVA_SEGUNDOS = "60"
python app.py
```

No publiques credenciales ni cadenas de conexión en GitHub. En Render, agrega estas mismas variables desde **Environment**. No subas un archivo `.env` con contraseñas al repositorio.

Si no configuras `MONGO_URI` en tu equipo local, la aplicación intenta usar MongoDB local en `127.0.0.1`. En Render, `MONGO_URI` es obligatorio.

## Cómo probar la actividad

1. Inicia sesión en Librería Honey.
2. En el panel Admin agrega al menos seis artículos de prueba en Merch, con stock mayor que cero. Puedes usar tipo **Libro físico**.
3. Abre Merch y verifica el stock disponible.
4. En una ventana normal, pulsa **Reservar durante 5 minutos** en un artículo.
5. Comprueba que aparece en el carrito con el contador y que el stock disponible disminuye.
6. Abre la aplicación en una ventana de incógnito y actualiza Merch. El artículo reservado ya no tendrá esa unidad disponible. Si el stock era 1, aparecerá agotado y la segunda reserva será rechazada.
7. En la primera ventana, prueba **Cancelar reserva** y verifica que vuelve la unidad al stock.
8. Reserva de nuevo y espera 5 minutos. Al actualizar el carrito o Merch, la reserva vencida se libera y la unidad regresa al stock.
9. Reserva otra vez e inicia sesión para confirmar la compra antes de que termine el contador. La reserva cambia a `comprado` y se registra el pedido como demostración.

**Importante:** para demostrar el rechazo de la segunda persona, utiliza un artículo con stock inicial de 1. Si el stock inicial es mayor, la segunda persona podrá reservar otra unidad legítimamente.

## Duración

La duración predeterminada es de 5 minutos. Se puede cambiar con la variable `RESERVA_SEGUNDOS`. La aplicación limita el valor entre 15 y 1800 segundos.

## Publicar en GitHub y Render

- Build Command: `pip install -r requirements.txt`
- Start Command: `gunicorn app:app`
- Variables de Render: `MONGO_URI`, `SECRET_KEY`, `ADMIN_EMAIL`, `ADMIN_PASSWORD` y `RESERVA_SEGUNDOS=300`.

En MongoDB Atlas, asegúrate de que el usuario tenga permisos de lectura y escritura y que Network Access permita la conexión del servicio Render.

## Aclaraciones de la demostración

- El identificador de comprador se guarda en la sesión del navegador. Una ventana normal y una ventana de incógnito funcionan como compradores distintos.
- La compra y el formulario de pago continúan siendo una simulación escolar. No se conecta una pasarela ni se realiza ningún cobro real.
- La liberación automática se ejecuta al consultar el carrito o Merch, o al cancelar una reserva; no requiere un proceso en segundo plano.


## Comprobante QR de compra

El código QR no aparece en las tarjetas de libros físicos ni mercancías. Solo se crea cuando el cliente confirma la compra. La página de comprobante muestra el detalle de artículos, cantidades, fecha, hora y total, y el QR permite abrir nuevamente ese comprobante. Configura `APP_BASE_URL` con la URL pública de Render. La compra permanece en modo demostración y no procesa pagos reales.
