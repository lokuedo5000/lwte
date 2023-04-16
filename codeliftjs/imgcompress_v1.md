### Descripcion
En este tutorial, aprenderás cómo crear un compresor de imágenes sin reducir su tamaño y sin pérdida significativa de calidad utilizando Node.js y la biblioteca sharp.
<br>
<br>
Sharp es una biblioteca de procesamiento de imágenes de alto rendimiento para Node.js que nos permitirá aplicar la compresión sin pérdida a nuestras imágenes.
<br>
<br>
El objetivo de este compresor de imágenes es reducir el tamaño del archivo de imagen sin afectar su calidad visual. Para ello, utilizaremos los algoritmos de compresión sin pérdida que ofrece sharp.
<br>
<br>
En la primera parte del tutorial, explicaremos los conceptos básicos de la compresión de imágenes y cómo funciona el algoritmo de compresión sin pérdida que utilizaremos. Luego, crearemos una aplicación de línea de comandos utilizando Node.js y sharp para procesar imágenes y aplicar la compresión sin pérdida.
<br>
<br>
En la aplicación, permitiremos al usuario especificar la calidad de la imagen comprimida utilizando un valor de 0 a 100. Utilizaremos el módulo fs de Node.js para leer y escribir archivos de imagen.
<br>
<br>
Al finalizar el tutorial, tendrás un compresor de imágenes funcional que te permitirá reducir el tamaño de los archivos de imagen sin afectar la calidad visual. Además, habrás aprendido cómo utilizar la biblioteca sharp de Node.js para aplicar la compresión sin pérdida en imágenes.
<br>
### Comencemos

#### Para iniciar con este tutorial debes seguir estos pasos:

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
      <div class="inputs_list">
        <label for="images">Seleccionar imagenes</label>
        <input type="file" id="images" name="images" multiple />
      </div>
      <br>
      <div class="inputs_list">
        <label for="png_calidad">Nivel de compresion de imagenes (png)</label>
        <br>
        <p>Un valor de compresión de 9 (máxima compresión) se traducirá en una mayor compresión de la imagen, lo que resultará en un tamaño de archivo más pequeño, pero también puede afectar la calidad de la imagen, especialmente si se usan imágenes con muchos detalles o gradientes.</p>
        <select name="compressionLevel" id="png_calidad">
          <option value="1">1%</option>
          <option value="2">2%</option>
          <option value="3">3%</option>
          <option value="4">4%</option>
          <option value="5">5%</option>
          <option value="6">6%</option>
          <option value="7">7%</option>
          <option value="8">8%</option>
          <option value="9">9%</option>
        </select>
      </div>
      <br>
      <div class="inputs_list">
        <label for="jpg_calidad">Nivel de compresion de imagenes (jpg)</label>
        <br>
        <p>El valor de calidad controla la cantidad de información que se guarda en la imagen. Un valor mayor de calidad (por ejemplo, 100) preservará más detalles de la imagen original, mientras que un valor menor (por ejemplo, 1) reducirá la calidad y aumentará la compresión. En otras palabras, a medida que disminuye la calidad, aumenta la compresión, lo que conduce a una imagen con menor tamaño de archivo pero con una menor calidad visual.</p>
        <select name="quality" id="jpg_calidad">
          <option value="20">20%</option>
          <option value="40">40%</option>
          <option value="60">60%</option>
          <option value="80">80%</option>
        </select>
      </div>

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
    // Validar que se hayan seleccionado imágenes
    if (!req.files || req.files.length === 0) {
      return res.status(400).send("Debe seleccionar al menos una imagen.");
    }

    // el valor de para comprimir las imagenes png
    var levelpng = req.body.compressionLevel;

    // el valor de para comprimir las imagenes jpg
    var qualityjpg = req.body.quality;

    // Validar que solo se hayan enviado archivos de imagen
    const allowedExtensions = ["jpg", "jpeg", "png"];
    const invalidFiles = req.files.filter(
      (file) =>
        !allowedExtensions.includes(
          file.originalname.split(".").pop().toLowerCase()
        )
    );
    if (invalidFiles.length > 0) {
      return res
        .status(400)
        .send("Solo se permiten archivos de imagen (JPG, JPEG, PNG).");
    }

    // Comprimir cada imagen utilizando Sharp
    const compressedImages = await Promise.all(
      req.files.map(async (file) => {
        // Datos de las imágenes
        const { originalname, path } = file;

        // Ruta donde se almacenará la imagen comprimida
        const outputPath = "pagina" + "/public" + `/compressed/${originalname}`;

        // Detectar el formato de archivo original y guardar la imagen en el mismo formato
        const image = sharp(path);
        const metadata = await image.metadata();
        const format = metadata.format;

        // Comprimir la imagen con una calidad {levelpng, qualityjpg}
        await image
          .toFormat(format)
          .png({ compressionLevel: parseInt(levelpng) })
          .jpeg({ quality:  parseInt(qualityjpg) })
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

app.set("views", path.join(__dirname, "pagina", "views"));
app.use(express.static(path.join(__dirname, "pagina", "public")));

// Pagina de Inicio
app.get("/", (req, res) => {
  res.sendFile(path.join(__dirname, "pagina", "views", "index.html"));
});

// Ruta para comprimir
app.post("/compress", upload.array("images"), async (req, res) => {
  try {
    // Validar que se hayan seleccionado imágenes
    if (!req.files || req.files.length === 0) {
      return res.status(400).send("Debe seleccionar al menos una imagen.");
    }

    // el valor de para comprimir las imagenes png
    var levelpng = req.body.compressionLevel;

    // el valor de para comprimir las imagenes jpg
    var qualityjpg = req.body.quality;

    // Validar que solo se hayan enviado archivos de imagen
    const allowedExtensions = ["jpg", "jpeg", "png"];
    const invalidFiles = req.files.filter(
      (file) =>
        !allowedExtensions.includes(
          file.originalname.split(".").pop().toLowerCase()
        )
    );
    if (invalidFiles.length > 0) {
      return res
        .status(400)
        .send("Solo se permiten archivos de imagen (JPG, JPEG, PNG).");
    }

    // Comprimir cada imagen utilizando Sharp
    const compressedImages = await Promise.all(
      req.files.map(async (file) => {
        // Datos de las imágenes
        const { originalname, path } = file;

        // Ruta donde se almacenará la imagen comprimida
        const outputPath = "pagina" + "/public" + `/compressed/${originalname}`;

        // Detectar el formato de archivo original y guardar la imagen en el mismo formato
        const image = sharp(path);
        const metadata = await image.metadata();
        const format = metadata.format;

        // Comprimir la imagen con una calidad del {levelpng, qualityjpg}
        await image
          .toFormat(format)
          .png({ compressionLevel: parseInt(levelpng) })
          .jpeg({ quality:  parseInt(qualityjpg) })
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
