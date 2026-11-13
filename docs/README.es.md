# Guía de precommit-review-skills

[precommit-review-skills](https://github.com/cku3987/precommit-review-skills) reúne habilidades independientes de Codex Agent para revisar cambios antes del commit en Git y SVN. Las versiones se publican conjuntamente a nivel de repositorio, pero cada habilidad se instala y se invoca por separado. El nombre `svn-precommit-review` y su `SKILL.md` se mantienen sin cambios; no renombre la habilidad SVN ya instalada.

- [Habilidad SVN](../svn-precommit-review/SKILL.md)
- [Habilidad Git](../git-precommit-review/SKILL.md)
- [Guía detallada de Git](git-precommit-review-guide.md)

## Git

`git-precommit-review` redacta los informes de revisión y los mensajes de commit en coreano.

- Por defecto revisa los cambios y añade automáticamente al área de preparación los nuevos archivos seguros cuya necesidad está confirmada. Excluye archivos ignorados, generados y con secretos.
- Una solicitud explícita de solo lectura impide cualquier escritura.
- Al solicitar un commit, prepara los cambios revisados de archivos versionados y conserva la preparación parcial existente. Los commits de alcance delimitado excluyen los cambios ajenos ya preparados.
- El commit requiere una única confirmación final del alcance y el mensaje. `push` solo se ejecuta mediante una solicitud aparte.

Ejemplos de invocación predeterminada, solicitud de commit en coreano (`커밋해줘`) y solo lectura explícita:

```text
$git-precommit-review
$git-precommit-review 변경사항을 검토하고 커밋해줘.
$git-precommit-review Revisa los cambios actuales de Git en modo de solo lectura. No realices ninguna escritura.
```

## SVN

`svn-precommit-review` es una habilidad de Codex Agent para revisar una copia de trabajo SVN/TortoiseSVN antes del commit y crear un mensaje de commit en coreano. Por defecto revisa y ejecuta automáticamente `svn add` para los nuevos archivos necesarios que cumplen los criterios de seguridad. El commit requiere una solicitud del usuario y una única confirmación final del alcance y el mensaje; solo incluye el alcance aprobado.

### Instalación

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

o:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

Copie `svn-precommit-review/SKILL.md` de este repositorio a la ubicación elegida.

### Uso

Revisión y preparación predeterminadas de archivos necesarios (una petición genérica como «revisa» o `검토해줘` hace lo mismo):

```text
$svn-precommit-review
```

Revisión de solo lectura:

```text
$svn-precommit-review Solo revisa los cambios actuales de SVN en modo de solo lectura. No ejecutes svn add/delete/commit.
```

Solicitar revisión y commit:

```text
$svn-precommit-review Revisa los cambios de <SVN_WORKING_COPY> y haz el commit.
```

Si solicitó un commit, compruebe los archivos incluidos y excluidos, los add/delete preparados, los riesgos pendientes y el mensaje, y responda una vez a la pregunta final. Basta con pedir «haz el commit» o `커밋해줘`; no es necesario decir `CLI`.

```text
네, 진행해
```

### Comportamiento

- Revisa `svn status --xml`, estado remoto, diff de contenido y diff de properties.
- Revisa archivos generados, archivos de IDE/usuario, registros, configuración local, datos sensibles, conflictos, archivos ausentes y falta de pruebas.
- En la revisión predeterminada, debe ejecutar automáticamente `svn add` para archivos sin versionar de código fuente, proyectos, recursos y configuración no exclusiva del entorno local cuya necesidad esté respaldada por referencias en archivos versionados o cambios revisados. Si se cumplen los criterios, no basta con recomendarlo.
- Las solicitudes explícitas de solo lectura, solo revisión, solo mensaje o ningún cambio no permiten ninguna escritura. Una petición genérica de revisión incluye la preparación predeterminada.
- Solo marca una adición como completada tras comprobar el éxito del comando add y el estado `A` en `svn status`. Las adiciones completadas figuran en el resumen de archivos, no como advertencias pendientes.
- Un add seguro e independiente puede continuar aunque quede un riesgo en el código. Ese riesgo se sigue notificando; el add no se presenta como una corrección del código.
- No agrega `bin`, `obj`, archivos generados, de IDE/usuario, registros, temporales, caché, configuración local, archivos comprimidos, volcados, copias de seguridad, capturas de pantalla, archivos ignorados, `.env`, archivos con posibles secretos ni ambiguos. Antes de agregar un directorio revisa todo su contenido recursivamente; si no es seguro en su totalidad, agrega solo las rutas concretas seguras.
- Cuando se solicita incluir candidatos de eliminación revisados, solo programa la eliminación de rutas locales versioned limpias.
- No elimina rutas modificadas localmente, en conflicto, switched, external, locked o ambiguas.
- El commit requiere una solicitud y la aprobación final del alcance exacto y el mensaje. Conserva la aprobación previa del mismo alcance y contenido, y vuelve a comprobar status y los diffs de contenido y propiedades antes del commit. Solo revisa y pide confirmación de nuevo si cambia sustancialmente el alcance o el contenido, o cambia un estado bloqueante.
- Si se elige explícitamente el commit manual con TortoiseSVN, termina con la revisión y la lista de comprobación, sin commit por CLI ni pregunta para ejecutarlo. La preparación segura predeterminada sigue activa salvo que se pida solo lectura.
- No usa `svn add`, `svn delete` ni `svn commit` de alcance amplio.

Las indicaciones durante la ejecución, la revisión, el mensaje de commit y los resultados se redactan siempre en coreano. Los resúmenes se agrupan por intención, sin un número fijo de puntos ni detalles excesivos. Solo reutiliza pruebas anteriores con evidencia real aplicable al contenido revisado sin cambios, y las identifica como resultados anteriores, no ejecutados de nuevo.

### Mensajes de commit en coreano

Para commits por CLI crea fuera de la copia de trabajo un archivo temporal único en UTF-8 sin BOM y con saltos de línea LF, y usa `--file` y `--encoding UTF-8`:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

El archivo temporal se elimina después de cada intento y se confirma su ausencia. Cuando el acceso al repositorio lo permite, compara carácter por carácter el mensaje del registro XML de la revisión real identificada en la salida del commit con el original. Si el coreano almacenado está dañado, informa un fallo de verificación de codificación y no reintenta el commit ni modifica revision properties.
