# 💀 NullSector — Blog técnico en Jekyll
![Texto alternativo](assets/img/site/neonsamurai.jpg)

**NullSector** es un blog personal construido con [Jekyll](https://jekyllrb.com/), enfocado en temas de seguridad ofensiva, criptografía, reversing y hardware hacking. El sitio está altamente personalizado para mantener una estética retro/underground mientras sirve como base de conocimiento y portafolio técnico.

---

## 🚀 Cómo levantar el sitio localmente

### ✅ Requisitos (recomendados)

- [Ruby](https://www.ruby-lang.org/) ~> 3.3.X (instalado previamente)
- [Bundler](https://bundler.io/) (`gem install bundler`) ~> 2.6.X (instalado previamente)
- [Jekyll](https://jekyllrb.com/) (`gem install jekyll`) ~> 7.3 (se instala automaticamente con las gemas)

### 🔧 Instrucciones

1. **Clona el repositorio:** Debemos clonar el repositorio y meternos dentro de la carpeta.

```bash
$ git clone https://github.com/cristobalonz/cristobalonz.github.io.git
$ cd cristobalonz.github.io
```

2. **Instalamos las gemas:** Creamos una carpeta donde guardar las gemas dentro del proyecto para mayor control de estas.
```bash
#Para configurar la ruta de las gemas
$ bundle config set --local path vendor/bundle
#Instalar las gemas en esa carpeta
$ bundle install
```

3. **Servimos la aplicacion en el localhost:** Deberia iniciar sin ningun problema.
```bash
$ bundle exec jekyll serve 
```

---

## Modificaciones al tema base

Para que el sitio quede mas personalizado se le hicieron las siguientes modificaciones al tema jekyll-chirpy version 7.3:

- **_layouts/home.html:** Se agrego una imagen de tokio con letras responsivas y un texto que dice "posts recientes".

---

## Licencia del blog

**Licencia MIT:**
Este sitio y su código fuente están bajo la Licencia MIT, lo que significa que puedes usar, copiar, modificar y compartir libremente el contenido y el código, incluso con fines comerciales. Solo te pedimos que mantengas la atribución original y el aviso de licencia si usas propiedad intelectual de codigo de este sitio. No ofrecemos garantías sobre el uso del software; se proporciona “tal cual”. Todo el contenido de este blog ha sido liberado al dominio público bajo [CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/). Puedes copiar, reutilizar o modificar libremente, sin necesidad de atribución. El conocimiento es libre.

---

## Licencias de imagenes utilizadas (solo externas)

Las licencias de las imagenes utilizadas copiadas de otro autor se listan a continuacion, las que he creado yo no la listo porque las libero al dominio publico con la licencia de contenido [CC0 1.0](https://creativecommons.org/publicdomain/zero/1.0/).

**assets/img/site/tokio-night.jpg**: Esta imagen fue tomada de WallpaperCave (https://wallpapercave.com/) con fines no comerciales y educativos. Si el autor desea que se retire o se atribuya, puede contactarme.

**assets/img/site/tokio-street.jpg**: Esta imagen fue tomada de Pexels (https://www.pexels.com/photo/dark-alley-1510610/) y es free to use.

Si alguna imagen no sale y es externa es porque ya no recuerdo de donde la saque pero si eres el autor y quieres que la baje contactame.

## Notas de desarrollo:

- Se debe mantener el workflow de github actions.
- Ver posibilidad preloader y mantener las imagenes en cache.





