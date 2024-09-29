
# Proyecto de Clasificación de Imágenes con Discord Bot y Teachable Machine

Este proyecto permite clasificar imágenes utilizando un modelo exportado desde Teachable Machine e integrado en un bot de Discord.

## Instrucciones Paso a Paso

### 1. **Exportar Tu Modelo desde Teachable Machine**

- Exporta **tu propio modelo** de Teachable Machine en formato Keras.
- Guarda los archivos `keras_model.h5` y `labels.txt`.
- Asegúrate de que estos archivos reflejan el entrenamiento de **tu modelo** personalizado.

### 2. **Configurar el Entorno en Visual Studio Code**

#### a) **Instalar Python 3.10**

- Descarga Python 3.10 desde [este enlace](https://www.python.org/downloads/release/python-3100/). 
- **Nota**: No es necesario desinstalar Python 3.12 si ya lo tienes instalado.

#### b) **Verificar ambas versiones de Python**

En una terminal, ejecuta el siguiente comando para verificar que ambas versiones están instaladas:

```bash
py -0p
```

#### c) **Crear el entorno virtual**

En Visual Studio Code, ejecuta los siguientes comandos:

```bash
pip install pipenv
pipenv --python "C:\Users\<tu_usuario>\AppData\Local\Programs\Python\Python310"
```

> **Nota**: Puedes encontrar la ruta de Python buscando en tu sistema archivos o usando `py -0p`.

#### d) **Activar el entorno virtual**

Ejecuta:

```bash
pipenv shell
```

### 3. **Instalar Dependencias**

En el entorno virtual, instala las dependencias necesarias:

```bash
pipenv install Pillow==9.1.0
pipenv install tensorflow==2.8.0
pipenv install discord.py
pipenv install numpy==1.21.6
pipenv install keras
pipenv install requests
pipenv install protobuf==3.20.3
```

### 4. **Crear los Archivos del Proyecto**

#### a) **Archivo `model.py`**

Crea un archivo `model.py` y pega el siguiente código que servirá para cargar **tu modelo** de Keras y hacer las predicciones. Este archivo se puede usar con cualquier modelo que hayas exportado desde Teachable Machine:

```python
from keras.models import load_model
from PIL import Image, ImageOps
import numpy as np

def get_class(model_path, labels_path, image_path):
    np.set_printoptions(suppress=True)
    model = load_model(model_path, compile=False)
    class_names = open(labels_path, "r").readlines()

    data = np.ndarray(shape=(1, 224, 224, 3), dtype=np.float32)
    image = Image.open(image_path).convert("RGB")
    size = (224, 224)
    image = ImageOps.fit(image, size, Image.Resampling.LANCZOS)
    image_array = np.asarray(image)
    normalized_image_array = (image_array.astype(np.float32) / 127.5) - 1
    data[0] = normalized_image_array

    prediction = model.predict(data)
    index = np.argmax(prediction)
    class_name = class_names[index]
    confidence_score = prediction[0][index]

    return (class_name[2:], confidence_score)
```

#### b) **Archivo `main.py`**

Crea el archivo `main.py` para tu bot de Discord, que usará **tu modelo** para hacer predicciones basadas en imágenes que se suben al canal de Discord:

```python
import discord
from discord.ext import commands
import os, random
from model import get_class
import requests

intents = discord.Intents.default()
intents.message_content = True
bot = commands.Bot(command_prefix='$', intents=intents)

@bot.event
async def on_ready():
    print(f'We have logged in as {bot.user}')

@bot.command()
async def hello(ctx):
    await ctx.send(f'Hi! I am a bot {bot.user}!')

@bot.command()
async def heh(ctx, count_heh = 5):
    await ctx.send("he" * count_heh)

@bot.command()
async def check(ctx):
    if ctx.message.attachments:
        for attachment in ctx.message.attachments:
            file_name = attachment.filename
            file_url = attachment.url
            await attachment.save(f"./img/{file_name}")
            await ctx.send(f"Guarda la imagen en ./img/{file_url}")
            class_name, confidence_score = get_class(model_path="keras_model.h5", labels_path="labels.txt", image_path=f"./img/{file_name}")
            response_message = f"**Predicción:** {class_name}\n**Confianza:** {confidence_score:.2f}"
            await ctx.send(response_message)
    else:
        await ctx.send("You forgot to upload the image :(")

bot.run("TU_TOKEN_DISCORD")
```

### 5. **Ejecutar el Proyecto**

1. Sube **tu modelo** `keras_model.h5` y el archivo `labels.txt` a la carpeta del proyecto.
2. Ejecuta el bot de Discord con:

```bash
python main.py
```

3. Sube una imagen en Discord y utiliza el comando `$check` para que el bot clasifique la imagen usando **tu modelo**.

---

### Notas:

- Cada miembro del equipo debe usar su propio modelo exportado desde Teachable Machine.
- Asegúrate de modificar `TU_TOKEN_DISCORD` con tu token de bot de Discord.
- Puedes subir imágenes de ejemplo a Discord para probar el modelo.

---
