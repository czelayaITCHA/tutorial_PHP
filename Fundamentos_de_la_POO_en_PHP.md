# Guía : Fundamentos de Programación Orientada a Objetos (POO) en PHP

## 1. Introducción
La **Programación Orientada a Objetos (POO)** es un paradigma que permite modelar problemas del mundo real mediante **clases** y **objetos**, facilitando la reutilización, mantenibilidad y escalabilidad del software. PHP, a partir de versiones modernas (7+ y 8+), ofrece un soporte robusto para POO.

Esta guía tiene como objetivo explicar los **fundamentos de la POO en PHP** y aplicarlos a un **caso práctico de cálculo de salarios**, usando **herencia y polimorfismo**.

---

## 2. Conceptos Fundamentales de POO

### 2.1 Programación Orientada a Objetos
La POO se basa en los siguientes principios:
- **Abstracción**
- **Encapsulamiento**
- **Herencia**
- **Polimorfismo**

---

### 2.2 Clase
Una **clase** es una plantilla que define las propiedades (atributos) y comportamientos (métodos) de un objeto.

Ejemplo conceptual:
```php
class Persona {
    public string $nombre;
    public function saludar() {}
}
```

---

### 2.3 Objeto
Un **objeto** es una instancia de una clase.

```php
$persona = new Persona();
```

---

### 2.4 Miembros de una Clase
Los miembros de una clase son:
- **Atributos (propiedades)**
- **Métodos (funciones)**

---

## 3. Tipos de Clases en PHP

### 3.1 Clases Normales
Son las más comunes y pueden ser instanciadas.

---

### 3.2 Clases Estáticas
Utilizan métodos o propiedades estáticas. No requieren instanciación.

```php
class Util {
    public static function mensaje() {
        return "Hola";
    }
}
```

---

### 3.3 Clases Abstractas
No pueden ser instanciadas directamente. Sirven como **clase base** para herencia.

Características:
- Pueden contener métodos abstractos
- Obligan a las clases hijas a implementarlos

---

### 3.4 Clases Finales
No pueden ser heredadas.

```php
final class Config {}
```

---

## 4. Constructores
El **constructor** se ejecuta automáticamente al crear un objeto.

```php
public function __construct() {}
```

Sirve para inicializar atributos.

---

## 5. Encapsulamiento
Consiste en proteger los atributos usando modificadores de acceso:
- `private`
- `protected`


Se accede a los atributos mediante **getters y setters**.

---

## 6. Herencia
Permite que una clase hija herede atributos y métodos de una clase padre.

```php
class Hija extends Padre {}
```

---

## 7. Polimorfismo
Permite que distintas clases respondan de manera diferente al mismo método.

Ejemplo:
```php
$empleado->calcSalario();
```

Cada tipo de empleado implementa su propia lógica.

---

## 8. Caso Práctico: Sistema de Planilla

### 8.1 Clase Abstracta Empleado

```php
<?php
abstract class Empleado{
    private int $id;
    private string $nombre;

    public function __construct(int $id, string $nombre){
        $this->id = $id;
        $this->nombre = $nombre;
    }

    abstract function calcSalario() : float;
    abstract function calcDescuentos(): float;

    public function getId(): int{
        return $this->id;
    }
    public function setId(int $id): void{
        $this->id = $id;
    }
    public function getNombre() : string {
        return $this->nombre;
    }
    public function setNombre(string $nombre) : void{
        $this->nombre = $nombre;
    }
}
```

---

### 8.2 Clase EmpleadoPorHora

```php
<?php
require_once('empleado.php');

class EmpleadoPorHora extends Empleado{
    private int $horasTrab;
    private float $valorHora;

    public function __construct(int $id, string $nombre, int $ht, float $vh){
        parent::__construct($id,$nombre);            
        $this->horasTrab = $ht;
        $this->valorHora = $vh;
    }

    public function calcSalario() : float{
        return $this->horasTrab * $this->valorHora;
    }

    public function calcDescuentos(): float{
        return $this->calcSalario() * 0.10;
    }
}
```

---

## 9. Interfaz Web (Formulario)

La interfaz web permite al usuario ingresar los datos del empleado y seleccionar el tipo de empleado (**Tiempo Completo** o **Por Hora**). Los datos se envían al backend usando **fetch (AJAX)** sin recargar la página.

### Archivo `salarios.php`

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <title>Fundamentos de PHP - POO</title>
</head>
<body>
<div class="container mt-4">
    <header class="mb-4">
        <h2>POO en PHP - Herencia</h2>
        <h4>Cálculo de Salario</h4>
    </header>

    <form id="form-emp">
        <div class="mb-3">
            <label class="form-label">Empleado</label>
            <input type="text" name="empleado" class="form-control" required>
        </div>

        <div class="mb-3">
            <label class="form-label">Tipo de Empleado</label><br>
            <input type="radio" name="tipo" value="tc" checked onchange="mostrarInputs(this.value)"> Tiempo Completo
            <input type="radio" name="tipo" value="ph" onchange="mostrarInputs(this.value)"> Por Hora
        </div>

        <div id="emp-tc" class="mb-3">
            <label class="form-label">Salario Base</label>
            <input type="text" name="salario" class="form-control" style="max-width:200px;">
        </div>

        <div id="emp-ph" class="mb-3" style="display:none;">
            <label class="form-label">Horas Trabajadas</label>
            <input type="text" name="horasTrab" class="form-control mb-2" style="max-width:200px;">

            <label class="form-label">Valor Hora</label>
            <input type="text" name="valorHora" class="form-control" style="max-width:200px;">
        </div>

        <button type="submit" class="btn btn-primary">Calcular Salario</button>

        <div id="result" class="mt-4"></div>
    </form>
</div>

<script>
const formulario = document.getElementById('form-emp');
const resultado = document.getElementById('result');

formulario.addEventListener('submit', async (e) => {
    e.preventDefault();
    const data = new FormData(formulario);

    const response = await fetch('calcularSalario.php', {
        method: 'POST',
        body: data
    });

    const res = await response.json();

    if (res === 'error') {
        resultado.innerHTML = '<div class="alert alert-danger">Complete los datos del formulario</div>';
    } else {
        resultado.innerHTML = '<div class="alert alert-success">' + res + '</div>';
    }
});

function mostrarInputs(op) {
    document.getElementById('emp-tc').style.display = op === 'tc' ? 'block' : 'none';
    document.getElementById('emp-ph').style.display = op === 'ph' ? 'block' : 'none';
}
</script>
</body>
</html>
```

**Aspectos didácticos a destacar:**
- Uso de `fetch` para comunicación asíncrona
- Separación frontend / backend
- Manipulación dinámica del DOM
- Integración de Bootstrap

---


## 10. Procesamiento Backend

### Archivo `calcularSalario.php`

```php
<?php
require_once('empleadohc.php');

if(isset($_POST['tipo'])){
    $tipoEmp = $_POST['tipo'];
    $result="";

    if ($tipoEmp == "") {
        $result= "error";
    } else if ($tipoEmp == "tc") {
        // TAREA PARA EL ESTUDIANTE
    } else {
        if($_POST["horasTrab"] != null && $_POST["valorHora"] != null){
            $horasTrab = $_POST["horasTrab"];
            $valorHora = $_POST["valorHora"];

            $empPH = new EmpleadoPorHora(rand(1,100), $_POST['empleado'], $horasTrab, $valorHora);

            $descuentoRenta = $empPH->calcDescuentos();

            $result  = 'Salario Devengado = $' . number_format($empPH->calcSalario(),2) . '<br>';
            $result .= 'Descuento Renta = $' . number_format($descuentoRenta,2) . '<br>';
            $result .= 'Total a pagar = $' . number_format($empPH->calcSalario() - $descuentoRenta,2);
        } else {
            $result = "error";
        }
    }
    echo json_encode($result);
} else {
    echo json_encode(["error"=> "Error al procesar los datos"]);
}
```

---

## 11. 📝 Ejercicio Propuesto (Evaluado)

### Empleado de Planta (Tiempo Completo)

**Objetivo:**
Implementar el cálculo de salario y descuentos para un **empleado de planta**, utilizando herencia.

### Requisitos:
1. Crear la clase `EmpleadoTiempoCompleto` que herede de `Empleado`.
2. El salario se calcula a partir de un **salario base**.
3. No considerar bonificaciones ni horas extra.

### Descuentos obligatorios en El Salvador:
- **ISSS**: 3% (máx. según ley)
- **AFP**: 7.25%
- **Renta**: según tramo (simplificado para el ejercicio, por ejemplo 10%)

### Actividades:
- Implementar los métodos `calcSalario()` y `calcDescuentos()`.
- Integrar la clase en `calcularSalario.php`.
- Mostrar salario devengado, descuentos y total a pagar.

---

## 12. Conclusión
Este ejercicio integra:
- Clases abstractas
- Herencia
- Polimorfismo
- Encapsulamiento
- Comunicación frontend–backend

Representa un caso real del desarrollo de **aplicaciones web con PHP orientado a objetos**.

---

📘 *Docente: Programación de Aplicaciones Web con PHP*

