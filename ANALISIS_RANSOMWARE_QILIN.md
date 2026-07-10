# Informe de Analisis Estatico - Ransomware Qilin (Target.exe)

**Fecha de analisis:** 10 de julio de 2026
**Analista:** Claudio Cortez
**Herramientas utilizadas:** IDA Pro 9.2 (Hex-Rays Decompiler), strings, Python3
**Plataforma de analisis:** Fedora 44 x86_64 (Linux) - Analisis estatico sin ejecucion

---

## 1. Resumen Ejecutivo

La muestra `Target.exe` corresponde a un **ransomware de la familia Qilin** (tambien conocida como "Agenda"), compilado en **Rust** con cross-compilation para Windows (MinGW/GCC). Es un encryptor empresarial de nivel avanzado, con capacidades de propagacion lateral (PsExec + PowerShell para vCenter/ESXi), cifrado hibrido (RSA-4096 + AES-CTR/ChaCha20), eliminacion de shadow copies, desbloqueo de archivos, evasion de sandbox, y auto-destruccion.

Pertenece al modelo **Ransomware-as-a-Service (RaaS)** con panel de negociacion en la red Tor.

---

## 2. Informacion del Binario

| Campo | Valor |
|-------|-------|
| **Nombre** | Target.exe |
| **Tamano** | 4.7 MB (4,915,200 bytes) |
| **Formato** | PE32 executable (console), Intel i386, stripped to external PDB |
| **Secciones** | 9 (.text, .data, .rdata, .eh_fram, .bss, .idata, .CRT, .tls, .reloc) |
| **Linker** | GNU ld 2.40 (MinGW cross-compiler) |
| **Lenguaje** | Rust (compilado con i686-pc-windows-gnu target) |
| **Timestamp** | 2026-01-06 17:46:56 UTC |
| **Entry Point** | 0x004014B0 |
| **Image Base** | 0x00400000 |
| **Subsistema** | WINDOWS_CUI (Consola) |
| **Caracteristicas** | EXECUTABLE_IMAGE, LARGE_ADDRESS_AWARE, 32BIT_MACHINE |

### 2.1 Hashes Criptograficos

| Algoritmo | Hash |
|-----------|------|
| **SHA-256** | `227ecf1a779cf7c19c0db869f66abf4b43d61b3700628d1d58b6eaeafe5496ba` |
| **SHA-1** | `fc3cab5aeea32162a6294cfc87fd16a81c068cfe` |
| **MD5** | `1c3da2e8855da39b0259b0515407b1b9` |

### 2.2 Secciones PE

| Nombre | Direccion Virtual | Tamano Virtual | Tamano Raw | Permisos |
|--------|-------------------|----------------|------------|----------|
| .text | 0x00001000 | 0x002C1018 | 0x002C1200 | CODE, EXEC, READ |
| .data | 0x002C3000 | 0x00000D60 | 0x00000E00 | DATA, READ, WRITE |
| .rdata | 0x002C4000 | 0x00195160 | 0x00195200 | DATA, READ |
| .eh_fram | 0x0045A000 | 0x0003D0C0 | 0x0003D200 | DATA, READ |
| .bss | 0x00498000 | 0x00000BD4 | 0x00000000 | UNINIT, READ, WRITE |
| .idata | 0x00499000 | 0x00001D54 | 0x00001E00 | DATA, READ, WRITE |
| .CRT | 0x0049B000 | 0x00000038 | 0x00000200 | DATA, READ, WRITE |
| .tls | 0x0049C000 | 0x00000008 | 0x00000200 | DATA, READ, WRITE |
| .reloc | 0x0049D000 | 0x0001AA30 | 0x0001AC00 | DATA, READ |

---

## 3. Atribucion y Familia

### 3.1 Identificacion: Qilin Ransomware

La atribucion se confirma por multiples indicadores:

1. **Nota de rescate** comienza con `"-- Qilin"`
2. **Infraestructura Tor** con dominios .onion conocidos del grupo Qilin
3. **Modelo RaaS** con panel de negociacion individual por victima
4. **Estructura del codigo fuente** en Rust con modulos `encryptor/src/` y `shared/`
5. **Capacidades avanzadas** consistentes con Qilin v3+ (propagacion vCenter, morph engine)

### 3.2 Grupo Amenaza

- **Nombre:** Qilin (anteriormente "Agenda")
- **Tipo:** Ransomware-as-a-Service (RaaS)
- **Primer avistamiento:** 2022
- **Lenguaje:** Migrado de Go a Rust (esta muestra es Rust)
- **Objetivos:** Empresas medianas y grandes, infraestructura VMware, hospitales, manufactura
- **Doble extorsion:** Si (robo de datos + cifrado)
- **Blog de filtraciones:** kbsqoivihgdmwczmxkbovk7ss2dcynitwhhfu5yw725dboqo5kthfaad.onion

---

## 4. Configuracion Embebida

El binario contiene una **configuracion JSON embebida** con los parametros operacionales del ataque:

### 4.1 Identificadores de la Campana

| Parametro | Valor |
|-----------|-------|
| **Company ID** | `0ziQdDa6WF` |
| **Extension cifrada** | `.0ziQdDa6WF` (tambien en blacklist para no re-cifrar) |
| **Password hash** | `4a35592a69a5e9602ddec358e4803b7142e6e3c9ff414ff10155590fca5c924a` |
| **Password operacional** | `2qoSubkqdMJC7W5ScupwuMJP04nyyRep` |

### 4.2 Clave Publica RSA-4096

```
-----BEGIN PUBLIC KEY-----
MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEA3utjtobIXIRRnKAYTRbs
7QxDs9ucFaOf4t57zcGUmMS4naen9bcC2QbHU39kTf4JVqcEyGdrHlwT+fvgScci
2+RYFT29sHzdNVqFu+WmKu4NnmqIpMn+SoDPdLwctt9v6XWNe0fY8qxdD0k/n2Z0
4Kl3MBXgpy/NNasPIWAyxYZeGkIxw+30tm5QpG7LIvbt9tIyzYFnjMb9kkOw7COY
RgjgpmxvcP5N3h2x8FSfQyhBDdyQrR7LlErp8HGIfLYZdasSA7bZAFE+nrZhDuLy
QGBddQ5n4uIakiBjFzUpEaklQSssmZZWdcYc4+VPwZxdg5S/6InAVxfym006B4Uz
LZsXN0B1TUw4qQGwr05dZAqkYOS31WemDFre73sxWjLLVguvm2xCFMHmWYeZL09p
uzJKqM9wdjwLiD6lmofsPYqQ6AJQQJPfXn0+H68o//59cCj37Zv8Iy6KEDm9KHzP
Z2OarPGj0PfAE6HfrjZcDvPz8wJZGg3eJEaUs9P62S2qSa2IT93RMnYWIkqzQRjl
eqvbF0WkQsgP3MlRruLDNE3aGjkDWTmgXbO8j5Xg9JhnhZCvP+BVbgHjFhMa09g9
fbsuW4O8mdMbHdP4YElNDHgi2XmdCk2+N3hI/b30cyayjaRzCjbRi3miWUlqdV1/
mn0vjTNM6nOV3Y0uyiQAgssCAwEAAQ==
-----END PUBLIC KEY-----
```

### 4.3 Parametros de Cifrado

| Parametro | Valor | Descripcion |
|-----------|-------|-------------|
| `n` | 0 | Modo Normal (cifrado completo) |
| `p` | 1 | Porcentaje para Skip-Percent |
| `fast` | 0 | Fast mode desactivado |
| `skip` | 0 | Bytes a saltar |
| `step` | 0 | Bytes a cifrar por ciclo |

### 4.4 Nota de Rescate

**Nombre del archivo:** `.README-RECOVER-.txt`

**Contenido:**
```
-- Qilin

Your network/system was encrypted.
Encrypted files have new extension.

-- Compromising and sensitive data

We have downloaded compromising and sensitive data from your system/network.
Our group cooperates with the mass media.
If you refuse to communicate with us and we do not come to an agreement,
your data will be reviewed and published on our blog and on the media page
(https://wikileaks2.site/)

Blog links:
http://kbsqoivihgdmwczmxkbovk7ss2dcynitwhhfu5yw725dboqo5kthfaad.onion
http://ijzn3sicrcy7guixkzjkib4ukbiilwc3xhnmby4mcbccnsd7j2rekvqd.onion

Data includes:
- Employees personal data, CVs, DL, SSN.
- Complete network map including credentials for local and remote services.
- Financial information including clients data, bills, budgets, annual reports, bank statements.
- Complete datagrams/schemas/drawings for manufacturing in solidworks format
- And more...

-- Recovery

1) Download tor browser: https://www.torproject.org/download/
2) Go to domain
3) Enter credentials

-- Credentials

Extension: 0ziQdDa6WF
Domain: spzq5vlkhohit3gddz2vu4gxw23m7czqx3hqx63jqpprf3cgdly5tkid.onion
login: L5PT0tjSvEjcBDLrF87OW8AdDoJ60Xs0
password: [redacted - embedded in binary]
```

### 4.5 Listas Negras de Exclusion

#### Procesos a Terminar (process_black_list)
```
vmms, vmwp, vmcompute, agntsvc, dbeng50, dbsnmp, encsvc, excel,
firefox, infopath, isqlplussvc, sql, msaccess, mspub, mydesktopqos,
mydesktopservice, notepad, ocautoupds, ocomm, ocssd, onenote, oracle,
outlook, powerpnt, sqbcoreservice, steam, synctime, tbirdconfig,
thebat, thunderbird, visio, winword, wordpad, xfssvccon, bedbh,
vxmon, benetns, bengien, pvlsvr, beserver, raw_agent_svc, vsnapvss,
cagservice, qbidpservice, qbdbmgrn, qbcfmonitorservice, sap,
teamviewer_service, teamviewer, tv_w32, tv_x64, cvmountd, cvd,
cvfwd, cvods, saphostexec, saposcol, sapstartsrv, avagent, avscc,
dellsystemdetect, enterpriseclient, veeamnfssvc, veeamtransportsvc,
veeamdeploymentsvc, mvdesktopservice
```

#### Servicios a Detener (win_services_black_list)
```
vmms, mepocs, memtas, veeam, backup, vss, sql, msexchange, sophos,
wsbexchange, pdvfsservice, backupexecvssprovider, backupexecagentaccelerator,
backupexecagentbrowser, backupexecdivecimediaservice, backupexecjobengine,
backupexecmanagementservice, backupexecrpcservice, gxblr, gxvss,
gxclmgrs, gxcvd, gxcimgr, gxmmm, gxvsshwprov, gxfwd, sapservice,
sap, sap$, sapd$, saphostcontrol, saphostexec, qbcfmonitorservice,
qbdbmgrn, qbidpservice, acronisagent, veeamnfssvc, veeamdeploymentservice,
veeamtransportsvc, mvarmor, mvarmor64, vsnapvss, acrsch2svc, (.*?)sql(.*?)
```

**Nota:** El patron `(.*?)sql(.*?)` es una regex que mata CUALQUIER servicio con "sql" en el nombre.

#### Directorios Excluidos (directory_black_list)
```
windows, system volume information, intel, admin$, ipc$, sysvol,
netlogon, $windows.~ws, application data, mozilla, program files (x86),
program files, $windows.~bt, msocache, tor browser, programdata, boot,
config.msi, google, perflogs, appdata, windows.old, .., .,
$recycle.bin
```

#### Archivos Excluidos (file_black_list)
```
desktop.ini, autorun.ini, ntldr, bootsect.bak, thumbs.db, boot.ini,
ntuser.dat, iconcache.db, bootfont.bin, ntuser.ini, ntuser.dat.log,
autorun.inf, bootmgr, bootmgr.efi, bootmgfw.efi, #recycle
```

#### Extensiones Excluidas (file_pattern_black_list)
```
themepack, nls, diapkg, msi, lnk, exe, scr, bat, drv, rtp, msp,
prf, msc, ico, key, ocx, diagcab, diagcfg, pdb, wpx, hlp, icns,
rom, dll, msstyles, mod, ps1, ics, hta, bin, cmd, ani, 386, lock,
cur, idx, sys, com, deskthemepack, shs, theme, mpa, nomedia, spl,
cpl, adv, icl, msu, 0ziQdDa6WF
```

**Nota:** La extension `0ziQdDa6WF` esta en la blacklist para evitar re-cifrar archivos ya cifrados.

#### Extensiones de Symlinks Permitidas (file_pattern_white_list)
```
mdf, ldf, bak, vib, vbk, vmv, rba, bkb, kz, sqb, trn, backup,
old, pfi, pbf, dim, gho, vpc, backup, arc, mtf, bkf
```

Estas son extensiones tipicas de **bases de datos y backups** - el ransomware sigue symlinks hacia ellas.

---

## 5. Arquitectura del Malware

### 5.1 Estructura del Codigo Fuente (Rust)

Basado en los paths de compilacion extraidos del binario:

```
/usr/src/myapp/windows-rust/
+-- encryptor/
|   +-- src/
|       +-- config/
|       |   +-- embedded_config.rs      # Configuracion embebida JSON
|       +-- pipelines/
|           +-- encryption_pipeline.rs  # Pipeline de cifrado por disco
+-- shared/
    +-- cryptography/
    |   +-- src/
    |       +-- encryption.rs           # Logica de cifrado (AES/ChaCha20)
    |       +-- config.rs               # Configuracion crypto
    +-- flags/
    |   +-- src/
    |       +-- flags/
    |           +-- mod.rs              # Parsing de flags CLI
    |           +-- flag_value.rs       # Valores de flags
    |           +-- unit_parsing.rs     # Parsing de unidades
    +-- logging/
    |   +-- src/lib.rs                  # Sistema de logging
    +-- networking/
    |   +-- src/lib.rs                  # Enumeracion de red
    +-- printing/
    |   +-- src/
    |       +-- printers.rs            # Enumeracion de impresoras
    |       +-- windows.rs             # Impresion en Windows
    +-- spreading/
    |   +-- src/
    |       +-- lib.rs                 # Modulo de propagacion
    |       +-- psexec.rs              # Propagacion via PsExec
    |       +-- vcenter.rs             # Propagacion via vCenter
    |       +-- windows.rs             # Propagacion Windows
    +-- windows/
        +-- src/
            +-- cmd/mod.rs             # Ejecucion de comandos
            +-- files/mod.rs           # Operaciones de archivos
            +-- helper/mod.rs          # Utilidades Windows
            +-- processes/
            |   +-- mod.rs             # Process killer
            |   +-- process_api.rs     # API de procesos
            +-- safemode/mod.rs        # Modo seguro
            +-- services/
            |   +-- mod.rs             # Service killer
            |   +-- service_api.rs     # API de servicios
            +-- storage/
            |   +-- shares.rs          # Shares de red
            |   +-- volumes.rs         # Volumenes locales
            +-- users/mod.rs           # Manejo de usuarios
            +-- wallpaper/mod.rs       # Cambio de wallpaper
```

### 5.2 Dependencias Rust (Crates)

| Crate | Version | Proposito |
|-------|---------|-----------|
| aes | 0.8.3 | Cifrado AES |
| cipher | 0.4.4 | Trait de cifrado |
| rsa | 0.7.2 | Cifrado RSA asimetrico |
| rand_chacha | 0.3.1 | ChaCha20 PRNG |
| rand | 0.8.5 | Generacion aleatoria |
| serde_json | 1.0.105 | Parsing de configuracion JSON |
| regex | 1.9.3 | Matching de patterns (servicios) |
| walkdir | 2.3.3 | Traversal de directorios |
| threadpool | 1.8.1 | Pool de hilos para cifrado |
| chrono | 0.4.26 | Timestamps |
| uuid | 1.4.1 | Generacion de UUIDs |
| gethostname | 0.4.3 | Obtener nombre de host |
| whoami | 1.4.1 | Info del usuario actual |
| ipconfig | 0.3.2 | Configuracion de red |
| winreg | 0.50.0 | Acceso al registro Windows |
| zstd | 0.11.2 | Compresion Zstandard |
| zip | 0.6.6 | Archivos ZIP (logs) |
| flate2 | 1.0.28 | Compresion DEFLATE |
| bzip2 | 0.4.4 | Compresion BZIP2 |
| base64ct | 1.6.0 | Codificacion Base64 |
| sha1_smol | 1.0.0 | Hash SHA-1 |
| num-bigint-dig | 0.8.4 | Aritmetica de precision arbitraria (RSA) |
| pem-rfc7468 | 0.6.0 | Parsing de claves PEM |
| zeroize | 1.6.0 | Limpieza segura de memoria |
| widestring | 1.0.2 | Strings UTF-16 (Windows API) |
| aho-corasick | 1.0.3 | Busqueda de patrones |
| once_cell | 1.19.0 | Inicializacion lazy |
| smallvec | 1.11.0 | Vectores stack-allocated |
| memchr | 2.5.0 | Busqueda de bytes |

### 5.3 Estadisticas IDA Pro

| Metrica | Valor |
|---------|-------|
| Total funciones | 6,604 |
| Funciones con nombre | 236 |
| Funciones decompiladas (Hex-Rays) | 62 |
| Total imports | 226 |
| Total strings | 7,271 |

---

## 6. Capacidades Tecnicas

### 6.1 Cifrado

**Esquema hibrido:**
- **Asimetrico:** RSA-4096 (clave publica embebida) para proteger la clave de sesion
- **Simetrico:** AES-CTR (si CPU soporta AES-NI) o ChaCha20 (fallback)
- **Seleccion automatica:** El binario detecta soporte de instrucciones AES-NI en el procesador

**Mensajes relevantes del binario:**
```
[INFO] AESNI support detected! Using AES-CTR mode
[WARNING] CPU doesn't support AESNI instructions. Using ChaCha20 mode.
[ERROR] Failed to query CPU info. Using ChaCha20 mode.
```

**Modos de cifrado (5 estrategias):**

1. **Auto (recomendado):** Seleccion automatica basada en tipo y tamano de archivo. Archivos <100MB se cifran completamente (Normal), archivos grandes usan Skip-Percent primero y luego Normal en segundo pase.
2. **Normal:** Cifrado completo del archivo entero.
3. **Step-Skip:** Cifra N bytes, salta M bytes, en ciclo.
4. **Skip-Percent:** Similar a Step-Skip pero el salto se calcula como porcentaje del tamano del archivo, redondeado a 512 KiB.

**Caracteristicas de resiliencia:**
- **Checkpoints:** Permite recuperar archivos si el proceso se interrumpe o la maquina se reinicia
- **Markers de inicio/fin:** Permiten descifrar aunque se haya anadido contenido al inicio o final del archivo
- **Extension temporal:** Durante el cifrado se usa una extension temporal, restaurando el nombre original al finalizar

### 6.2 Escalacion de Privilegios

1. **Verificacion de admin:** Requiere privilegios de administrador (abort sin `--no-admin`)
2. **Escalacion a SYSTEM:** Se eleva automaticamente al nivel NT AUTHORITY\SYSTEM
3. **Impersonacion de tokens:** Duplica tokens de otros procesos para acceder a recursos protegidos
4. **Multi-SID:** Recopila informacion a traves de multiples identidades (SIDs) impersonadas

**APIs utilizadas:**
- `OpenProcessToken` / `DuplicateTokenEx`
- `AdjustTokenPrivileges` (SeDebugPrivilege, SeShutdownPrivilege)
- `SetThreadToken` / `RevertToSelf`
- `CreateProcessAsUserW` / `CreateProcessWithLogonW`

### 6.3 Propagacion Lateral

#### PsExec (Windows -> Windows)
- PsExec de Sysinternals **embebido completamente** en el binario
- PDB paths: `D:\a\1\s\psexec\exe\Win32\Release\psexec.pdb`
- Se extrae (drop) al sistema de archivos antes de usarse
- Soporta lista de hosts con credenciales (`host:user:pass`)
- Si no se proporciona lista, **escanea TODOS los hosts del dominio** via Active Directory:
  ```powershell
  Import-Module ActiveDirectory;
  Get-ADComputer -Filter * | Select-Object -ExpandProperty DNSHostName
  ```
- Cada host se procesa en hilo separado
- Comando de ejecucion: `/C -accepteula \\ -c -f -h -d -i`

#### vCenter/ESXi (PowerShell)
- Script PowerShell completo embebido para propagacion a VMware
- **Flujo:**
  1. Instala .NET Framework 4.7.2 si es necesario
  2. Instala modulos PowerShell: `VMware.PowerCLI`, `Posh-SSH`
  3. Conecta a vCenter
  4. Desactiva HA/DRS en todos los clusters
  5. Obtiene lista de hosts ESXi
  6. Cambia password de root en cada ESXi
  7. Habilita SSH en cada host
  8. Sube payload via SCP
  9. Ejecuta payload en cada ESXi

### 6.4 Eliminacion de Shadow Copies

```
/C vssadmin.exe delete shadows /all /quiet
```

**Flujo:**
1. Habilita servicio VSS (si esta deshabilitado)
2. Elimina todas las shadow copies silenciosamente
3. Deshabilita VSS despues de la purga

### 6.5 Modo Seguro (Safe Mode)

Capacidad de reiniciar en **modo seguro con red** para evadir antivirus:

```
BCDEdit.exe /set {current} safeboot network
```

**Flujo:**
1. Cambia la password del usuario actual al valor de `--password`
2. Configura AutoLogin (registro `Winlogon`)
3. Configura autostart en modo seguro
4. Reinicia en safe mode con networking
5. El cifrado se ejecuta automaticamente al iniciar en safe mode

### 6.6 Terminacion de Procesos y Servicios

- **Procesos:** Termina 65+ procesos (bases de datos, backup, oficina, virtualizacion)
- **Servicios:** Detiene y deshabilita 50+ servicios (Veeam, SQL, Exchange, Sophos, VSS)
- **VMs:** Detiene maquinas virtuales Hyper-V (vmms, vmwp, vmcompute)
- **Clusters:** Opcion de detener clusters (`Stop-Cluster -Force`)
- Deteccion de Hyper-V para excluir procesos propios del host

### 6.7 File Unlocker (Restart Manager)

Utiliza la API de **Restart Manager** de Windows para desbloquear archivos en uso:

- `RmStartSession` - Inicia sesion
- `RmRegisterResources` - Registra archivo bloqueado
- `RmGetList` - Obtiene lista de procesos que bloquean
- `RmEndSession` - Cierra sesion

### 6.8 Enumeracion de Red y Shares

- **Shares locales montados:** `WNetOpenEnumW` / `WNetEnumResourceW`
- **Shares de dominio:** `NetShareEnum` + enumeracion Active Directory
- **Subnet scanning:** Escaneo de toda la subred local
- **Shortcuts de red:** Busca accesos directos a shares en el escritorio
- Detecta y omite IPv6
- Filtra shares admin$ para evitar danarse a si mismo

### 6.9 Almacenamiento y Discos

- **Monta discos offline:** Trae online discos fisicos desconectados
- **Elimina read-only:** Quita flags de solo lectura
- **DeviceIoControl:** Para operaciones de disco de bajo nivel
- **SetupDi API:** Enumeracion de dispositivos de almacenamiento
- **Tipo de disco:** Detecta tipo (HDD/SSD) para optimizar threads

### 6.10 Persistencia

- **Autostart:** Se agrega al registro de autostart para TODOS los usuarios
- **Flags preservados:** Al autostart se pasan los mismos flags de la ejecucion original
- **Limpieza:** Elimina claves de autostart anteriores antes de crear nueva

### 6.11 Anti-Forense

1. **Eliminacion de Event Logs:**
   ```powershell
   $logs = Get-WinEvent -ListLog * | Where-Object {$_.RecordCount} |
   Select-Object -ExpandProperty LogName;
   ForEach ($l in $logs | Sort | Get-Unique) {
       [System.Diagnostics.Eventing.Reader.EventLogSession]::GlobalSession.ClearLog($l)
   }
   ```
   - Se ejecuta multiples veces (durante y despues del cifrado)
   - Hilo dedicado para eliminacion continua

2. **Zeroing de espacio libre:** Escribe ceros en todo el espacio libre de todos los discos
3. **Auto-destruccion:** Se elimina a si mismo despues de completar el cifrado
4. **Logging en %TEMP%:** Logs en `C:\Windows\Temp\QLOGS` (desactivable con `--no-logs`)

### 6.12 Cambio de Wallpaper e Intimidacion

- **Wallpaper:** Cambia el fondo de escritorio con imagen embebida (JPEG con metadata Adobe Photoshop 23.2)
- **Lockscreen:** Modifica la imagen de pantalla de bloqueo via registro
- **Impresion:** Opcion de imprimir la nota de rescate en TODAS las impresoras disponibles:
  ```powershell
  Get-Printer | Format-List Name,DriverName
  Get-Content -Path '<nota>' | Out-Printer -Name '<printer>'
  ```

### 6.13 Deteccion de Sandbox

- Detecta si se ejecuta en entorno virtualizado
- Si detecta sandbox, **no mata** ciertos procesos/servicios para mantener estabilidad
- Desactivable con `--no-sandbox`

### 6.14 Mutex

- Crea mutex nombrado para prevenir multiples ejecuciones en el mismo host
- `CreateMutexA` / `CreateMutexW`
- Desactivable con `--force`

---

## 7. Imports por DLL

### KERNEL32.dll (124 funciones)
Funciones criticas: `CreateFileW`, `CreateProcessW`, `CreateThread`, `DeviceIoControl`, `FindFirstFileW`, `GetLogicalDrives`, `LoadLibraryA`, `MoveFileExW`, `OpenProcess`, `SetFileAttributesW`, `TerminateProcess`, `VirtualProtect`, `Wow64DisableWow64FsRedirection`

### advapi32.dll (22 funciones)
Funciones criticas: `AdjustTokenPrivileges`, `ControlService`, `DuplicateTokenEx`, `EnumServicesStatusW`, `OpenProcessToken`, `RegSetValueExW`, `RevertToSelf`

### netapi32.dll (3 funciones)
`NetShareEnum`, `NetApiBufferFree`, `NetUserSetInfo`

### mpr.dll (4 funciones)
`WNetOpenEnumW`, `WNetEnumResourceW`, `WNetCloseEnum`, `WNetGetLastErrorA`

### ws2_32.dll (5 funciones)
`WSAStartup`, `WSACleanup`, `getaddrinfo`, `freeaddrinfo`, `WSAGetLastError`

### rstrtmgr.dll (4 funciones)
`RmStartSession`, `RmGetList`, `RmRegisterResources`, `RmEndSession`

### setupapi.dll (6 funciones)
Enumeracion completa de dispositivos de almacenamiento

### ntdll.dll (3+1 funciones)
`NtReadFile`, `NtWriteFile`, `NtSetInformationProcess`, `RtlNtStatusToDosError`

### Otras DLLs
- **bcrypt.dll:** `BCryptGenRandom` (generacion de numeros aleatorios criptograficos)
- **iphlpapi.dll:** `GetAdaptersAddresses` (enumeracion de interfaces de red)
- **ole32.dll:** COM (CoCreateInstance para WMI/VSS)
- **psapi.dll:** `EnumProcesses`, `GetProcessImageFileNameW`
- **shell32.dll:** `SHGetKnownFolderPath` (Desktop, etc.), `ShellExecuteA`
- **user32.dll:** `SystemParametersInfoW` (wallpaper), `ExitWindowsEx` (reinicio)

---

## 8. Indicadores de Compromiso (IOCs)

### 8.1 Hashes

| Tipo | Valor |
|------|-------|
| SHA-256 | `227ecf1a779cf7c19c0db869f66abf4b43d61b3700628d1d58b6eaeafe5496ba` |
| SHA-1 | `fc3cab5aeea32162a6294cfc87fd16a81c068cfe` |
| MD5 | `1c3da2e8855da39b0259b0515407b1b9` |

### 8.2 Infraestructura de Red (Dominios Tor)

| Proposito | Dominio |
|-----------|---------|
| Panel de negociacion | `spzq5vlkhohit3gddz2vu4gxw23m7czqx3hqx63jqpprf3cgdly5tkid.onion` |
| Blog de filtraciones #1 | `kbsqoivihgdmwczmxkbovk7ss2dcynitwhhfu5yw725dboqo5kthfaad.onion` |
| Blog de filtraciones #2 | `ijzn3sicrcy7guixkzjkib4ukbiilwc3xhnmby4mcbccnsd7j2rekvqd.onion` |
| Pagina de medios | `wikileaks2.site` (clearnet) |

### 8.3 Credenciales del Panel

| Campo | Valor |
|-------|-------|
| Login | `L5PT0tjSvEjcBDLrF87OW8AdDoJ60Xs0` |
| Password | [embebido en nota - no extraido] |

### 8.4 Archivos Creados

| Archivo | Descripcion |
|---------|-------------|
| `.README-RECOVER-.txt` | Nota de rescate (en cada directorio) |
| `%TEMP%\QLOGS\*` | Logs de ejecucion |
| `*.0ziQdDa6WF` | Extension temporal durante cifrado |

### 8.5 Claves de Registro Modificadas

| Clave | Proposito |
|-------|-----------|
| `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\DefaultUserName` | AutoLogin |
| `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\DefaultPassword` | AutoLogin |
| `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon\AutoAdminLogon` | AutoLogin |
| `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Personalization` | Wallpaper |
| `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\PersonalizationCSP` | Lockscreen |

### 8.6 Comandos Ejecutados

```
cmd.exe /d /c "vssadmin.exe delete shadows /all /quiet"
cmd.exe /d /c "fsutil behavior set SymlinkEvaluation R2R:1"
cmd.exe /d /c "fsutil behavior set SymlinkEvaluation R2L:1"
cmd.exe /d /c "wmic service where name='<service>' call ChangeStartMode Manual"
BCDEdit.exe /set {current} safeboot network
powershell -Command "Import-Module ActiveDirectory; Get-ADComputer -Filter *..."
powershell -Command "Get-WinEvent -ListLog * | ... ClearLog($l)"
powershell -Command "Set-ItemProperty -Path '<path>' -Name Wallpaper -Value '<path>'"
powershell -Command "Stop-Cluster -Force"
powershell -Command "Get-Printer | Format-List Name,DriverName"
powershell -Command "Get-Content -Path '<note>' | Out-Printer -Name '<printer>'"
```

### 8.7 Mutex

Crea un mutex nombrado (nombre especifico no extraido de strings - generado en runtime).

### 8.8 PsExec Embebido

- PDB: `D:\a\1\s\psexec\exe\Win32\Release\psexec.pdb`
- PDB servicio: `D:\a\1\s\psexec\svc\Win32\Release\psexesvc.pdb`
- Se droppea temporalmente al disco para propagacion

---

## 9. Flujo de Ejecucion

```
[INICIO]
    |
    v
[1. Verificar password CLI] ---> FATAL si incorrecto
    |
    v
[2. Verificar admin] ---> FATAL sin --no-admin
    |
    v
[3. Crear mutex] ---> FATAL si ya existe (sin --force)
    |
    v
[4. Escalacion a SYSTEM]
    |
    v
[5. Inicializar host info]
    |
    v
[6. Ir a background] (por defecto)
    |
    +---> [7a. Matar VMs/Procesos/Servicios]
    |         (espera 15s para servicios, 5s para procesos)
    |
    +---> [7b. Montar discos offline + quitar read-only]
    |
    +---> [7c. Eliminar shadow copies]
    |
    +---> [7d. Habilitar autostart]
    |
    +---> [7e. Establecer prioridad CPU/IO maxima]
    |
    v
[8. Pipeline de cifrado]
    |
    +---> [8a. Discos locales] (threads = num_cores x tipo_disco)
    +---> [8b. Shares montados]
    +---> [8c. Shortcuts de red]
    +---> [8d. Shares de dominio]
    +---> [8e. Shares de subred]
    |
    v
[9. Esperar finalizacion de todos los jobs]
    |
    v
[10. Post-cifrado]
    +---> [10a. Cambiar wallpaper/lockscreen]
    +---> [10b. Imprimir nota (si --print-image)]
    +---> [10c. Limpiar event logs]
    +---> [10d. Zeroing espacio libre (background)]
    +---> [10e. Auto-destruccion]
    |
    v
[FIN]
```

---

## 10. MITRE ATT&CK Mapping

| Tactica | Tecnica | ID | Descripcion |
|---------|---------|----|-------------|
| Execution | Command and Scripting Interpreter: PowerShell | T1059.001 | Scripts PS para propagacion y limpieza |
| Execution | Command and Scripting Interpreter: cmd | T1059.003 | Ejecucion de comandos via cmd.exe |
| Execution | System Services: Service Execution | T1569.002 | PsExec para ejecucion remota |
| Persistence | Boot or Logon Autostart | T1547.001 | Autostart en registro |
| Privilege Escalation | Access Token Manipulation | T1134 | Impersonacion de tokens SYSTEM |
| Defense Evasion | Indicator Removal: Clear Windows Event Logs | T1070.001 | Limpieza de event logs |
| Defense Evasion | File Deletion | T1070.004 | Auto-destruccion |
| Defense Evasion | Virtualization/Sandbox Evasion | T1497 | Deteccion de sandbox |
| Defense Evasion | Safe Mode Boot | T1562.009 | Reinicio en safe mode |
| Credential Access | Credentials from Password Stores | T1555 | Uso de credenciales embebidas |
| Discovery | Process Discovery | T1057 | Enumeracion de procesos |
| Discovery | System Service Discovery | T1007 | Enumeracion de servicios |
| Discovery | Network Share Discovery | T1135 | Enumeracion de shares |
| Discovery | Remote System Discovery | T1018 | Escaneo de dominio/subred |
| Discovery | System Information Discovery | T1082 | Info de CPU, discos, red |
| Lateral Movement | Remote Services: SMB/Admin Shares | T1021.002 | PsExec via admin$ |
| Lateral Movement | Exploitation of Remote Services | T1210 | vCenter/ESXi propagation |
| Impact | Data Encrypted for Impact | T1486 | Cifrado de archivos |
| Impact | Inhibit System Recovery | T1490 | Eliminacion de shadow copies |
| Impact | Service Stop | T1489 | Detencion de servicios criticos |
| Impact | System Shutdown/Reboot | T1529 | Reinicio a safe mode |
| Impact | Defacement: Internal | T1491.001 | Cambio de wallpaper |

---

## 11. Recomendaciones de Deteccion

### 11.1 Reglas YARA (Sugeridas)

```yara
rule Qilin_Ransomware_Rust {
    meta:
        description = "Detecta ransomware Qilin compilado en Rust"
        author = "Analisis Interno"
        date = "2026-07-10"
        hash = "227ecf1a779cf7c19c0db869f66abf4b43d61b3700628d1d58b6eaeafe5496ba"

    strings:
        $rust_path1 = "encryptor/src/config/embedded_config.rs"
        $rust_path2 = "encryptor/src/pipelines/encryption_pipeline.rs"
        $rust_path3 = "shared/cryptography/src/encryption.rs"
        $rust_path4 = "shared/spreading/src/psexec.rs"
        $note = "-- Qilin" ascii wide
        $readme = ".README-RECOVER-.txt"
        $config1 = "process_black_list"
        $config2 = "win_services_black_list"
        $config3 = "company_id"
        $config4 = "password_hash"
        $psexec_pdb = "psexec\\exe\\Win32\\Release\\psexec.pdb"
        $aesni = "AESNI support detected"
        $chacha = "Using ChaCha20 mode"

    condition:
        uint16(0) == 0x5A4D and
        filesize < 10MB and
        (3 of ($rust_path*) or
         ($note and 2 of ($config*)) or
         ($psexec_pdb and $aesni))
}
```

### 11.2 Indicadores de Red

- Monitorear trafico DNS hacia dominios .onion
- Alertar sobre escaneos masivos de puertos SMB (445) internos
- Detectar uso de PsExec hacia multiples hosts en corto tiempo
- Monitorear conexiones SSH hacia hosts ESXi desde servidores Windows

### 11.3 Indicadores de Endpoint

- Creacion masiva de archivos `.README-RECOVER-.txt`
- Ejecucion de `vssadmin delete shadows /all /quiet`
- Modificacion de `BCDEdit.exe /set safeboot`
- Creacion de directorio `QLOGS` en `%TEMP%`
- Proceso con prioridad Realtime de CPU/IO
- Llamadas masivas a Restart Manager API
- Escritura masiva a disco con patron de rename temporal

---

## 12. Conclusiones

Esta muestra de **Qilin ransomware** representa una amenaza de **nivel critico** para entornos empresariales:

1. **Madurez tecnica:** Codigo Rust bien estructurado, modular, con manejo robusto de errores
2. **Amplitud de impacto:** Cifra local + red + ESXi + propagacion lateral automatica
3. **Resiliencia:** Checkpoints, markers, multiple retry, auto-recovery
4. **Anti-forense:** Triple limpieza (logs, zeroing, auto-delete)
5. **Evasion:** Safe mode, sandbox detection, impersonacion
6. **Optimizacion:** Seleccion automatica de algoritmo, threads por tipo de disco
7. **Intimidacion:** Wallpaper, impresion, doble extorsion

La presencia de credenciales para el panel Tor y la clave RSA-4096 publica confirman que este build es **operacional** y fue generado para un objetivo especifico con company_id `0ziQdDa6WF`.

---

## 13. Archivos de Soporte

| Archivo | Descripcion |
|---------|-------------|
| `ida_analysis/functions.json` | 6,604 funciones identificadas por IDA |
| `ida_analysis/imports.json` | 226 imports organizados por DLL |
| `ida_analysis/strings.json` | 7,271 strings extraidos |
| `ida_analysis/decompiled.json` | 62 funciones decompiladas (Hex-Rays) |
| `ida_analysis/summary.json` | Resumen del analisis IDA |
| `password.txt` | Password operacional de la muestra |
| `usage.txt` | Manual de uso original del operador (RU/EN) |

---

*Documento generado mediante analisis estatico. No se ejecuto el malware en ningun momento.*
