Sistema de Reserva de Canchas Sintéticas (C++)

Descripcion del Problema y Solución
En un sistema tradicional de reserva de canchas, el manejo de diferentes tipos de canchas, el cálculo de tarifas con servicios adicionales y las notificaciones de cambios de estado suelen resolverse con bloques masivos de `if-else` o un alto acoplamiento. 

Esta aplicación resuelve ese problema integrando (tres patrones de diseño) que trabajan juntos en un mismo flujo para permitir crear canchas dinámicamente, agregar costos de servicios extra de forma flexible y notificar eventos del sistema sin acoplar la lógica[cite: 2].

---
Patrones de Diseño Utilizados

1.Creacional: Factory Method[cite: 2]
   - Rol:Centraliza la creación de los distintos tipos de canchas (`CanchaFutbol`, `CanchaTenis`, `CanchaPadel`)[cite: 2].
   - Integración: Produce las instancias de `Cancha` que la aplicación gestiona en memoria sin acoplar el código a clases concretas[cite: 2].

2. Estructural: Decorator[cite: 2]
   -Rol: Permite envolver una reserva base para añadir cobros adicionales dinámicos (`DecoradorIluminacion`, `DecoradorEquipamiento`)[cite: 2].
   -Integración: Toma el precio base calculado por la cancha creada en el *Factory Method* y le suma servicios opcionales durante la transacción[cite: 2].

3. De Comportamiento: Observer[cite: 2]
   - Rol: Notifica a múltiples suscriptores (`NotificadorPantalla`, `NotificadorHistorial`) cada vez que una reserva se crea o cancela[cite: 2].
   - Integración: Se dispara automáticamente en el flujo de `realizarReserva()` y `cancelarReserva()`, registrando los eventos generados por el *Decorator* y el *Factory Method*[cite: 2].

---
Compilación y Ejecución

Para compilar y ejecutar el proyecto desde la terminal usando `c++`:

```bash
# Compilar
c++ -std=c++11 main.cpp -o reservas

# Ejecutar
./reservas

====================================
   SISTEMA DE RESERVA DE CANCHAS    
====================================
1. Ver estado de canchas
2. Realizar reserva
...
¡Reserva registrada con exito!
Número de reserva: 101
Detalle: Reserva Base (14:00-15:00) + Iluminacion Nocturna + Alquiler Balon/Raquetas
Costo Total: $75
[NOTIFICACION EN SISTEMA]: Reserva #101 creada por Juan para la Cancha 1
