# Sistema Autónomo de Refrigeración Inteligente para PODs de Data Center

---

## Introducción

Este proyecto propone un **sistema autónomo y automático de control térmico** para PODs de Data Center, tanto **confinados** (CAC / HAC) como  **no confinados** , orientado a:

* Proteger la carga IT
* Optimizar el consumo energético
* Mantener continuidad operativa ante fallas
* Reducir la intervención manual

El sistema está diseñado para controlar  **4 unidades InRow** , cada una refrigerando  **3 racks** , utilizando sensores críticos de temperatura de entrada ( **INLET** ) y sensores de contexto según el escenario operativo.

---

## Objetivos del sistema

* Mantener la temperatura de entrada al rack dentro de un rango óptimo
* Ajustar automáticamente los setpoints de las unidades InRow
* Adaptar dinámicamente el intervalo de control según la criticidad térmica
* Detectar fallas de sensores y operar de forma segura
* Emitir alertas ante condiciones anómalas
* Funcionar como una  **política automática** , no como un simple script

---

## Arquitectura conceptual del sistema

### Ciclo operativo continuo

El sistema opera bajo un  **loop de control autónomo** , que  **nunca se detiene** .

Ante eventos anómalos, cambia su  **modo de operación** , no su ejecución.

**Estados posibles:**

* 🟢 **Modo Normal** → Control fino y eficiente
* 🟡 **Modo Degradado** → Control conservador
* 🔴 **Modo Seguro** → Prioridad operativa
* 

## Diagrama del Ciclo de Control Autónomo

El siguiente diagrama representa el ciclo completo de control térmico del sistema,
incluyendo validación de sensores, modos operativos y toma de decisiones.

```mermaid
flowchart TD
    A[Inicio del Ciclo de Control] --> B[Lectura de Sensores]

    B --> C[Validación de lecturas]

    C --> D{Estado del sensor}

    D -->|OK| E[Modo Normal]
    D -->|Inestable| F[Modo Degradado]
    D -->|Caído| G[Modo Seguro]

    E --> H[Leer setpoint actual InRow]
    F --> H
    G --> H

    H --> I[Evaluar temperatura de entrada al rack]

    I --> J[Calcular setpoint según modo activo]

    J --> K[Aplicar setpoint al InRow]

    K --> L[Registrar log / auditoría]

    L --> M[Definir próximo intervalo Δt]

    M --> N[Esperar Δt]

    N --> A
```

---

## Sensores y criterios de medición

### Sensor crítico

* **Temperatura de entrada al rack (INLET)**

  Es el sensor principal sobre el cual se toman todas las decisiones de control térmico.

### Sensores de contexto (según escenario)

* Ambiente (CAC / sin confinamiento)
* Pasillo caliente (HAC)
* Estado de puertas de confinamiento

> **Importante**
>
> La temperatura del aire caliente es informativa, pero **nunca reemplaza** la temperatura de entrada al rack como variable de control.

---

## Gestión inteligente de fallas de sensores

El sistema  **no reacciona ante una única lectura inválida** .

### Política de validación

* Cada sensor posee un contador de fallas consecutivas
* Una lectura válida reinicia el contador
* Al superar un umbral:
  * El sensor se marca como no confiable
  * Se genera una alerta
  * El sistema entra en **modo seguro**

### Esto evita:

* Oscilaciones innecesarias
* Falsas alarmas
* Ajustes térmicos peligrosos

---

## Lógica general de control térmico

1. Se leen los sensores
2. Se valida la calidad de las lecturas
3. Se determina el estado operativo
4. Se calcula el setpoint adecuado
5. Se aplica el setpoint al InRow
6. Se define el próximo intervalo de control
7. Se registra todo para auditoría

---

## Fragmentos clave del código (explicados)

### Configuración general del sistema

<pre class="overflow-visible! px-0!" data-start="4101" data-end="4456"><div class="contain-inline-size rounded-2xl corner-superellipse/1.1 relative bg-token-sidebar-surface-primary"><div class="sticky top-[calc(--spacing(9)+var(--header-height))] @w-xl/main:top-9"><div class="absolute end-0 bottom-0 flex h-9 items-center pe-2"><div class="bg-token-bg-elevated-secondary text-token-text-secondary flex items-center gap-4 rounded-sm px-2 font-sans text-xs"></div></div></div><div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-python"><span><span># 🎯 OBJETIVOS TÉRMICOS</span><span>
TEMPERATURA_OBJETIVO = </span><span>22.0</span><span>
BANDA_MUERTA = </span><span>0.5</span><span>

</span><span># 🔒 LÍMITES OPERATIVOS</span><span>
SETPOINT_MINIMO = </span><span>17.0</span><span>
SETPOINT_MAXIMO = </span><span>25.0</span><span>

</span><span># ⏱️ INTERVALOS DE CONTROL</span><span>
INTERVALO_NORMAL = </span><span>20</span><span> * </span><span>60</span><span>
INTERVALO_ALERTA = </span><span>5</span><span> * </span><span>60</span><span>
INTERVALO_CRITICO = </span><span>2</span><span> * </span><span>60</span><span>

</span><span># 🔧 AJUSTES</span><span>
AJUSTE_SUAVE = </span><span>0.5</span><span>
AJUSTE_RAPIDO = </span><span>1.0</span><span>

</span><span># 🏗️ ESCENARIO</span><span>
ESCENARIO_POD = </span><span>"HAC"</span><span>
</span></span></code></div></div></pre>

---

### Lectura del sensor crítico (INLET)

<pre class="overflow-visible! px-0!" data-start="4629" data-end="4783"><div class="contain-inline-size rounded-2xl corner-superellipse/1.1 relative bg-token-sidebar-surface-primary"><div class="sticky top-[calc(--spacing(9)+var(--header-height))] @w-xl/main:top-9"><div class="absolute end-0 bottom-0 flex h-9 items-center pe-2"><div class="bg-token-bg-elevated-secondary text-token-text-secondary flex items-center gap-4 rounded-sm px-2 font-sans text-xs"></div></div></div><div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-python"><span><span>def</span><span></span><span>leer_sensor_inlet</span><span>(</span><span>inrow_id</span><span>):
    """
    Lee la temperatura de entrada de aire al rack.
    Sensor CRÍTICO del sistema.
    """
    </span><span>pass</span><span>
</span></span></code></div></div></pre>

**Concepto**

Mide la temperatura real que recibe la carga IT.

Es la referencia principal del sistema.

---

### Sensor de contexto

<pre class="overflow-visible! px-0!" data-start="4927" data-end="5118"><div class="contain-inline-size rounded-2xl corner-superellipse/1.1 relative bg-token-sidebar-surface-primary"><div class="sticky top-[calc(--spacing(9)+var(--header-height))] @w-xl/main:top-9"><div class="absolute end-0 bottom-0 flex h-9 items-center pe-2"><div class="bg-token-bg-elevated-secondary text-token-text-secondary flex items-center gap-4 rounded-sm px-2 font-sans text-xs"></div></div></div><div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-python"><span><span>def</span><span></span><span>leer_sensor_contexto</span><span>(</span><span>inrow_id</span><span>):
    """
    Sensores complementarios según escenario:
    - CAC → Ambiente
    - HAC → Pasillo caliente
    - SIN → Ambiente
    """
    </span><span>pass</span><span>
</span></span></code></div></div></pre>

**Concepto**

Aporta diagnóstico y contexto, pero **no gobierna** el control.

---

### Estado del confinamiento

<pre class="overflow-visible! px-0!" data-start="5241" data-end="5403"><div class="contain-inline-size rounded-2xl corner-superellipse/1.1 relative bg-token-sidebar-surface-primary"><div class="sticky top-[calc(--spacing(9)+var(--header-height))] @w-xl/main:top-9"><div class="absolute end-0 bottom-0 flex h-9 items-center pe-2"><div class="bg-token-bg-elevated-secondary text-token-text-secondary flex items-center gap-4 rounded-sm px-2 font-sans text-xs"></div></div></div><div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-python"><span><span>def</span><span></span><span>puertas_confinamiento_cerradas</span><span>():
    """
    Verifica si las puertas del confinamiento están cerradas.
    Aplica a CAC / HAC.
    """
    </span><span>pass</span><span>
</span></span></code></div></div></pre>

**Concepto**

El sistema considera variables físicas reales del entorno.

---

### Función central de control

<pre class="overflow-visible! px-0!" data-start="5523" data-end="5772"><div class="contain-inline-size rounded-2xl corner-superellipse/1.1 relative bg-token-sidebar-surface-primary"><div class="sticky top-[calc(--spacing(9)+var(--header-height))] @w-xl/main:top-9"><div class="absolute end-0 bottom-0 flex h-9 items-center pe-2"><div class="bg-token-bg-elevated-secondary text-token-text-secondary flex items-center gap-4 rounded-sm px-2 font-sans text-xs"></div></div></div><div class="overflow-y-auto p-4" dir="ltr"><code class="whitespace-pre! language-python"><span><span>def</span><span></span><span>evaluar_y_controlar_inrow</span><span>(</span><span>inrow_id</span><span>):
    """
    Función principal del sistema:
    - Lee sensores
    - Valida datos
    - Define modo operativo
    - Calcula y aplica setpoint
    - Ajusta el intervalo de control
    """
    </span><span>pass</span><span>
</span></span></code></div></div></pre>

**Concepto**

Esta función representa la  **política automática de refrigeración** .

---

## Modo seguro operativo

Ante fallas persistentes:

* Se fija un setpoint conservador
* Se prioriza la protección de la carga IT
* El sistema sigue operando

Ejemplo:

> Setpoint fijo en 20 °C hasta restaurar sensores confiables.

---

## Beneficios del sistema

✔ Reducción de riesgo térmico

✔ Menor intervención humana

✔ Mayor eficiencia energética

✔ Resiliencia ante fallas

✔ Escalabilidad

✔ Auditoría y trazabilidad

---

## Conclusión

Este proyecto no busca solo automatizar equipos, sino  **establecer una política autónoma de refrigeración** , capaz de adaptarse al contexto real del Data Center.

> **La automatización no reemplaza al operador: le devuelve control estratégico.**
>
> **El sistema no automatiza equipos. Automatiza decisiones.**

---

## Mejoras y Próximos pasos

* Integración con DCIM
* Alertas reales (Mail / Teams / WhatsApp)
* Históricos y dashboards
* Predicción térmica (ML)
