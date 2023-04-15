## Comencemos

#### Para iniciar con este tutorial debes seguir los pasos

1. Abre una terminal o línea de comandos en tu equipo.
2. Crea una nueva carpeta para tu proyecto e ingresa a ella:

```bash
cd Desktop
mkdir nombre-del-proyecto
cd nombre-del-proyecto
```

3. Inicializa un nuevo proyecto de Node.js usando el comando npm init. Este comando creará un archivo package.json en la carpeta de tu proyecto:

```bash
$ npm init -y
```

4. Ahora, puedes instalar los paquetes que necesites para tu proyecto. vamos a instalar (Express, Sharp, multer), puedes instalarlo con el siguiente comando:

```bash
$ npm i express sharp multer
```

5. Crea una carpeta dentro de tu proyecto y la llama `pagina` y en la carpeta `pagina` crean dos carpetas mas `views`, `public` y en `public` crea una carpeta `compressed`
6. En la carpeta `views` crea un archivo `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>nombre-del-proyecto</title>
  </head>
  <body>
    <form action="/compress" method="post" enctype="multipart/form-data">
      <input type="file" id="images" name="images" multiple />
      <button type="submit">Comprimir imágenes</button>
    </form>

    <!-- jQuery -->
    <script src="https://code.jquery.com/jquery-3.6.4.js"></script>
    <!-- Main -->
    <script src="/main.js"></script>
  </body>
</html>
```

7. Crea un archivo `main.js` en la carpeta `public`:

```javascript
$(document).ready(function () {
  $("form").submit(function (event) {
    event.preventDefault();
    $.ajax({
      url: "/compress",
      type: "POST",
      data: new FormData($("form")[0]),
      processData: false,
      contentType: false,
      success: function (data) {
        if (data) {
          console.log(data);
        }
      },
    });
  });
});
```

<br>

8. Crea un archivo `index.js` en la carpeta de tu proyecto y comienza a escribir tu código:

### Ahora te explico el codigo

El middleware `multer` se encarga de procesar cualquier archivo que se envíe en el formulario y los almacena temporalmente en el directorio `uploads/`.

```javascript
// Modulo de multer
const multer = require("multer");
// Donde se guarda las imagenes temporalmente
const upload = multer({ dest: "uploads/" });
```

Luego, el código utiliza `Sharp` para comprimir cada imagen que se ha cargado. Primero, se recorre el array de archivos para obtener la información de cada archivo. A continuación, se usa `Sharp` para cargar cada archivo y comprimirlo con una calidad del 80% en el caso de las imágenes JPEG y con un nivel de compresión del 9 en el caso de las imágenes PNG. Después de comprimir cada imagen, se guarda la imagen comprimida en el directorio compressed/.

```javascript
try {
  // Archivos seleccionados
  const files = req.files;

  // Comprimir cada imagen utilizando Sharp
  const compressedImages = await Promise.all(
    files.map(async (file) => {
      // datos de las imagenes
      const { originalname, path } = file;

      const outputPath = "pagina" + "/public" + `/compressed/${originalname}`;

      // Detectar el formato de archivo original y guardar la imagen en el mismo formato
      const image = sharp(path);
      const metadata = await image.metadata();
      const format = metadata.format;

      // Comprimir la imagen con una calidad del 80%
      await image
        .toFormat(format)
        .jpeg({ quality: 80 })
        .png({ compressionLevel: 9 })
        .toFile(outputPath);

      // Obtener el nuevo peso y enviar la URL de la imagen comprimida al cliente
      const compressedMetadata = await sharp(outputPath).metadata();
      const compressedSize = fs.statSync(outputPath).size;
      const compressedImageUrl = `http://localhost:3000/compressed/${originalname}`;

      // Eliminar el archivo original
      fs.unlinkSync(path);

      // Devolver un objeto con la información de la imagen comprimida
      return {
        originalname,
        compressedUrl: compressedImageUrl,
        compressedSize,
      };
    })
  );

  // Enviar una respuesta al cliente con la lista de imágenes comprimidas
  res.json(compressedImages);
} catch (error) {
  console.error(error);
  res.status(500).send("Ocurrió un error al comprimir las imágenes.");
}
```

<br>

### Codigo completo

```javascript
// Dependencias
const path = require("path");
const fs = require("fs");

// Sharp
const sharp = require("sharp");

// Express
const express = require("express");
const app = express();

// Modulo de multer
const multer = require("multer");

// Donde se guarda las imagenes temporalmente
const upload = multer({ dest: "uploads/" });

// Middleware para parsear el cuerpo de las peticiones HTTP
app.use(express.urlencoded({ extended: true }));
app.use(express.json());


app.use(express.static(path.join(__dirname, "pagina", "public")));

// Pagina de Inicio
app.get("/", (req, res) => {
  res.sendFile(path.join(__dirname, "pagina", "views", "index.html"));
});

// Ruta para comprimir
app.post("/compress", upload.array("images"), async (req, res) => {
  try {
    // Archivos seleccionados
    const files = req.files;

    // Comprimir cada imagen utilizando Sharp
    const compressedImages = await Promise.all(
      files.map(async (file) => {
        // datos de las imagenes
        const { originalname, path } = file;

        const outputPath = "pagina" + "/public" + `/compressed/${originalname}`;

        // Detectar el formato de archivo original y guardar la imagen en el mismo formato
        const image = sharp(path);
        const metadata = await image.metadata();
        const format = metadata.format;

        // Comprimir la imagen con una calidad del 80%
        await image
          .toFormat(format)
          .jpeg({ quality: 80 })
          .png({ compressionLevel: 9 })
          .toFile(outputPath);

        // Obtener el nuevo peso y enviar la URL de la imagen comprimida al cliente
        const compressedMetadata = await sharp(outputPath).metadata();
        const compressedSize = fs.statSync(outputPath).size;
        const compressedImageUrl = `http://localhost:3000/compressed/${originalname}`;

        // Eliminar el archivo original
        fs.unlinkSync(path);

        // Devolver un objeto con la información de la imagen comprimida
        return {
          originalname,
          compressedUrl: compressedImageUrl,
          compressedSize,
        };
      })
    );

    // Enviar una respuesta al cliente con la lista de imágenes comprimidas
    res.json(compressedImages);
  } catch (error) {
    console.error(error);
    res.status(500).send("Ocurrió un error al comprimir las imágenes.");
  }
});

app.listen(3000, () => {
  console.log("El servidor está funcionando en el puerto 3000");
});
```

<br>

### Finalizar

En nuestro archivo `package.json` en `scripts` borramos `"test": "echo \"Error: no test specified\" && exit 1"` y Agregamos:

```javascript
"scripts": {
    "start": "node index.js"
}
```

En la consola iniciamos nuestro proyecto de la siguiente forma:
```bash
$ npm start
```