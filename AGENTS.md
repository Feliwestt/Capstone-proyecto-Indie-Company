# AGENTS.md

## Método de push a Git (opcional)

- **main = producción.** Nunca hagas push a main sin permiso explícito del usuario.
- **dev = rama de trabajo diario.** Los cambios se suben ahí de forma segura.
- **Deploy:** fusionar dev → main solo cuando el usuario diga "push to main" o "deploy".
- Nunca hagas `git push --force` ni reescribas historial de ramas compartidas.
- Nunca commitees secretos ni credenciales (.env, tokens, contraseñas).
- Haz `git pull` antes de `git push` para evitar conflictos.
- Mensajes de commit descriptivos, ej: "agregar evidencias fase 2".
- Para cambios grandes, trabajar en ramas `feature/*` y fusionarlas a dev.
- No commitees archivos basura: `node_modules/`, `*.tmp`, salidas de build (usar .gitignore).
- Antes de cada commit, revisa `git status` y `git diff` para no subir archivos por accidente.
