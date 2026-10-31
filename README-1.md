<div align="center">

# 🍎 OpenCore EFI for Atermiter X79G

**Xeon E5-2650 v2 · RX 570 · macOS Monterey / Ventura+**

![OpenCore](https://img.shields.io/badge/OpenCore-1.0.8-blue)
![macOS](https://img.shields.io/badge/macOS-Monterey%2012.7-success)
![Ventura+](https://img.shields.io/badge/Ventura%2B-with%20OCLP%20patches-orange)
![SMBIOS](https://img.shields.io/badge/SMBIOS-MacPro6%2C1%20%2F%20MacPro7%2C1-lightgrey)


<img width="586" height="351" alt="window" src="https://github.com/user-attachments/assets/9a79c7ed-6826-4825-b74b-31be96dd8f94" /> 

<img width="327" height="596" alt="window" src="https://github.com/user-attachments/assets/65c55f48-d3f4-4c1c-892a-bbaaf6c87594" />

[English](#english) · [Русский](#русский)

</div>

---

## English

A working OpenCore EFI for a budget Chinese **X79** board (Atermiter X79G rev 1.6, AMI Aptio IV) with a single **Xeon E5-2650 v2**. It includes a custom DSDT cut down to one socket, plus the fixes that were needed to boot with four DIMMs.

> ⚠️ This EFI is made for **my exact hardware and BIOS dump**. Please read the notes below before using it on anything else.

### 🖥 Hardware

| Part | Model |
|------|-------|
| Board | Atermiter X79G rev 1.6 (BIOS `X79GA00R`) |
| CPU | Intel Xeon E5-2650 v2 (8C/16T, Ivy Bridge-EP) |
| RAM | 4 × 8 GB DDR3 ECC RDIMM (2 × Samsung + 2 × Micron), BIOS reports 1866 MT/s |
| GPU | MSI Armor RX 580 2048SP 8GB (flashed as RX 570) |
| SSD | Tesla 512 GB |
| SMBIOS | MacPro6,1 (Monterey), MacPro7,1 (Ventura and newer) |

### ✅ Status

| Feature | Status |
|---------|:------:|
| CPU: 8C/16T, turbo (~3.1 GHz under light load), power management via custom SSDT-PM | ✅ |
| RAM: all 32 GB (via `CustomMemory`) | ✅ |
| USB (EHCI) | ✅ |
| Audio (Realtek ALC662, layout 11) | ✅ |
| GPU acceleration, Ethernet, other devices (Monterey 12.7.x) | ✅ |
| Ventura and newer: install with acceleration disabled, then apply OCLP root patches (see [below](#-installing-ventura-and-newer)) | ⚠️ |
| Sleep (S3 is not wired on this board; disabled via `pmset`) | ❌ |

Please note that on Ventura and newer some minor graphical artifacts may remain even after patching (for example, a pixelated Dock or a translucent window during the minimize animation). They do not affect everyday use. If you run into graphics problems, or find the cause or a fix, please let us know by opening an [Issue](../../issues).

### 📥 Which release to download

| macOS | Release |
|-------|---------|
| Monterey | `1.0.8` |
| Ventura and newer | `1.0.8V` |

Please download the release that matches your macOS version from the **Releases** page.

### 🍏 Installing Ventura and newer

**Why are extra steps needed?** The E5-2650 v2 has AVX but no AVX2, and the Polaris graphics driver in Ventura and newer requires AVX2. Without a workaround the system has no video output. The known solution is to install with GPU acceleration disabled, and then apply the OpenCore Legacy Patcher (OCLP) root patches, which remove the AVX2 requirement.

1. **Back up your current EFI** and download the **`1.0.8V`** release.
2. In `config.plist`, please enable `CryptexFixup.kext`, and add the following to `boot-args`:
   ```
   -amd_no_dgpu_accel revpatch=sbvmm,f16c
   ```
   `-amd_no_dgpu_accel` disables GPU acceleration (software rendering, so you get a picture). `revpatch=f16c` is needed on a clean install: without it, Setup Assistant crashes with an illegal-instruction error and the system keeps rebooting.
3. Install macOS Ventura (or newer) as usual and complete the first setup.
4. Once you are at the desktop, run **OpenCore Legacy Patcher → Post-Install Root Patch** and apply the **AMD Polaris** patches. An internet connection is required. Reboot when it asks.
5. After the reboot, **remove `-amd_no_dgpu_accel` from `boot-args` in `config.plist`**, save it, reset NVRAM from the OpenCore menu and boot again.
6. If the picture disappears after step 5, please put the flag back and repeat step 4.

> 🔁 Root patches are removed by every macOS update. After updating, please apply them again (with the flag in place if needed).

> ℹ️ For OCLP, SIP and AMFI must be set accordingly. In this config that is `csr-active-config` = `03080000` and `amfi_get_out_of_my_way=1`.

### 🔧 What is special in this EFI

**Custom DSDT (`EFI/OC/ACPI/DSDT_*.aml`)**

The stock DSDT describes a 4-socket machine with 120 processor objects. macOS failed while enumerating CPUs, so the table was cut down:

- removed `SCK1`–`SCK3` and the extra threads `C010`–`C01D`; only `SCK0` with `C000`–`C00F` (16 threads) remains;
- `EUSB` → `EH01` and `USBE` → `EH02` renames are **baked into the table**;
- the HPET `_CRS` → `XCRS` rename and the TMR / PIC / RTC0 IRQ fixes are **baked in** too (they are paired with `SSDT-HPET.aml`).

> OpenCore `ACPI → Patch` rules did **not** apply to the replaced DSDT in my setup. That is why the renames and fixes live inside the table itself. The matching patches in `config.plist` are harmless leftovers.

The DSDT was made from the `X79GA00R` dump. Other BIOS versions or boards need their own table.

**`CustomMemory`**

With all 4 DIMMs installed the BIOS produced a wrong SMBIOS memory table: a 16 GB mapped range instead of 32 GB, and both Micron modules shown as empty. macOS hung at `EXITBS:START`. Filling `PlatformInfo → CustomMemory` with the real 4 × 8 GB fixed it. It is hardcoded for my DIMMs, so please **edit it for your RAM**.

**ACPI / kexts**

- `SSDT-PM-2650V2.aml`: generated with ssdtPRGen for the E5-2650 v2.
- `SSDT-EC`, `SSDT-HPET`, `SSDT-SBUS-MCHC`, `SSDT-IMEI`, `SSDT-HDEF`.
- Kexts: Lilu, VirtualSMC (+ SMCProcessor / SMCSuperIO), AppleALC, CpuTscSync, USBInjectAll, NVMeFix, RestrictEvents. See the `Kexts` folder for the full list.
- Boot args that matter: `alcid=11 npci=0x3000`. The rest are debug or Ventura-specific (see the Ventura section above).

### 📌 Before you use it

1. **`PlatformInfo` is empty.** Please generate your own `SystemSerialNumber`, `MLB`, `SystemUUID` and `ROM` (for example with OpenCore Auxiliary Tools or `macserial`).
2. **Edit `CustomMemory`** for your DIMMs.
3. **BIOS:** UEFI boot, CFG Lock disabled, VT-d disabled, Above 4G Decoding enabled, EHCI Hand-off enabled.
4. Make a backup of your current EFI first.

### 🔌 Notes

- Hold the power button for ~2 seconds to get the shutdown dialog (a short press does nothing because sleep is disabled).
- `ASPP-Override` is disabled in the config and kept only for reference. `CryptexFixup` is needed for Ventura and newer, so please enable it for those versions.

### 🙏 Credits

Many thanks to [OpenCore / Acidanthera](https://github.com/acidanthera), [Dortania guides](https://dortania.github.io/), [OpenCore Legacy Patcher](https://github.com/dortania/OpenCore-Legacy-Patcher), ssdtPRGen, and [Антико 4PDA](https://4pda.to/forum/index.php?showuser=4709617) for helping to create the EFI folder.

### ⚖️ Disclaimer

Provided **as is**, without any warranty, and you use it at your own risk. All third-party components (kexts, drivers, OpenCore) belong to their authors and keep their own licenses. No Apple software is included.

---

## Русский

Рабочий EFI на OpenCore для недорогой китайской платы на **X79** (Atermiter X79G rev 1.6, AMI Aptio IV) с одним процессором **Xeon E5-2650 v2**. Внутри свой DSDT, урезанный до одного сокета, и все фиксы, которые понадобились, чтобы система грузилась с четырьмя планками памяти.

> ⚠️ EFI собран под **моё железо и мой дамп BIOS**. Пожалуйста, прочитайте заметки ниже, прежде чем ставить его на что-то другое.

### 🖥 Железо

| Компонент | Модель |
|-----------|--------|
| Плата | Atermiter X79G rev 1.6 (BIOS `X79GA00R`) |
| Процессор | Intel Xeon E5-2650 v2 (8 ядер / 16 потоков, Ivy Bridge-EP) |
| Память | 4 × 8 ГБ DDR3 ECC RDIMM (2 × Samsung + 2 × Micron), BIOS показывает 1866 MT/s |
| Видеокарта | MSI Armor RX 580 2048SP 8GB (прошита как RX 570) |
| SSD | Tesla 512 ГБ |
| SMBIOS | MacPro6,1 (Monterey), MacPro7,1 (Ventura и новее) |

### ✅ Статус

| Функция | Статус |
|---------|:------:|
| Процессор: 8C/16T, турбо (~3.1 ГГц при лёгкой нагрузке), управление частотой через свой SSDT-PM | ✅ |
| ОЗУ: все 32 ГБ (через `CustomMemory`) | ✅ |
| USB (EHCI) | ✅ |
| Звук (Realtek ALC662, layout 11) | ✅ |
| Ускорение GPU, Ethernet и остальное (Monterey 12.7.x) | ✅ |
| Ventura и новее: установка с отключённым ускорением, затем root-патчи OCLP (см. [ниже](#-установка-ventura-и-новее)) | ⚠️ |
| Сон (S3 на плате не заведён, отключён через `pmset`) | ❌ |

Обратите внимание: на Ventura и новее после патчей могут остаться небольшие графические артефакты (например, «пиксельный» Dock или прозрачное окно при анимации сворачивания). На повседневную работу они не влияют. Если вы столкнётесь с проблемами графики или найдёте причину или решение, пожалуйста, напишите об этом в [Issue](../../issues).

### 📥 Какой релиз скачать

| macOS | Релиз |
|-------|-------|
| Monterey | `1.0.8` |
| Ventura и новее | `1.0.8V` |

Пожалуйста, скачайте на странице **Releases** тот релиз, который соответствует вашей версии macOS.

### 🍏 Установка Ventura и новее

**Зачем нужны дополнительные шаги?** У E5-2650 v2 есть AVX, но нет AVX2, а графический драйвер Polaris в Ventura и новее требует AVX2. Без обхода картинки не будет. Известное решение: установить систему с отключённым ускорением GPU, а затем применить root-патчи OpenCore Legacy Patcher (OCLP), которые убирают требование AVX2.

1. **Сделайте бэкап своего текущего EFI** и скачайте релиз **`1.0.8V`**.
2. В `config.plist` включите `CryptexFixup.kext` и добавьте в `boot-args`:
   ```
   -amd_no_dgpu_accel revpatch=sbvmm,f16c
   ```
   `-amd_no_dgpu_accel` отключает ускорение GPU (программный рендер, зато есть картинка). `revpatch=f16c` нужен при чистой установке: без него Setup Assistant падает с ошибкой недопустимой инструкции, и система уходит в бесконечную перезагрузку.
3. Установите macOS Ventura (или новее) как обычно и пройдите первоначальную настройку.
4. Когда попадёте на рабочий стол, запустите **OpenCore Legacy Patcher → Post-Install Root Patch** и примените патчи для **AMD Polaris**. Нужен доступ в интернет. Когда попросит, перезагрузитесь.
5. После перезагрузки **уберите `-amd_no_dgpu_accel` из `boot-args` в `config.plist`**, сохраните, сбросьте NVRAM в меню OpenCore и загрузитесь снова.
6. Если после пункта 5 картинка пропала, пожалуйста, верните флаг на место и повторите пункт 4.

> 🔁 Каждое обновление macOS снимает root-патчи. После обновления, пожалуйста, примените их заново (при необходимости снова с флагом).

> ℹ️ Для OCLP нужны соответствующие настройки SIP и AMFI. В этом конфиге это `csr-active-config` = `03080000` и `amfi_get_out_of_my_way=1`.

### 🔧 Что особенного в этом EFI

**Свой DSDT (`EFI/OC/ACPI/DSDT_*.aml`)**

Стоковый DSDT описывает четырёхсокетную машину со 120 объектами процессоров. macOS не справлялась с их перечислением, поэтому таблица урезана:

- удалены `SCK1`–`SCK3` и лишние потоки `C010`–`C01D`; остался только `SCK0` с `C000`–`C00F` (16 потоков);
- переименования `EUSB` → `EH01` и `USBE` → `EH02` **вшиты прямо в таблицу**;
- переименование HPET `_CRS` → `XCRS` и IRQ-фиксы TMR / PIC / RTC0 тоже **вшиты** (работают в паре с `SSDT-HPET.aml`).

> Правила `ACPI → Patch` из OpenCore у меня **не применились** к подменённому DSDT. Поэтому переименования и фиксы лежат внутри самой таблицы. Такие же патчи в `config.plist` остались просто так и вреда не делают.

DSDT собран по дампу `X79GA00R`. Для другой версии BIOS или другой платы нужна своя таблица.

**`CustomMemory`**

С четырьмя планками BIOS отдавала неверную SMBIOS-таблицу памяти: диапазон 16 ГБ вместо 32 ГБ и обе Micron показаны пустыми. macOS зависала на `EXITBS:START`. Заполнение `PlatformInfo → CustomMemory` реальными 4 × 8 ГБ это вылечило. Таблица зашита под мои планки, поэтому, пожалуйста, **поправьте её под свою память**.

**ACPI / кексты**

- `SSDT-PM-2650V2.aml`: сгенерирован ssdtPRGen под E5-2650 v2.
- `SSDT-EC`, `SSDT-HPET`, `SSDT-SBUS-MCHC`, `SSDT-IMEI`, `SSDT-HDEF`.
- Кексты: Lilu, VirtualSMC (+ SMCProcessor / SMCSuperIO), AppleALC, CpuTscSync, USBInjectAll, NVMeFix, RestrictEvents. Полный список смотрите в папке `Kexts`.
- Важные аргументы загрузки: `alcid=11 npci=0x3000`. Остальные отладочные или нужны для Ventura (см. раздел про Ventura выше).

### 📌 Перед использованием

1. **`PlatformInfo` пустой.** Пожалуйста, сгенерируйте свои `SystemSerialNumber`, `MLB`, `SystemUUID` и `ROM` (например, в OpenCore Auxiliary Tools или через `macserial`).
2. **Поправьте `CustomMemory`** под свои планки.
3. **BIOS:** UEFI boot, CFG Lock отключён, VT-d отключён, Above 4G Decoding включён, EHCI Hand-off включён.
4. Сначала сделайте бэкап своего текущего EFI.

### 🔌 Заметки

- Чтобы выключить компьютер, удерживайте кнопку питания около 2 секунд (короткое нажатие ничего не делает, потому что сон отключён).
- `ASPP-Override` в конфиге выключен и лежит только для справки. `CryptexFixup` нужен для Ventura и новее, поэтому для этих версий, пожалуйста, включите его.

### 🙏 Благодарности

Большое спасибо [OpenCore / Acidanthera](https://github.com/acidanthera), [руководствам Dortania](https://dortania.github.io/), [OpenCore Legacy Patcher](https://github.com/dortania/OpenCore-Legacy-Patcher), ssdtPRGen и [Антико 4PDA](https://4pda.to/forum/index.php?showuser=4709617) за помощь в создании EFI-папки.

### ⚖️ Отказ от ответственности

Предоставляется **как есть**, без каких-либо гарантий, и вы используете его на свой страх и риск. Все сторонние компоненты (кексты, драйверы, OpenCore) принадлежат своим авторам и сохраняют свои лицензии. Софта Apple внутри нет.
