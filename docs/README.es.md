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

- `svn status`, `svn status -u`, `svn diff --summarize`, `svn diff`
- artefactos de build, archivos de IDE, logs, temporales y configuración local
- riesgos de `.env`, tokens, contraseñas, API keys y otros secretos
- marcadores de conflicto y archivos missing/deleted
- `svn add` seguro para archivos necesarios de build solo con solicitud explícita
- `svn delete` seguro solo para rutas exactas revisadas tras una nueva solicitud explícita
- commit CLI protegido solo con solicitud explícita y aprobación final

## Codificación de mensajes coreanos

Evita esto en PowerShell:

```powershell
svn commit -m "한글 커밋 메시지"
```

Para commits por CLI usa un archivo UTF-8 y `--encoding UTF-8`:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

Verifica el log almacenado con:

```powershell
svn log --xml --limit 1
```

## Detalles

Guía principal en inglés: [`svn-precommit-review-guide.md`](svn-precommit-review-guide.md)
