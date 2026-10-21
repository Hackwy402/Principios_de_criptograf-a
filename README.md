# Taller: acceso a un servidor con llaves SSH

**Principios de Criptografía**

En este taller vas a:

1. Generar tu propio **par de llaves** (privada + pública).
2. Entregar **solo la llave pública**.
3. Verificar la **identidad del servidor** antes de conectarte.
4. Descargar por **SFTP**, sin usar contraseñas, el libro *A Graduate Course in Applied Cryptography* (Dan Boneh y Victor Shoup, versión 0.6). Sus autores lo distribuyen gratis en [toc.cryptobook.us](https://toc.cryptobook.us).

| Dato | Valor |
|---|---|
| Servidor | `54.157.214.128` |
| Tu usuario | `est_nombre_apellido` (el docente te lo confirma) |
| Archivo | `/libros/libro.pdf` |

> ⚠️ **El servidor cambió de dirección** (antes `158.23.160.9`). Usa la IP y las huellas de esta guía. Si tu `known_hosts` guarda la IP vieja, no te afecta: la nueva es un servidor distinto.

**Actividades:**
> - Primera: descargar el libro por SFTP.
> - Laboratorio forense I — integridad y cadena de custodia: **[LAB_FORENSE.md](LAB_FORENSE.md)**.
> - Laboratorio forense II — reconstruir un incidente (Autopsy): **[LAB_INCIDENTE.md](LAB_INCIDENTE.md)**.

---

## 0. Antes de empezar: ¿tengo el cliente SSH?

| Sistema | Qué hacer |
|---|---|
| **Windows 10/11** | Abre **PowerShell** y escribe `ssh -V`. Si no aparece la versión, ve a *Configuración → Aplicaciones → Características opcionales → Agregar* e instala **Cliente OpenSSH**. |
| **macOS** | Abre **Terminal**. Ya viene instalado (`ssh -V`). |
| **Linux** | Abre una terminal. Si `ssh -V` falla: `sudo apt install openssh-client`. |

---

## 1. Generar tu par de llaves

Usa **una** de estas dos opciones. Escribe tu nombre real en el formato `nombre.apellido`: con ese comentario se crea tu cuenta.

**ECDSA sobre la curva NIST P-256:**

```
ssh-keygen -t ecdsa -b 256 -C "nombre.apellido"
```

**Ed25519 (curva de Edwards, recomendada):**

```
ssh-keygen -t ed25519 -C "nombre.apellido"
```

El comando es el mismo en Windows (PowerShell), macOS y Linux.

- Cuando pregunte *"Enter file in which to save the key"*, pulsa **Enter**.
- Cuando pida una **passphrase**, escribe una frase que recuerdes. Con ella la llave privada **se guarda cifrada** en tu disco. Si alguien copia el archivo, no le sirve sin la frase.

Se crean dos archivos:

| Archivo | Qué es | ¿Se comparte? |
|---|---|---|
| `id_ecdsa` / `id_ed25519` | **Llave privada** | **NUNCA** |
| `id_ecdsa.pub` / `id_ed25519.pub` | **Llave pública** | Sí, es la que entregas |

Ubicación: `~/.ssh/` en macOS y Linux; `C:\Users\TU_USUARIO\.ssh\` en Windows.

**Sobre los parámetros:**
- `-t` es el algoritmo.
- `-b 256` es el tamaño de la curva. Una llave ECDSA de 256 bits ofrece una seguridad comparable a RSA de 3072 bits.
- `-C` es un comentario. No es secreto ni forma parte de las matemáticas de la llave: sirve para identificarla.

---

## 2. Ver y enviar SOLO la llave pública

**Windows (PowerShell):**

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ecdsa.pub            # o id_ed25519.pub
Get-Content $env:USERPROFILE\.ssh\id_ecdsa.pub | Set-Clipboard   # la copia al portapapeles
```

**macOS:**

```
cat ~/.ssh/id_ecdsa.pub
pbcopy < ~/.ssh/id_ecdsa.pub     # la copia al portapapeles
```

**Linux:**

```
cat ~/.ssh/id_ecdsa.pub
```

Debe verse como **una sola línea**:

```
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAA... nombre.apellido
```

### Entrega: pega tu llave pública en el pad del curso

👉 **https://pad.riseup.net/p/Principios_de_criptograf%C3%ADa**

Reglas:

1. Pega tu llave en **una sola línea**, al final del pad: el contenido completo de tu `.pub`, tal cual sale del comando anterior.
2. **Una línea por estudiante.** No agregues nada más en esa línea (ni tu nombre aparte, ni guiones, ni viñetas). Tu nombre ya va en la llave, al final (`nombre.apellido`).
3. **No modifiques ni borres las líneas de tus compañeros.**
4. Si te equivocaste, **corrige solo tu propia línea**.

Ejemplo de cómo debe quedar el pad:

```
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBB... maria.lopez
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIGq7... juan.perez
```

**Si lo que vas a pegar empieza con `-----BEGIN OPENSSH PRIVATE KEY-----`, DETENTE: es tu llave privada.** Si la pegaste por error, bórrala del pad y genera un par nuevo: esa llave ya no es secreta.

> **Pregunta para pensar:** el pad lo puede editar cualquiera que tenga el enlace. ¿Qué impide que alguien reemplace tu llave por la suya y entre al servidor con tu usuario? La respuesta está en la verificación de huellas de la siguiente sección.

### ¿Por qué la privada nunca se comparte?

- La autenticación por llave es una **firma digital**. El servidor te envía un reto y tú lo firmas con tu llave privada. El servidor verifica la firma con tu llave pública.
- Quien tenga tu llave privada **puede firmar como tú**: para el servidor, *es* tú. No hay forma de distinguirlo.
- La llave pública solo sirve para **verificar** firmas. Que la conozca todo el mundo no es un problema: de ella no se puede obtener la privada (ese es el problema del logaritmo discreto en curvas elípticas).
- Por eso nadie legítimo te pedirá jamás la llave privada: ni el docente, ni un administrador, ni "soporte técnico".

### Verificación de tu huella

Cuando el docente cargue las llaves, te pedirá que leas en voz alta la **huella** (fingerprint) de tu llave pública:

```
ssh-keygen -lf ~/.ssh/id_ecdsa.pub
```

En Windows: `ssh-keygen -lf $env:USERPROFILE\.ssh\id_ecdsa.pub`

Así el docente confirma que la llave que recibió es la tuya y que nadie la cambió en el camino. Es la misma idea que el paso 3, pero en sentido contrario.

---

## 3. Primera conexión: verificar la identidad del servidor

La primera vez que te conectes verás algo así:

```
The authenticity of host '54.157.214.128 (54.157.214.128)' can't be established.
ED25519 key fingerprint is SHA256:acKi2UauHa2YeEyFn5Y0D7OG+2NhBi/Hf57ZCgbQNHE.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

**Antes de escribir `yes`, compara la huella carácter por carácter** con las huellas oficiales del servidor:

| Tipo | Huella SHA256 |
|---|---|
| ED25519 | `SHA256:acKi2UauHa2YeEyFn5Y0D7OG+2NhBi/Hf57ZCgbQNHE` |
| ECDSA | `SHA256:6NawMFlx1KJ3Hh7rlADUKmxuaFcvsw+X+nAwDADxglk` |
| RSA | `SHA256:aAvS+5lE4BJjYe7wtxqilMzQZHz55jsPXN6z+buHTwI` |

- **Si coincide**, escribe `yes`.
- **Si NO coincide, escribe `no`** y avisa al docente.

### TOFU: *Trust On First Use*

SSH no usa autoridades certificadoras como HTTPS. La primera vez **confías** en la llave que presenta el servidor y la guardas en `~/.ssh/known_hosts`. En las conexiones siguientes, SSH comprueba que la llave sea **la misma**.

El punto débil es **esa primera vez**.

### Hombre en el medio (MITM)

Si un atacante controla la red (por ejemplo, un Wi-Fi falso), puede hacerse pasar por el servidor y presentarte **su** llave.

- Si aceptas sin comparar, te conectas al atacante.
- Por eso la huella oficial se reparte **por otro canal** (esta guía, el tablero). Comparar la huella convierte el "confío a ciegas" en "confío porque lo verifiqué".

Tu llave privada nunca sale de tu equipo, ni siquiera ante un MITM. Lo que sí podría hacer el atacante es entregarte un archivo falso. Por eso, además, verificarás el hash del libro en el paso 4.

---

## 4. Conectarte y descargar el libro

Reemplaza `est_nombre_apellido` por tu usuario:

```
sftp est_nombre_apellido@54.157.214.128
```

Si tu llave tiene passphrase, te la pedirá: es **la de tu llave**, no una contraseña del servidor.

Dentro de SFTP:

```
sftp> ls
libros
sftp> cd libros
sftp> ls -l
sftp> get libro.pdf
sftp> bye
```

El archivo queda en la carpeta desde donde ejecutaste `sftp`.

**Prueba (y entiende) las restricciones:**
- `put algo.txt` falla: el servidor está en **solo lectura**.
- `cd /etc` falla: estás en una **jaula (chroot)**. Para ti, `/` es la carpeta del taller.
- `ssh est_nombre_apellido@54.157.214.128` responde *"This service allows sftp connections only."*

### Verifica la integridad del libro

**Windows:**

```powershell
Get-FileHash .\libro.pdf -Algorithm SHA256
```

**macOS:**

```
shasum -a 256 libro.pdf
```

**Linux:**

```
sha256sum libro.pdf
```

Debe coincidir con este SHA-256:

```
67bc4507f7558c99f0ecc331651da224f0a73b2453fdd8ff5e67c8d4e56cc7dd
```

Es el hash del archivo oficial que publican los autores en `toc.cryptobook.us/book.pdf` y en `crypto.stanford.edu`; los dos sitios entregan exactamente el mismo archivo. Si no coincide, el archivo cambió en algún punto del camino.

---

## 5. Errores comunes y qué significan

### `Permission denied (publickey).`

El servidor no aceptó ninguna de tus llaves. Causas posibles:

- **Usuario mal escrito.** Revisa `est_nombre_apellido`.
- **Estás usando otra llave.** Indícala explícitamente:

  ```
  sftp -i ~/.ssh/id_ecdsa est_nombre_apellido@54.157.214.128
  ```

  En Windows: `-i $env:USERPROFILE\.ssh\id_ecdsa`
- **Generaste una llave nueva después de entregar la pública.** La que está en el servidor ya no corresponde a tu privada.
- **Tu llave fue revocada.**

Para ver el detalle: `sftp -v ...` (busca `Offering public key` y `Server accepts key`).

### `WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!` / `Host key verification failed.`

La llave del servidor **no es la que guardaste** en `known_hosts`. Puede ser:

- que el servidor se reinstaló, o
- **un ataque de hombre en el medio**.

**No lo ignores. No borres la línea "para que funcione".** Avisa al docente y compara de nuevo con las huellas oficiales. Solo si el docente confirma un cambio legítimo:

```
ssh-keygen -R 54.157.214.128
```

### `Too many authentication failures`

Tu cliente ofreció demasiadas llaves (por ejemplo, desde un agente SSH) y el servidor cortó. Usa solo la llave correcta:

```
sftp -o IdentitiesOnly=yes -i ~/.ssh/id_ecdsa est_nombre_apellido@54.157.214.128
```

### `WARNING: UNPROTECTED PRIVATE KEY FILE!` / `Permissions ... are too open`

Tu llave privada puede ser leída por otros usuarios de tu equipo, y SSH se niega a usarla.

- **macOS/Linux:** `chmod 600 ~/.ssh/id_ecdsa`
- **Windows:** la llave debe estar en tu carpeta `.ssh`, y solo tu usuario debe tener permisos sobre el archivo.

### `This service allows sftp connections only.`

No es un error: usaste `ssh` en vez de `sftp`. Tu cuenta solo permite SFTP.

### `Connection timed out` / `Connection refused`

No llegas al puerto 22 del servidor:

- el puerto puede estar cerrado fuera del horario del taller,
- tu red puede bloquearlo, o
- puedes tener mal la IP.

### `Enter passphrase for key ...`

Es la frase que pusiste al generar **tu** llave. Sirve para descifrarla localmente; **nunca viaja al servidor**.
