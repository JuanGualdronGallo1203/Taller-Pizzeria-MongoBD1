# Taller: Modelado NoSQL Documental con MongoDB para una Pizzería

Este documento presenta una propuesta completa para el diseño de una base de datos NoSQL documental utilizando MongoDB para gestionar una pizzería. El objetivo es explorar y justificar un modelo de datos flexible que cubra los requerimientos de productos, combos, pedidos y clientes, adaptándose a las características de una base de datos documental.

---

## 1. Investigación

### ¿Qué es una base de datos NoSQL?
Una base de datos NoSQL (Not Only SQL) es un sistema de gestión de datos que no sigue el modelo relacional tradicional de tablas, filas y columnas. Está diseñada para manejar datos no estructurados o semi-estructurados, ofreciendo:
- **Flexibilidad de esquema**: No requiere un esquema fijo, permitiendo cambios dinámicos en la estructura de los datos.
- **Escalabilidad horizontal**: Facilita la distribución de datos en múltiples servidores.
- **Alto rendimiento**: Optimizada para grandes volúmenes de datos y consultas específicas.
Existen varios tipos de bases NoSQL, como documentales (MongoDB), clave-valor (Redis), de columnas (Cassandra) y de grafos (Neo4j).

### ¿Qué es MongoDB?
MongoDB es una base de datos NoSQL de tipo documental, de código abierto, que almacena datos en documentos con formato JSON (o BSON, una versión binaria de JSON). Cada documento es una unidad de datos que puede contener campos anidados, listas y objetos, lo que lo hace ideal para modelar datos complejos y jerárquicos. MongoDB permite consultas rápidas, índices personalizados y escalabilidad horizontal mediante particionamiento (sharding).

### ¿Qué diferencia hay entre una base de datos relacional (como MySQL) y una base de datos documental como MongoDB?
- **Estructura**: 
  - Relacional (MySQL): Usa tablas con esquemas rígidos, donde los datos se dividen en filas y columnas. Las relaciones se gestionan mediante claves primarias y foráneas.
  - Documental (MongoDB): Almacena datos en documentos JSON/BSON, que son flexibles y no requieren un esquema predefinido. Los datos relacionados pueden estar incrustados o referenciados.
- **Escalabilidad**:
  - MySQL escala verticalmente (mejor hardware), lo que puede ser costoso.
  - MongoDB escala horizontalmente (más servidores), ideal para grandes volúmenes de datos.
- **Flexibilidad**:
  - MySQL requiere definir tablas y relaciones antes de insertar datos.
  - MongoDB permite agregar o modificar campos dinámicamente.
- **Consultas**:
  - MySQL usa SQL para consultas estructuradas.
  - MongoDB usa un lenguaje de consultas basado en JavaScript, más flexible para datos anidados.
- **Casos de uso**:
  - MySQL es ideal para sistemas con datos estructurados y relaciones complejas (bancos, ERP).
  - MongoDB es mejor para aplicaciones con datos variables, como catálogos de productos o sistemas de pedidos.

### ¿Qué son documentos y colecciones en MongoDB?
- **Documento**: Es una unidad de datos en MongoDB, similar a un objeto JSON. Representa una entidad (como una pizza o un pedido) y contiene pares clave-valor, que pueden incluir listas, objetos anidados o referencias a otros documentos.
- **Colección**: Es un grupo de documentos relacionados, análogo a una tabla en MySQL, pero sin un esquema fijo. Por ejemplo, una colección `pedidos` puede contener todos los documentos de pedidos de la pizzería.

---

## 2. Diseño de la Propuesta

### Colecciones Propuestas
Para la pizzería, proponemos las siguientes colecciones en MongoDB, justificadas según los requerimientos del negocio:

1. **Productos**: Almacena información de pizzas, panzarottis, bebidas, postres y adiciones. Cada documento representa un producto individual con sus detalles (nombre, precio, ingredientes, categoría).
2. **Combos**: Guarda los combos predefinidos que combinan productos (por ejemplo, pizza + bebida + postre) con un precio especial.
3. **Pedidos**: Registra los pedidos realizados, incluyendo información del cliente, productos pedidos, personalizaciones, tipo de pedido (en sitio o para recoger) y estado.
4. **Clientes**: Contiene datos de los clientes (nombre, teléfono, dirección) para facilitar la gestión de pedidos recurrentes.
5. **Ingredientes**: Almacena los ingredientes disponibles para personalización, con detalles como nombre, costo adicional y disponibilidad.

**Justificación**: Separar los datos en estas colecciones permite una organización clara y modular. Usar colecciones distintas para productos y combos evita redundancia, ya que los combos referencian productos existentes. La colección de clientes permite reutilizar datos en múltiples pedidos, mientras que los ingredientes se separan para facilitar la gestión de personalizaciones.

### Estructura de los Documentos

#### Documento de Pedido
Un documento de pedido debe contener toda la información necesaria para procesar un pedido, incluyendo referencias al cliente y productos, así como personalizaciones. Proponemos incrustar los productos pedidos con sus detalles (en lugar de solo referencias) para reducir consultas y mejorar el rendimiento en la lectura, ya que los pedidos suelen consultarse completos. Sin embargo, los datos del cliente se referencian mediante un ID para evitar duplicar información que podría cambiar (como la dirección).

**Estructura propuesta**:
- `_id`: Identificador único del pedido.
- `cliente_id`: ID del cliente (referencia a la colección `clientes`).
- `productos`: Lista de objetos con detalles de los productos pedidos, incluyendo personalizaciones (ingredientes adicionales o eliminados).
- `tipo`: Tipo de pedido ("en_sitio" o "para_recoger").
- `estado`: Estado del pedido ("pendiente", "en_preparacion", "entregado").
- `fecha`: Fecha y hora del pedido.
- `total`: Precio total calculado.

#### Documento de Producto
Un documento de producto representa un ítem del menú (pizza, bebida, etc.) con sus características. Los ingredientes se incrustan como una lista para facilitar consultas rápidas, ya que los ingredientes de un producto no cambian frecuentemente.

**Estructure propuesta**:
- `_id`: Identificador único del producto.
- `nombre`: Nombre del producto (ej. "Pizza Margarita").
- `categoria`: Categoría ("pizza", "bebida", "postre", "adicion").
- `precio`: Precio base del producto.
- `ingredientes`: Lista de IDs de ingredientes (referencias a la colección `ingredientes`).
- `disponible`: Booleano que indica si el producto está disponible.

#### Decisiones de Diseño
- **Incrustado vs. Referenciado**:
  - **Incrustado**: Los productos en un pedido se incrustan para evitar múltiples consultas al consultar un pedido completo. Esto mejora el rendimiento en aplicaciones donde los pedidos se leen frecuentemente.
  - **Referenciado**: Los clientes y los ingredientes se referencian mediante IDs para evitar duplicación de datos y facilitar actualizaciones (por ejemplo, cambiar la dirección de un cliente o el costo de un ingrediente).
- **Listas y objetos anidados**: Los productos en un pedido incluyen una lista de personalizaciones (ingredientes añadidos o eliminados) como objetos anidados, lo que permite modelar la flexibilidad de las personalizaciones sin crear colecciones adicionales.
- **Flexibilidad**: Al no tener un esquema fijo, podemos agregar nuevos campos (como descuentos o comentarios) sin modificar la estructura de la base de datos.

---

## 3. Ejemplos de Documentos JSON

A continuación, se presentan tres documentos de ejemplo en formato JSON, validados para cumplir con los requerimientos de la pizzería.

### Ejemplo 1: Producto (Pizza Margarita)
```json
{
  "_id": "prod001",
  "nombre": "Pizza Margarita",
  "categoria": "pizza",
  "precio": 12.99,
  "ingredientes": ["ing001", "ing002", "ing003"],
  "disponible": true
}
```
**Justificación**: Este documento representa una pizza con ingredientes predefinidos (referenciados por ID). La categoría permite filtrar productos en el menú, y el campo `disponible` facilita la gestión de inventario.

### Ejemplo 2: Combo
```json
{
  "_id": "combo001",
  "nombre": "Combo Familiar",
  "productos": [
    { "producto_id": "prod001", "cantidad": 1 },
    { "producto_id": "prod002", "cantidad": 2 },
    { "producto_id": "prod003", "cantidad": 1 }
  ],
  "precio": 24.99,
  "disponible": true
}
```
**Justificación**: El combo referencia productos existentes y especifica cantidades, lo que permite reutilizar productos del catálogo. El precio es fijo para el combo, independientemente de los precios individuales de los productos.

### Ejemplo 3: Pedido
```json
{
  "_id": "ped001",
  "cliente_id": "cli001",
  "productos": [
    {
      "producto_id": "prod001",
      "nombre": "Pizza Margarita",
      "precio": 12.99,
      "personalizaciones": [
        { "ingrediente_id": "ing004", "accion": "agregar", "costo_adicional": 1.50 },
        { "ingrediente_id": "ing002", "accion": "eliminar" }
      ],
      "cantidad": 1
    },
    {
      "producto_id": "prod002",
      "nombre": "Bebida Cola",
      "precio": 2.50,
      "personalizaciones": [],
      "cantidad": 2
    }
  ],
  "tipo": "para_recoger",
  "estado": "pendiente",
  "fecha": "2025-08-01T10:30:00Z",
  "total": 17.99
}
```
**Justificación**: Este documento incluye productos incrustados con sus personalizaciones, lo que facilita consultar el pedido completo sin necesidad de uniones. El cliente se referencia por ID para evitar redundancia.

---

## 4. Reflexión

### ¿Qué fue lo más difícil de imaginar sin tablas?
El mayor desafío fue abandonar el pensamiento relacional basado en tablas normalizadas. En MySQL, hubiéramos creado múltiples tablas (pedidos, productos, clientes, etc.) con relaciones mediante claves foráneas. En MongoDB, decidir qué datos incrustar y qué referenciar requirió analizar el caso de uso: priorizamos incrustar productos en pedidos para consultas rápidas, pero referenciar clientes para mantener la consistencia. Esta transición mental fue compleja, pero enriquecedora.

### ¿Qué les gustó del enfoque con documentos?
- **Flexibilidad**: La posibilidad de agregar campos dinámicamente (como descuentos o notas) sin modificar un esquema fijo es ideal para una pizzería con necesidades cambiantes.
- **Datos anidados**: Incrustar personalizaciones y productos en un pedido simplifica las consultas y refleja mejor la estructura natural de los datos.
- **Intuitivo**: Los documentos JSON son fáciles de entender, ya que representan entidades completas (como un pedido) de forma similar a cómo las visualizamos en la vida real.

### ¿Qué dudas surgieron al pensar en este nuevo tipo de base de datos?
- **Rendimiento**: ¿Cómo afecta incrustar muchos datos (como productos en pedidos) al tamaño de la base de datos y al rendimiento a largo plazo?
- **Consistencia**: Al referenciar clientes o ingredientes, ¿cómo garantizar que las referencias sean válidas si un cliente o ingrediente se elimina?
- **Consultas complejas**: ¿Cómo manejar consultas que requieren combinar datos de múltiples colecciones (equivalente a un JOIN en SQL)?
- **Escalabilidad**: Aunque MongoDB es escalable, ¿cuáles son las mejores prácticas para estructurar colecciones en una pizzería con miles de pedidos diarios?

---

## Conclusión
El diseño propuesto aprovecha la flexibilidad de MongoDB para modelar los datos de una pizzería de manera intuitiva y eficiente. Al combinar documentos incrustados (para productos en pedidos) y referenciados (para clientes e ingredientes), logramos un equilibrio entre rendimiento y mantenimiento. Este taller permitió explorar las ventajas de las bases de datos documentales y reflexionar sobre los desafíos de abandonar el modelo relacional, sentando las bases para un aprendizaje continuo en el uso de NoSQL.

---

## Autores
- Juan Sebastian Gualdron
- Cristian Miguel Perez
- Yhwrr Samuel Vega