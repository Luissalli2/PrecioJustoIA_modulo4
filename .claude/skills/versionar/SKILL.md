---
name: versionar
description: >-
  Versiona el trabajo en Git con la disciplina del curso: revisar el diff →
  commit con mensaje claro → push a main. También hace "nacer" un repo nuevo
  desde los 3 docs (PRD/AGENTS/CLAUDE) y lo publica en GitHub. Usar cuando el
  usuario pida commitear, pushear, guardar cambios, "versioná esto", publicar
  en GitHub, iniciar/armar un repo nuevo, o deshacer un desastre con Git.
---

# Versionar — Git con red y disciplina

Empaqueta el workflow de versionado del curso. Tres reglas de oro, tres
workflows (nacer un repo, guardar cambios, deshacer un desastre) y una lista de
qué NO hacer. Todos los comandos usan `git -C "<ruta-del-repo>"` para no
depender del directorio actual.

## Reglas de oro (siempre)

1. **Diffs chicos.** Antes de guardar, SIEMPRE `git diff`. Revisás lo viejo
   (rojo) vs lo nuevo (verde). Un cambio, una intención.
2. **Commit como checkpoint.** Cada unidad de trabajo terminada = un commit con
   mensaje honesto y claro. Mensaje en modo "qué hace este cambio"
   (ej. `docs: aclara criterio de aceptación`, `feat: agrega carga de ticket`).
3. **Revertir, no parchar.** Si algo se rompió, se vuelve al último buen estado
   (`git restore` / `git revert`), NO se apilan parches encima del desastre.
4. **Ritmo del proyecto:** funcionalidad lista → commit → push a main. El
   usuario dirige, el agente ejecuta los comandos.

## Antes de tocar nada

- Confirmá en qué carpeta estás y si ya es repo:
  ```
  git -C "<ruta>" rev-parse --is-inside-work-tree 2>/dev/null && echo "YA es repo" || echo "NO es repo"
  ```
- ⚠️ **Nunca** hagas `git init` en una carpeta que tiene código/`node_modules`
  sin confirmarlo con el usuario: podés estar versionando la carpeta equivocada.

---

## Workflow A — Hacer nacer un repo nuevo

Para arrancar un proyecto v2 desde cero, versionado desde el día uno. Solo
viajan los **documentos** (contrato + reglas); el código se reconstruye.

1. **Carpeta nueva + solo los 3 docs** (sin código, sin `node_modules`, sin
   carpetas sueltas):
   ```
   mkdir -p "<ruta-nueva>"
   cp "<origen>/PRD.md" "<origen>/AGENTS.md" "<origen>/CLAUDE.md" "<ruta-nueva>/"
   ```
2. **Init + rama main:**
   ```
   git -C "<ruta-nueva>" init
   git -C "<ruta-nueva>" branch -M main
   ```
3. **Identidad** (si Git pide "who you are"). Preguntá al usuario nombre y si la
   querés global (todos sus repos) o local (solo este):
   ```
   git -C "<ruta-nueva>" config user.name "<nombre>"
   git -C "<ruta-nueva>" config user.email "<email>"
   ```
4. **Primer commit:**
   ```
   git -C "<ruta-nueva>" add .
   git -C "<ruta-nueva>" commit -m "estado inicial: PRD + guardrails"
   git -C "<ruta-nueva>" log --oneline
   ```
   La foto tiene que ser liviana (solo los docs). Está bien que así sea.
5. **GitHub (lo crea el usuario en el navegador):** github.com → New repository
   → mismo nombre → Private/Public → **NO** marcar README/.gitignore/license
   (el repo debe quedar vacío del lado de GitHub). Pedí la URL `.git`.
6. **Conectar y publicar:**
   ```
   git -C "<ruta-nueva>" remote add origin https://github.com/<user>/<repo>.git
   git -C "<ruta-nueva>" push -u origin main
   ```

### Autenticación del push (importante)

- El prefijo `!` de Claude Code corre sin terminal interactiva → un push que
  pide usuario/token falla con `could not read Username`.
- En Windows/WSL, si existe el **Git Credential Manager**
  (`/mnt/c/Program Files/Git/mingw64/bin/git-credential-manager.exe`),
  configuralo y el login se hace por navegador (sin manejar tokens a mano):
  ```
  git -C "<ruta>" config credential.helper '!"/mnt/c/Program Files/Git/mingw64/bin/git-credential-manager.exe"'
  ```
- Sin GCM: el usuario genera un Personal Access Token (scope `repo`) y corre el
  `push` él mismo desde una terminal real.

---

## Workflow B — Guardar cambios (ritual diario)

El ciclo que se repite el resto del proyecto. Se puede pedir en palabras
("commiteá esto y pusheá a main") y el agente ejecuta:

1. **Revisar** qué cambió (hábito de diffs chicos):
   ```
   git -C "<ruta>" diff
   ```
2. **Guardar** con mensaje claro:
   ```
   git -C "<ruta>" add .
   git -C "<ruta>" commit -m "<tipo>: <qué hace el cambio>"
   ```
3. **Publicar** a main:
   ```
   git -C "<ruta>" push origin main
   ```
4. **Verificar** historial y sincronía con la nube:
   ```
   git -C "<ruta>" log --oneline
   git -C "<ruta>" status -sb    # "## main...origin/main" sin ahead/behind = sincronizado
   ```

Convención de mensajes (prefijos sugeridos): `feat:` nueva funcionalidad ·
`fix:` corrección · `docs:` documentación/PRD · `refactor:` reorganización sin
cambiar comportamiento · `chore:` tareas varias.

---

## Workflow C — Deshacer un desastre (botón de pánico)

Cuando la IA (o vos) rompen algo y todavía **no** se commiteó:

- Descartar cambios sin guardar de un archivo o de todo:
  ```
  git -C "<ruta>" restore <archivo>     # uno
  git -C "<ruta>" restore .             # todo, vuelve al último commit
  ```
- Ver qué está sucio antes de decidir:
  ```
  git -C "<ruta>" status --short
  ```

Si el desastre YA está commiteado (o pusheado), revertí en vez de borrar
historia:
```
git -C "<ruta>" revert <hash-del-commit-malo>
```
`revert` crea un commit nuevo que deshace el malo (historia honesta, seguro para
lo que ya está en GitHub). Evitá `reset --hard` sobre commits ya pusheados.

---

## Qué NO hacer

- ❌ `git init` en la carpeta de la v1 / carpeta "testigo" con código: no se
  toca, queda como estaba.
- ❌ Copiar código viejo al repo nuevo: solo viajan los docs; el código se
  reconstruye.
- ❌ Marcar README/.gitignore/license al crear el repo en GitHub (rompe el
  `push` del repo local existente).
- ❌ Commitear `node_modules`, `.next`, builds o secretos. Si aparecen, sumá un
  `.gitignore` antes del primer commit.
- ❌ `commit`/`push` sin haber mirado el `diff` primero.
- ❌ `git config --global` sin preguntar: puede pisar la identidad de todos los
  repos del usuario. Ante la duda, usá `--local`.

## Los 5 comandos base (referencia rápida)

| Comando | Para qué |
|---------|----------|
| `git status` | qué cambió / estado del árbol |
| `git add .` | preparar cambios para el commit |
| `git commit -m "..."` | guardar el checkpoint |
| `git push` | subir a GitHub |
| `git log --oneline` | ver el historial de fotos |

Extra imprescindibles: `git diff` (revisar antes de guardar), `git restore`
(deshacer sin commitear), `git revert` (deshacer algo ya commiteado).
