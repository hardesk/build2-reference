# build2 reference

A half-assed attempt to make a concise build2 reference.

buildfile contains directives, target declarations and variable assignmens

## Targets

- `exe` executable
- `lib[a|s]` library, also can be specifically set as archive or shared
- `libu[e|l]` utility library for executable or library (wtf is this???)
- `obj[e|a|s]` object, also executable, archive or shared specific

After the target introducer, its name follows eg. `exe{tool}` or `obje{util}`. Dependencies follow the target: `exe{tool}: {hxx cxx}{tool}`. Properties on target can be changed like so:

```
exe{tool}:
{
	install = false
}
```

When the specific suffix is used, it allows to change options only when building the target for a parent of that type. Eg we may want compile a .cpp with different options when building an archive vs a shared library. For example:
```
libs{tool}: cxx{string}

objs{string}:
{
	cxx.poptions += -DLIB_DYNAMIC_EXPORT
}
```


Also, scope can be introduced which is equivalent as if reading a buildfile from a folder with a name that matches the scope's name. Here in upper level buildfile we use files that reside in ./tool subdirectory (?).

```
tool/
{
	exe{hammer}: {hxx cxx}{*}
}
```

## Directives

These resemble functions in a programming language.

### import, import?, import!
Import target from another project. Syntax is `import <name> = [<project>%]<target>`. Project is optional.
Example:
```
import mylib = libhello%lib{hello}
import anotherlib = lib{world}
```
	
### assert, assert!
Check condition, write a message if condition evaluates to false. assert! inverts the condition.
```
assert ($cxx.target.cpu == 'x86_64') 'Only 64bit x86 is supported'
```

### print
Write argument to stdout
```
print $(cxx.id)
```

### fail, warn, info, text
Write diagnostics message to stderr. Fail also terminates the program.
```
info 'will get an error next'
fail 'the end'
info 'unreached'
```

### dump
Print the contents of the current scope (including nested scope) or target passed as argument to stderr.
```
exe{hello}: {hxx cxx}{**}
cxx.poptions =+ "-I$out_root" "-I$src_root"
dump
dump exe{hello}
```

### source

### include
Include a buildfile. Basically pull in text from the file specified, but additionally including it creates a new scope for the file content. Eg.
```
include ../libhello
```
this will include ../libhello/buildfile

### run
Launch an external process with arguments as process args and parse output as a buildfile.
```
run <name> [<arg>...]
```

### export
Export target something something..
```
export $out_root/libhello/$import.target
```

**build/export.build** files describes how to export a target out of a project. A few variables are defined while parsing it:

| variable | description |
| --- | --- |
| `src_root` | source root of the project being imported (this project) |
| `out_root` | output root of the project being imported (this project) |
| `import.target` | name of the target being imported. eg lib{tool} |

The file should load the buildfile for the target being exported and actually export it using `export` directive. eg.

See variables that are relevant to target [exportation](#export-variables)


```
$out_root { include buildfile }
export $out_root/libtool/$import.target
```

### using, using?
Use a build2 module, eg.
```
using cxx
```

### define
Define an alias for target (?).
```
define derived : base
```

### if, if!, else, elif, elif!
Branch execution depending on condition result. `!` version inverts condition.
```
if ($c.target.system == 'windows')
	info 'target is windows'
```

```
if! true
{
	info 'will not print'
}
```


### switch <arg, arg..> [: match-function]
### case, default
Pattern matching switch statement. Multiple values can be matched against a pattern consisting of multiple values as well. Also pattern can combine the values in different ways:
- ',', eg. case 'a', 'b' will match two arguments 'a' and 'b'.
- '|', eg. case 'a' | 'b' will match either 'a' or 'b'

match function can be
- `regex.match` - regex match of the pattern against the full argument string 
- `regex.search` - regex search of a substring in argument
- `path.match` - use path-style matching
- `icase` - add this to ignore case
- also any function taking `value, pattern` and returning bool can be specified

```
switch 'x' {
	case 'x'
		info 'got x'
}

switch $cxx.target.class : regex.match icase
{
	case 'A.*'
		info 'A architecture'
}
```

### for
For loop.

```
for n: foo bar baz
{
  exe{$n}: cxx{$n}
}
```

### config
Declare a special config variable. The name/path has to conform to `config[.**].<project>.**`


```
config [bool] config.libworld.greet ?= true
```


## Special variables

| variable | description |
| :---: | ---- |
| `src_root` | path to the root of the *project* |
| `src_base` | path to the current *target/scope* |
| `out_root` | path to the output root for the *project* |
| `out_base` | path to the current output for the *target/scope* |
| `version` | version, also available .major, .minor and .patch |

cxx module

| variable	| desc |
| :------:	| ---- |
| `cxx.std` 	| C++ std version (default to latest) |
| `cxx.id`		| toolset |
| `cxx.target.class`	| triplet describing compilation target |
| `cxx.target.cpu`	| target cpu |
| `cxx.target.system`	| target cpu |
| `cxx.target.system`	| target cpu |
| `cxx.export.*` | target 'usage' properties. |

### Export Variables
The prefix is either `c.`, `cxx.` or `cc.` for C, C++ or both respectively.

| var | desc |
| :---: | --- |
| `export.poptions` | preprocessor opts |
| `export.coptions` | compiler options |
| `export.loptions` | linker options |
| `export.libs` | libraries depend on |
| `export.imp_libs` | libraries to import |

### Bin Module Variables

| var | desc |
| :---: | --- |
| config.bin.target |  |
| config.bin.pattern
| [config.]bin.lib | library types to build |
| [config.]bin.{exe|liba|libs}.lib | library type to use |
| [config.]bin.rpath | |
| [config.]bin.rpath.auto | |
| [config.]bin.rpath_link | |
| [config.]bin.rpath_link.auto | |
| config.bin.{prefix|suffix} | |
| [config.]bin.{lib|exe}.{prefix|suffix} | prefix, suffix for library or executable |
| bin.whole | link dependencies into library |
| bin.binless | library is binless (header only) |
| bin.lib.load_suffix | |
| bin.lib.load_suffix_pattern | |
| bin.lib.version | |
| bin.lib.version_pattern | |

```
[project_name] project
```

There are more:
```
[string] project.summary, [string] project.url,

[target_triplet] cxx.target = x86_64-w64-mingw32
[string] cxx.target.class = windows
[string] cxx.target.cpu = x86_64
[string] cxx.target.system = mingw32
hxx{*}: [string] extension = hxx
cxx{*}: [string] extension = cxx
```

## Configuration

```
@config.import@

config.import.<proj>[.<name>[.<type>]
config.<proj>
import.<name>.<type>
import.<name>
```

	path to project, possibly target name and type


