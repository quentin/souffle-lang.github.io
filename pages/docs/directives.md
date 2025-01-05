---
title: Directives
permalink: /directives
sidebar: docs_sidebar
folder: docs
---
Soufflé has directives controlling the I/O behaviour of relations, and their execution behaviour.
The I/O system for relations supports various data-sources for input and output using various formats.
Soufflé supports terminal output, file I/O, and I/O utilising a SQLite as a database. 
For a relation in a Datalog program, several input and output directives can be issued. 
Soufflé supports directives to limit the number of tuples per relation. 

## Input Directive
A Soufflé program may load the facts of a relation (aka. as EDB) from various input sources.
The input source is specified by the `.input` directive of a relation.
The default input source is a tab-separated file for a relation where
each row in the tab-separated file represents a fact in the relation. 

For example, 
```prolog
.decl A(a:number,b:number)
.input A 
```
defines an input relation `A` with two number attributes. 
The input directive `.input` make the EDB read from 
a tab-separated file `A.facts`.


For example,
```prolog
.decl A(a:number,b:number)
.input A(IO=file, filename="path/to/infile", columns="4:7", delimiter=",")
```
This will load the relation data from file (the default if no IO type is specified) using the filename "path/to/infile" and a comma as a delimiter, and will use columns four and seven (zero-indexed) for input data.
```prolog
.decl A(a:number,b:number)
.input A(IO=stdin, columns="4:7", delimiter=",")
```
Data can also be read from stdin, and again the delimiter and columns to read can be specified.
```prolog
.decl A(a:number,b:number)
.input A(IO=sqlite, dbname="path/to/sqlite3db")
```
Data will now be read from an sqlite3 database at the given path.
The data is expected to be stored in a table matching the relation name
prefixed by an underscore and the sqlite3 database is expected to contain a
view matching the relation name. For example, for the relation `edge`, the
sqlite3 database should have a table named `_edge` containing the data and a
view named `edge` that returns the data from `_edge`.

A more detailed specification for each relation can be defined in the input directive. 
```prolog
.input <relation id> (IO=[file|stdin|sqlite] [optional parameters])
```

### IO=file
`filename`
Note that if the `-F<path>` command line option is used, that path will be prepended to the filename, unless the filename path is absolute.
 
`delimiter`
Used to specify the delimiter to separate columns in the input file. The default value is a tab character.

`columns`
Particular columns can be selected from an input file. `columns="2:0"` will use the third input file column for the first attribute of the relation, and the first input file column for the second attribute. The default is to use the first columns of the input file until all attributes are described.

`compress`
Input is assumed to be in gzip compressed format. By default, gzip compressed input is automatically detected.

`headers`
The header can be enabled by `headers=true` or surpressed by `headers=false` (which is default) for CSV files. 

`rfc4180=[false|true]`
Enables the [rfc4180](https://datatracker.ietf.org/doc/html/rfc4180) compatibility mode. Delimiter is set to `,` by default and double-quotes are only allowed to enclose fields. Two consecutive double-quote characters `""` in a double-quote enclosed field encode for a single double-quote in the actual value. The `"` delimiter is forbidden.

### IO=stdin
`delimiter`
Used to specify the delimiter to separate columns in the input file. The default value is a tab character. The order of evaluation of relations is not fixed. This method is not reliable when reading more than one relation from stdin.

### IO=sqlite
`filename`
The path to the sqlite3 database. Note that if the `-F<path>` command line option is used, that path will be prepended to the filename, unless the filename path is absolute.

## Output Directive
The output relations of a Datalog program are, by default, written to a tab separated file with name `<relation name>.csv`, located in the current directory. If the parameter `-D<output-dir>` is given then the default output directory will be changed to that given. `-D-` can be used to redirect all output to stdout.

For example, the relation  
```prolog
.decl result(a:number, b:number, c:symbol)
.output result
```
has three number columns that are written to the file `result.csv` in the current directory, or `<output-dir>` when using the flag `-D <output-dir>`.

As for input more detailed specification for each relation can be defined in the output directive. For example,
```prolog
.decl A(a:number,b:number)
.output A(IO=file, filename="path/to/outfile", delimiter=":")
```
This will store the relation data to file (the default if no IO type is specified) using the filename "path/to/outfile" and a colon as a delimiter.
```prolog
.decl A(a:number,b:number)
.output A(IO=stdout, delimiter=",")
```
Data can also be sent to stdout, and again the delimiter can be specified.
```prolog
.decl A(a:number,b:number)
.output A(IO=sqlite, dbname="path/to/sqlite3db")
```
Data will be written to an sqlite3 database at the given path.
The data will be stored with a view that shows the symbols and numbers contained in the relation.
This is backed by a database table to store the symbol table and a table to store the relations numbers or symbol indices.

The output can be parameterised:

```prolog
.output <relation id> (IO=[file|stdout|sqlite] [optional parameters])
```

### IO=file
`filename`
Note that if the `-D<path>` command line option is used, that path will be prepended to the filename, unless the filename path is absolute.

`delimiter`
Used to specify the delimiter to separate columns in the input file. The default value is a tab character.

`compress`
Output is in gzip compressed format.

`rfc4180=[false|true]`
Produces an rfc4180 compliant CSV file. Delimiter is set to `,` by default and all fields are double-quote delimited.  The `"` delimiter is forbidden.

### IO=stdout
`delimiter`
Used to specify the delimiter to separate columns in the input file. The default value is a tab character.

### IO=sqlite
`filename`
The path to the sqlite3 database. Note that if the `-D<path>` command line option is used, that path will be prepended to the filename, unless the filename path is absolute.

### I/O Types
Attributes types are signed numbers, unsigned numbers, float, strings, and records. If for primitive types (i.e. numbers, unsigned, and float) wrong input values are provided while reading from a data-source, an error will be issued. 

Records are written in a recursive format for input and output directives. A recursive data-structure is expanded completely
and printed.  For example:
```prolog
.type List = [data:number, next:List]
.decl A(l:List,y:number)
A([1,[2,[3,nil]]],10). 
.output A
```

Produces following output:
```
---------------
A
l	y
===============
[1, [2, [3, nil]]]	10
===============
```

ADTs are written in a serialized format for input and output directives. An ADT is expanded with its branch name and printed. For example:
```prolog
.type Location = City {name: symbol} | Country {name: symbol}
.decl Resident(l:Location, p:symbol)
Resident($City("Sydney"), "Bob").
Resident($Country("Australia"), "Bob").
.output Resident
```

Produces the following output:
```
---------------
Resident
l       p
===============
$City(Sydney)   Bob
$Country(Australia)     Bob
===============
```

To read or write arbitrary ADTs, complex records, and symbols without ambiguity, you may use the `rfc4180=true` I/O option. For example:
```prolog
.type ListOfSymbols = [data:symbol, next:ListOfSymbols]
.type ADT = One {a:number} | Two {b:number, c:number}
.decl A(l:ListOfSymbols,y:symbol,z:ADT)
A(["comma=,",["double-quote=\"",["space= ",["bracket=]",["nil",nil]]]]],"double-quote=\" comma=,",$Two(10,20)).
.output A(rfc4180=true,delimiter="\t")
```

Produces following output:
```
---------------
A
l       y       z
===============
"[""comma=,"", [""double-quote=\"""", [""space= "", [""bracket=]"", [""nil"", nil]]]]]"	"double-quote=\"" comma=,"	"$Two(10, 20)"
===============
```

Compared to (without the `rfc4180` option):

```
---------------
A
l       y       z
===============
[comma=,, [double-quote=", [space= , [bracket=], [nil, nil]]]]]	double-quote=" comma=,	$Two(10, 20)
===============
```

## Printsize Directive

The printsize directive prints the name and number of tuples in the relation (separated by a tabulator symbol). 

For example,
```prolog
.decl A(x:number)
A(1).
A(2).
.printsize A
```
prints the name `A` and the number 2, i.e., 
```
A   2
```

## Limit-Size Directive 
Soufflé can stop the fix-point calculation after a certain number of 
tuples have been computed. This number is a soft-limit and should be
used for debugging purposes only.

The main application for the limit-size directive is to debug non-terminating or slow-terminating programs.
After the limit-size directive is set, the program will terminate early and the output of the program
can be inspected.  For example, the program
```prolog
.decl A(x:number)
A(1).
A(x+1) :- A(x), x< 1000.
.output A

.limitsize A(n=47)
```
computes the first 47 numbers in set `A` rather than 1000 numbers. 
This is a consequence of the limitsize directive that stops the fix-point computation when the size of `A` reaches 47 tuples or more. 

Note that if there are other mutual recursive relations in the same stratum, they will be stopped as well. 

## Syntax 
In the following, we define directive declarations in Soufflé more formally using [syntax diagrams](https://en.wikipedia.org/wiki/Syntax_diagram) and [EBNF](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form). The syntax diagrams were produced with [Bottlecaps](https://www.bottlecaps.de/rr/ui).

### Qualified Name

![Qualifier Name](https://souffle-lang.github.io/img/qualified_name.svg)

A qualified name is a sequence of identifiers separated by `.` to disambiguate relations that are instantiated by components.

```ebnf
qualified_name ::= IDENT ( '.' IDENT )*
```

### Directive 

![Directive](https://souffle-lang.github.io/img/directive.svg)

A directive changes the I/O or execution behaviour for a relation or a list or relations. The relation qualifier specifies the type of the directive, i.e., input, output, printsize, and limitsize. Relations in a directive are separated by a comma symbol. Each directive may have parameters. The parameters can be ommitted. Parameters are listed in parenthesis. Parameter has an identifier which is assigned a directive value. 

```ebnf
directive ::= directive_qualifier qualified_name ( ',' qualified_name )* ( '(' ( IDENT '=' directive_value ( ',' IDENT '=' directive_value )* )? ')' )?
```

### Directive Qualifier

A directive qualifier defines the type of the directive. 

![Directive Qualifier](https://souffle-lang.github.io/img/directive_qualifier.svg)

```ebnf
directive_qualifier  ::= '.input' | '.output' | '.printsize' | '.limitsize'
```

### Directive Value

A directive value defines values of parameters. They can either be identifiers, symbols, numbers, and `true` and `false` values.  

![Directive Value](https://souffle-lang.github.io/img/directive_value.svg)

```ebnf
directive_value ::= STRING | IDENT | NUMBER | 'true' | 'false'
```

### Legacy Syntax
Older versions supported I/O qualifiers in the relation declaration such as 
```prolog
.decl A(x:number, y:symbol) input
.decl B(x:number, y:symbol) output
```
that should be rewritten to 
```prolog
.decl A(x:number, y:symbol)
.input A
.decl B(x:number, y:symbol) 
.output B
```
Soufflé still supports the legacy syntax with command line option `--legacy`, but a warning message will be issued.
