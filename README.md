[README.md](https://github.com/user-attachments/files/33350190/README.md)
# DENSIFICA — Simulador de Edificación Vertical

Aplicación académica interactiva para explorar geometría, volumetría y presupuesto de una edificación vertical.

## Ejecutar localmente

Por usar módulos JavaScript y Three.js desde CDN, abre el proyecto mediante un servidor local (no con doble clic en `index.html`):

```bash
python -m http.server 8000
```

Después visita `http://localhost:8000`.

También puedes usar la extensión Live Server de Visual Studio Code.

## Publicar en GitHub Pages

1. Crea un repositorio y sube todos los archivos conservando esta estructura.
2. En GitHub, abre **Settings → Pages**.
3. Selecciona **Deploy from a branch**, la rama `main` y la carpeta `/ (root)`.
4. Guarda y espera a que se publique.

No requiere proceso de compilación. El HTML carga Three.js como módulo desde jsDelivr; por tanto, el visor 3D requiere conexión a internet y un navegador con WebGL habilitado.

## Funciones

- Pantalla de bienvenida y dashboard responsive.
- Edición del ancho/largo del lote, tarifas por uso y costos fijos.
- Agregar, editar y eliminar niveles (máximo 20; mínimo 1).
- Altura predeterminada por uso, editable por nivel.
- Visor 3D con rotación, zoom, perspectiva/ortográfica, cuadrícula y selección de niveles.
- Métricas y gráficas SVG generadas desde los datos del proyecto.
- Proyecto de demostración y reinicio.
- Formato numérico colombiano.

## Modelo matemático y supuestos

- Altura total: `H = Σ hᵢ`.
- Área construida bruta: `A_c = ancho × largo × número de niveles`.
- Volumen: `V = ancho × largo × H`.
- Costo por nivel: `Cᵢ = área de planta × tarifa del uso`.
- Presupuesto total: `C = Σ Cᵢ + C₀`.
- Núcleo: `A(x) = −x² + bx`, limitada a un rango físicamente plausible entre 0 y el 30% del área de planta. Esta limitación se aplica para evitar áreas negativas o irreales, pero significa que el valor mostrado es un modelo académico acotado, no un dimensionamiento de ingeniería.
- Área útil: área construida bruta menos el área del núcleo una sola vez. Para un proyecto real, se debería especificar si el núcleo se expresa por planta o como suma de todos los niveles.

## Limitaciones

- El modelo 3D es volumétrico y conceptual; no es un modelo BIM ni un proyecto constructivo.
- Los precios por m² son valores de ejemplo editables, no cotizaciones oficiales.
- No modela retranqueos, patios, estructura, fachadas detalladas, normas urbanísticas ni áreas comunes.
- La escena usa dimensiones normalizadas para mantener una vista legible; modificar las dimensiones del lote actualiza los cálculos, no la escala visual exacta del modelo.
- Las gráficas son SVG generados localmente y no requieren biblioteca externa.
