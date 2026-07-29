# Guía de svn-precommit-review

`svn-precommit-review` es una habilidad de Codex Agent para revisar una copia de trabajo SVN/TortoiseSVN antes del commit y crear un mensaje de commit en coreano. Por defecto funciona en modo de solo lectura. Cuando el usuario solicita explícitamente un commit mediante Codex CLI, puede hacer add de los nuevos archivos fuente necesarios y, tras una única confirmación final, confirmar solo el alcance aprobado.

## Instalación

```text
%USERPROFILE%\.codex\skills\svn-precommit-review\SKILL.md
```

o:

```text
<project-root>\.agents\skills\svn-precommit-review\SKILL.md
```

Copie `svn-precommit-review/SKILL.md` de este repositorio a la ubicación elegida.

## Uso

Revisión de solo lectura:

```text
$svn-precommit-review Revisa los cambios actuales de SVN antes del commit y crea un mensaje de commit en coreano.
```

Solicitar un commit por CLI:

```text
$svn-precommit-review Revisa los cambios de <SVN_WORKING_COPY>, haz add de los nuevos archivos fuente necesarios y, tras la confirmación final, confirma por CLI solo el alcance aprobado.
```

Después de mostrar el alcance y el mensaje, responda una vez de forma clara.

```text
네, 진행해
```

## Comportamiento

- Revisa `svn status --xml`, estado remoto, diff de contenido y diff de properties.
- Revisa archivos generados, archivos de IDE/usuario, registros, configuración local, datos sensibles, conflictos, archivos ausentes y falta de pruebas.
- Ante una solicitud de commit por CLI, puede ejecutar `svn add` para archivos fuente, proyectos, recursos y configuración necesaria cuya intención sea clara.
- No agrega `bin`, `obj`, registros, caché, archivos comprimidos, `.env`, archivos con posibles secretos, generados o ambiguos.
- Cuando se solicita incluir candidatos de eliminación revisados, solo programa la eliminación de rutas locales versioned limpias.
- No elimina rutas modificadas localmente, en conflicto, switched, external, locked o ambiguas.
- Antes del commit vuelve a revisar el estado y los diffs objetivo; solo se detiene y revisa de nuevo si cambió el alcance aprobado, el contenido significativo o un estado SVN bloqueante.
- No usa `svn add`, `svn delete` ni `svn commit` de alcance amplio.

## Mensajes de commit en coreano

Para commits por CLI usa un archivo temporal UTF-8 sin BOM y `--encoding UTF-8`:

```powershell
svn commit --file <message-file> --encoding UTF-8 <approved-paths>
```

El archivo temporal se elimina después de cada intento. Cuando el acceso al repositorio lo permite, compara el mensaje XML de la revisión realmente creada. Si el coreano almacenado está dañado, informa un fallo de verificación de codificación y no reintenta el commit ni modifica revision properties.
