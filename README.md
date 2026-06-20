# DSTRT — Prototipo 1

Módulo de efectos send/return para DJ, con arquitectura híbrida analógica/digital, inspirado en el formato de pedal de efectos.

## Descripción

DSTRT es un módulo de efectos pensado para conectarse al lazo de envío/retorno (send/return) de una mezcladora de DJ, igual que un pedal de efectos en una pedalera de guitarra. Combina tres efectos clásicos —**filtro**, **delay** y **reverb**— que pueden activarse de forma independiente y combinarse libremente entre sí mediante footswitches.

La cadena de señal es híbrida:

- **Filtro:** completamente analógico, con bypass por relé.
- **Delay y reverb:** procesados digitalmente sobre un núcleo DSP (Electrosmith Daisy Seed), activables/desactivables por software.
- **Bypass maestro:** relé true bypass, cableado en modo *fail-safe* (passthrough directo si se pierde alimentación), pensado para uso en vivo.

Este repositorio documenta el desarrollo del **Prototipo 1**: diseño de la etapa analógica, integración del DSP, diseño de PCB en EasyEDA y pruebas de validación.

## Estado del proyecto

En desarrollo — Fase 0 (especificaciones) 

## Stack

- Hardware: PCB diseñada en EasyEDA
- DSP: Electrosmith Daisy Seed (DaisySP, C++)
- Firmware: control de footswitches, potenciómetros y bypass por relé
