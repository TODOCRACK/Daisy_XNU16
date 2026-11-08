# Daisy_XNU16
(Dame creditos si haces uso del REPO porfis, gracias <3)

Xiaomi Redmi 6 Pro (Daisy / Qualcomm MSM8953 Snapdragon 625), targeting iOS 16

The actual REPO where i'm working on updates is on private (for now) :P


Pruebas de bring-up de un shim de arranque para el **Xiaomi Mi A2 Lite (daisy, MSM8953)**,
con la intención de hacer funcionar iOS 16 mediante un kernel XNU. 
**Estado: experimental. No es un sistema operativo estable ni usable, puede dejar tu móvil como un ladrillo :P.**

El shim se carga siempre con `fastboot boot` (solo RAM). Nada de este repositorio
escribe en las particiones del móvil.

## Qué funciona (verificado en el dispositivo)

| Paso | Resultado |
|---|---|
| Carga con `fastboot boot` (imagen de página 2048) | OK |
| Entrada en EL1, mapa de hardware (FDT) | OK |
| Pantalla de texto 1080x2280, 24 bpp (azul-verde-rojo) | OK |
| MMU de 4 KiB activa | OK (requiere RAM mapeada como normal desde `0x10000000`) |
| GICv2 y temporizador virtual | OK |
| Lectura de la eMMC (GPT, `userdata` 107069407 sectores) | OK |
| Reloj raíz y de interfaz del SDCC2 (SD) | Configurado y leído |
| Detección de tarjeta por GPIO 133 | Funciona (1 = tarjeta insertada) |
| Identificación de la tarjeta SD (CMD0) | **No funciona**: el comando no se genera (`INT=0`) |

## Hallazgos del hardware

- RAM libre bajo `0xA0000000`..`0xF53FFFFF` (~1,31 GB) y RAM desde `0x10000000` (el shim se ejecuta en `0x10080000`).
- El mapa de `0x00000000`..`0x0FFFFFFF` debe tratarse como **memoria de dispositivos** (MMIO), no ejecutable.
- GIC en `0x0B000000` / `0x0B002000` (GICv2).
- eMMC: SDHCI en `0x07824900`. SD: SDHCI en `0x07864900`, núcleo en `0x07864000`.
- Pantalla: framebuffer en `0x90001000`, 1080x2280, 24 bpp.

## Cómo compilar (Windows)

1. Instalar LLVM: `winget install LLVM.LLVM`, y añadir `C:\Program Files\LLVM\bin` al PATH de la ventana.
2. `cd shim` y `python build.py`. Genera `artifacts\vince-boot.img`.
3. Probar **solo** con `fastboot boot artifacts\vince-boot.img`. Nunca `fastboot flash`.

`build.py` puede necesitar herramientas adicionales (`mkbootimg`, empaquetado del payload) que
no se incluyen aquí: obtenerlas de los proyectos de origen (ver Créditos).

## Estructura

- `shim/`: código del shim (MMU, GIC, temporizador, consola de texto, lectura de eMMC y pruebas de SD).
- `tools/mkadt_daisy.py`, `tools/verify_adt_daisy.py`: borrador del generador del árbol de dispositivo (ADT) para daisy. Aún no validado de extremo a extremo.
- `research/device_dumps/live_fdt.dtb`: mapa de hardware del Mi A2 Lite (`/sys/firmware/fdt`).
- `docs/`: notas de la investigación y procedimiento de copia de seguridad.

## Qué NO está en este R.E.P.O

- Copias de particiones (`userdata`, `gpt`, `ssd`, `keystore`, `boot`...). Son MIOSSS ;P
- IPSW de iOS ni el árbol de dispositivo de Apple (`DeviceTree.d22ap`). Hay que obtenerlos del propio IPSW de Apple.
- Binarios del kernel XNU compilados.

## Advertencias

- Probar siempre con `fastboot boot`. Hacer copia de seguridad de las particiones antes de cualquier `flash`.
- Ejecutables de herramientas de flasheo de terceros (por ejemplo `XiaoMiFlash.exe`) pueden borrar el móvil. No ejecutar nada que no se pueda verificar.

## Créditos

- [arabaci0711-ux/ios-vince](https://github.com/arabaci0711-ux/ios-vince): shim y parches XNU para vince (MSM8953), base de este trabajo.
- [dipa-studio/ios-sweet](https://github.com/dipa-studio/ios-sweet): herramientas de empaquetado del sistema de iOS.
- Dumped files from https://github.com/frost2k5-org/xiaomi_daisy_sprout_dump
- postmarketOS y msm8953-mainline: referencia de hardware.
- Apple: XNU de código abierto.
