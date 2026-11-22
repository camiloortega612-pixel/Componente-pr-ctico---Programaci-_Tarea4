# Sistema Integral de Gestión de Clientes, Servicios y Reservas — Software FJ

Proyecto de la **Fase 4** del curso *Programación (213023)* de la UNAD.
Aplicación de consola desarrollada en Python 3.12, completamente orientada a
objetos y **sin motor de base de datos**: toda la información se gestiona con
objetos y listas en memoria, y el único archivo que se escribe es la bitácora
de eventos y errores.

---

## 1. Objetivo

Construir un sistema estable, modular y extensible que implemente de forma
rigurosa **abstracción, herencia, polimorfismo, encapsulación y manejo avanzado
de excepciones**, garantizando que la aplicación continúe funcionando incluso
cuando se presenten errores durante su ejecución.

## 2. Requisitos

- Python 3.10 o superior (probado en Python 3.12).
- No requiere librerías externas: solo se usa la biblioteca estándar.

```bash
python --version          # verificar la versión
python main.py            # ejecutar la demostración
```

La bitácora se crea automáticamente en `logs/sistema_fj.log`.

## 3. Estructura del proyecto

```text
SistemaSoftwareFJ/
├── main.py                  # Punto de entrada y guion de 28 operaciones
├── requirements.txt         # Sin dependencias externas
├── README.md
├── logs/
│   └── sistema_fj.log       # Bitácora generada en tiempo de ejecución
└── sistema_fj/              # Paquete con el modelo de dominio
    ├── __init__.py          # API pública del paquete
    ├── excepciones.py       # Jerarquía de excepciones personalizadas
    ├── registro.py          # Bitácora de eventos y errores
    ├── entidad.py           # Clase abstracta Entidad (raíz del modelo)
    ├── cliente.py           # Cliente con encapsulación y validaciones
    ├── servicio.py          # Servicio abstracto + 3 especializaciones
    ├── reserva.py           # Reserva y su ciclo de vida
    └── sistema.py           # Fachada que coordina todo el sistema
```

## 4. Diseño orientado a objetos

### 4.1 Abstracción

`Entidad` (ABC) define el contrato común: identidad, fecha de creación y el
método abstracto `describir()`. `Servicio` también es abstracta y obliga a sus
subclases a implementar `_calcular_base()` y `describir()`.

### 4.2 Herencia y polimorfismo

```text
Entidad (ABC)
├── Cliente
├── Reserva
└── Servicio (ABC)
    ├── ReservaSala
    ├── AlquilerEquipo
    └── AsesoriaEspecializada
```

El sistema recorre la lista de servicios y llama `calcular_costo()` y
`describir()` sin conocer el tipo concreto: cada subclase responde con su
propia tarifa y su propia descripción.

### 4.3 Encapsulación

`Cliente` y `Servicio` guardan sus datos en atributos privados (`__nombre`,
`__documento`, `__correo`, `__telefono`, `__tarifa_base`, …) y solo permiten
modificarlos mediante propiedades con validación. El estado de una reserva
(`__estado`) solo cambia a través de `confirmar()`, `procesar()` y `cancelar()`.

### 4.4 Métodos sobrecargados

`Servicio.calcular_costo()` simula la sobrecarga de otros lenguajes: acepta uno,
dos o tres argumentos, posicionales o nominales, sin duplicar la lógica.

```python
servicio.calcular_costo(4)                       # costo base
servicio.calcular_costo(4, 0.19)                 # + IVA 19 %
servicio.calcular_costo(4, 0.19, 0.10)           # + descuento 10 %
servicio.calcular_costo(horas=4, descuento=0.10) # argumentos nominales
```

### 4.5 Manejo de excepciones

- **Excepciones personalizadas** con código de error estable (`FJ-100` … `FJ-500`).
- **`try/except`**: captura de errores de negocio en cada operación.
- **`try/except/else`**: el bloque `else` solo actualiza el estado si no hubo error.
- **`try/except/finally`**: el `finally` garantiza la trazabilidad en la bitácora.
- **Encadenamiento** (`raise ... from error`): los errores técnicos (`ValueError`,
  `KeyError`, `TypeError`, `OSError`) se convierten en errores de dominio sin
  perder la causa original.

## 5. Bitácora de eventos

Cada línea del archivo `logs/sistema_fj.log` tiene el formato:

```text
2026-11-21 14:49:02 | ERROR       | FJ-304  | Reserva rechazada: ...
```

Se registran los eventos informativos (registros, confirmaciones, facturación),
así como **todos** los errores y advertencias. Si el archivo no se puede
escribir, el sistema no se detiene: informa por consola y conserva el evento en
memoria.

## 6. Demostración

`python main.py` ejecuta 28 operaciones: 10 exitosas y 18 rechazos
controlados, entre ellos cliente con documento inválido, correo inválido,
cliente duplicado, sala con capacidad 0, servicio inexistente, reserva de un
cliente no registrado, duración inválida, solapamiento de franjas, fecha
pasada, reserva fuera del horario de atención, servicio suspendido, cliente
inactivo, doble facturación y cancelación de una reserva ya procesada.

Al final se imprime un resumen con los indicadores del sistema, el conteo de
eventos por severidad y las últimas líneas de la bitácora.
