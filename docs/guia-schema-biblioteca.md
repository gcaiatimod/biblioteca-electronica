# Guía de Schema.org (Microdatos HTML), Separación de Datos y Maquetación — Biblioteca Electrónica

Documentación de referencia basada en la implementación validada en:
- [index.html](file:///opt/lampp/htdocs/WWW/biblioteca-electronica/index.html) (versión con Poncho local en `dist/css/`)
- [catalogo-cdn.html](file:///opt/lampp/htdocs/WWW/biblioteca-electronica/catalogo-cdn.html) (versión con Poncho desde CDN de argentina.gob.ar)
- [detalle.html](file:///opt/lampp/htdocs/WWW/biblioteca-electronica/detalle.html) (ficha de detalle)

---

## 1. Parámetros generales de maquetación y accesibilidad (Poncho + Bootstrap 3)

1. **Sin CSS personalizado:** No agregar etiquetas `<style>` ni atributos `style="..."`. Utilizar únicamente clases nativas de Bootstrap 3.4.1 y Poncho.
2. **Negritas semánticas:** No utilizar `<b>` ni `<strong>`; usar `<span class="fw-bold">`.
3. **Tamaño de botones:** Usar `.btn.btn-default.btn-sm` en el botón secundario de la cabecera de tarjeta (`Agregar a Mi catálogo`) para mantenerlo compacto, y `.btn.btn-default` en tamaño estándar para acciones como `Resumen disponible`.
4. **Título como enlace accesible al detalle:**
   - El `<h3>` contiene directamente el `<a>` hacia la ficha del catálogo (`/catalogo/{id}`), aprovechando la regla nativa de Poncho (`h3 a`) con subrayado y estados `:hover` / `:focus`.
   - Se incluye `<span class="sr-only"> — Ver más información</span>` dentro del `<a>` para contextualizar el destino en lectores de pantalla **sin alterar** el valor de `itemprop="name"`.
5. **Iconos informativos en tablas (`Descargá` / `Solicitá`):**
   - Si el icono no ejecuta una acción ni enlaza a otro recurso, no debe llevar `<a>` ni `potentialAction`.
   - Usar `aria-hidden="true"` y `title="..."` en el `<i>`, acompañado de `<span class="sr-only">...</span>` (por ejemplo: `"Descargá directamente"` y `"No disponible para solicitar"`).

---

## 2. Mapeo dinámico de `itemtype` según tipo de catálogo

Cada tarjeta de resultado es un contenedor `<div class="panel panel-default m-b-2" itemscope itemtype="...">` cuyo `itemtype` se asigna dinámicamente según el tipo de publicación:

| Tipo en catálogo (`genre`) | `itemtype` de Schema.org |
| :--- | :--- |
| `libros` | `https://schema.org/Book` |
| `revistas` | `https://schema.org/Periodical` |
| `actas de congreso` | `https://schema.org/ScholarlyArticle` |
| `tesis` | `https://schema.org/Thesis` |

---

## 3. Reglas de separación y limpieza de datos para Microdatos

Al renderizar datos provenientes de base de datos o cadenas concatenadas, se deben aislar los valores puros dentro de los nodos `itemprop` para que los validadores (Schema.org Validator / Google Rich Results) extraigan datos limpios sin texto auxiliar ni separadores.

### 3.1. Tipo y Acceso (`genre` y `conditionsOfAccess`)
Se colocan directamente en las etiquetas visuales superiores:
```html
<span class="label label-primary" itemprop="genre">libros</span>
<span class="label label-warning" itemprop="conditionsOfAccess">Acceso Abierto</span>
```

### 3.2. Título y URL (`name` y `url`)
Separar `<span itemprop="name">` del texto accesible `.sr-only` dentro del mismo `<a>`:
```html
<h3 class="m-t-0 m-b-1">
    <a href="https://biblioteca.sicyt.gob.ar/catalogo/203180" itemprop="url">
        <span itemprop="name">: Exploring Its Past to Protect Its Future: : Exploring Its Past to Protect Its Future</span>
        <span class="sr-only"> — Ver más información</span>
    </a>
</h3>
```
- `itemprop="url"` toma el `href` del `<a>`.
- `itemprop="name"` toma **solo** el título limpio (excluyendo `" — Ver más información"`).

### 3.3. ISSNs (`issn`) — solo en revistas (`Periodical`)
Aislar únicamente el código numérico `XXXX-XXXX` dentro de `<span itemprop="issn">`, dejando las aclaraciones `(impreso)` y `(en línea)` por fuera:
```html
<p class="m-b-1 font-sans-serif small">
    <span class="fw-bold">ISSNs</span>
    <span itemprop="issn">1647-6158</span> (impreso)
    <span itemprop="issn">1647-7316</span> (en línea)
</p>
```

### 3.4. Autores/as (`author` → `Person`)
Cuando la base devuelve una lista o cadena separada por `;`, iterar cada autor en su propio nodo `Person` y dejar el `;` fuera de las etiquetas:
```html
<p class="m-b-1 font-sans-serif small">
    <span class="fw-bold">Autores/as:</span>
    <span class="text-muted">
        <span itemprop="author" itemscope itemtype="https://schema.org/Person">
            <span itemprop="name">Mariana Da Silva Lorenz</span>
        </span>
        ;
        <span itemprop="author" itemscope itemtype="https://schema.org/Person">
            <span itemprop="name">Mariana Galvani</span>
        </span>
    </span>
</p>
```

### 3.5. Tabla de Disponibilidad
Cada fila `<tr>` representa una fuente/repositorio disponible para la obra:

- **Institución detectada:** **Sin `itemprop`**. Valores como `"No requiere"` indican validación de acceso/IP, no el proveedor de la publicación.
- **Año de publicación (`datePublished`):**
  - Si tiene valor cargado: `<td data-label="Año de publicación" itemprop="datePublished">2022</td>`.
  - Si viene vacío de la base: **no agregar** `itemprop="datePublished"` (`<td data-label="Año de publicación"></td>`).
- **Período (`temporalCoverage`, en revistas):**
  - Toma el texto concatenado `"desde ... / hasta ..."` directamente en el `<td>`:
    `<td data-label="Período" itemprop="temporalCoverage">desde ene. 2010 / hasta nov. 2026</td>`.
- **Navegá (`provider` → `Organization`):**
  - Se anida `Organization` en el `<td>` combinando la URL externa (`itemprop="url"`) y el nombre del repositorio (`itemprop="name"`):
    ```html
    <td data-label="Navegá" itemprop="provider" itemscope itemtype="https://schema.org/Organization">
        <a href="https://directory.doabooks.org/handle/20.500.12854/93074" target="_blank" itemprop="url">
            <span itemprop="name">Directory of Open access Books</span>
        </a>
    </td>
    ```
- **Descargá y Solicitá:** Sin microdatos ni enlaces (solo iconos informativos con `title` y `.sr-only`).

### 3.6. Cobertura temática (`about`)
En Schema.org, colocar `itemprop="about"` directamente sobre un `<a href="...">Texto</a>` hace que el extractor tome la **URL** del `href` en lugar del nombre del tema. Por eso se debe anidar un tipo estructurado:

- **En el listado (`index.html` / `catalogo-cdn.html`) usando `Thing`:**
  Cada disciplina se envuelve en `Thing` y el separador ` - ` queda fuera de la etiqueta `<a>`:
  ```html
  <p class="m-t-05 m-b-05 small">
      <span class="fw-bold">Cobertura temática:</span>
      <a href="https://biblioteca.sicyt.gob.ar/catalogo?discipline_id%5B%5D=6"
          itemprop="about" itemscope itemtype="https://schema.org/Thing">
          <span itemprop="name">Humanidades y artes</span>
      </a> -
      <a href="https://biblioteca.sicyt.gob.ar/catalogo?discipline_id%5B%5D=6.1"
          itemprop="about" itemscope itemtype="https://schema.org/Thing">
          <span itemprop="name">Historia y arqueología</span>
      </a>
  </p>
  ```
- **En la vista de detalle (`detalle.html`) usando `DefinedTerm`:**
  Cuando se dispone de la URI de taxonomía (por ejemplo FORD / BeCyT), se utiliza `https://schema.org/DefinedTerm` con `url`, `name` e `inDefinedTermSet`:
  ```html
  <div class="media-body media-middle" itemprop="about" itemscope itemtype="https://schema.org/DefinedTerm">
      <a href="https://biblioteca.sicyt.gob.ar/catalogo?discipline_id%5B%5D=6" itemprop="url">
          <span itemprop="name">Humanidades y artes</span>
          <link itemprop="inDefinedTermSet" href="https://purl.org/becyt/ford/6">
      </a>
  </div>
  ```

### 3.7. Resumen desplegable (`abstract`)
El atributo `itemprop="abstract"` se coloca en el contenedor del texto del resumen dentro de un componente nativo `<details>` y `<summary>` (estilizado automáticamente por Poncho):
```html
<details class="m-t-1" id="abstract-116252">
    <summary>Resumen disponible</summary>
    <div class="ar-details__content text-justify" itemprop="abstract">
        Texto del resumen...
    </div>
</details>
```

---

## 4. Checklist de implementación en plantilla / backend

- [ ] Asignar `itemtype` dinámicamente según el tipo de publicación (`Book`, `Periodical`, `ScholarlyArticle`, `Thesis`).
- [ ] Envolver el título en `<h3><a href="..." itemprop="url"><span itemprop="name">...</span><span class="sr-only"> — Ver más información</span></a></h3>`.
- [ ] Omitir `itemprop="datePublished"` cuando el año venga vacío.
- [ ] Separar los códigos ISSN del texto `(impreso)` / `(en línea)`.
- [ ] Iterar autores separando `;` fuera de cada nodo `Person`.
- [ ] Envolver cada disciplina de `Cobertura temática` en `Thing` (listado) o `DefinedTerm` (detalle), dejando el separador ` - ` fuera del `<a>`.
- [ ] Verificar que la ficha pase el Schema.org Validator con **0 errores y 0 advertencias**.
