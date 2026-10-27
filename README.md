# Envios Service — Análisis de Métricas de Calidad

## Entorno de análisis
* **Decisión de diseño 1 (Entorno de análisis):** Se eligió **SonarCloud (servicio administrado en la nube)** en lugar de SonarQube local ejecutado con Docker. Esta decisión se fundamenta en dos criterios de diseño:
  1. *Cero sobrecarga de recursos locales e instalación:* No requiere mantener un contenedor local de Docker consumiendo memoria RAM y procesador en la máquina de desarrollo, evitando conflictos derivados de la virtualización y asegurando estabilidad.
  2. *Accesibilidad centralizada y persistencia:* El dashboard queda almacenado de forma permanente en un servicio SaaS accesible mediante un enlace público web permanente, integrándose de forma nativa con los repositorios de GitHub. Como compensación técnica, se asume la necesidad obligatoria de disponer de conectividad a internet durante la ejecución del análisis.

* **Decisión de diseño 2 (Exclusiones del análisis):** Se decidió **no configurar exclusiones (`sonar.exclusions`)** en este proyecto. `envios-service` es un proyecto Maven plano enfocado exclusivamente en la lógica de negocio del cálculo de tarifas y carece de clases de arranque con método `main`, archivos de configuración de frameworks o DTOs autogenerados sin lógica propia. Excluir cualquier archivo distorsionaría artificialmente el porcentaje global de cobertura sin aportar información real sobre la calidad del código.

## Estado inicial del análisis

| Métrica | Valor real observado |
|---------|----------------------|
| Complejidad ciclomática de `calcularCosto()` | 20 |
| Cobertura de líneas de `ServicioCalculoEnvio` | 32.4% |
| Cobertura de ramas de `ServicioCalculoEnvio` | 21.3% (8 de 38 ramas cubiertas) |
| Duplicación de código | 0.0% |
| Code Smells reportados | 0 |

## Interpretación
El método `ServicioCalculoEnvio.calcularCosto()` concentra de forma crítica tanto la complejidad ciclomática más alta (**CC = 20**) como una cobertura de ramas deficiente (**21.3%**).

Al revisar el reporte HTML de JaCoCo (`target/site/jacoco/index.html`), se comprueba que la suite de pruebas inicial únicamente ejercita el camino de tarifa `"ESTANDAR"` con peso menor o igual a 5 kg, distancia menor a 500 km y tipos de cliente `"REGULAR"` y `"PREMIUM"`. Quedaron sin cubrir:
* Los tramos de peso medio (`5 < peso <= 20`) y alto (`peso > 20`) de `"ESTANDAR"`, así como su recargo por distancia superior a 500 km.
* La totalidad de las ramas de cálculo para los métodos `"EXPRESS"`, `"OVERNIGHT"` e `"INTERNACIONAL"`.
* La validación de excepción para envíos `"OVERNIGHT"` con distancia mayor a 300 km y la excepción por método de envío no soportado.
* El recargo por paquete frágil (`esFragil = true`) y el descuento para clientes de categoría `"CORPORATIVO"`.

* **Decisión de diseño 3 (Métrica usada como señal principal):** Se seleccionó la **Complejidad Ciclomática (CC = 20)** en conjunto con la **Cobertura de Ramas** como señal principal para guiar la intervención. La complejidad ciclomática mide la cantidad mínima de caminos linealmente independientes que debe transitar la lógica: una cobertura baja solo evidencia pruebas ausentes, pero una CC de 20 refleja una violación estructural de modularidad que dificulta la mantenibilidad. Desacoplar primero el método mediante refactorización reduce los caminos independientes y permite que las pruebas unitarias cubran casos atómicos sin duplicación.

