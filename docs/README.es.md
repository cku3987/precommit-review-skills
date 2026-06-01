# Guía de svn-precommit-review

`svn-precommit-review` es una Codex Agent Skill para revisar cambios de SVN/TortoiseSVN antes de confirmar y preparar un borrador de mensaje de commit en coreano.

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

## Qué revisa

- `svn status`, `svn status -u`, `svn diff --summarize`, `svn diff`
- artefactos de build, archivos de IDE, logs, temporales y configuración local
- riesgos de `.env`, tokens, contraseñas, API keys y otros secretos
- marcadores de conflicto y archivos missing/deleted
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
