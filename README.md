# Real timestamp via HTTP `Date` header (no NTP, no local clock trust)

*Leer en español más abajo / Spanish below.*

## English

### Why

Sometimes you need a timestamp you can trust without depending on:

- The local system clock (which can be wrong, unsynced, or manually changed).
- NTP servers (often blocked by restrictive network/firewall/proxy setups, e.g. sandboxed environments that only allow HTTPS to a fixed list of domains).

This one-liner gets the current time straight from an HTTP `Date` response header — every HTTP/1.1 server is required to send one, and `github.com` is reliable, fast, and usually reachable even from locked-down networks.

### The command

```bash
RAW_DATE=$(curl -sI https://github.com | grep -i '^date:' | sed 's/[Dd]ate: //' | tr -d '\r') \
  && EPOCH=$(TZ="UTC" date -d "$RAW_DATE" +"%s") \
  && TZ="Europe/Madrid" date -d "@$EPOCH" +"%Y-%m-%d %H:%M:%S GMT%z" \
  | sed -E 's/GMT\+0([0-9])00/GMT+\1/; s/GMT-0([0-9])00/GMT-\1/'
```

Example output:

```
2026-08-19 12:26:27 GMT+2
```

### How it works

1. `curl -sI` sends a HEAD request to `github.com` and captures only the response headers.
2. `grep`/`sed` extract the `Date:` header value (an HTTP-date, always in GMT/UTC).
3. `date -d "$RAW_DATE" +"%s"` (GNU `date`) converts that string to a Unix epoch.
4. A second `date` call converts the epoch to any target timezone (`Europe/Madrid` here — change to whatever `TZ` you need).
5. The final `sed` just cosmetically shortens `GMT+0200` to `GMT+2` (drop the leading zero and trailing `00`).

### Requirements

- GNU `date` (Linux). **This will NOT work as-is on macOS/BSD `date`**, which uses `-j -f` and `-r` instead of `-d`. A BSD-compatible version is left as an exercise (or open an issue/PR).
- `curl` reachable to `github.com` over HTTPS (port 443 only — no NTP/UDP needed).

### Limitations

- Precision is to the second, and depends on network latency to GitHub (typically well under a second of drift for this use case).
- If `github.com` is unreachable, swap in any other HTTPS server you trust — any HTTP/1.1 response has a `Date` header.

### Motivation

Built for an AI assistant (Claude) running in a sandboxed shell with an egress allowlist that includes `github.com` but no NTP access — this was the simplest reliable way to get a real, current timestamp instead of guessing or trusting a possibly stale system clock.

---

## Español

### Por qué

A veces necesitas un timestamp fiable sin depender de:

- El reloj local del sistema (puede estar mal, desincronizado o modificado a mano).
- Servidores NTP (a menudo bloqueados por firewalls/proxies restrictivos, por ejemplo entornos sandbox que solo permiten HTTPS a una lista fija de dominios).

Este comando obtiene la hora actual directamente de la cabecera HTTP `Date` de una respuesta — todo servidor HTTP/1.1 está obligado a enviarla, y `github.com` es fiable, rápido y normalmente accesible incluso desde redes muy restringidas.

### El comando

```bash
RAW_DATE=$(curl -sI https://github.com | grep -i '^date:' | sed 's/[Dd]ate: //' | tr -d '\r') \
  && EPOCH=$(TZ="UTC" date -d "$RAW_DATE" +"%s") \
  && TZ="Europe/Madrid" date -d "@$EPOCH" +"%Y-%m-%d %H:%M:%S GMT%z" \
  | sed -E 's/GMT\+0([0-9])00/GMT+\1/; s/GMT-0([0-9])00/GMT-\1/'
```

Salida de ejemplo:

```
2026-08-19 12:26:27 GMT+2
```

### Cómo funciona

1. `curl -sI` hace una petición HEAD a `github.com` y captura solo las cabeceras de respuesta.
2. `grep`/`sed` extraen el valor de la cabecera `Date:` (una fecha HTTP, siempre en GMT/UTC).
3. `date -d "$RAW_DATE" +"%s"` (GNU `date`) convierte ese texto a epoch Unix.
4. Una segunda llamada a `date` convierte el epoch a cualquier zona horaria de destino (`Europe/Madrid` aquí — cámbiala por la `TZ` que necesites).
5. El `sed` final solo acorta estéticamente `GMT+0200` a `GMT+2` (quita el cero inicial y los `00` finales).

### Requisitos

- `date` de GNU (Linux). **No funciona tal cual en macOS/BSD `date`**, que usa `-j -f` y `-r` en vez de `-d`. Una versión compatible con BSD queda como ejercicio (o abre un issue/PR).
- `curl` con acceso a `github.com` por HTTPS (solo puerto 443, no hace falta NTP/UDP).

### Limitaciones

- Precisión al segundo, y depende de la latencia de red hasta GitHub (normalmente muy por debajo de un segundo de desviación para este uso).
- Si `github.com` no es accesible, se puede usar cualquier otro servidor HTTPS de confianza — cualquier respuesta HTTP/1.1 trae cabecera `Date`.

### Motivación

Creado para un asistente de IA (Claude) ejecutándose en una shell en sandbox con una lista blanca de salida que incluye `github.com` pero no acceso NTP — esta fue la forma más simple y fiable de obtener un timestamp real y actual, en vez de suponerlo o fiarse de un reloj de sistema posiblemente desincronizado.


