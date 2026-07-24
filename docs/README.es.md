# Guía de svn-precommit-review

`svn-precommit-review` es una Codex Agent Skill para revisar cambios de SVN/TortoiseSVN antes de confirmar y preparar un borrador de mensaje de commit en coreano. Por defecto la revisión es de solo lectura; si el usuario pide explícitamente preparar el commit o auto-add, puede ejecutar `svn add` solo para archivos nuevos necesarios que podrían romper build/runtime y que pasen los criterios de seguridad. Después de la revisión, `svn delete` solo se permite para rutas versionadas exactas cuya eliminación el usuario vuelve a solicitar y que pasan los criterios de seguridad.

## Instalación

A nivel de usuario:

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

A nivel de proyecto:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

Copia `svn-precommit-review/SKILL.md` desde este repositorio a la ubicación deseada.

## Uso

```text
$svn-precommit-review Revisa los cambios SVN actuales antes del commit y prepara un mensaje de commit en coreano.
```

Para una working copy específica:

```text
$svn-precommit-review Revisa svn status y svn diff en <SVN_WORKING_COPY> y recomienda si se puede hacer commit.
```

Safe add de archivos necesarios:

```text
$svn-precommit-review Revisa los cambios en <SVN_WORKING_COPY> y agrega con svn add solo los archivos nuevos necesarios si podrían romper el build y pasan los criterios de seguridad.
```

## Qué revisa

- revisión estructurada con `svn status --xml`, `svn status -u --xml`, diffs de contenido y propiedades
- artefactos de build, archivos de IDE, logs, temporales y configuración local
- riesgos de `.env`, tokens, contraseñas, API keys y otros secretos; los valores siempre se ocultan como `[REDACTED]`
- conflictos de texto, propiedades y árbol, además de estados replaced, obstructed, switched, external, nested working copy y lock
- `svn add` seguro para archivos necesarios de build solo con solicitud explícita
- `svn delete` seguro solo para rutas locales exactas y sin cambios; nunca para archivos modificados localmente ni URLs del repositorio
- snapshot SHA-256 de estado, diffs, propiedades, objetivos y contenido de archivos antes de aprobar y justo antes del commit
- commits por archivo de forma predeterminada; los directorios requieren revisar todos los descendientes y límites SVN
- las categorías vacías se muestran brevemente como `- 없음`
- en el modo manual de TortoiseSVN, Codex no ejecuta `svn commit` ni solicita aprobación para ejecutar la CLI
- el commit CLI de Codex solo se ejecuta después de una solicitud explícita y de una respuesta afirmativa clara a la pregunta directa de ejecución tras revisar los objetivos y el mensaje

Respuestas breves y claras como `sí`, `yes` o `continúa` autorizan el commit CLI solo si el modo CLI ya se solicitó explícitamente y Codex acaba de hacer la pregunta directa de ejecución. Una solicitud de solo revisión termina sin esa pregunta.

## Codificación de mensajes coreanos

Evita esto en PowerShell:

```powershell
svn commit -m "한글 커밋 메시지"
```

Para commits por CLI usa un archivo UTF-8 sin BOM, con saltos LF, y `--encoding UTF-8`:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

Obtén la revisión del resultado del commit y verifica exactamente esa revisión:

```powershell
svn log --xml -r <revision-number> <repository-root-url>
```

Compara el `<msg>` XML exactamente con el mensaje original. `--limit 1` no es una prueba válida cuando puede haber commits simultáneos.

## Detalles

Guía principal en inglés: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
