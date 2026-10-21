# Laboratorio forense II: reconstruir un incidente

**Principios de Criptografía — Unidad 2**

Eres el analista. Un servidor fue comprometido y recibes una imagen de disco
(`incidente.img`) con los registros del equipo. Tu misión: **reconstruir la
línea de tiempo del ataque** a partir de la evidencia, sin alterarla.

> Igual que en el primer lab: se verifica el hash y se analiza en **solo
> lectura**. La evidencia no se monta en escritura.

---

## 1. Descargar la evidencia

```
sftp est_nombre_apellido@54.157.214.128
sftp> cd libros
sftp> get incidente.img
sftp> get incidente.sha256
sftp> bye
```

## 2. Verificar integridad

El SHA-256 oficial de `incidente.img` es:

```
15f22f7c3368968c08b42f500ee1bf4a438bb80018a5aebd1f2831d34a000387
```

Compáralo (debe decir `OK`):

- **Windows:** `Get-FileHash .\incidente.img -Algorithm SHA256`
- **macOS:** `shasum -a 256 -c incidente.sha256`
- **Linux:** `sha256sum -c incidente.sha256`

Si falla, vuelve a descargar. No analices evidencia que no verifica.

---

## 3. Analizar con Autopsy (recomendado)

[Autopsy](https://www.autopsy.com/download/) es gratuito (Windows/Mac/Linux) y
hace lo mismo que EnCase/FTK para este nivel.

1. **New Case** → nombre del caso → tu nombre como examinador.
2. **Add Data Source** → *Disk Image or VM File* → selecciona `incidente.img`.
3. Deja correr los *Ingest Modules* (Hash Lookup, Keyword Search, File Type).
4. Explora en el árbol de la izquierda:
   - **File System → /var/log/** → `auth.log`, `apache2/access.log`
   - **/etc/passwd**
   - **/home/devops/** → archivos ocultos (empiezan por `.`)
   - **Deleted Files** → lo que se borró pero sigue en disco.

> Autopsy abre la imagen en solo lectura: es imposible alterarla desde la interfaz.

### Alternativa por terminal (The Sleuth Kit)

```
fls -r -p incidente.img        # lista archivos, incluidos borrados (marcados con *)
icat incidente.img <inode>     # muestra el contenido de un archivo por su inodo
strings incidente.img | grep -Ei "failed password|accepted|scp|flag\{"
```

---

## 4. Lo que debes responder

Reconstruye la historia y entrega:

| # | Pregunta | Dónde buscar |
|---|---|---|
| 1 | ¿Desde qué **IP** se hizo la fuerza bruta y el acceso exitoso? | `auth.log` |
| 2 | ¿A qué hora logró entrar el atacante (`Accepted password`)? | `auth.log` |
| 3 | ¿Qué **usuario de persistencia** creó? | `auth.log` y `/etc/passwd` |
| 4 | ¿Qué **archivo** exfiltró y a qué **IP**? | `auth.log`, `.bash_history` |
| 5 | ¿Cuál es la **flag**? | `/home/devops/.bash_history` |
| 6 | El **SHA-256** de la imagen, antes y después de tu análisis | igual = cadena de custodia intacta |

**Pregunta de cierre:** el atacante ejecutó `history -c` para borrar su rastro.
¿Por qué los investigadores igual pudieron leer sus comandos? (Pista: el archivo
en disco y lo que `history -c` borra no son lo mismo.)

---

## 5. Confirma que no alteraste la evidencia

```
sha256sum -c incidente.sha256     # macOS: shasum -a 256 -c incidente.sha256
```

Debe seguir diciendo `OK`. Como solo leíste, el hash no cambió.
