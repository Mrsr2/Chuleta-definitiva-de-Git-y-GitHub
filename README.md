# Chuleta definitiva de Git y GitHub

**Objetivo:** trabajar con Git sin perder cambios, contribuir a proyectos reales y mejorar la visibilidad de tu perfil y tus repositorios.

> **Idea clave:** Git gestiona versiones en tu equipo; GitHub aloja repositorios y añade perfiles, issues, pull requests, métricas y colaboración.

## Índice

1. Conceptos y configuración
2. Flujo diario de Git
3. Historial, diferencias y búsqueda
4. Ramas, merge y rebase
5. Deshacer cambios y recuperarlos
6. Remotos, forks y contribuciones
7. Stash, etiquetas y limpieza
8. Seguridad

---

## 1. Conceptos y configuración

### Los cuatro lugares que debes conocer

- **Directorio de trabajo:** archivos que estás editando.
- **Staging o índice:** instantánea de los cambios que has seleccionado para el próximo commit.
- **Repositorio local:** historial de commits guardado en tu equipo.
- **Repositorio remoto:** copia alojada en GitHub u otro servidor. `origin` y `upstream` son nombres de remotos, no ramas.

```bash
git --version
git config --global user.name "Tu nombre"
git config --global user.email "tu-correo-verificado@ejemplo.com"
git config --global init.defaultBranch main
git config --list --show-origin
```

Usa un correo asociado a tu cuenta de GitHub o el correo `noreply` que aparece en los ajustes de correo de GitHub. `user.name` es el nombre que figura en los commits; no tiene que coincidir con tu nombre de usuario de GitHub. Si quieres una configuración solo para un proyecto, ejecuta `git config user.email "..."` dentro de ese repositorio, sin `--global`.

**Autenticación:** para HTTPS, GitHub no acepta la contraseña de la cuenta para operaciones Git: utiliza un gestor de credenciales, GitHub CLI o un token; también puedes usar SSH. No pegues tokens en URLs, scripts o capturas.

## 2. Flujo diario de Git

### Crear o descargar un proyecto

```bash
mkdir mi-proyecto && cd mi-proyecto
git init                         # Crea un repositorio local en la carpeta actual
# O, para un proyecto que ya existe:
git clone https://github.com/USUARIO/REPOSITORIO.git
cd REPOSITORIO
```

**Elige una de las dos opciones:** `git init` para un proyecto nuevo o `git clone` para uno existente; no necesitas ejecutar ambas para el mismo proyecto.

### Revisar y guardar

```bash
git status                       # Estado de archivos, staging y rama
git status -sb                   # Resumen compacto
git add archivo.txt              # Prepara la versión actual de un archivo
git add .                        # Prepara cambios de la carpeta actual hacia abajo
git add -A                       # Prepara altas, modificaciones y borrados de todo el repo
git diff --staged                # Revisa lo que entrará en el commit
git commit -m "docs: agrega guía de instalación"
git log --oneline -5
```

`git add` no pone archivos «a escuchar»: añade una instantánea al staging. Si vuelves a editar un archivo después, debes hacer `git add` otra vez para incluir esa nueva versión. Revisa `git status` y `git diff --staged` antes de confirmar.

```bash
git commit -am "fix: corrige validación"
```

Este atajo prepara y confirma cambios en archivos **ya rastreados**, incluidos borrados; **no** añade archivos nuevos sin seguimiento. Los mensajes `feat:`, `fix:` y `docs:` son una convención opcional, no una obligación de Git.

### Publicar un repositorio nuevo

1. Crea en GitHub un repositorio vacío, preferiblemente sin README ni licencia si ya tienes un historial local.
2. Comprueba la URL real y reemplaza los marcadores:

```bash
git branch -M main
git remote add origin https://github.com/USUARIO/REPOSITORIO.git
git remote -v
git push -u origin main
```

`-u` establece la rama remota de seguimiento para futuros `git push` y `git pull`. Si el remoto ya tiene contenido, **no fuerces el envío**: consulta la sección de soluciones.

## 3. Historial, diferencias y búsqueda

```bash
git log                          # Historial de la rama actual
git log --all --graph --decorate --oneline
git log --oneline -- archivo.txt # Historial que afecta a un archivo
git show                         # Detalles del último commit
git show HASH                    # Detalles de un commit específico
git show HASH:archivo.txt        # Contenido del archivo en ese commit
git diff                         # Trabajo vs. staging
git diff --staged                # Staging vs. último commit
git diff HASH1 HASH2             # Diferencias entre dos commits
git diff HASH1 HASH2 -- archivo.txt
git diff main...mi-rama          # Cambios desde el ancestro común hasta mi-rama
```

`git show` sin argumentos muestra el último commit, **no todo el historial**; usa `git log` para el historial.

```bash
git grep -n "palabra"            # Coincidencias y líneas en archivos rastreados
git grep -c "palabra"            # Número de líneas coincidentes por archivo
git log --grep="palabra"         # Busca en mensajes de commit
git log -S"palabra" --oneline    # Commits que cambian el número de apariciones
git log -G"expresión" --oneline  # Commits con líneas modificadas que coinciden
git shortlog -sn HEAD            # Número de commits por autor en esta historia
git blame archivo.txt            # Autor y commit de cada línea actual
```

`git grep -c` cuenta **líneas coincidentes**, no necesariamente todas las apariciones. `git log -s` suprime el diff: no busca una palabra.

## 4. Ramas, merge y rebase

```bash
git branch                       # Lista ramas locales
git branch -a                    # Incluye referencias remotas
git switch -c feat/mi-mejora     # Crea y cambia a una rama
git switch main                  # Cambia de rama
git checkout main                # Alternativa tradicional a switch
git branch -d feat/mi-mejora     # Borra rama local ya fusionada
```

### Fusionar una rama en main

```bash
git switch main
git pull --ff-only origin main   # Solo si main sigue a origin/main
git merge feat/mi-mejora
git push origin main
```

`merge` integra historias; puede crear un commit de merge o avanzar directamente con *fast-forward*. En proyectos colaborativos normalmente se abre un pull request en lugar de hacer push directo a `main`.

### Rebase de tu rama sobre main

```bash
git fetch origin
git switch feat/mi-mejora
git rebase origin/main
# Si hay conflictos: editar archivos, git add ARCHIVO, git rebase --continue
# Para cancelar todo el rebase: git rebase --abort
```

`rebase` vuelve a aplicar commits sobre otra base y cambia sus identificadores. Evita reescribir ramas compartidas. Si tu rama personal ya estaba publicada y necesitas actualizarla, coordínate y, solo si es apropiado, usa `git push --force-with-lease`, que es más seguro que `--force`, pero todavía puede sobrescribir trabajo. Las ramas protegidas pueden bloquearlo.

**Conflicto en un merge:** edita los archivos señalados por `git status`, elimina los marcadores de conflicto, ejecuta `git add ARCHIVO` y después `git commit` si Git lo pide. Para cancelar el merge en curso: `git merge --abort`.

## 5. Deshacer cambios y recuperarlos

⚠️ Comprueba `git status` y haz una copia antes de comandos destructivos. `restore`, `reset --hard` y `clean` pueden eliminar trabajo que nunca se confirmó.

```bash
git restore --staged archivo.txt # Quita del staging; conserva el archivo editado
git restore archivo.txt          # Descarta cambios NO preparados del archivo
git restore --source=HEAD --staged --worktree archivo.txt
                                 # Descarta cambios preparados y no preparados
git rm --cached archivo.txt      # Deja de rastrear el archivo, conservándolo en disco
git rm archivo.txt               # Lo elimina del disco y prepara el borrado
git revert HASH                  # Crea un commit que revierte otro; útil en ramas compartidas
```

Para dejar de rastrear un archivo y evitar que se vuelva a añadir, inclúyelo también en `.gitignore`. `git rm --cached` no borra información del historial. `git revert` aplicado a un commit de merge necesita opciones adicionales.

```bash
git commit --amend               # Modifica el último commit con lo preparado
git commit --amend -m "nuevo mensaje"
git reset --soft HEAD~1          # Deshace el último commit; conserva cambios en staging
git reset --mixed HEAD~1         # Deshace el commit; conserva cambios fuera de staging
git reflog                       # Movimientos locales recientes de HEAD
git switch -c rescate HASH       # Crea una rama para rescatar un commit encontrado
```

`--amend` y `reset` reescriben el historial: evita usarlos en commits que otros ya comparten. El reflog es local y sus entradas pueden caducar; no es una copia de seguridad.

```bash
# PELIGRO: mueve la rama y descarta cambios rastreados de staging y directorio de trabajo.
# Solo después de comprobar el hash y guardar cualquier trabajo necesario:
git reset --hard HASH
```

## 6. Remotos, forks y contribuciones

### Actualizar tu copia

```bash
git remote -v
git fetch origin                 # Descarga referencias; no fusiona cambios
git pull --ff-only               # Descarga y avanza sin crear merge; falla si divergió
git push                         # Envía la rama con seguimiento configurado
git push -u origin feat/mi-mejora
```

Si `pull --ff-only` falla por divergencia, revisa `git status` y `git log --all --graph --oneline`, y decide entre merge o rebase según el proyecto. No uses `--allow-unrelated-histories` como solución automática: solo se usa cuando **realmente** deseas unir dos historias independientes y entiendes las consecuencias.

### Contribuir a un proyecto con fork

1. En GitHub, crea un **fork** del repositorio original.
2. Clona tu fork y añade el original como `upstream`:

```bash
git clone https://github.com/TU_USUARIO/PROYECTO.git
cd PROYECTO
git remote add upstream https://github.com/AUTOR_ORIGINAL/PROYECTO.git
git remote -v
git fetch upstream
git switch main                   # Sustituye main si la rama principal se llama distinto
git merge --ff-only upstream/main
git switch -c fix/descripcion-corta
# Edita, prueba y revisa las instrucciones del proyecto.
git add ARCHIVO
git commit -m "fix: describe la corrección"
git push -u origin fix/descripcion-corta
```

3. En GitHub, abre un **pull request** desde tu fork y rama hacia la rama principal del proyecto original. Explica qué cambia, por qué y cómo lo probaste; enlaza el issue si existe.
4. Responde a la revisión y realiza más commits en la misma rama. El PR se actualizará con cada push.
5. Cuando se integre, actualiza tu `main` desde `upstream` antes de empezar otra tarea. `upstream` es un remoto, no una rama.

Busca issues `good first issue` o `help wanted`, lee `CONTRIBUTING.md` y pide aclaraciones antes de hacer cambios grandes. No envíes PRs triviales o masivos solo para inflar estadísticas.

### GitHub CLI: opcional

Requiere instalar [GitHub CLI](https://cli.github.com/) y autenticarse con `gh auth login`. Estos son comandos de **GitHub CLI**, no de Git:

```bash
gh auth status
gh repo view --web
gh issue list --label "good first issue"
gh pr create --web
gh pr status
```

## 7. Stash, etiquetas y limpieza

### Guardado temporal

```bash
git stash push -m "trabajo pendiente"     # Guarda cambios rastreados
git stash push -u -m "incluye nuevos"     # También archivos no rastreados
git stash list
git stash show -p stash@{0}
git stash apply stash@{0}                 # Aplica y conserva el stash
git stash pop                             # Aplica y elimina si tiene éxito
git stash branch rescate stash@{0}        # Crea rama desde su base y aplica
git stash drop stash@{0}                  # Elimina esa entrada
```

`stash` es local y temporal, no reemplaza una copia de seguridad; por defecto no guarda archivos no rastreados ni ignorados. La aplicación puede producir conflictos. En algunas terminales quizá debas citar `'stash@{0}'`.

### Versiones con tags

```bash
git tag                                   # Lista etiquetas
git tag -a v1.0.0 -m "Primera versión"   # Etiqueta anotada en HEAD
git tag -a v0.1.0 -m "Versión inicial" HASH
git show v1.0.0
git push origin v1.0.0                   # Sube una etiqueta específica
git push origin --tags                    # Sube todas las etiquetas locales
git tag -d v1.0.0                         # Borra la etiqueta local
git push origin --delete v1.0.0           # Borra la etiqueta remota
```

Etiqueta solo versiones listas y documenta los cambios. Un tag no crea automáticamente una GitHub Release.

### Limpiar archivos no rastreados

```bash
git clean -nd    # Simulación: archivos y carpetas no rastreados
git clean -fd    # PELIGRO: los borra permanentemente
```

No uses `-x` salvo que sepas lo que haces: también incluiría archivos ignorados, que podrían contener secretos o datos locales. Revisa siempre la simulación primero.

## 8. Seguridad y buenas prácticas

Ejemplo mínimo de `.gitignore` (adáptalo al lenguaje):

```gitignore
.env
.env.*
!.env.example
node_modules/
__pycache__/
.DS_Store
```

- Nunca subas contraseñas, claves privadas ni tokens. Comparte un `.env.example` sin secretos reales.
- Si expones un secreto, **revócalo o rótalo inmediatamente**. Borrar el archivo o añadirlo a `.gitignore` no elimina el secreto del historial ni de otras copias. Sigue la [guía de GitHub para datos sensibles](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/removing-sensitive-data-from-a-repository).
- Activa 2FA, revisa permisos de tokens y, cuando esté disponible, usa secret scanning, push protection y Dependabot.
- Protege la rama principal cuando el plan y permisos lo permitan; exige PRs y pruebas en proyectos colaborativos.
- Antes de integrar contribuciones, ejecuta las pruebas y documenta cómo reproducirlas.

## 9. Perfil de GitHub, contribuciones, visitas y estrellas

### Primero, qué mide cada cosa

- **Contribuciones:** actividad que GitHub reconoce bajo sus criterios; no es un indicador directo de calidad.
- **Visitas:** GitHub ofrece métricas de tráfico **por repositorio**, no un contador nativo equivalente de visitas del perfil. Accede a `Repositorio → Insights → Traffic`; la disponibilidad depende del plan, visibilidad y permisos. La ventana mostrada suele ser de 14 días. Los contadores externos del perfil son opcionales, dependen de terceros y no equivalen a analítica oficial.
- **Estrellas:** otros usuarios marcan repositorios que les interesan. Se otorgan a **repositorios**, no directamente al perfil. Tu perfil puede mostrar proyectos fijados y sus estrellas; tú no puedes crear legítimamente estrellas de otras personas con un comando Git.

### Haz que tus contribuciones se atribuyan correctamente

1. Verifica que el correo del autor de los commits esté asociado a tu cuenta de GitHub:

```bash
git config user.email
git log -5 --format='%h | %an <%ae> | %s'
```

2. Haz push a GitHub. Los commits deben cumplir los requisitos de GitHub: en general, estar en la rama predeterminada o `gh-pages` de un repositorio independiente, no solo en una rama secundaria o un fork. Un PR integrado en el proyecto original puede hacer que tus commits sean elegibles; el método de integración puede afectar la atribución.
3. Issues, PRs, revisiones y otras actividades también pueden contar según sus reglas. Para actividad privada, activa la opción de mostrar contribuciones privadas en el perfil si quieres que aparezcan de forma anonimizada.
4. Puede tardar hasta 24 horas en actualizarse. Revisa la [guía oficial de contribuciones ausentes](https://docs.github.com/en/account-and-profile/how-tos/contribution-settings/troubleshooting-missing-contributions).

**Importante:** cambiar `git config user.email` afecta a futuros commits, no corrige retroactivamente los antiguos. Si puedes, añade y verifica en GitHub la dirección usada en los commits anteriores. No alteres fechas ni generes commits vacíos solo para pintar el gráfico.



## Soluciones rápidas

| Problema | Qué comprobar o hacer |
|---|---|
| No aparecen contribuciones | Correo del autor asociado, push realizado, rama predeterminada, fork y espera de hasta 24 horas. Consulta la guía oficial. |
| `push` rechazado | Haz `git fetch`, revisa el historial y el estado; integra los cambios siguiendo el flujo del proyecto. No fuerces sin entender el motivo. |
| `remote origin already exists` | Ejecuta `git remote -v`; si corresponde, actualiza con `git remote set-url origin URL_CORRECTA`. |
| `src refspec main does not match any` | Comprueba que exista al menos un commit y que la rama se llame `main`: `git status` y `git branch`. |
| Historias no relacionadas | Ocurre a menudo al crear el remoto con un README y también iniciar localmente. Inspecciona ambas historias; decide si conservar e integrar ambas. `--allow-unrelated-histories` es excepcional, no un arreglo universal. |
| Subí un secreto | Revócalo de inmediato, avisa a los responsables y sigue la guía de limpieza; un commit que lo borre no basta. |
| Necesito recuperar un commit | Usa `git reflog`, identifica el hash y crea `git switch -c rescate HASH`. Si el trabajo nunca se guardó, puede no ser recuperable. |

## Referencias oficiales

- [Documentación de Git](https://git-scm.com/doc)
- [Configuración del README de perfil](https://docs.github.com/en/account-and-profile/how-tos/profile-customization/managing-your-profile-readme)
- [Contribuciones que faltan](https://docs.github.com/en/account-and-profile/how-tos/contribution-settings/troubleshooting-missing-contributions)
- [Tráfico de un repositorio](https://docs.github.com/en/repositories/viewing-activity-and-data-for-your-repository/viewing-traffic-to-a-repository)
- [Buenas prácticas para repositorios](https://docs.github.com/en/repositories/creating-and-managing-repositories/best-practices-for-repositories)
- [Cómo actuar ante un secreto filtrado](https://docs.github.com/en/code-security/tutorials/remediate-leaked-secrets/remediating-a-leaked-secret)
