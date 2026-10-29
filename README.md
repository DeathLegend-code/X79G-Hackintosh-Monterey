<div align="center">

# 🍎 OpenCore EFI for Atermiter X79G

**Xeon E5-2650 v2 · RX 570 · macOS Monterey**

![OpenCore](https://img.shields.io/badge/OpenCore-1.0.8-blue)
![macOS](https://img.shields.io/badge/macOS-Monterey%2012.7-success)
![Ventura+](https://img.shields.io/badge/Ventura%2B-no%20video-red)
![SMBIOS](https://img.shields.io/badge/SMBIOS-MacPro6%2C1-lightgrey)

<img width="586" height="351" alt="window" src="https://github.com/user-attachments/assets/9a79c7ed-6826-4825-b74b-31be96dd8f94" />


[English](#english) · [Русский](#русский)

</div>

---

## English

A working OpenCore EFI for a cheap Chinese **X79** board (Atermiter X79G rev 1.6, AMI Aptio IV) with a single **Xeon E5-2650 v2**. It includes a custom DSDT cut down to one socket, plus the fixes that were needed to boot with four DIMMs.

> ⚠️ This EFI is made for **my exact hardware and BIOS dump**. Read the notes below before using it on anything else.

### 🖥 Hardware

| Part | Model |
|------|-------|
| Board | Atermiter X79G rev 1.6 (BIOS `X79GA00R`) |
| CPU | Intel Xeon E5-2650 v2 (8C/16T, Ivy Bridge-EP) |
| RAM | 4 × 8 GB DDR3 ECC RDIMM (2 × Samsung + 2 × Micron), BIOS reports 1866 MT/s |
| GPU | MSI Armor RX 580 2048SP 8GB (flashed as RX 570) |
| SSD | Tesla 512 GB |
| SMBIOS | MacPro6,1 |

### ✅ Status (macOS Monterey 12.7.x)

| Feature | Status |
|---------|:------:|
| CPU: 8C/16T, turbo (~3.1 GHz under light load), power management via custom SSDT-PM | ✅ |
| RAM: all 32 GB (via `CustomMemory`) | ✅ |
| USB (EHCI) | ✅ |
| Audio (Realtek ALC662, layout 11) | ✅ |
| GPU acceleration, Ethernet, other devices | ✅ |
| Sleep (S3 is not wired on this board; disabled via `pmset`) | ❌ |
| Ventura and newer: installs, but **no video output** | ❌ |

**Why no video on Ventura+?** The E5-2650 v2 has AVX but no AVX2, and the Polaris driver in Ventura and newer needs AVX2. OpenCore Legacy Patcher root patches are the known workaround and are not included here.

### 🔧 What is special in this EFI

**Custom DSDT (`EFI/OC/ACPI/DSDT_*.aml`)**

The stock DSDT describes a 4-socket machine with 120 processor objects. macOS failed while enumerating CPUs, so the table was cut down:

- removed `SCK1`–`SCK3` and the extra threads `C010`–`C01D`; only `SCK0` with `C000`–`C00F` (16 threads) remains;
- `EUSB` → `EH01` and `USBE` → `EH02` renames are **baked into the table**;
- the HPET `_CRS` → `XCRS` rename and the TMR / PIC / RTC0 IRQ fixes are **baked in** too (they are paired with `SSDT-HPET.aml`).

> OpenCore `ACPI → Patch` rules did **not** apply to the replaced DSDT in my setup. That is why the renames and fixes live inside the table itself. The matching patches in `config.plist` are harmless leftovers.

The DSDT was made from the `X79GA00R` dump. Other BIOS versions or boards need their own table.

**`CustomMemory`**

With all 4 DIMMs installed the BIOS produced a wrong SMBIOS memory table: a 16 GB mapped range instead of 32 GB, and both Micron modules shown as empty. macOS hung at `EXITBS:START`. Filling `PlatformInfo → CustomMemory` with the real 4 × 8 GB fixed it. It is hardcoded for my DIMMs, so **edit it for your RAM**.

**ACPI / kexts**

- `SSDT-PM-2650V2.aml`: generated with ssdtPRGen for the E5-2650 v2.
- `SSDT-EC`, `SSDT-HPET`, `SSDT-SBUS-MCHC`, `SSDT-IMEI`, `SSDT-HDEF`.
- Kexts: Lilu, VirtualSMC (+ SMCProcessor / SMCSuperIO), AppleALC, CpuTscSync, USBInjectAll, NVMeFix, RestrictEvents. See the `Kexts` folder for the full list.
- Boot args that matter: `alcid=11 npci=0x3000`. The rest are debug or Ventura-specific.

### 📌 Before you use it

1. **`PlatformInfo` is empty.** Generate your own `SystemSerialNumber`, `MLB`, `SystemUUID` and `ROM` (for example with OpenCore Auxiliary Tools or `macserial`).
2. **Edit `CustomMemory`** for your DIMMs.
3. **BIOS:** UEFI boot, CFG Lock disabled. VT-d Disabled, 4G Above Decoding Enabled, EHCI-Handoff Enabled
4. Make a backup of your current EFI first.

### 🔌 Notes

- Hold the power button for ~2 seconds to get the shutdown dialog (a short press does nothing because sleep is disabled).
- Disabled in the config and kept only for reference: `ASPP-Override`, `CryptexFixup` (needed for Ventura and newer).

### 🙏 Credits

[OpenCore / Acidanthera](https://github.com/acidanthera), [Dortania guides](https://dortania.github.io/), [OpenCore Legacy Patcher](https://github.com/dortania/OpenCore-Legacy-Patcher), ssdtPRGen, [Антико 4PDA](https://4pda.to/forum/index.php?showuser=4709617) for helping to create the EFI-folder
### ⚖️ Disclaimer

Provided **as is**, without any warranty. You use it at your own risk. All third-party components (kexts, drivers, OpenCore) belong to their authors and keep their own licenses. No Apple software is included.

---

## Русский

Рабочий EFI на OpenCore для дешёвой китайской платы на **X79** (Atermiter X79G rev 1.6, AMI Aptio IV) с одним процессором **Xeon E5-2650 v2**. Внутри свой DSDT, урезанный до одного сокета, и все фиксы, которые понадобились, чтобы система грузилась с четырьмя планками памяти.

> ⚠️ EFI собран под **моё железо и мой дамп BIOS**. Прочитай заметки ниже, прежде чем ставить на что-то другое.

### 🖥 Железо

| Компонент | Модель |
|-----------|--------|
| Плата | Atermiter X79G rev 1.6 (BIOS `X79GA00R`) |
| Процессор | Intel Xeon E5-2650 v2 (8 ядер / 16 потоков, Ivy Bridge-EP) |
| Память | 4 × 8 ГБ DDR3 ECC RDIMM (2 × Samsung + 2 × Micron), BIOS показывает 1866 MT/s |
| Видеокарта | MSI Armor RX 580 2048SP 8GB (прошита как RX 570) |
| SSD | Tesla 512 ГБ |
| SMBIOS | MacPro6,1 |

### ✅ Статус (macOS Monterey 12.7.x)

| Функция | Статус |
|---------|:------:|
| Процессор: 8C/16T, турбо (~3.1 ГГц при лёгкой нагрузке), управление частотой через свой SSDT-PM | ✅ |
| ОЗУ: все 32 ГБ (через `CustomMemory`) | ✅ |
| USB (EHCI) | ✅ |
| Звук (Realtek ALC662, layout 11) | ✅ |
| Ускорение GPU, Ethernet и остальное | ✅ |
| Сон (S3 на плате не заведён, отключён через `pmset`) | ❌ |
| Ventura и новее: ставится, но **нет картинки** | ❌ |

**Почему на Ventura+ нет картинки?** У E5-2650 v2 есть AVX, но нет AVX2, а драйвер Polaris в Ventura и новее требует AVX2. Известный обход — root-патчи OpenCore Legacy Patcher, в этот EFI они не входят.

### 🔧 Что особенного в этом EFI

**Свой DSDT (`EFI/OC/ACPI/DSDT_*.aml`)**

Стоковый DSDT описывает четырёхсокетную машину со 120 объектами процессоров. macOS не справлялась с их перечислением, поэтому таблица урезана:

- удалены `SCK1`–`SCK3` и лишние потоки `C010`–`C01D`; остался только `SCK0` с `C000`–`C00F` (16 потоков);
- переименования `EUSB` → `EH01` и `USBE` → `EH02` **вшиты прямо в таблицу**;
- переименование HPET `_CRS` → `XCRS` и IRQ-фиксы TMR / PIC / RTC0 тоже **вшиты** (работают в паре с `SSDT-HPET.aml`).

> Правила `ACPI → Patch` из OpenCore у меня **не применились** к подменённому DSDT. Поэтому переименования и фиксы лежат внутри самой таблицы. Такие же патчи в `config.plist` остались просто так и вреда не делают.

DSDT собран по дампу `X79GA00R`. Для другой версии BIOS или другой платы нужна своя таблица.

**`CustomMemory`**

С четырьмя планками BIOS отдавала неверную SMBIOS-таблицу памяти: диапазон 16 ГБ вместо 32 ГБ и обе Micron показаны пустыми. macOS зависала на `EXITBS:START`. Заполнение `PlatformInfo → CustomMemory` реальными 4 × 8 ГБ это вылечило. Таблица зашита под мои планки, поэтому **поправь её под свою память**.

**ACPI / кексты**

- `SSDT-PM-2650V2.aml`: сгенерирован ssdtPRGen под E5-2650 v2.
- `SSDT-EC`, `SSDT-HPET`, `SSDT-SBUS-MCHC`, `SSDT-IMEI`, `SSDT-HDEF`.
- Кексты: Lilu, VirtualSMC (+ SMCProcessor / SMCSuperIO), AppleALC, CpuTscSync, USBInjectAll, NVMeFix, RestrictEvents. Полный список смотри в папке `Kexts`.
- Важные аргументы загрузки: `alcid=11 npci=0x3000`. Остальные — отладочные или нужны для Ventura.

### 📌 Перед использованием

1. **`PlatformInfo` пустой.** Сгенерируй свои `SystemSerialNumber`, `MLB`, `SystemUUID` и `ROM` (например, в OpenCore Auxiliary Tools или через `macserial`).
2. **Поправь `CustomMemory`** под свои планки.
3. **BIOS:** UEFI boot, CFG Lock Отключен. VT-d Отключен, 4G Above Decoding Включен, EHCI-Handoff Включен
4. Сначала сделай бэкап своего текущего EFI.

### 🔌 Заметки

- Чтобы выключить компьютер, удерживай кнопку питания около 2 секунд (короткое нажатие ничего не делает, потому что сон отключён).
- В конфиге выключены и лежат только для справки: `ASPP-Override`, `CryptexFixup` (нужны для Ventura и выше).

### 🙏 Благодарности

[OpenCore / Acidanthera](https://github.com/acidanthera), [руководства Dortania](https://dortania.github.io/), [OpenCore Legacy Patcher](https://github.com/dortania/OpenCore-Legacy-Patcher), ssdtPRGen, [Антико 4PDA](https://4pda.to/forum/index.php?showuser=4709617) за помощь в создании EFI-папки

### ⚖️ Отказ от ответственности

Предоставляется **как есть**, без каких-либо гарантий. Используешь на свой страх и риск. Все сторонние компоненты (кексты, драйверы, OpenCore) принадлежат своим авторам и сохраняют свои лицензии. Софта Apple внутри нет.
