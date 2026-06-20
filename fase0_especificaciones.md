# Fase 0 — Especificaciones del proyecto
## Módulo de efectos send/return para DJ (híbrido analógico/digital)

---

## 1. Resumen de arquitectura

Cadena de señal fija en serie, con bloques individualmente activables:

```
Entrada (send) → Filtro analógico [bypass relé] → ADC → DSP digital (Delay → Reverb) [on/off software] → DAC → Bypass maestro [relé] → Salida (return)
```

- **Filtro:** analógico, activo/inactivo por relé.
- **Delay y reverb:** digitales, corren en el mismo DSP, cada uno con su propio flag de encendido (footswitch → GPIO → firmware). Se pueden combinar libremente entre sí y con el filtro.
- **Bypass maestro:** saca todo el módulo de la cadena de señal (relé true bypass).

---

## 2. Estrategia de bypass

| Bloque | Tipo de bypass | Justificación |
|---|---|---|
| Filtro analógico | Relé (DPDT) | Sin degradación de señal, sin fugas (vs. switches CMOS tipo 4066) |
| Delay | Software (flag + crossfade ~10-20ms) | No requiere hardware adicional, evita clicks al activar/desactivar |
| Reverb | Software (flag + crossfade ~10-20ms) | Igual que delay |
| Módulo completo | Relé (DPDT), normalmente cerrado en modo passthrough | **Fail-safe**: si se pierde alimentación o el firmware se cuelga, el audio sigue pasando físicamente. Crítico en contexto de DJ en vivo. |

**Regla de diseño:** ningún bypass debe depender únicamente del firmware para mantener el audio vivo. El relé maestro es la única garantía física de continuidad de señal.

---

## 3. Niveles de señal e impedancias

Los send/return de mezcladoras DJ no están estandarizados entre fabricantes: hay equipos con nivel de línea de consumo (~-10 dBV) y equipos pro con nivel balanceado (~+4 dBu). Para no limitar el módulo a una sola marca:

- **Rango de entrada objetivo:** -20 dBV a +4 dBu (cubre consumo y pro), con etapa de ganancia/atenuación ajustable (trimmer) antes del ADC.
- **Impedancia de entrada:** ≥10 kΩ (no cargar la salida del send del mixer).
- **Impedancia de salida:** ≤1 kΩ (manejar cualquier entrada de return sin pérdida de nivel).
- **Protección de entrada:** clamping con diodos antes de la primera etapa activa (proteger contra niveles inesperados o conexión en caliente).
- **Referencia del ADC/DAC:** el codec del Daisy Seed trabaja en torno a una polarización de ~1.65V (alimentación 3.3V), por lo que la etapa de entrada necesita atenuación + bias de DC (red de polarización a midpoint) para acoplar correctamente la señal de línea profesional al rango del codec, y la etapa de salida necesita el desacople de DC inverso + ajuste de nivel de vuelta a línea profesional.

*(Los valores exactos de ganancia y componentes se calculan en la Fase 1, al diseñar la etapa analógica.)*

---

## 4. Conectores

- **Entrada y salida:** Jack TRS 1/4" (6.35mm). TRS aunque se use en modo unbalanced, para compatibilidad con mezcladoras que sí envían balanceado.
- **Alimentación:** Jack DC barril 5.5x2.1mm, centro negativo (estándar de pedales de efectos).

---

## 5. Plataforma digital

**Recomendación: Electrosmith Daisy Seed**

Justificación:
- Códec de audio integrado (24-bit / 48kHz), pensado específicamente para procesamiento de audio embebido.
- Librería `DaisySP` con delay, reverb (`ReverbSc`) y filtros ya implementados como punto de partida — permite enfocar el esfuerzo en la etapa analógica y en el ajuste fino de los algoritmos, no en reinventar el manejo de DMA/I2S desde cero.
- Se integra como módulo (headers o pads castellated) directamente en tu PCB de EasyEDA — no necesitas diseñar tu propio mínimo sistema con STM32 desde cero (aunque es una opción futura si quieres llevar el control total del firmware a otro nivel).
- Bajo costo (~$25-30 USD).

**Sample rate:** 48 kHz / 24-bit (estándar de la plataforma).
**Latencia presupuestada:** objetivo <10 ms total (códec + procesamiento), para que el módulo no se note "atrasado" en contextos de mezcla en vivo.

---

## 6. Alimentación

- **Entrada:** 9V DC, centro negativo (estándar de pedal).
- **Regulación necesaria:**
  - 9V → 5V para alimentar el VIN del Daisy Seed (su regulador interno baja a 3.3V).
  - Generación de **virtual ground** (~4.5V) para polarizar las etapas analógicas de un solo riel (filtro, buffers de entrada/salida) — patrón clásico de pedales de efectos.
- Considerar un regulador de bajo ruido (LDO) para no inyectar ruido de conmutación en la etapa analógica.

---

## 7. Interfaz de control

| Elemento | Cantidad estimada | Función |
|---|---|---|
| Footswitch | 4 | Filtro on/off, Delay on/off, Reverb on/off, Bypass maestro |
| Potenciómetros | ~6 | Filtro: frecuencia de corte + resonancia. Delay: tiempo + feedback + mix. Reverb: tiempo + mix |
| LEDs indicadores | 4 | Uno por cada footswitch (estado activo) |
| Conexión a MCU | — | Footswitches y pots se leen por GPIO/ADC del propio Daisy Seed |

*(El número exacto de pots se ajusta en Fase 1 según cuántos parámetros quieras exponer físicamente vs. fijos en firmware.)*

---

## 8. Consideraciones para el diseño de PCB en EasyEDA

- Verificar si existe footprint del Daisy Seed en la librería de EasyEDA/LCSC; si no, crear footprint personalizado (castellated edge o pines THT según la versión que compres).
- Separar físicamente en el layout la zona analógica (filtro, buffers, virtual ground) de la zona digital (Daisy Seed, líneas de reloj/I2S) para minimizar acoplamiento de ruido — plano de tierra único pero con buen criterio de retorno de corriente.
- Dejar pads de prueba (test points) en los nodos clave: salida del filtro, entrada/salida del codec, virtual ground. Te van a salvar la vida en la fase de pruebas.
- Pensar el tamaño de la PCB en función del chasis tipo pedal que planeas usar (define esto antes de rutear, no después).

---

## 9. Próximos pasos (Fase 1)

1. Diseño y cálculo de la etapa de entrada (atenuación, bias DC, protección).
2. Diseño del filtro analógico (topología: Sallen-Key vs. state-variable — a decidir según si quieres resonancia controlable).
3. Diseño de la etapa de salida (desacople DC, ajuste de nivel de vuelta a línea).
4. Validación en simulación SPICE (EasyEDA tiene simulador integrado) antes de pasar a prototipo físico.
