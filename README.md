# build2 reference

A half-assed attempt to make a build2 reference. Information is extracted from the manual novel. Read it first (at least basics).

buildfile contains directives, target declarations and variable assignmens. newlines matter.

## Files

### ./build/bootstrap.build
A file read first. Names the project and must contain using directives to load modules.

|variable|description|
|:---:|---|
|project|name of the project|
|subprojects|local paths to subprojects. Listed dirs will be searched for buildfiles|
|amalgamation| ??? |


### ./build/root.build
In addition to what bootstrap contains, holds shared properties for targets in the project. Here `src_root` is defined while `src_base` is not (because no project is loaded yet).

When running buildfile from a subfolder, module import paths are based off buildfile. This means that import must be prefixed with `$src_root` if it's project relative e.g.
```
config.import.fmt=$src_root/libs/fmt # will make fmt%lib{fmt} available to import
```

## Functions

For path and most other functions, the argument can be `path`, `dir_path` or lists thereof. When the arg is `dir_path`, directory separator is stuck/appended at the end of the resulting value. 

| function | description |
| :---: | --- |
|`$type(arg)`| return type of `arg` |
|`$null(arg)`|check `arg` for null (return `true`)|
|`$empty(a)`||
|`$identity(a)`||
|`$getenv(s)`| get value of env variable `s`|
|`$path_search(pattern [, directories])`| search for paths in `directories` using `pattern` |
|`$icasecmp(a,b), $stirng.icasecmp(a,b)`|compare strings `a` and `b` case insensitive|
|`$trim(a), $string.trim(a)`|trim whitespace around for `a`|
|`$install.resolve(p) `| resolve relative path `p` to absolute path where `p` is supposed to be installed|
|`$canonicalize(p), $path.canonicalize(p) `| make path adhere to OS path separators|
|`$normalize(p), $path.normalize(p)`| collapse .. in path. `$normalize(a/../b)` -> b |
|`$directory(p), $path.directory(p)`| return directory part of `path`. `$directory(a/b/c)` returns `a/b/` |
|`$base(p), $path.base(p)`| base of filename, ie. name with extension dropped |
|`$extension()`||
|`$name()`||
|`$leaf(p), $path.leaf(p)`| return the most leaf element of the path. everything except the most root element, `$leaf(a/b/c)` -> `c` (or `c/` if dir_path) |
|`$path.match(s, re[, ?]), $match(s, re[, ?])`| match `s` against regexp `re` |
|`$regex.match(s, re[, flags])`| |
|`$regex.search(s, re[, flags])`| |
|`$regex.replace(s, re, rep[, flags])`| replace `re` in `s` with `rep`. eg. `$regex.replace('a.cxx\nb.cpp\n', '(.*).cxx', '\1.cpp', return_lines)` |
|`$regex.replace_lines(s, re, rep[, flags])`| replace `re` in `s` with `rep`. eg. `$regex.replace_lines('a.cxx\nb.cpp\n', '(.*).cxx', '\1.cpp', return_lines)` |
|`$regex.apply(s, re, ?)`| |
|`$regex.find_match(s, re)`| |
|`$regex.find_search(s, re)`| search using `re` in `s` |
|`$regex.merge(s, ...)`| |
|`$regex.split(s, re, ?)`| | split `s` using regexp `re` |
|`$process.run(p)`| launch process `p` (with optional arguments) |
|`$process.run_regex(p, re [,repl])`| launch process `p` and parse output using regex `re`. Can also replace output by specifying `repl` |
|`$name.name()`||
|`$name.extension()`||
|`$name.directory()`||
|`$name.target_type()`||
|`$name.project()`||

Possible flags for regex functions:
icase, format_first_only, format_no_copy, return_lines, return_subs, return_match.

## Targets

- `exe` executable
- `lib[a,s]` library, also can be specifically set as archive or shared
- `libu[e,l]` utility library for executable or library (wtf is this???)
- `obj[e,a,s]` object, also executable, archive or shared specific

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

objs{string}: # as opposed to obja{string}
{
	cxx.poptions += -DLIB_DYNAMIC_EXPORT
}
```


Also, scope can be introduced which is equivalent to reading a buildfile from a folder named as scope. Here in ./buildfile we reference ./tool/iron.hpp/cpp.

```
tool/
{
	exe{hammer}: {hxx cxx}{iron}
}
```

## Directives

These resemble functions in a programming language.

### import, import?, import!
Import target from a project. Syntax is `import <name> = [<project>%]<target>`. Target can may be in another project: `config.import.<project>=<path>` must be declared is such case. <project> is skipped if target is in "this" project. This is called project-local importation.
Example:
```
import mylib = libos%lib{threading} # threading target from libos project
import lib = lib{printer} # printer target from this project
```
	
### assert, assert!
Check condition, write a message if condition evaluates to false. assert! inverts the condition.
```
assert ($cxx.target.cpu == 'x86_64') 'Only 64bit x86 is supported'
```

### print
Write argument to *stdout*
```
print $(cxx.id)
```

### fail, warn, info, text
Write diagnostics message to *stderr*. Fail also terminates the program.
```
info 'will get an error next'
fail 'the end'
info 'unreached'
```

### dump
Print the contents of the current scope (including nested scope) or of the target passed as an argument to stderr.
```
exe{hello}: {hxx cxx}{**}
cxx.poptions =+ "-I$out_root" "-I$src_root"
dump
dump exe{hello}
```

### source
Include a buildfile as if copy-pasting the contents of the file. File can be included multiple times.

### include
Include a buildfile guarding against multi-include. Pull in text from the file specified (or implied), but additionally create a new scope for the file content. Eg.
```
include ../libhello
```
this will include ../libhello/buildfile

### run
Launch an external process passing arguments and parse output as a buildfile.
```
run <name> [<arg>...]
```

### export
Export target something something..
```
export $out_root/libhello/$import.target
```

**build/export.build** file describes how to export a target out of a project. Or rather, how to import it, as the file is read and parsed by the importing pary when the target (project?) is used in *another project*. A few variables are defined while parsing it (`src_base` and `out_base` are undefined):

| variable | description |
| --- | --- |
| `src_root` | source root of the project being imported (this project) |
| `out_root` | output root of the project being imported (this project) |
| `import.target` | name of the target being imported (file is sourced by the **importing** party), eg `lib{tool}` |

The file should include the buildfile for the target being exported/imported (this actually describes the target) and actually export it using `export` directive. See variables that are relevant to target [exportation](#export-variables), e.g.:

```
$out_root/
{
	include buildfile
}
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
config [string] config.libworld.greet ?= "hello"

# in libworld/buildfile
cc.poptions += "-DGREET=$config.libworld.greet"
```

Afterwards when importing the target from the project, the variable can be set and it will be "passed" to the imported target.
```
config.libworld.greet = "hi"
config.import.libworld=$src_root/libs/libworld
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
| `<P>.export.poptions` | preprocessor opts |
| `<P>.export.coptions` | compiler options |
| `<P>.export.loptions` | linker options |
| `<P>.export.libs` | libraries depend on |
| `<P>.export.imp_libs` | libraries to import |

### Bin Module Variables

| var | desc |
| :---: | --- |
| `config.bin.target` |  |
| `config.bin.pattern`
| `[config.]bin.lib` | library types to build |
| `[config.]bin.{exe,liba,libs}.lib` | library type to use |
| `[config.]bin.rpath` | |
| `[config.]bin.rpath.auto` | |
| `[config.]bin.rpath_link` | |
| `[config.]bin.rpath_link.auto` | |
| `config.bin.{prefix,suffix}` | |
| `[config.]bin.{lib,exe}.{prefix,suffix}` | prefix, suffix for library or executable |
| `bin.whole` | link dependencies into library |
| `bin.binless` | library is binless (header only) |
| `bin.lib.load_suffix` | |
| `bin.lib.load_suffix_pattern` | |
| `bin.lib.version` | |
| `bin.lib.version_pattern` | |

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


