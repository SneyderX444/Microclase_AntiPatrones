## Parte 1. Tabla de Antipatrones

| Fragmento | Nombre del Antipatrón | Síntoma que lo delata | Consecuencia en el consulado | 
| ----- | ----- | ----- | ----- | 
| **Fragmento 1** | **God Object** | La clase `OficinaDeTodo` valida el pasaporte, cobra la tarifa, guarda el expediente y envía el correo. | Si ocurre un fallo en cualquiera de los procesos o si el componente principal no responde, el consulado entero se detiene. El código no es reutilizable y viola el Principio de Responsabilidad Única. | 
| **Fragmento 2** | **Copy-Paste Programming** | La fórmula de cálculo de tarifas (`50 * meses + 10`) está duplicada exactamente en los métodos `visaTurista`, `visaEstudiante` y `visaTrabajo`. | Modificar la tarifa base requiere editar múltiples líneas en el código. Si se olvida actualizar un método, esa visa mantendrá el precio antiguo generando inconsistencias. | 
| **Fragmento 3** | **Magic Numbers** | Presencia de literales numéricos sin explicación como `tipo == 1, 2, 3`, rangos de edad `17`, `15`, `31` o multiplicadores `0.5`. | Dificulta la lectura y mantenimiento. Ante la consulta de un tipo no registrado (como `9`), la versión heredada lo rechaza en silencio sin notificar la causa del error. | 
| **Fragmento 4** | **Acoplamiento Fuerte** | La clase `SolicitudAcoplada` crea instancias con `new` y llama secuencialmente a `CorreoServicio`, `CitasServicio` y `PagosServicio`. | Al realizar una cancelación, el olvido manual de invocar `PagosServicio` provoca que el viajero pierda su reembolso. Añadir nuevos servicios requiere modificar el código interno de la solicitud. | 

## Parte 2. Estructura de Paquetes

```
src/
├── heredado/
│   ├── OficinaDeTodo.java
│   ├── CalculadoraDeTarifas.java
│   ├── EvaluadorMagico.java
│   └── SolicitudAcoplada.java
├── rescatado/
│   ├── ValidadorPasaporte.java
│   ├── CajaPagos.java
│   ├── GestorExpedientes.java
│   ├── NotificadorCorreo.java
│   ├── TramitadorVisas.java
│   ├── TipoVisa.java
│   ├── EvaluadorRescatado.java
│   ├── strategy/
│   │   ├── EstrategiaTarifa.java
│   │   ├── TarifaTurista.java
│   │   ├── TarifaEstudiante.java
│   │   ├── TarifaTrabajo.java
│   │   └── CalculadoraTarifasRescatada.java
│   └── observer/
│       ├── ObservadorCancelacion.java
│       ├── ServicioCorreoObserver.java
│       ├── ServicioCitasObserver.java
│       ├── ServicioPagosObserver.java
│       └── SolicitudRescatada.java
└── main/
    └── Main.java
```

## Parte 3. Salida de los Casos de Prueba en Consola

```
===============================================================================
       UNIVERSIDAD DISTRITAL FRANCISCO JOSÉ DE CALDAS - INGENIERÍA DE SISTEMAS
              TALLER: LA OFICINA DE VISAS DEL CAOS (ANTIPATRONES)
===============================================================================

-------------------------------------------------------------------------------
CASO 1: God Object (OficinaDeTodo vs Clases Separadas por Responsabilidad)
-------------------------------------------------------------------------------

>>> VERSIÓN HEREDADA (God Object - Ramiro lo hace todo):
Cobrando $150.0 a Juan Perez
Guardando expediente de Juan Perez
Correo para Juan Perez: su visa va en camino

>>> VERSIÓN RESCATADA (Principio SRP - Clases con responsabilidad única):
[CajaPagos] Cobrando $150.0 a Juan Perez
[GestorExpedientes] Guardando expediente de Juan Perez
[NotificadorCorreo] Correo para Juan Perez: su visa va en camino

-------------------------------------------------------------------------------
CASO 2: Copy-Paste Programming (Modificación de Tarifa Base)
-------------------------------------------------------------------------------

>>> VERSIÓN HEREDADA (Lógica duplicada):
Tarifas (Base $60, 2 meses): Turista=$130.00 | Estudiante=$65.00 | Trabajo=$195.00
Lugares/Archivos modificados manualmente en el código: 3

>>> VERSIÓN RESCATADA (Patrón Strategy):
Tarifas (Base $60, 2 meses): Turista=$130.00 | Estudiante=$65.00 | Trabajo=$195.00
Lugares/Archivos modificados en el código: 1
Resultado: El precio final coincide exactamente ($130.0 == $130.0), pero la mantenibilidad se optimizó a 1 solo punto.

-------------------------------------------------------------------------------
CASO 3: Magic Numbers (Números Misteriosos vs Enum TipoVisa)
-------------------------------------------------------------------------------

>>> PRUEBA 1: Tipo 2 (Estudiante) con Edad 20
Heredado (evaluar(2, 20)): APROBADA
Rescatado (evaluar(TipoVisa.ESTUDIANTE, 20)): APROBADA

>>> PRUEBA 2: Tipo inválido/desconocido 9
Heredado (evaluar(9, 20)): RECHAZADA (Rechazo en silencio, no explica causa)
Rescatado (evaluarPorCodigo(9, 20)): [EvaluadorRescatado] ERROR: El código 9 no corresponde a ningún TipoVisa válido.
Resultado: RECHAZADA (TIPO INVÁLIDO)

-------------------------------------------------------------------------------
CASO 4: Acoplamiento Fuerte (Llamadas manuales vs Patrón Observer)
-------------------------------------------------------------------------------

>>> VERSIÓN HEREDADA (Acoplamiento fuerte):
[CorreoServicio] Envío correo de cancelación.
[CitasServicio] Cupo de cita liberado.
ALERTA HEREDADA: ¡Se olvidó llamar a PagosServicio! El viajero perdió su reembolso.

>>> VERSIÓN RESCATADA (Patrón Observer):
[Solicitud] Cancelando solicitud SOL-2026-99 de Sofia Gomez...
[CorreoObserver] Notificando vía email la cancelación a Sofia Gomez
[CitasObserver] Liberando cupo de la cita asignada para Sofia Gomez
[PagosObserver] Procesando reembolso automático para Sofia Gomez
Resultado: La clase SolicitudRescatada no conoce los servicios concretos, pero todos se enteraron y se ejecutó el reembolso.

-------------------------------------------------------------------------------
CASO 5: Integración del Viaje Completo y Tabla Resumen
-------------------------------------------------------------------------------

--- SIMULACIÓN DE FLUJO COMPLETO CON LA VERSIÓN RESCATADA ---
1. EVALUACIÓN:
   Resultado evaluación para Carlos Mendoza (Visa de Estudiante): APROBADA

2. CÁLCULO DE TARIFA:
   Costo total calculado para 6 meses: $155.0

3. TRÁMITE Y COBRO:
   [CajaPagos] Cobrando $155.0 a Carlos Mendoza
   [GestorExpedientes] Guardando expediente de Carlos Mendoza
   [NotificadorCorreo] Correo para Carlos Mendoza: su visa va en camino

4. SOLICITUD Y CANCELACIÓN POSTERIOR:
   [Solicitud] Cancelando solicitud SOL-2026-777 de Carlos Mendoza...
   [CorreoObserver] Notificando vía email la cancelación a Carlos Mendoza
   [CitasObserver] Liberando cupo de la cita asignada para Carlos Mendoza
   [PagosObserver] Procesando reembolso automático para Carlos Mendoza

=======================================================================================================================
                                      TABLA FINAL DE ANTIPATRONES CAPTURADOS
=======================================================================================================================
| ANTIPATRÓN             | SÍNTOMA EN EL CONSULADO                      | SOLUCIÓN APLICADA                        |
=======================================================================================================================
| 1. God Object          | Ramiro/OficinaDeTodo asume todas las tareas  | Principio SRP (Clases por responsabilidad)|
| 2. Copy-Paste          | Cálculo de tarifado duplicado 3 veces        | Patrón Strategy (EstrategiaTarifa)       |
| 3. Magic Numbers       | Números misteriosos como 1, 2, 3 o 0.5       | Enum TipoVisa con constantes claras      |
| 4. Acoplamiento Fuerte | Llamadas manuales a servicios olvidan pagos  | Patrón Observer (Suscriptores desacoplados)|
=======================================================================================================================
```

## Parte 4. Explicación de Prevención de Antipatrones

La refactorización evita la reaparición de estos antipatrones mediante la aplicación directa de principios de diseño y patrones orientados a objetos. Al delegar cada tarea a clases específicas (`ValidadorPasaporte`, `CajaPagos`, `GestorExpedientes`, `NotificadorCorreo`), se cumple el Principio de Responsabilidad Única (SRP) e impide la creación de un nuevo God Object. La sustitución de literales por el enum `TipoVisa` elimina los Magic Numbers al tipar y validar de forma explícita las propiedades de cada visa. Asimismo, la implementación del patrón Strategy centraliza el cálculo tarifario permitiendo modificar o añadir reglas financieras modificando un único punto del sistema. Por último, el patrón Observer desarticula el acoplamiento fuerte al reemplazar las llamadas manuales internas por un mecanismo de suscripción en el que los servicios reaccionan a los eventos de la solicitud sin que esta necesite instanciarlos o conocer su implementación.