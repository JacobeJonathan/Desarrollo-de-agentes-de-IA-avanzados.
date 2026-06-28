# Claude Code + tmux: Equipos de Sub-Agentes en Ubuntu / Linux

> **Objetivo:** Hacer correr múltiples agentes Claude Code en paralelo, cada uno en su propio panel tmux visible, en cualquier sistema Linux (Ubuntu, Debian, Fedora, Arch, etc.).

---

## ¿Por qué tmux?

Claude Code detecta automáticamente si está dentro de una sesión tmux y activa el modo **split-pane**: cada sub-agente (teammate) aparece en su propio panel visible. Sin tmux, los agentes trabajan en segundo plano sin que puedas observarlos. tmux también da persistencia (las sesiones sobreviven desconexiones SSH), historial de scroll, y la capacidad de coordinar múltiples sesiones de agentes simultáneamente.

---

## Requisitos previos

- Ubuntu 20.04+ / Debian 11+ / cualquier distro Linux moderna
- Suscripción Claude Pro o Max (con acceso a Opus 4.6)
- Acceso a terminal con permisos de `sudo`

---

## Paso 1 — Instalar dependencias del sistema

### Ubuntu / Debian

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y tmux git curl build-essential
```

### Fedora / RHEL

```bash
sudo dnf install -y tmux git curl gcc gcc-c++ make
```

### Arch Linux

```bash
sudo pacman -Syu tmux git curl base-devel
```

Verifica la instalación de tmux:

```bash
tmux -V   # debe mostrar tmux 3.x
```

---

## Paso 2 — Instalar Node.js (requerido por Claude Code)

Usando **nvm** (recomendado para mantener múltiples versiones):

```bash
# Instalar nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# Recargar el shell
source ~/.bashrc   # o source ~/.zshrc si usas Zsh

# Instalar Node.js LTS
nvm install --lts

# Verificar
node --version   # debe mostrar v22.x o similar
npm --version
```

---

## Paso 3 — Instalar Claude Code

```bash
npm install -g @anthropic-ai/claude-code

# Verificar instalación
claude --version
```

Si es tu primera vez usando Claude Code, inicia la autenticación:

```bash
claude
```

Sigue las instrucciones en pantalla para conectar tu cuenta Anthropic.

---

## Paso 4 — Configurar tmux para agentes

Crea el archivo `~/.tmux.conf` optimizado para sesiones de agentes:

```bash
cat > ~/.tmux.conf << 'EOF'
# ──────────────────────────────────────────
# CONFIGURACIÓN BÁSICA
# ──────────────────────────────────────────

# Soporte de ratón: clic en paneles, redimensionar, scroll
set -g mouse on

# Historial amplio para logs largos de agentes
set -g history-limit 50000

# Colores de 256 bits
set -g default-terminal "tmux-256color"

# Actualizar la barra de estado cada 5 segundos
set -g status-interval 5

# Renumerar ventanas automáticamente
set -g renumber-windows on

# Cerrar panel automáticamente cuando el proceso termina
set -g remain-on-exit off

# ──────────────────────────────────────────
# ATAJOS DE TECLADO
# ──────────────────────────────────────────

# Dividir paneles y mantener el directorio de trabajo actual
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"

# Navegación estilo vim entre paneles
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# Recargar config sin reiniciar
bind r source-file ~/.tmux.conf \; display "~/.tmux.conf recargado"

# ──────────────────────────────────────────
# ESTÉTICA DE LA BARRA DE ESTADO
# ──────────────────────────────────────────

set -g status-bg colour235
set -g status-fg white
set -g pane-active-border-style fg=green
EOF
```

Para aplicar la config dentro de una sesión tmux activa:

```bash
tmux source-file ~/.tmux.conf
```

---

## Paso 5 — Habilitar Agent Teams (experimental)

Los equipos de agentes son experimentales y vienen desactivados por defecto. Edita `~/.claude/settings.json`:

```bash
# Si la carpeta no existe, créala
mkdir -p ~/.claude

# Editar settings.json
nano ~/.claude/settings.json
```

Agrega o modifica el bloque `env`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Guarda con `Ctrl+O`, `Enter`, `Ctrl+X`.

También puedes hacerlo con un solo comando:

```bash
# Si el archivo no existe o quieres crearlo desde cero
echo '{"env": {"CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"}}' > ~/.claude/settings.json
```

---

## Paso 6 — Iniciar una sesión tmux y lanzar Claude Code

Crea una sesión tmux apuntando a tu proyecto:

```bash
# Opción A — navegar primero
cd ~/dev/mi-proyecto
tmux new-session -s claude -c .

# Opción B — pasar la ruta directamente
tmux new-session -s claude -c ~/dev/mi-proyecto

# Opción C — sesión con nombre descriptivo del proyecto
tmux new-session -s "car-sistema" -c ~/proyectos/car-sistema
```

Una vez dentro de la sesión tmux (verás la barra verde en la parte inferior), inicia Claude Code:

```bash
claude
```

Claude Code detecta automáticamente que está dentro de tmux y usará el modo split-pane cuando se spawnen sub-agentes. No necesitas configuración adicional.

---

## Paso 7 — Crear tu equipo de agentes

### Pedirle a Claude que cree un equipo en lenguaje natural

```
Crea un equipo de agentes para analizar el repositorio. Asigna:
- Un agente (Investigador) para analizar la estructura del proyecto
- Un agente (Implementador) para proponer mejoras al código
- Un agente (Revisor) para validar la calidad del código
Usa el modelo Haiku 4.5 para cada agente.
```

Los paneles tmux se dividirán automáticamente, uno por agente.

### Prompt de prueba rápida

```
Crea un equipo de agentes para traducir "Hola mundo" al inglés, francés y alemán.
Un agente por idioma. Reporta los resultados en un archivo resultado.md.
Usa Haiku 4.5 para cada agente.
```

### Script de lanzamiento automatizado

Puedes crear un script para iniciar siempre con la misma configuración:

```bash
cat > ~/bin/claude-team.sh << 'EOF'
#!/bin/bash
SESSION="claude-team"
PROJECT_DIR="${1:-$HOME/dev}"

# Terminar sesión previa si existe
tmux kill-session -t $SESSION 2>/dev/null

# Crear nueva sesión
tmux new-session -d -s $SESSION -c "$PROJECT_DIR"

# Ir dentro y lanzar Claude Code
tmux send-keys -t $SESSION "claude" Enter

# Conectarse a la sesión
tmux attach -t $SESSION
EOF

chmod +x ~/bin/claude-team.sh
```

Uso:

```bash
claude-team.sh ~/proyectos/mi-repo
```

---

## Atajos de teclado tmux esenciales

El prefijo por defecto es `Ctrl+B`. Presiónalo, suéltalo, luego la segunda tecla.

| Acción | Teclas |
|--------|--------|
| Dividir panel vertical | `Ctrl+B` luego `\|` |
| Dividir panel horizontal | `Ctrl+B` luego `-` |
| Navegar entre paneles | `Ctrl+B` luego `H` / `J` / `K` / `L` (vim) o flechas |
| Cerrar panel actual | `Ctrl+B` luego `X` |
| Nueva ventana | `Ctrl+B` luego `C` |
| Renombrar ventana | `Ctrl+B` luego `,` |
| Listar ventanas | `Ctrl+B` luego `W` |
| Modo scroll/copia | `Ctrl+B` luego `[` → flechas para mover, `Q` para salir |
| Buscar en historial | `Ctrl+B` luego `[` luego `Ctrl+S` para buscar |
| Desconectarse (sin cerrar sesión) | `Ctrl+B` luego `D` |
| Matar ventana actual | `Ctrl+B` luego `&` |
| Recargar config | `Ctrl+B` luego `R` (con el atajo configurado arriba) |

---

## Gestión de sesiones

```bash
# Ver todas las sesiones activas
tmux ls

# Reconectarse a una sesión existente
tmux attach -t claude

# Reconectarse a la última sesión activa
tmux attach

# Matar una sesión específica
tmux kill-session -t claude

# Matar todas las sesiones
tmux kill-server

# Ver sesiones y ventanas activas (dentro de tmux)
# Ctrl+B luego S
```

---

## Panel de agentes en modo in-process

Si lanzas Claude Code sin estar dentro de tmux, los agentes corren en **modo in-process**:

- Aparecen en el **panel de agentes** debajo del prompt de entrada.
- `↑` / `↓` para seleccionar un agente.
- `Enter` para ver su output y escribirle directamente.
- Un agente que desaparece no se detuvo — se ocultó tras 30 segundos de inactividad. Escribe su nombre para traerlo de vuelta.
- Para forzar el modo split-pane: lanza `claude` desde dentro de una sesión tmux.

---

## Estructura recomendada de CLAUDE.md

Crea un archivo `CLAUDE.md` en la raíz del proyecto antes de lanzar el equipo de agentes:

```markdown
# Proyecto: [Nombre del Proyecto]

## Descripción
[Descripción breve del proyecto]

## Estructura de módulos y responsabilidades
- `src/db/`       → Agente DB (solo modifica esta carpeta)
- `src/api/`      → Agente API (solo modifica esta carpeta)
- `src/frontend/` → Agente Frontend (solo modifica esta carpeta)
- `tests/`        → Agente QA (solo modifica esta carpeta)

## Reglas del equipo
1. Cada agente trabaja SOLO en su módulo asignado
2. El archivo de salida compartido es `docs/reporte-final.md`
3. Nunca modificar archivos fuera del módulo propio
4. Documentar cada cambio con un comentario explicativo

## Convenciones de código
- Lenguaje: [Python / JS / PHP / etc.]
- Estilo: [PEP8 / Airbnb / PSR-12]
```

---

## Cuándo usar Agent Teams vs Sub-agentes

| Situación | Recomendación |
|-----------|---------------|
| 3+ módulos independientes que pueden construirse en paralelo | **Agent Teams con tmux** |
| Tarea puntual: investigación, verificación | **Sub-agente simple (inline)** |
| Módulos que comparten los mismos archivos | **Sesión única** (evita conflictos) |
| Proyecto pequeño (1-2 módulos) | **Sesión única de Claude Code** |
| Proyecto grande (5+ módulos) | **4-5 agentes + 1 lead** (retorno decreciente más allá) |

---

## Configuración avanzada: tmux con plugins (opcional)

Para una experiencia más visual, instala el gestor de plugins TPM:

```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

Agrega al final de `~/.tmux.conf`:

```bash
# Plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-resurrect'   # guardar/restaurar sesiones
set -g @plugin 'tmux-plugins/tmux-cpu'          # CPU y RAM en la barra

# Inicializar TPM (siempre al final del archivo)
run '~/.tmux/plugins/tpm/tpm'
```

Dentro de tmux, presiona `Ctrl+B` luego `I` (mayúscula) para instalar los plugins.

---

## Solución de problemas comunes

**Los agentes no abren paneles separados**
→ Verifica que estás dentro de tmux antes de lanzar `claude`:
```bash
echo $TMUX   # debe mostrar algo; si está vacío, no estás en tmux
```

**`command not found: claude`**
→ El PATH de nvm puede no estar cargado. Agrega a `~/.bashrc`:
```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
```
Luego: `source ~/.bashrc && claude --version`

**Agent Teams no se activa**
→ Verifica `~/.claude/settings.json` y que la variable esté correctamente establecida:
```bash
cat ~/.claude/settings.json
```

**Sesión tmux persistente después de que Claude terminó**
```bash
tmux ls                          # listar sesiones
tmux kill-session -t claude      # matar la sesión específica
```

**Alto consumo de memoria con múltiples agentes**
→ Monitorea con `htop` desde otro panel tmux (`Ctrl+B` luego `|` para dividir, luego `htop`).

**Agentes se detienen al encontrar errores**
→ Selecciona el agente en el panel, presiona `Enter` para ver su output y decide si continuar:
```
Continúa con la tarea, el error anterior fue ignorado.
```
