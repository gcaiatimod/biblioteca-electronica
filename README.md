# Biblioteca Electrónica

Maquetación frontend para el catálogo y fichas de recursos de la Biblioteca Electrónica, desarrollada con el Sistema de Diseño Poncho (Argentina.gob.ar) y Bootstrap 3.4.1.

## Características

- **Diseño institucional Poncho**: Interfaz adaptada a las pautas de estilo y componentes de la Administración Pública Nacional.
- **Estructura semántica y accesible**: Implementación de buenas prácticas de accesibilidad (WAI-ARIA, textos para lectores de pantalla `.sr-only`).
- **Datos estructurados (Schema.org)**: Marcado con microdatos HTML para publicaciones (libros, revistas, tesis y artículos académicos).
- **Diseño responsivo**: Grilla fluida basada en Bootstrap 3.

## Estructura del proyecto

```text
biblioteca-electronica/
├── index.html              # Vista principal del catálogo (recursos locales de Poncho)
├── catalogo-cdn.html       # Vista del catálogo utilizando CDNs institucionales
├── detalle.html            # Ficha de detalle de una publicación
├── dist/                   # Estilos y dependencias locales de Poncho
│   └── css/
│       ├── poncho.min.css
│       └── icono-arg.css
└── docs/                   # Documentación técnica de referencia
    └── guia-schema-biblioteca.md
```

## Dependencias

- **Bootstrap**: 3.4.1
- **Poncho**: Sistema de diseño de Argentina.gob.ar
- **Font Awesome**: 4.7.0
- **jQuery**: 3.x
- **Tipografía**: Encode Sans (Google Fonts)

## Visualización

Para previsualizar el proyecto, servir el directorio mediante un servidor web local (por ejemplo Apache/XAMPP, PHP Built-in Server o extensiones tipo Live Server):

```bash
# Ejemplo con servidor embebido de PHP:
php -S localhost:8000
```

Acceder a `http://localhost:8000/index.html` desde cualquier navegador.
