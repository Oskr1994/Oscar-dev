# tmux — Sesiones persistentes para trabajar por SSH

Guía de referencia para mantener vivo el trabajo en el VPS (Claude Code, despliegues,
migraciones de Odoo, scripts largos) aunque se cierre la terminal o se caiga el internet.

---

## 1. El problema y la solución

| Sin tmux | Con tmux |
|---|---|
| Abro `ssh` → ejecuto `claude` → el proceso es **hijo de la sesión SSH**. | Abro `ssh` → entro a tmux → ejecuto `claude` → el proceso es **hijo del servidor tmux**, que vive en el VPS. |
| Si cierro la laptop, se corta el wifi o cambio de red, SSH muere y **arrastra a Claude**. | Si se cae la conexión, tmux solo se queda "sin cliente". **Claude sigue trabajando**. |
| Vuelvo a conectar y perdí el contexto y el avance. | Vuelvo a conectar, hago `tmux attach` y **veo la pantalla tal como la dejé**. |

> Regla mental: **primero tmux, después claude**. Nunca al revés.

---

## 2. Instalación

| Sistema | Comando |
|---|---|
| Ubuntu / Debian | `sudo apt update && sudo apt install -y tmux` |
| CentOS / Rocky / Alma | `sudo dnf install -y tmux` |
| Alpine | `apk add tmux` |
| Verificar versión | `tmux -V` |

---

## 3. Conceptos básicos

| Concepto | Qué es | Analogía |
|---|---|---|
| **Servidor** | Proceso que corre en el VPS y sostiene todo. Arranca solo con la primera sesión. | El edificio |
| **Sesión** | Espacio de trabajo con nombre (ej. `claude`, `odoo`, `logs`). Es lo que sobrevive a la desconexión. | Una oficina |
| **Ventana** | Pestaña dentro de una sesión. Cada una con su propia terminal. | Un escritorio dentro de la oficina |
| **Panel** | División de una ventana (pantalla partida). | Dos monitores en el mismo escritorio |
| **Attach / Detach** | Conectarse o soltarse de una sesión **sin matarla**. | Entrar o salir de la oficina dejando la PC prendida |

---

## 4. La tecla prefijo

Todos los atajos internos empiezan con el **prefijo**, que por defecto es `Ctrl+b`.

Se escribe en dos tiempos: se suelta `Ctrl+b` y **luego** se presiona la tecla.

| Notación en esta guía | Cómo se teclea |
|---|---|
| `prefijo d` | `Ctrl+b`, soltar, luego `d` |
| `prefijo %` | `Ctrl+b`, soltar, luego `Shift+5` |

---

## 5. Comandos desde la terminal (fuera de tmux)

### 5.1 Crear y entrar

| Comando | Qué hace |
|---|---|
| `tmux` | Crea una sesión sin nombre (`0`, `1`, `2`…). No recomendado para trabajo largo. |
| `tmux new -s claude` | Crea la sesión `claude` y entra. |
| `tmux new -s claude -d` | Crea la sesión en segundo plano, **sin** entrar. |
| `tmux new -A -s claude` | **El más útil:** si `claude` existe se conecta, si no existe la crea. |
| `tmux new -s claude -c /ruta/proyecto` | Crea la sesión ya parada en ese directorio. |
| `tmux new -s deploy 'bash deploy.sh'` | Crea la sesión ejecutando directo ese comando. |

### 5.2 Listar y reconectar

| Comando | Qué hace |
|---|---|
| `tmux ls` | Lista las sesiones vivas y cuáles están `(attached)`. |
| `tmux attach -t claude` | Se conecta a la sesión `claude`. |
| `tmux a -t claude` | Igual que el anterior (forma corta). |
| `tmux a` | Se conecta a la última sesión usada. |
| `tmux attach -d -t claude` | Se conecta y **expulsa** a cualquier otro cliente pegado (útil tras una caída fantasma). |
| `tmux attach -r -t claude` | Se conecta en **solo lectura**: ves lo que pasa sin poder escribir ni romper nada. |
| `tmux has-session -t claude 2>/dev/null && echo existe` | Comprobar desde un script si la sesión existe. |

### 5.3 Administrar y cerrar

| Comando | Qué hace |
|---|---|
| `tmux rename-session -t 0 claude` | Le pone nombre a una sesión anónima. |
| `tmux kill-session -t claude` | Mata **esa** sesión y todo lo que corre dentro. |
| `tmux kill-server` | Mata **todas** las sesiones del usuario. Úsalo con cuidado. |
| `tmux ls \| grep -q claude \|\| tmux new -d -s claude` | Crea la sesión solo si no existe. |

### 5.4 Leer y guardar lo que pasó

| Comando | Qué hace |
|---|---|
| `tmux capture-pane -p -t claude` | Imprime lo visible del panel en la terminal actual. |
| `tmux capture-pane -p -S -100000 -t claude > ~/claude-log.txt` | Guarda todo el historial de la sesión en un archivo. |
| `tmux pipe-pane -t claude -o 'cat >> ~/claude-vivo.log'` | Empieza a grabar en vivo todo lo que salga en ese panel. |
| `tmux pipe-pane -t claude` | Detiene la grabación anterior. |

---

## 6. Atajos dentro de tmux

### 6.1 Sesiones

| Atajo | Qué hace |
|---|---|
| `prefijo d` | **Detach**: sale dejando todo corriendo. El atajo más importante de todos. |
| `prefijo s` | Lista interactiva de sesiones (flechas + `Enter` para saltar). |
| `prefijo $` | Renombrar la sesión actual. |
| `prefijo (` / `prefijo )` | Sesión anterior / siguiente. |
| `prefijo :` | Abre la línea de comandos de tmux (ej. `kill-session`). |
| `prefijo ?` | Muestra todos los atajos. Se sale con `q`. |

### 6.2 Ventanas (pestañas)

| Atajo | Qué hace |
|---|---|
| `prefijo c` | Crear ventana nueva. |
| `prefijo ,` | Renombrar la ventana actual (ej. `claude`, `logs`, `odoo`). |
| `prefijo n` / `prefijo p` | Ventana siguiente / anterior. |
| `prefijo 0` … `prefijo 9` | Saltar a la ventana por número. |
| `prefijo w` | Lista interactiva de ventanas. |
| `prefijo &` | Cerrar la ventana actual (pide confirmación). |

### 6.3 Paneles (pantalla partida)

| Atajo | Qué hace |
|---|---|
| `prefijo %` | Partir en vertical (izquierda / derecha). |
| `prefijo "` | Partir en horizontal (arriba / abajo). |
| `prefijo ←↑↓→` | Moverse entre paneles. |
| `prefijo o` | Saltar al siguiente panel. |
| `prefijo z` | **Zoom**: agranda el panel a pantalla completa. Se repite para volver. |
| `prefijo x` | Cerrar el panel actual. |
| `prefijo espacio` | Cambiar la distribución de los paneles. |
| `prefijo Ctrl+←↑↓→` | Redimensionar el panel (de a 1 celda). |
| `prefijo !` | Convertir el panel actual en una ventana propia. |
| `prefijo q` | Muestra el número de cada panel; teclea el número para saltar. |

### 6.4 Modo copia (para leer hacia arriba)

Dentro de tmux la rueda del mouse no hace scroll normal: hay que entrar al modo copia.

| Atajo | Qué hace |
|---|---|
| `prefijo [` | Entra al modo copia (ahí ya puedes subir con flechas, `PgUp`, rueda). |
| `q` o `Esc` | Sale del modo copia. |
| `Ctrl+u` / `Ctrl+d` | Media página arriba / abajo. |
| `g` / `G` | Ir al inicio / al final del historial (con `mode-keys vi`). |
| `/texto` + `Enter` | Buscar hacia atrás; `n` repite la búsqueda (con `mode-keys vi`). |
| `espacio` → `Enter` | Marcar selección y copiarla al portapapeles de tmux (con `mode-keys vi` la marca es `v` y copia `y`). |
| `prefijo ]` | Pegar lo copiado. |

> Para copiar al portapapeles **de tu PC**, mantén `Shift` mientras seleccionas con el
> mouse: eso salta tmux y usa la selección de tu terminal local.

---

## 7. Flujos de trabajo

### Flujo A — Lanzar una tarea larga con Claude y poder desconectarse

| # | Paso | Comando / acción |
|---|---|---|
| 1 | Entrar al VPS | `ssh usuario@mi-vps` |
| 2 | Abrir (o recuperar) la sesión de trabajo | `tmux new -A -s claude` |
| 3 | Ir al proyecto | `cd ~/proyectos/mi-app` |
| 4 | Lanzar Claude | `claude` |
| 5 | Darle la tarea larga | (escribir el prompt normalmente) |
| 6 | **Soltarse sin matar nada** | `Ctrl+b` luego `d` |
| 7 | Cerrar SSH tranquilo | `exit` |
| 8 | Volver más tarde | `ssh usuario@mi-vps` → `tmux a -t claude` |

Si la conexión se cae sola en el paso 5, no hay que hacer nada: el paso 6 ocurre
automáticamente y el trabajo continúa.

### Flujo B — Retomar después de una caída de conexión

| # | Situación | Comando |
|---|---|---|
| 1 | Reconectar al VPS | `ssh usuario@mi-vps` |
| 2 | Ver qué sesiones quedaron vivas | `tmux ls` |
| 3a | La sesión aparece libre | `tmux a -t claude` |
| 3b | La sesión aparece `(attached)` pero no eres tú (cliente fantasma de la caída) | `tmux attach -d -t claude` |
| 4 | Revisar lo que pasó mientras no estabas | `prefijo [` y subir con las flechas |
| 5 | Si la sesión ya no existe (se reinició el VPS) | `tmux new -A -s claude` y volver a lanzar `claude` |

### Flujo C — Varias tareas en paralelo

| # | Paso | Comando / acción |
|---|---|---|
| 1 | Entrar a la sesión | `tmux new -A -s trabajo` |
| 2 | Ventana 1 para Claude | `prefijo ,` → nombrarla `claude` → ejecutar `claude` |
| 3 | Ventana 2 para logs | `prefijo c` → `prefijo ,` → `logs` → `journalctl -fu odoo` |
| 4 | Ventana 3 para git / comandos sueltos | `prefijo c` → `prefijo ,` → `git` |
| 5 | Saltar entre ellas | `prefijo 1`, `prefijo 2`, `prefijo 3` |
| 6 | Ver Claude y los logs a la vez | en la ventana 1: `prefijo "` y en el panel nuevo `tail -f archivo.log` |
| 7 | Agrandar un panel un momento | `prefijo z` (y otra vez para volver) |

### Flujo D — Vigilar el avance desde otro dispositivo (celular, otra PC)

| # | Paso | Comando |
|---|---|---|
| 1 | Conectar por SSH desde el otro equipo | `ssh usuario@mi-vps` |
| 2 | Entrar en **solo lectura** para no escribir por accidente | `tmux attach -r -t claude` |
| 3 | Salir sin tocar nada | `prefijo d` |

> Dos clientes conectados a la misma sesión ven **lo mismo en vivo**. Sirve también para
> mostrarle a alguien lo que está corriendo.

### Flujo E — Cerrar el trabajo y limpiar

| # | Paso | Comando / acción |
|---|---|---|
| 1 | Guardar el historial si te interesa | `tmux capture-pane -p -S -100000 -t claude > ~/claude-log.txt` |
| 2 | Salir de Claude | `/exit` o `Ctrl+d` dentro de Claude |
| 3 | Cerrar la shell de la ventana | `exit` |
| 4 | Si quedaron sesiones colgadas | `tmux ls` y luego `tmux kill-session -t <nombre>` |
| 5 | Limpieza total (cuidado) | `tmux kill-server` |

---

## 8. Configuración recomendada (`~/.tmux.conf`)

```bash
# Ratón: scroll, cambiar de panel y redimensionar con clic
set -g mouse on

# Historial amplio (por defecto solo guarda 2000 líneas)
set -g history-limit 50000

# Numerar ventanas desde 1 y renumerar al cerrar alguna
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on

# Modo copia estilo vi: v para seleccionar, y para copiar
setw -g mode-keys vi

# Colores correctos dentro de tmux (para que Claude Code se vea bien)
set -g default-terminal "tmux-256color"
set -ga terminal-overrides ",xterm-256color:Tc"

# Esc instantáneo (Claude Code usa Esc para interrumpir)
set -sg escape-time 10

# Los paneles nuevos abren en el directorio actual
bind '"' split-window -v -c "#{pane_current_path}"
bind %  split-window -h -c "#{pane_current_path}"

# Recargar la configuración con prefijo + r
bind r source-file ~/.tmux.conf \; display "tmux.conf recargado"
```

| Línea | Por qué importa |
|---|---|
| `mouse on` | Permite hacer scroll con la rueda y cambiar de panel con clic. |
| `history-limit 50000` | Sin esto se pierden las respuestas largas de Claude al subir. |
| `mode-keys vi` | Hace el modo copia navegable y buscable (`/`, `n`, `g`, `G`). |
| `default-terminal` + `terminal-overrides` | Evita colores rotos o texto ilegible en la interfaz de Claude. |
| `escape-time 10` | Sin esto, `Esc` tarda ~0.5 s y se siente que Claude no responde. |
| `split-window -c` | Abre el panel nuevo en la misma carpeta del proyecto. |

Aplicar los cambios:

| Situación | Comando |
|---|---|
| Estando dentro de tmux | `prefijo r` (con el bind de arriba) |
| Desde fuera | `tmux source-file ~/.tmux.conf` |
| Si algo quedó raro | `tmux kill-server` y volver a abrir |

> Si el VPS es antiguo y `tmux-256color` da error (`missing or unsuitable terminal`),
> cambia esa línea por `set -g default-terminal "screen-256color"`.

---

## 9. Automatizar el arranque

### 9.1 Alias útiles (`~/.bashrc` del VPS)

```bash
alias tl='tmux ls'
alias ta='tmux attach -t'
alias tn='tmux new -A -s'
alias tk='tmux kill-session -t'
alias tc='tmux new -A -s claude'
```

Recargar con `source ~/.bashrc`.

### 9.2 Entrar siempre a tmux al conectarse por SSH

Al final del `~/.bashrc` del VPS:

```bash
if command -v tmux >/dev/null && [ -n "$PS1" ] && [ -z "$TMUX" ] && [ -n "$SSH_CONNECTION" ]; then
  tmux new-session -A -s principal
fi
```

| Condición | Para qué está |
|---|---|
| `command -v tmux` | No falla si tmux no está instalado. |
| `[ -n "$PS1" ]` | Solo en shells interactivas (no rompe `scp` ni `rsync`). |
| `[ -z "$TMUX" ]` | Evita abrir tmux dentro de tmux. |
| `[ -n "$SSH_CONNECTION" ]` | Solo al entrar por SSH, no en la consola local. |

### 9.3 Menos desconexiones desde tu PC (`~/.ssh/config` local)

```
Host mi-vps
    HostName 123.45.67.89
    User oscar
    ServerAliveInterval 60
    ServerAliveCountMax 3
    TCPKeepAlive yes
```

Con eso conectas escribiendo solo `ssh mi-vps`, y el `ServerAliveInterval` evita que
la conexión se corte sola por inactividad.

---

## 10. Notas específicas al usar Claude Code dentro de tmux

| Situación | Qué hacer |
|---|---|
| Lanzar Claude de forma segura | `tmux new -A -s claude` y recién ahí `claude`. |
| Irse a media tarea | `prefijo d`. Claude sigue trabajando y escribiendo archivos. |
| Leer una respuesta larga que se fue hacia arriba | `prefijo [`, subir con flechas o rueda, `q` para salir. |
| Buscar algo en lo que ya pasó | `prefijo [` → `/texto` → `Enter` → `n` para repetir. |
| Guardar la conversación completa en un archivo | `tmux capture-pane -p -S -100000 -t claude > ~/claude-log.txt` |
| `Esc` (interrumpir a Claude) se siente lento | Falta `set -sg escape-time 10` en `~/.tmux.conf`. |
| Pegar un prompt largo | Pegar normal; si tu terminal lo parte raro, usa `prefijo ]` tras copiar dentro de tmux. |
| Dejar corriendo y revisar desde el celular | `ssh` al VPS y `tmux attach -r -t claude` (solo lectura). |

> **Límite importante:** tmux sobrevive a la caída de SSH, pero **no a un reinicio del VPS**.
> Si el servidor se reinicia, las sesiones se pierden y hay que volver a lanzarlas.

---

## 11. Solución de problemas

| Síntoma | Causa | Solución |
|---|---|---|
| `no server running on /tmp/tmux-1000/default` | No hay ninguna sesión viva. | `tmux new -s claude` |
| `duplicate session: claude` | Ya existe esa sesión. | `tmux a -t claude` o usar `tmux new -A -s claude` |
| `sessions should be nested with care, unset $TMUX` | Ya estás dentro de tmux. | Usar `prefijo c` para otra ventana, o `prefijo d` y volver a entrar. |
| La sesión dice `(attached)` pero no la estás viendo | Cliente fantasma de una caída anterior. | `tmux attach -d -t claude` |
| La rueda del mouse manda el historial de bash en vez de hacer scroll | Falta el modo ratón. | `set -g mouse on` en `~/.tmux.conf` |
| Colores raros, cuadros o texto ilegible en Claude | `TERM` mal configurado. | Ajustar `default-terminal` y `terminal-overrides` (sección 8). |
| `Esc` tarda en responder | `escape-time` por defecto (500 ms). | `set -sg escape-time 10` |
| `Ctrl+b` choca con un atajo de tu terminal | Prefijo por defecto ocupado. | Cambiarlo: `set -g prefix C-a` + `unbind C-b` + `bind C-a send-prefix` (ojo: `Ctrl+a` es "inicio de línea" en bash). |
| Perdí todo tras reiniciar el VPS | tmux no persiste reinicios. | Volver a crear las sesiones; para tareas críticas usar un servicio `systemd` en lugar de tmux. |
| No puedo copiar texto a mi PC | El mouse lo captura tmux. | Mantener `Shift` mientras seleccionas, o volcar con `capture-pane`. |

---

## 12. Chuleta rápida

Lo que se usa el 90% del tiempo:

| Acción | Qué tecleo |
|---|---|
| Abrir o recuperar mi sesión de trabajo | `tmux new -A -s claude` |
| Soltarme dejando todo corriendo | `Ctrl+b` luego `d` |
| Ver qué hay vivo | `tmux ls` |
| Volver a entrar | `tmux a -t claude` |
| Nueva pestaña | `Ctrl+b` luego `c` |
| Saltar de pestaña | `Ctrl+b` luego `1` / `2` / `3` |
| Partir la pantalla | `Ctrl+b` luego `%` o `"` |
| Subir a leer | `Ctrl+b` luego `[` (salir con `q`) |
| Matar una sesión | `tmux kill-session -t claude` |
