# Laboratorio 8 — Construir en Amazon Quick: Flows, Apps y Automate

> **Todos los datos de este laboratorio son ficticios**, igual que el resto del kit `exito-quick-lab`. Los correos usan el dominio reservado `exito-lab.example`, que no recibe mensajes.

Laboratorio de profundización para **todos los participantes**. Parte de lo construido en los Labs 3, 5, 6 y 7 y lo lleva un paso más allá: el mismo problema de reposición entre almacenes resuelto con las tres herramientas de construcción de Quick.

| | |
| --- | --- |
| **Duración** | 75 a 90 minutos (Partes A, B y C de 25 minutos cada una, más 10 minutos de cierre) |
| **Formato** | Individual: cada participante construye las tres piezas, las prueba y resuelve las tareas de aprobación de su propia automatización |
| **Resultado** | Un Flow programado, una app publicada y una automatización desplegada con aprobación humana |

## El caso

Un almacén sin producto en la góndola es un cliente que quizá no regresa. Cada lunes, Abastecimiento revisa qué almacenes quedaron por debajo del stock mínimo en las categorías críticas, propone transferencias entre almacenes según la política **POL-OPS-003** y espera que los gerentes regionales las aprueben. Hoy se hace con hojas de cálculo y correos. En este laboratorio se construyen tres piezas:

| Parte | Herramienta | Qué resuelve | Quién lo usa |
| --- | --- | --- | --- |
| A | **Quick Flows** | Plan semanal de transferencias de una zona, con borrador de correo | Gerente regional, cada lunes |
| B | **Apps en Quick** | Centro de reposición: ver solicitudes y registrar nuevas con validación | Gerentes de almacén, todos los días |
| C | **Quick Automate** | Validar un lote de solicitudes y enviarlas a aprobación del gerente regional | Abastecimiento, en cada lote |

## Requisitos

- Haber completado el **Lab 3** (Space "Operaciones de almacenes — piloto" con `politicas/`). El **Lab 5** (tablero) y el **Lab 6** (conector de Outlook) hacen más completas las Partes A y B.
- **Apps en Quick** está disponible en los planes Plus, Professional y Enterprise.
- **Quick Automate** solo está disponible en cuentas de Quick creadas desde la consola de AWS; no en Free ni Plus. Sin acceso, el facilitador muestra la Parte C como demostración y los participantes siguen la guía en pantalla.
- Usa solo la cuenta y el buzón de pruebas. Ninguna acción se envía a buzones reales.

## Archivos de este laboratorio

| Archivo | Contenido | Se usa en |
| --- | --- | --- |
| `stock_cierre_2026-10-18.csv` | Foto del stock al cierre del 18-oct-2026: 20 almacenes × 24 SKU (480 filas), con ciudad, zona operativa, stock final, stock mínimo y marca `bajo_minimo` | Partes A, B y C |
| `solicitudes_lote_2026-10-19.csv` | Lote de 9 solicitudes de transferencia del 19-oct-2026, sin validar | Parte C |
| `proceso_aprobacion_transferencias.md` / `.docx` | Proceso PRO-OPS-003: reglas de validación, decisiones y aprobación humana | Parte C (documento de proceso) |

También se usan del kit: `almacenes.csv`, `gerentes_regionales.csv`, `solicitudes_reposicion.csv` y `politicas/politica_reposicion_entre_almacenes.docx`.

**Regla de cálculo** (POL-OPS-003, igual en las tres partes):

- **Necesario** = stock mínimo del destino − stock final del destino.
- **Donante válido** = almacén de la **misma ciudad** con stock **mayor a 2 veces** su mínimo.
- **Disponible del donante** = stock − 1,5 × mínimo, redondeado hacia abajo.
- **Cantidad a transferir** = el menor valor entre lo necesario y lo disponible. Si no hay donante válido, se **escala al centro de distribución**.

---

## Parte A — Quick Flows: plan semanal de transferencias

**Objetivo:** pasar del Flow simple del Lab 7 a uno con entradas, conocimiento del Space, lógica de control, una acción y un horario.

### A.1 Crear el Flow desde una descripción

En **Flows**, crea un Flow nuevo y descríbelo en lenguaje natural:

> Crea un Flow llamado "Plan semanal de transferencias por zona". Entradas: un archivo con el stock al cierre y el nombre de una zona operativa (Centro-Oriente, Antioquia-Eje Cafetero o Caribe-Pacífico). Pasos: 1) consulta en el Space "Operaciones de almacenes — piloto" la política POL-OPS-003 de reposición entre almacenes; 2) filtra los SKU de Abarrotes básicos, Desayuno y bebidas calientes y Lácteos y refrigerados de la zona con `bajo_minimo` = Sí; 3) para cada uno, busca un almacén donante de la misma ciudad con stock mayor a 2 veces su mínimo y calcula la cantidad sin dejar al donante por debajo de 1,5 veces su mínimo; si no hay donante, márcalo como "Escalar a centro de distribución"; 4) arma una tabla con destino, SKU, stock, mínimo, donante y cantidad; 5) prepara un borrador de correo para el gerente regional de la zona con la tabla y la referencia a la política.

### A.2 Revisar y ajustar los pasos

Revisa en el editor que el Flow tenga, como mínimo:

| Paso esperado | Tipo de paso en Quick |
| --- | --- |
| Archivo de stock | **Files** (entrada del usuario) |
| Zona operativa | **Text** (entrada del usuario) |
| Política del Space | **Quick data** (Spaces o bases de conocimiento) |
| Validar la entrada y recorrer los SKU bajo el mínimo | **Reasoning group**, con instrucciones en lenguaje natural y referencias `@` a los pasos anteriores |
| Tabla de transferencias | **General knowledge** (respuesta de IA) |
| Borrador de correo | **Application actions** (conector de Outlook del Lab 6) o, sin conector, un paso de IA que redacte el correo |

Ajustes que debes hacer a mano:

1. En el **Reasoning group**, agrega una validación: "Si la zona no es Centro-Oriente, Antioquia-Eje Cafetero ni Caribe-Pacífico, detente y pide una zona válida".
2. Pide que la tabla tenga una columna **"Faltante"** con lo que no alcanza a cubrir el donante.
3. En el paso del correo, deja la confirmación del usuario activada antes de ejecutar la acción.

### A.3 Ejecutar y probar

1. En **Run mode**, sube `stock_cierre_2026-10-18.csv` y escribe la zona **Antioquia-Eje Cafetero**. Compara la tabla con la regla de cálculo: deben salir 7 SKU bajo el mínimo, de los cuales 4 tienen donante y 3 se escalan.
2. Vuelve a ejecutarlo con la zona **Caribe-Pacífico** y revisa qué SKU quedan para escalar al centro de distribución: deben ser 3 de 5.
3. Ejecútalo con la zona **"Orinoquía"**: el Flow debe detenerse y pedir una zona válida.

Resultado esperado:

| Zona | Destino | SKU | Stock / mínimo | Donante (disponible) | Cantidad | Faltante |
| --- | --- | --- | --- | --- | --- | --- |
| Antioquia-Eje Cafetero | CA-MDE-01 | DES-002 | 39 / 80 | EX-MDE-01 (121) | 41 | 0 |
| Antioquia-Eje Cafetero | CA-MDE-01 | DES-003 | 61 / 62 | Sin donante | Escalar | 1 |
| Antioquia-Eje Cafetero | SI-PEI-01 | DES-003 | 40 / 56 | Sin donante (único almacén en Pereira) | Escalar | 16 |
| Antioquia-Eje Cafetero | SM-MDE-01 | ABA-001 | 83 / 111 | EX-MDE-01 (213) | 28 | 0 |
| Antioquia-Eje Cafetero | SM-MDE-01 | DES-001 | 39 / 49 | EX-MDE-02 (73) | 10 | 0 |
| Antioquia-Eje Cafetero | SM-MDE-01 | DES-003 | 30 / 33 | Sin donante | Escalar | 3 |
| Antioquia-Eje Cafetero | SM-MDE-02 | LAC-002 | 52 / 57 | EX-MDE-01 (178) | 5 | 0 |
| Caribe-Pacífico | CA-CLO-01 | ABA-001 | 100 / 126 | SI-CLO-01 (130) | 26 | 0 |
| Caribe-Pacífico | CA-CLO-01 | DES-002 | 69 / 75 | Sin donante | Escalar | 6 |
| Caribe-Pacífico | SI-CLO-01 | DES-003 | 58 / 59 | Sin donante | Escalar | 1 |
| Caribe-Pacífico | SM-BAQ-01 | ABA-003 | 0 / 72 | SI-BAQ-01 (81) | 72 | 0 |
| Caribe-Pacífico | SM-BAQ-01 | LAC-002 | 40 / 56 | Sin donante | Escalar | 16 |

Nota: la zona Centro-Oriente tiene 13 SKU bajo el mínimo y varios comparten donante (CA-BOG-02 para ABA-001). El Flow de esta parte no descuenta lo comprometido entre filas; ese control es parte del reto de la Parte C.

### A.4 Programar y compartir

1. En **Run mode**, abre el ícono de programación y elige **Create schedule**: semanal, lunes a las 07:00, zona horaria de Colombia (America/Bogota), con `stock_cierre_2026-10-18.csv` y "Caribe-Pacífico" como entradas por defecto.
2. Deja **desactivada** la opción **Run with no confirmation**: el correo se revisa antes de enviarse.
3. Comparte el Flow con el grupo. Ten en cuenta que el horario es personal y no se comparte; cada persona debe crear el suyo.
4. Revisa el historial de ejecuciones del Flow.

**Éxito:** el Flow corre de punta a punta para Antioquia-Eje Cafetero y Caribe-Pacífico, rechaza una zona inválida y queda programado sin envío automático.

---

## Parte B — Apps en Quick: centro de reposición entre almacenes

**Objetivo:** construir una herramienta interna que los gerentes de almacén usen todos los días, con datos, validación, IA y almacenamiento.

### B.1 Describir la app

Abre **Apps**, crea una app nueva, adjunta `stock_cierre_2026-10-18.csv`, `solicitudes_reposicion.csv` y `almacenes.csv`, y pide:

> Construye una app llamada "Centro de reposición Éxito" para gerentes de almacén y gerentes regionales. Tiene tres secciones:
> 1. **Solicitudes**: tabla con las solicitudes de `solicitudes_reposicion.csv`, filtros por estado, ciudad y categoría, y un contador de pendientes.
> 2. **Nueva solicitud**: formulario con almacén destino, SKU y motivo. Al elegir almacén y SKU, muestra el stock final y el mínimo de `stock_cierre_2026-10-18.csv`, calcula lo necesario y sugiere el donante de la misma ciudad con más unidades disponibles según POL-OPS-003 (stock mayor a 2 veces el mínimo; no dejar al donante por debajo de 1,5 veces su mínimo). Si no hay donante, indica "Escalar a centro de distribución". Si la categoría no es Abarrotes básicos, Desayuno y bebidas calientes ni Lácteos y refrigerados, o el stock no está bajo el mínimo, no permite enviar y explica por qué. Al guardar, la solicitud queda en estado "Pendiente" y se conserva entre sesiones.
> 3. **Resumen**: un botón que genere con IA un párrafo para el gerente regional con las solicitudes pendientes de su zona.
>
> Estilo sobrio, en español, con una paleta de amarillo y negro propia de una cadena de supermercados.

### B.2 Iterar en la conversación

Pide al menos tres cambios, uno por mensaje:

1. "Agrega la zona operativa y el correo del gerente regional (de `almacenes.csv`) en el detalle de cada solicitud."
2. "En Nueva solicitud, muestra también el segundo mejor donante como alternativa."
3. (Si hiciste el Lab 5) "Agrega una pestaña **Tablero** con el visual de días con quiebre por almacén del tablero de ventas y disponibilidad."

### B.3 Probar con casos conocidos

Registra estas solicitudes en la vista previa y compara con el resultado esperado:

| # | Almacén destino | SKU | Resultado esperado |
| --- | --- | --- | --- |
| 1 | CA-MDE-01 | DES-002 | Stock 39, mínimo 80, necesario 41. Donante EX-MDE-01 (121 disponibles); alternativa EX-MDE-02 (72). Cantidad 41 |
| 2 | SM-BOG-01 | ABA-002 | Stock 0, mínimo 66, necesario 66. Donante CA-BOG-02 (65 disponibles); alternativa SM-BOG-02 (49). Cantidad 65, faltan 1 |
| 3 | EX-BOG-01 | DES-003 | Stock 103, mínimo 108. Ningún almacén de Bogotá supera 2 veces su mínimo en este SKU: "Escalar a centro de distribución" |
| 4 | SM-BOG-02 | BEB-001 | No permite enviar: Bebé no es categoría crítica y el stock (25) está sobre el mínimo (22) |

Después cierra la app, vuelve a abrirla y confirma que las solicitudes que guardaste siguen ahí.

### B.4 Publicar

1. Publica la app y compártela con el grupo.
2. Registra una solicitud nueva y usa el botón de **Resumen**.

**Éxito:** la app publicada valida los 4 casos de prueba, guarda las solicitudes entre sesiones y genera el resumen con IA.

---

## Parte C — Quick Automate: validación y aprobación de un lote

**Objetivo:** automatizar un proceso de punta a punta con casos, un agente que aplica reglas y aprobación humana del gerente regional.

### C.1 Crear el proyecto desde el documento de proceso

1. En **Automate**, crea (o usa) el grupo de automatizaciones **"Operaciones de almacenes"**.
2. Crea el proyecto **"Aprobación de transferencias entre almacenes"**, describe el caso de negocio en una línea y adjunta `proceso_aprobacion_transferencias.docx`.
3. Pide a Quick Automate que genere el plan a partir del documento. Revisa los pasos propuestos antes de aceptarlos.

### C.2 Revisar y completar la automatización

En el **Visual Designer**, la automatización debe quedar con esta estructura:

| # | Paso | Acción de Automate |
| --- | --- | --- |
| 1 | Leer `solicitudes_lote_2026-10-19.csv` y `stock_cierre_2026-10-18.csv` como tablas | Acciones de lectura de archivos y tablas de datos |
| 2 | Un caso por solicitud | **Create multiple cases**: tipo de caso "Transferencias entre almacenes", columna de referencia `id_solicitud` |
| 3 | Recorrer los casos | **Process cases** |
| 4 | Validar R1 a R4 y calcular la cantidad | **Custom agent** con las reglas de la sección 3 de PRO-OPS-003 |
| 5 | Guardar la decisión sugerida, la cantidad, el donante y el motivo | **Update case data** |
| 6 | Pedir aprobación al gerente regional ("Aprobar" y "Aprobar con ajuste") | **Create user task** |
| 7 | Cerrar el caso con la respuesta del gerente regional | Al volver el caso, leer `latest_task_resolution` y actualizar el estado |
| 8 | Resumen del lote y borrador de correo | **Search cases** + paso de IA (+ acción de correo si está el conector) |

Detalles que debes configurar:

- **Instrucciones del agente:** pega las reglas R1 a R4 y los cálculos de PRO-OPS-003. Indica que procese en orden de `id_solicitud` y descuente del donante lo comprometido en solicitudes anteriores del lote. Pide la salida como datos estructurados: `decision_sugerida`, `cantidad_aprobada`, `id_almacen_donante_final`, `regla_aplicada`, `explicacion`.
- **Formulario de la tarea:**
  - Texto de solo lectura (*Display text*) con la solicitud original, la decisión sugerida y el motivo.
  - Desplegable (*Dropdown*) "Decisión" con Aprobar, Rechazar y Escalar a centro de distribución.
  - Campo de texto "Comentario".
- **Severidad y plazo de la tarea:** severidad **Critical** para Lácteos y refrigerados y **Moderate** para el resto; plazo de 4 horas.
- **Orden de los pasos:** no agregues acciones después de **Create user task** dentro del ciclo. El caso pasa a *Pending Resolution* y vuelve a *Ready* cuando el gerente regional responde.

### C.3 Probar

1. Ejecuta una prueba con el lote completo y revisa los registros de cada caso.
2. Comprueba que el agente:
   - rechaza la solicitud que no es de una categoría crítica;
   - escala las que no tienen donante válido;
   - ajusta la cantidad cuando el mismo donante aparece en dos solicitudes.
3. Resuelve las tareas desde tu lista como gerente regional: aprueba una, rechaza otra con comentario y escala una tercera. Verifica que cada caso termine con el estado correcto.

Resultado esperado del lote:

| Solicitud | Destino | SKU | Solicitado | Decisión sugerida | Cantidad | Donante | Regla |
| --- | --- | --- | --- | --- | --- | --- | --- |
| SOL-20260019 | SM-BOG-01 | ABA-001 | 60 | Aprobar | 60 | CA-BOG-02 | R1 a R4 sin cambios |
| SOL-20260020 | SM-BOG-02 | ABA-001 | 45 | Aprobar con ajuste | 25 | CA-BOG-02 | R4: el donante ya comprometió 60 de sus 85 disponibles |
| SOL-20260021 | SM-BOG-01 | ABA-003 | 70 | Aprobar con ajuste | 53 | CA-BOG-01 | R4: solo tiene 53 disponibles |
| SOL-20260022 | SM-MDE-01 | ABA-001 | 45 | Aprobar con ajuste | 28 | EX-MDE-01 | R4: lo necesario es 28 |
| SOL-20260023 | CA-CLO-01 | ABA-001 | 26 | Aprobar | 26 | SI-CLO-01 | R1 a R4 sin cambios |
| SOL-20260024 | SM-BAQ-01 | LAC-002 | 16 | Escalar a centro de distribución | 0 | — | R3: ningún donante supera 2 veces su mínimo en Barranquilla |
| SOL-20260025 | SI-PEI-01 | DES-003 | 16 | Escalar a centro de distribución | 0 | — | R2: Pereira tiene un solo almacén; el donante propuesto es de Medellín |
| SOL-20260026 | SM-BOG-01 | BEV-002 | 20 | Rechazar | 0 | — | R1: Bebidas no es categoría crítica y el stock está sobre el mínimo |
| SOL-20260027 | SM-MDE-01 | DES-001 | 10 | Aprobar con ajuste | 10 | EX-MDE-02 | R3: el donante propuesto (EX-MDE-01) no supera 2 veces su mínimo |

Son 6 tareas de aprobación (Centro-Oriente 3, Antioquia-Eje Cafetero 2, Caribe-Pacífico 1) con 202 unidades aprobadas en total.

### C.4 Desplegar

1. Confirma (*commit*) la versión probada: solo las versiones confirmadas se pueden desplegar.
2. En el despliegue, asigna como responsables de las tareas a los usuarios o al grupo de gerentes regionales de prueba.
3. Revisa que las credenciales e integraciones estén seleccionadas.
4. En la página de despliegue, elige **Create Trigger**: todos los días hábiles a las 08:00.

**Éxito:** las 9 solicitudes terminan en un estado final y cada decisión explica qué regla de PRO-OPS-003 aplicó. Las tareas llegan al gerente regional con la cantidad correcta y la automatización queda desplegada con un disparador.

---

## Cierre — ¿Flow, App o Automate?

Clasifica estos casos de Grupo Éxito y justifica tu elección en una línea:

| Caso | Flow | App | Automate |
| --- | --- | --- | --- |
| Resumen semanal de ventas por marca (Éxito, Carulla, Surtimax y Super Inter) para el gerente comercial | | | |
| Formulario para que los almacenes reporten productos dañados en la recepción | | | |
| Conciliar cada noche las facturas de proveedores contra las órdenes de compra, con revisión de diferencias | | | |
| Borrador de respuesta a un reclamo de cliente usando la política de devoluciones | | | |
| Portal de capacitación para nuevos coordinadores de calidad con su avance | | | |
| Alta de un nuevo almacén: crear usuarios, cargar surtido y notificar a cinco áreas, con aprobaciones | | | |

**Guía rápida:**

- **Flow:** tarea corta que una persona ejecuta o programa, con IA y alguna acción.
- **App:** una interfaz que el equipo usa a diario para ver y registrar información.
- **Automate:** proceso de punta a punta, de alto volumen, con reglas, varios sistemas, excepciones y aprobación humana.

## Referencias

- [Using Amazon Quick Flows](https://docs.aws.amazon.com/quick/latest/userguide/using-amazon-quick-flows.html)
- [Scheduling your Amazon Quick Flows](https://docs.aws.amazon.com/quick/latest/userguide/schedules-in-quick-flows.html)
- [Build web applications with apps in Amazon Quick](https://docs.aws.amazon.com/quick/latest/userguide/using-amazon-quick-apps.html)
- [Using Amazon Quick Automate](https://docs.aws.amazon.com/quick/latest/userguide/using-amazon-quick-automate.html)
- [Orchestration (casos y human-in-the-loop)](https://docs.aws.amazon.com/quick/latest/userguide/actions-orchestration.html)
- [Deploying automations](https://docs.aws.amazon.com/quick/latest/userguide/deploying-automations.html)
