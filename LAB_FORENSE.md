# Laboratorio forense: evidencia, integridad y cadena de custodia

**Principios de Criptografía — Unidad 2**

Vas a actuar como analista forense. Recibes una imagen de disco (`caso.img`) de un
equipo bajo investigación. Tu trabajo:

1. **Descargar** la evidencia por SFTP.
2. **Verificar su integridad** con un hash SHA-256 (cadena de custodia).
3. **Extraer tres datos** sin modificar la imagen.

> Regla de oro del forense: **nunca se analiza el original en modo escritura.**
> Se verifica su hash y se trabaja sobre una copia de solo lectura. Si la evidencia
> cambia, aunque sea un bit, deja de ser válida ante un tribunal.

---

## 1. Descargar la evidencia

Usa tu usuario del taller (`est_nombre_apellido`):

```
sftp est_nombre_apellido@<IP>
sftp> cd libros
sftp> get caso.img
sftp> get caso.sha256
sftp> bye
```

---

## 2. Verificar la integridad (cadena de custodia)

El archivo `caso.sha256` contiene la **huella oficial** que el docente calculó al
sellar la evidencia. Comprueba que tu copia es idéntica.

El SHA-256 oficial de `caso.img` es:

```
46ae3118d18449c3b0031a0506e8ec9b0e330338135e68b13577f954553c5a02
```

**Windows (PowerShell):**

```powershell
Get-FileHash .\caso.img -Algorithm SHA256
```
Compara el resultado, a mano, con el contenido de `caso.sha256` (ábrelo con `notepad caso.sha256`).

**macOS:**

```
shasum -a 256 -c caso.sha256
```

**Linux:**

```
sha256sum -c caso.sha256
```

Debe responder `caso.img: OK`.

- **Si dice OK:** la evidencia llegó íntegra. Puedes analizarla.
- **Si dice FAILED:** el archivo se corrompió o fue alterado en el camino. **Detente**
  y vuelve a descargarlo. No analices evidencia que no pasó la verificación.

**Pregunta:** ¿por qué publicamos `caso.sha256` también en GitHub, por un canal
distinto al SFTP? Porque si un atacante pudiera alterar la imagen *y* el hash en el
mismo sitio, la verificación no serviría. Dos canales independientes lo dificultan.

---

## 3. Extraer la evidencia SIN alterar la imagen

No vamos a montar la imagen en modo escritura. La leemos tal cual, byte a byte.
Las herramientas de abajo **solo leen**; no modifican `caso.img`.

### Dato 1 — La cadena oculta (flag)

Hay un archivo oculto con una "flag". Búscala leyendo las cadenas de texto de la imagen:

**macOS / Linux:**

```
strings caso.img | grep "flag{"
```

**Windows (PowerShell):** usa el módulo del taller o abre la imagen con un visor hex;
si tienes WSL o Git Bash, el comando anterior funciona igual.

### Dato 2 — La IP a la que se exfiltraron datos

Los registros del sistema (`auth.log`) quedaron dentro de la imagen. Busca la línea
de la transferencia sospechosa:

```
strings caso.img | grep -i "scp"
```

Anota la **dirección IP** de destino.

### Dato 3 — El archivo que el atacante creyó borrar

En el equipo se creó un archivo, se usó y se **borró**. Pero en disco los datos
siguen ahí hasta que otro los sobrescribe: borrar no es destruir. Recupéralo:

```
strings caso.img | grep -i "exfiltrado"
```

Anota el **nombre del archivo** mencionado.

---

## 4. Comprueba que NO alteraste la evidencia

Vuelve a calcular el hash de `caso.img` después de tu análisis:

```
sha256sum -c caso.sha256        # macOS: shasum -a 256 -c caso.sha256
```

Debe seguir diciendo `OK`. Como solo usaste comandos de lectura (`strings`, `grep`),
el hash no cambió: **demostraste que tu análisis respetó la cadena de custodia.**

---

## Entrega

Reporta al docente, con una frase que explique cómo lo obtuviste:

| # | Hallazgo | Dónde estaba |
|---|---|---|
| 1 | La flag `flag{...}` | archivo oculto |
| 2 | La IP de exfiltración | `auth.log` |
| 3 | El nombre del archivo exfiltrado | fichero borrado, recuperado de disco |
| 4 | El SHA-256 de la imagen, antes y después | igual = cadena de custodia intacta |
