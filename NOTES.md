# Notas de trabajo

## Mapa de memoria
- Desde `/proc/iomem` del kernel de TWRP (3.18.71): RAM del sistema en `0x10000000`..`0x849FFFFF`, reservas del módem y de otros DSP, y `0xA0000000`..`0xFFFFFFFF`.
- El shim ejecutado en `0x10080000` con MMU activa demuestra que la zona desde `0x10000000` es RAM.
- Otro origen (notas de la comunidad) indica RAM desde `0x40000000`. **No confirmado en este dispositivo**: contrastar siempre con `/proc/iomem`.

## Almacenamiento
- `userdata` en la eMMC: 54.819.536.384 bytes (107069407 sectores, LBA `0xE60000`). Confirmado con la tabla GPT y con `fastboot getvar partition-size:userdata`.
- Sistema de archivos de `userdata` no identificable por cabecera (ni f2fs ni ext4 en claro): probablemente cifrado.
- Tamaño de la eMMC: 62.537.072.640 bytes (122142720 sectores).

## Tarjeta SD
- Detección por GPIO 133 (1 = insertada). Confirmado.
- Reloj raíz SDCC2 en `0x43004` (cfg `0x2017` a 400 kHz, M=1, N=0xfc, D=0xfb). Confirmado.
- Reloj de interfaz en `0x4301C` y reloj de tarjeta en `0x43018`. Confirmados.
- Alimentación del bus: el controlador acepta `0x0F` en el registro de potencia. Sin regulador explícito en el driver (`No vmmc regulator found` en Linux).
- Pendiente: el controlador no genera comandos (`INT=0x00000000`, `PSTATE=0x0`). Causa no identificada.
- Hipótesis descartadas: pines (la SD no tiene selector de función), reloj raíz, modo SDHCI, confirmación de bus (PWRCTL), alimentación.
- Hipótesis sin probar: voto de ancho de banda del bus (requiere el driver de interconexión del kernel).
