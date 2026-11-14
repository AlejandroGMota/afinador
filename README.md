# Afinador

Afinador web de guitarra (afinación estándar E A D G B E). Un solo `index.html`, todo corre en el navegador; detección de tono con [pitchy](https://github.com/ianprime0509/pitchy) (McLeod Pitch Method).

Producción: https://afinador.alejandrogmota.com (VM de Oracle, nginx, carpeta `/var/www/afinador`).

## Probar en local
```sh
python3 -m http.server 8000
```
`http://localhost:8000` cuenta como origen seguro, así que el micrófono funciona sin HTTPS.

## Subir cambios
```sh
scp -i ~/.ssh/oracle_chavarria_vm index.html robots.txt sitemap.xml ubuntu@160.34.222.215:/var/www/afinador/
```

## Calibración
`MIN_RMS` y `MIN_CLARITY` al inicio del script. Si una guitarra eléctrica sin conectar no se detecta, bajar `MIN_RMS`.
