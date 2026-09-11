# CodeForge

Panel de escritorio para Windows que junta en una sola ventana los issues de Linear, el estado real de las ramas de git en tu máquina y tus pull requests de GitHub, y desde el que arrancas el trabajo de un issue con un agente de código sin cambiar de aplicación.

Está pensado para una persona trabajando en su propia máquina. No hay servidor, no hay cuentas propias, no hay nube: la app lee lo que ya existe (git, Linear, GitHub) y hace un conjunto pequeño y explícito de escrituras.

Este repositorio solo publica los instaladores. El código fuente vive en un repositorio privado.

## Descargar

Entra a [Releases](../../releases) y descarga el `*-setup.exe` más reciente. Las versiones con guion (`v0.1.0-beta`) son previas.

El instalador no está firmado, así que Windows SmartScreen puede frenarlo la primera vez: **Más información → Ejecutar de todas formas**.

La app revisa al arrancar si hay una versión más nueva y te avisa con un enlace a la descarga. También puedes buscarla a mano en **Ajustes → General → Buscar actualizaciones**. Nunca se actualiza sola.

## Qué hace

La idea central es que una rama local, su issue de Linear, su PR de GitHub y la sesión del agente que trabaja en ella son la misma fila. El cruce se hace por el nombre de rama que sugiere Linear.

- **Hoy.** Lo que está a medias, las alertas (ramas muy atrasadas, PRs esperando) y la cola de issues asignados sin rama, ordenada por prioridad y estado.
- **Revisión.** Lo que un agente dejó terminado, los PRs que esperan tu review y las ramas con cambios sin commitear.
- **Issues.** Tus issues de uno o varios equipos de Linear. Estado, prioridad, ciclo y proyecto se editan en la misma fila; también puedes comentar y crear issues nuevos, con un borrador redactado por Claude si quieres.
- **Ramas.** Cada rama con cuántos commits va adelante y atrás de la principal, dos diffs separados (lo commiteado contra la principal y lo vivo sin commitear), su worktree, su issue, su PR y su sesión. Desde aquí creas ramas nuevas, las borras o limpias en lote las ya fusionadas.
- **Diff de rama.** El diff completo archivo por archivo, con comentarios de revisión que se copian como prompt para el agente.
- **PRs y proyectos.** Tus PRs abiertos con su CI y su estado de review, y los proyectos activos de Linear con su salud y su último update.
- **Sesiones.** Terminales integradas donde corre Claude Code (o Codex, o Gemini CLI, si están instalados) dentro del worktree de cada rama. Varias a la vez en una cuadrícula, con cola cuando se llega al máximo, notificación cuando un agente pregunta algo y reanudación de sesiones anteriores de Claude Code.
- **Skills.** Las skills de Claude Code del repo y de tu perfil, para leerlas, editarlas o crear nuevas.
- **Trabajar en un issue.** Un solo botón: hace `fetch`, crea el worktree desde la rama principal remota (o apilado sobre otra rama) con el nombre que sugiere Linear, escribe el contexto del issue dentro, opcionalmente lo mueve a "En curso" y arranca la sesión del agente con la skill o el prompt que elijas.

También trae paleta de comandos (`Ctrl+K`), apertura rápida de archivos, búsqueda en los worktrees, vista previa de archivos, envío de una rama a GitHub Desktop, tema claro y oscuro, e interfaz en español e inglés.

## Tecnologías

| Capa | Qué usa |
| --- | --- |
| Shell de escritorio | [Tauri 2](https://tauri.app) sobre WebView2 |
| Backend | Rust con `tokio`; `reqwest` para las APIs; `keyring` para guardar credenciales en el Credential Manager de Windows |
| Frontend | React 19 con TypeScript, Tailwind CSS 4 y componentes de shadcn/ui sobre Radix |
| Terminales | `portable-pty` sobre ConPTY, pintado con xterm.js y su renderer WebGL |
| Diffs y markdown | `react-diff-view`, `react-markdown` y Shiki para el resaltado |
| Git | El `git.exe` instalado, invocado directo; sin bibliotecas de git ni shell intermedio |
| GitHub | `gh` si está instalado; si no, la API REST con un token |
| Linear | API GraphQL con un API key personal |
| Tipos compartidos | `ts-rs` genera los tipos de TypeScript desde los structs de Rust |

No hay base de datos: el estado vive en memoria y se reconstruye en cada arranque. La configuración es un `config.toml` en `%APPDATA%\CodeForge\`.

## Requisitos

- Windows 10 1809 o superior (WebView2 ya viene en Windows 11)
- [Git para Windows](https://git-scm.com/download/win) con `git.exe` en el `PATH`
- Un API key personal de Linear (Linear → Settings → Security & access → Personal API keys)
- [GitHub CLI](https://cli.github.com) con sesión iniciada, o un token personal de GitHub
- Para las sesiones: [Claude Code](https://docs.anthropic.com/claude-code) en el `PATH` nativo de Windows (no solo dentro de WSL). Codex y Gemini CLI son opcionales.

## Primer arranque

Al abrir la app por primera vez te lleva a Ajustes:

1. Pega el API key de Linear y elige tus equipos.
2. Agrega las carpetas de los repos que quieres seguir.
3. Elige la carpeta raíz de los worktrees (por defecto `C:\wt`). Van fuera del repo a propósito: ruta del repo, más nombre de rama, más `node_modules` choca rápido con el límite de 260 caracteres de Windows.
4. Si no usas `gh`, pega un token de GitHub.

Las credenciales se guardan en el Credential Manager de Windows, nunca en un archivo.

Tres ajustes que evitan la mayoría de los problemas:

- `git config --global core.longpaths true`
- Excluye las carpetas de repos y la de worktrees de Windows Defender. Git es lento cuando Defender revisa cada archivo de `.git`, y la app lo consulta cada 30 segundos.
- No tengas los repos dentro de una carpeta sincronizada por OneDrive: git y la sincronización se pelean por los archivos.

## Qué escribe y qué no

Son límites deliberados:

- **Nunca** hace `push`, `--force` ni `merge`.
- En git solo escribe `fetch`, crea y quita worktrees, y crea o borra ramas locales. Borrar siempre pide confirmación.
- En Linear cambia estado, prioridad, ciclo y proyecto, comenta, crea issues y publica updates de proyecto. No borra ni archiva nada. Cada escritura queda registrada en `%APPDATA%\CodeForge\logs\writes.log`.
- Las sesiones de los agentes solo arrancan cuando tú lo pides. Nada corre por su cuenta, por webhook ni por temporizador, y mueren al cerrar la app, con aviso antes.

## Reportar un problema

Los reportes y las sugerencias van en [Issues](../../issues/new/choose), con un formulario que pide lo necesario.

- **Desde la app:** **Ajustes → General → Reportar un problema** abre el formulario en el navegador con la versión ya puesta.
- **Si la app tuvo un error interno o se cerró sola:** al volver a abrirla aparece un aviso con **Reportar**, que abre el formulario con el error ya pegado.
- **Si la ventana se queda en blanco por un error de la interfaz:** en su lugar aparece un mensaje con **Recargar** y **Reportar**. Recargar no corta las sesiones de los agentes: corren en el proceso de la app, no en la ventana.

Nada se envía solo: siempre ves el formulario completo en GitHub y decides si lo mandas.

Los errores de la app y de la interfaz quedan en `%APPDATA%\CodeForge\logs\app.<fecha>.log` (se guardan los últimos 7 días), y también se pueden ver en **Ajustes → General → Ver registro**. Si pegas parte del registro en un reporte, revísalo antes: incluye rutas de tus repos y nombres de ramas, y este repositorio es público.

## Problemas comunes

- **Git tarda segundos en cada refresco.** Faltan las exclusiones de Defender.
- **Errores de git sobre archivos que no existen.** Casi siempre es el límite de ruta: revisa `core.longpaths` y que los worktrees estén en una carpeta corta.
- **Archivo bloqueado o en uso.** El repo está en una carpeta de OneDrive.
- **La zona de Linear dice que falló.** El API key venció; reemplázalo en Ajustes.
- **La sesión no arranca.** `claude` no está en el `PATH` de Windows. Compruébalo con `where claude` desde PowerShell.
