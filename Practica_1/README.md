# MODELO MULTIDIMENSIONAL PARA SG-FOOD


SG-FOOD es una empresa que gestiona la compra, distribución y comercialización de productos alimenticios a través de varias sucursales en el país. Con el objetivo de mejorar el análisis de información y la toma de decisiones, se desarrolló un proceso ETL y un modelo de datos para análisis OLAP sobre ventas y compras.

---

## 1. CONCEPTOS BÁSICOS

- **OLAP (Online Analytical Processing):** Técnica que permite explorar y analizar grandes volúmenes de datos desde múltiples perspectivas (dimensiones).
- **Dimensiones:** Ejes de análisis como tiempo, producto, sucursal, cliente, proveedor.
- **Medidas:** Variables numéricas a analizar, como unidades vendidas, costos, ingresos.
- **Jerarquías:** Niveles de agregación dentro de una dimensión (por ejemplo, Año > Mes > Día).

---

## 2. PROCESO ETL

1. **Extracción:** Se recopilaron datos de compras y ventas desde archivos proporcionados por las sucursales.
2. **Transformación:** Se normalizaron formatos, eliminaron duplicados y se homologaron las columnas para integrarlas.
3. **Carga:** Los datos limpios fueron cargados al modelo multidimensional en SQL Server (u otro motor compatible).

---

## 3. DISEÑO DE MODELO MULTIDIMENSIONAL

Se eligió un **modelo constelacion** por su simplicidad y eficiencia en consultas OLAP.

### Diseño representado en diagrama

![alt text](Diagrama.png)

### Modelo SQL

```
-- Dimensión Fecha
CREATE TABLE Dim_Fecha (
    IdFecha INT PRIMARY KEY,
    Fecha DATE,
    Anio INT,
    Mes INT,
    Dia INT
);

-- Dimensión Producto
CREATE TABLE Dim_Producto (
    IdProducto INT PRIMARY KEY,
    CodProducto VARCHAR(50),
    NombreProducto VARCHAR(100),
    MarcaProducto VARCHAR(50),
    Categoria VARCHAR(50)
);

-- Dimensión Sucursal
CREATE TABLE Dim_Sucursal (
    IdSucursal INT PRIMARY KEY,
    CodSucursal VARCHAR(50),
    NombreSucursal VARCHAR(100),
    Region VARCHAR(50),
    Departamento VARCHAR(50)
);

-- Dimensión Cliente (solo para ventas)
CREATE TABLE Dim_Cliente (
    IdCliente INT PRIMARY KEY,
    CodCliente VARCHAR(50),
    NombreCliente VARCHAR(100),
    TipoCliente VARCHAR(50)
);

-- Dimensión Proveedor (solo para compras)
CREATE TABLE Dim_Proveedor (
    IdProveedor INT PRIMARY KEY,
    CodProveedor VARCHAR(50),
    NombreProveedor VARCHAR(100)
);

-- Tabla de hechos de ventas
CREATE TABLE Hecho_Ventas (
    IdVenta INT PRIMARY KEY IDENTITY(1,1),
    IdFecha INT NOT NULL,
    IdProducto INT NOT NULL,
    IdSucursal INT NOT NULL,
    IdCliente INT NOT NULL,
    Unidades INT,
    PrecioUnitario DECIMAL(18,2),
    FOREIGN KEY (IdFecha) REFERENCES Dim_Fecha(IdFecha),
    FOREIGN KEY (IdProducto) REFERENCES Dim_Producto(IdProducto),
    FOREIGN KEY (IdSucursal) REFERENCES Dim_Sucursal(IdSucursal),
    FOREIGN KEY (IdCliente) REFERENCES Dim_Cliente(IdCliente)
);

-- Tabla de hechos de compras
CREATE TABLE Hecho_Compras (
    IdCompra INT PRIMARY KEY IDENTITY(1,1),
    IdFecha INT NOT NULL,
    IdProducto INT NOT NULL,
    IdSucursal INT NOT NULL,
    IdProveedor INT NOT NULL,
    Unidades INT,
    CostoUnitario DECIMAL(18,2),
    FOREIGN KEY (IdFecha) REFERENCES Dim_Fecha(IdFecha),
    FOREIGN KEY (IdProducto) REFERENCES Dim_Producto(IdProducto),
    FOREIGN KEY (IdSucursal) REFERENCES Dim_Sucursal(IdSucursal),
    FOREIGN KEY (IdProveedor) REFERENCES Dim_Proveedor(IdProveedor)
);

```

### Justificacion, atributos y jerarquia

#### Justificacion y atributos
Dim_Fecha:
Permite el análisis temporal de ventas y compras. Incluye jerarquía: Año > Mes > Día para facilitar agrupamientos y comparaciones históricas.

Dim_Producto:
Identifica cada producto y su contexto (marca, categoría), útil para análisis de ventas y compras por tipo de producto o categoría.

Dim_Sucursal:
Permite ver el desempeño geográfico y comparar entre regiones, departamentos y sucursales.

Dim_Cliente:
Esencial para entender el comportamiento de los distintos clientes (por tipo, nombre o código) y segmentar el análisis de ventas.

Dim_Proveedor:
Facilita la evaluación y comparación de proveedores según el volumen y valor de las compras.

Hecho_Ventas:
Almacena cada venta realizada, relacionando fecha, producto, sucursal y cliente. Incluye las medidas clave: unidades vendidas y precio unitario. Permite calcular ingresos y analizar ventas desde varias dimensiones.

Hecho_Compras:
Registra cada compra, vinculando la fecha, el producto, la sucursal y el proveedor. Guarda las unidades y el costo unitario, permitiendo evaluar gastos y analizar compras según proveedores, regiones y categorías de productos.

#### Jerarquías
Fecha: Año > Mes > Día (útil para reportes y tendencias)

Producto: Categoría > Marca > Producto (permite análisis agregados o detallados)

Sucursal: Región > Departamento > Sucursal (para comparar zonas y ubicaciones)

Cliente/Proveedor: Posibilidad de jerarquizar por tipo, región, etc., si se añaden más atributos.

