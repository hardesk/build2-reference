# build2 reference

buildfile contains directives, target declarations and variable assignmens

## Directives

These seem to resemble functions in a programming language

### import, import?, import!
Import target from another project. Syntax is `import <name> = [<project>%]<target>`. Project is optional
Example:

`import mylib = libhello%lib{hello}
import anotherlib = lib{world}`
	
### assert, assert!
Check condition, write a message if condition evaluates to false. assert! inverts the condition

`assert ($cxx.target.cpu == 'x86_64') 'Only 64bit x86 is supported'

### print
Write argument to stdout
`print $(cxx.id)`

### fail, warn, info, text
Write diagnostics message to stderr. Fail also terminates the program.
`info 'will get an error next'`
`fail 'the end'`
`info 'unreached'`

### dump
Print the contents of the current scope (including nested scope) or target passed as argument to stderr.
`exe{hello}: {hxx cxx}{**}
cxx.poptions =+ "-I$out_root" "-I$src_root"
dump
dump exe{hello}`

### source

### include
Include a buildfile. Basically pull in text from the file specified, but additionally including it creates a new scope for the file content. Eg.
`include ../libhello` this will include ../libhello/buildfile

### run
Launch an external process with arguments as process args and parse output as a buildfile.
`run <name> [<arg>...]`

### export
Export target something something..
`export $out_root/libhello/$import.target`

### using, using?
Use a build2 module, eg.
`using cxx`

### define
Define an alias for target (?).
`define derived : base`

### if, if!, else, elif, elif!
Branch execution depending on condition result. `!` version inverts condition.
`if ($c.target.system == 'windows')
	info 'target is windows'`

`if false
{
	info 'will not print'
}`


### switch <arg, arg..> [: match-function]
### case, default
Pattern matching switch statement. Multiple values can be matched against a pattern consisting of multiple values as well. Also pattern can combine the values in different ways:
',', eg. case 'a', 'b' will match two arguments 'a' and 'b'.
'|', eg. case 'a' | 'b' will match either 'a' or 'b'

match function can be
	regex.match - regex match of the pattern against the full argument string 
	regex.search - regex search of a substring in argument
	path.match - use path-style matching
	icase - add this to ignore case
	also any function taking value, pattern and returning bool can be specified

switch 'x' {
	case 'x'
		info 'got x'
}

switch $cxx.target.class : regex.match icase
{
	case 'A.*'
		info 'A architecture'
}

### for
For loop.

`for n: foo bar baz
{
  exe{$n}: cxx{$n}
}`

### config
Declare a special config variable. The name/path has to conform to `config[.**].<project>.**`


`config [bool] config.libworld.greet ?= true`



### Configuration

@config.import@

	config.import.<proj>[.<name>[.<type>]
	config.<proj>
	import.<name>.<type>
	import.<name>

		path to project, possibly target name and type



