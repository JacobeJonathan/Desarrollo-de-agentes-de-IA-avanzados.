<img width="1662" height="1174" alt="Desarrollo de agentes de IA avanzados" src="https://github.com/user-attachments/assets/65d25d04-593b-4196-84d3-593c2bb43041" />

### Manual de Usuario: Uso de paneles y subagentes en Codex CLI
#### Objetivo
Trabajar con varios subagentes en paralelo utilizando Windows Terminal/CMD y Codex CLI.
<img width="848" height="461" alt="ssssdd" src="https://github.com/user-attachments/assets/faafb9f2-18e7-475f-8827-987b6e849883" />

 
- 1. Abrir paneles
- 1. Abre Windows Terminal.
- 2. Presiona Alt + Shift + D para dividir la ventana.
- 3. Repite el atajo hasta tener tres paneles.
- 2. Ir a la carpeta del proyecto
- En cada panel ejecuta:

cd C:\ruta\de\tu\proyecto
## 3. Iniciar un subagente por panel
Panel 1 - HTML
codex "Lee proceso.md. Actúa únicamente como subagente HTML. Modifica solo archivos HTML."
Panel 2 - CSS
codex "Lee proceso.md. Actúa únicamente como subagente CSS. Modifica solo archivos CSS."
Panel 3 - JSON
codex "Lee proceso.md. Actúa únicamente como subagente JSON. Modifica solo archivos JSON."
## 4. Recomendaciones
- Todos los paneles deben estar en la misma carpeta del proyecto.
- Cada subagente debe trabajar en archivos diferentes para evitar conflictos.
- Usa Git para revisar los cambios antes de integrarlos.
Ejemplo de flujo
Panel 1: index.html
Panel 2: styles.css
Panel 3: config.json
Cuando los tres terminen, revisa los cambios y realiza el commit.
