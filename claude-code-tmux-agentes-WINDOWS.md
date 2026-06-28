# Claude Code + tmux: Equipos de Sub-Agentes en Windows (WSL2)

> **Objetivo:** Hacer correr múltiples agentes Claude Code en paralelo, cada uno en su propio panel tmux visible, usando Windows 10/11 con WSL2.

---

## ¿Por qué necesitas WSL2 + tmux?

Claude Code detecta automáticamente si está dentro de una sesión tmux y activa el modo **split-pane**: cada sub-agente (teammate) aparece en su propio panel visible. Sin tmux, los agentes trabajan en segundo plano sin que puedas verlos. El modo split-pane **no funciona** en Windows Terminal nativo, VS Code integrado ni Ghostty — requiere tmux vía WSL2.

---

## Requisitos previos

- Windows 10 (build 19041+) o Windows 11
- Suscripción Claude Pro o Max (con acceso a Opus 4.6)
- Conexión a internet

---

## Paso 1 — Activar WSL2 e instalar Ubuntu

Abre **PowerShell como Administrador** y ejecuta:

```powershell
wsl --install
```

Reinicia cuando se te pida. Ubuntu terminará su configuración en el primer lanzamiento: crea tu usuario y contraseña de Linux cuando se te solicite.

Si ya tienes WSL1, actualiza a WSL2:

```powershell
wsl --set-default-version 2
```

Verifica la versión activa:

```powershell
wsl -l -v
```

---

## Paso 2 — Instalar dependencias dentro de WSL (Ubuntu)

Abre tu terminal Ubuntu y ejecuta:

```bash
# Actualizar paquetes
sudo apt update && sudo apt upgrade -y

# Instalar tmux y herramientas de compilación
sudo apt install -y tmux git curl build-essential

# Instalar Node.js via nvm (requerido por Claude Code)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc
nvm install --lts
```

Verifica las instalaciones:

```bash
tmux -V        # debe mostrar tmux 3.x
node --version  # debe mostrar v22.x o similar
```

---

## Paso 3 — Instalar Claude Code

```bash
npm install -g @anthropic-ai/claude-code
claude --version   # verifica que instaló correctamente
```

Si es tu primera vez, inicia sesión:

```bash
claude
```

Sigue las instrucciones en pantalla para autenticar con tu cuenta Anthropic.

---

## Paso 4 — Configurar tmux para agentes

Crea el archivo de configuración `~/.tmux.conf`:

```bash
cat > ~/.tmux.conf << 'EOF'
# Soporte de ratón: clic en paneles, redimensionar, scroll
set -g mouse on

# Historial amplio para logs de agentes
set -g history-limit 50000

# Colores correctos
set -g default-terminal "tmux-256color"

# Cerrar panel automáticamente cuando el proceso termina
set -g remain-on-exit off

# Dividir paneles y mantener el directorio actual
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"

# Navegación estilo vim entre paneles
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# Actualizar barra de estado cada 5 segundos
set -g status-interval 5

# Intervalo de refresco y numeración de ventanas
set -g renumber-windows on
EOF
```

> **Nota:** No ejecutes `tmux source-file ~/.tmux.conf` todavía; ese comando solo funciona dentro de una sesión tmux activa. La config se aplicará automáticamente en el Paso 6.

---

## Paso 5 — Sincronizar configuración `.claude` entre Windows y WSL (opcional pero recomendado)

Claude Code guarda agentes, skills, equipos y configuración en `~/.claude`. Para no mantener dos configs separadas, haz un symlink de WSL hacia la carpeta de Windows:

```bash
# Elimina la carpeta ~/.claude vacía que creó WSL
rm -rf ~/.claude

# Crea el symlink (reemplaza "TuUsuarioWindows" con tu usuario real)
ln -s /mnt/c/Users/TuUsuarioWindows/.claude ~/.claude

# Verifica que funcionó — debes ver agents, skills, settings.json, etc.
ls ~/.claude
```

---

## Paso 6 — Habilitar Agent Teams (experimental)

Los equipos de agentes son experimentales y vienen desactivados por defecto. Edita `~/.claude/settings.json`:

```bash
# Si el archivo no existe, créalo
cat ~/.claude/settings.json 2>/dev/null || echo '{}' > ~/.claude/settings.json
```

Agrega el bloque `env`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Puedes editar con nano:

```bash
nano ~/.claude/settings.json
```

---

## Paso 7 — Iniciar tmux y lanzar Claude Code

Crea una sesión tmux apuntando a tu proyecto:

```bash
# Opción A — navegar primero
cd /mnt/c/dev/github/mi-proyecto
tmux new-session -s claude -c .

# Opción B — pasar la ruta directamente
tmux new-session -s claude -c /mnt/c/dev/github/mi-proyecto
```

> **Consejo de rendimiento:** El acceso a archivos a través de `/mnt/c/` es más lento que rutas Linux nativas. Para trabajo intensivo, clona el repo dentro del filesystem WSL: `~/dev/mi-proyecto`.

Ya dentro de la sesión tmux, inicia Claude Code:

```bash
claude
```

Claude Code detecta automáticamente que está dentro de tmux y usará el modo split-pane cuando se spawnen sub-agentes.

---

## Paso 8 — Crear tu equipo de agentes

### Pedirle a Claude que cree un equipo

Una vez dentro de Claude Code, escribe en lenguaje natural:

```
Crea un equipo de agentes para analizar el repositorio. Asigna:
- Un agente para revisar la arquitectura del backend
- Un agente para revisar el frontend
- Un agente para revisar las pruebas
Usa el modelo Haiku 4.5 para cada agente.
```

Verás tu ventana tmux dividirse automáticamente en múltiples paneles, uno por agente.

### Prompt de prueba mínimo

```
Crea un equipo de agentes para traducir "Hola mundo" al inglés, francés y alemán.
Asigna un agente por idioma. Reporta los resultados.
Usa el modelo Haiku 4.5 para cada agente.
```

---

## Atajos de teclado tmux esenciales

El prefijo por defecto es `Ctrl+B`. Presiónalo, suéltalo, luego la segunda tecla.

| Acción | Teclas |
|--------|--------|
| Dividir panel vertical | `Ctrl+B` luego `\|` |
| Dividir panel horizontal | `Ctrl+B` luego `-` |
| Navegar entre paneles | `Ctrl+B` luego flecha (↑↓←→) |
| Cerrar panel actual | `Ctrl+B` luego `X` |
| Nueva ventana | `Ctrl+B` luego `C` |
| Listar ventanas | `Ctrl+B` luego `W` |
| Modo copia/scroll | `Ctrl+B` luego `[` → navega con flechas, `Q` para salir |
| Desconectarse (sin cerrar) | `Ctrl+B` luego `D` |
| Recargar config tmux | `Ctrl+B` luego `:source-file ~/.tmux.conf` |

---

## Gestión de sesiones

```bash
# Ver sesiones activas
tmux ls

# Reconectarse a una sesión
tmux attach -t claude

# Matar una sesión
tmux kill-session -t claude

# Si tmux quedó colgado después de que Claude terminó
tmux kill-server
```

---

## Panel de agentes (in-process mode)

Si no estás dentro de tmux o prefieres el modo integrado:

- Los agentes aparecen en el **panel de agentes** debajo del prompt.
- Usa `↑` y `↓` para seleccionar un agente.
- Presiona `Enter` para ver su output.
- Un agente que desaparece no se detuvo — se ocultó después de 30 segundos de inactividad. Escribe su nombre para traerlo de vuelta.

---

## Estructura recomendada de CLAUDE.md

Crea un archivo `CLAUDE.md` en la raíz del proyecto antes de lanzar el equipo:

```markdown
# Proyecto: [Nombre del Proyecto]

## Estructura de módulos
- `src/db/` → responsabilidad del Agente DB
- `src/api/` → responsabilidad del Agente API
- `src/frontend/` → responsabilidad del Agente Frontend

## Reglas del equipo
- Cada agente trabaja SOLO en su módulo asignado
- El archivo de salida compartido es `docs/resultado.md`
- No modificar archivos fuera del módulo propio
```

---

## Cuándo usar Agent Teams vs Sub-agentes

| Situación | Recomendación |
|-----------|---------------|
| 3+ módulos independientes en paralelo | **Agent Teams** |
| Tarea puntual de investigación/verificación | **Sub-agente simple** |
| Módulos que comparten archivos | Un solo agente (para evitar conflictos) |
| Proyecto pequeño (1-2 módulos) | Sesión única de Claude Code |

---

## Solución de problemas comunes

**Los agentes no se muestran en paneles separados**
→ Verifica que estás dentro de una sesión tmux antes de lanzar `claude`. Ejecuta `echo $TMUX` — debe mostrar algo si estás dentro.

**tmux no reconocido**
→ Asegúrate de instalar dentro de WSL (Ubuntu), no en PowerShell: `sudo apt install -y tmux`.

**Claude Code no encuentra la API key**
→ Ejecuta `claude` y sigue el flujo de autenticación, o establece: `export ANTHROPIC_API_KEY="sk-ant-..."`.

**Sesión tmux persistente después de que Claude terminó**
→ `tmux ls` para listar, luego `tmux kill-session -t nombre`.

**Archivos lentos en /mnt/c/**
→ Clona el repositorio dentro del filesystem Linux (`~/dev/`) para mejor rendimiento en tareas intensivas.
