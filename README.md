Fantomas
========

![Fantomas logo](fantomas_logo.png)

F# source code formatter, inspired by [scalariform](https://github.com/mdr/scalariform) for Scala, [ocp-indent](https://github.com/OCamlPro/ocp-indent) for OCaml and [PythonTidy](https://github.com/acdha/PythonTidy) for Python.

[![Build Status Travis](https://travis-ci.org/fsprojects/fantomas.png)](https://travis-ci.org/fsprojects/fantomas)
[![Build Status AppVeyor](https://ci.appveyor.com/api/projects/status/github/nojaf/fantomas)](https://ci.appveyor.com/project/nojaf/fantomas)

## Purpose
This project aims at formatting F# source files based on a given configuration.
Fantomas will ensure correct indentation and consistent spacing between elements in the source files.
We assume that the source files are *parsable by F# compiler* before feeding into the tool.
Fantomas follows the formatting guideline being described in [A comprehensive guide to F# Formatting Conventions](docs/FormattingConventions.md).

## Use cases
The project is developed with the following use cases in mind:

 - Reformatting an unfamiliar code base. It gives readability when you are not the one originally writing the code. 
To illustrate, the following example

	```fsharp
	type Type
	    = TyLam of Type * Type
	    | TyVar of string
	    | TyCon of string * Type list
	    with override this.ToString () =
	            match this with
	            | TyLam (t1, t2) -> sprintf "(%s -> %s)" (t1.ToString()) (t2.ToString())
	            | TyVar a -> a
	            | TyCon (s, ts) -> s
	```
	will be rewritten to

	```fsharp
	type Type = 
	    | TyLam of Type * Type
	    | TyVar of string
	    | TyCon of string * Type list
	    override this.ToString() = 
	        match this with
	        | TyLam(t1, t2) -> sprintf "(%s -> %s)" (t1.ToString()) (t2.ToString())
	        | TyVar a -> a
	        | TyCon(s, ts) -> s
	```

 - Converting from verbose syntax to light syntax. 
Feeding a source file in verbose mode, Fantomas will format it appropriately in light mode.
This might be helpful for code generation since generating verbose source files is much easier.
For example, this code fragment

	```fsharp
	let Multiple9x9 () = 
	    for i in 1 .. 9 do
	        printf "\n";
	        for j in 1 .. 9 do
	            let k = i * j in
	            printf "%d x %d = %2d " i j k;
	        done;
	    done;;
	Multiple9x9 ();;
	```	
	is reformulated to 
	
	```fsharp
	let Multiple9x9() = 
	    for i in 1..9 do
	        printf "\n"
	        for j in 1..9 do
	            let k = i * j
	            printf "%d x %d = %2d " i j k
	
	Multiple9x9()
	```

 - Formatting F# signatures, especially those generated by F# compiler and F# Interactive.

For more complex examples, please take a look at F# outputs of [20 language shootout programs](tests/languageshootout_output) and [10 CodeReview.SE source files](tests/stackexchange_output).

## How to use
### VS 2015
Fantomas is a part of [Visual F# Power Tools][vfpt] extension compatible with
Visual Studio 2015. The extension could be installed from [Visual Studio
Gallery][vfpt-gallery]. The usage instructions are the same as for VS 2012
extension.

[vfpt]: https://github.com/fsprojects/VisualFSharpPowerTools
[vfpt-gallery]: https://visualstudiogallery.msdn.microsoft.com/136b942e-9f2c-4c0b-8bac-86d774189cff

### VS 2012 and 2013 extension
[Ivan Towlson](https://github.com/itowlson) kindly contributes the initial version of [Fantomas VS extension](src/Fantomas.VisualStudio). The user guide can be found [here](docs/Documentation.md#using-visual-studio-2012-extension).
This is available in the [Visual Studio Gallery](http://visualstudiogallery.msdn.microsoft.com/24ef5c87-b4e3-4c3b-b126-1064cc66e148) - search for "fantomas" in "Tools --> Extensions and Updates --> Online". 

    Ctrl + K D   -- format document
    Ctrl + K F   -- format selection / format cursor position

You can also use Fantomas extension in Ivan's [fsharp-vs-commands](https://github.com/itowlson/fsharp-vs-commands) project.

### Command line tool / API 
Use this command to install Fantomas as a dotnet SDK global tool:

```
dotnet tool install -g fantomas-tool
```

For detailed guidelines, please read [Fantomas: How to use](docs/Documentation.md#using-the-command-line-tool).

### FAKE build system
Fantomas can be easily integrated with FAKE build system. Here is a sample `build.fsx`:

```fsharp
#r "packages/FAKE/tools/FakeLib.dll"
#r "packages/Fantomas/lib/FantomasLib.dll"

open Fake
open Fantomas.FakeHelpers
open Fantomas.FormatConfig

// Properties
let buildDir = "./build/"
let fantomasConfig =
    { FormatConfig.Default with
            PageWidth = 120
            ReorderOpenDeclaration = true }

Target "CheckCodeFormat" (fun _ ->
    !! "src/**/*.fs"
      |> checkCode fantomasConfig
)

Target "FormatCode" (fun _ ->
    !! "src/**/*.fs"
      |> formatCode fantomasConfig
      |> Log "Formatted files: "
)

RunTargetOrDefault "CheckCodeFormat"
```

### Trying Fantomas online
[FantomasWeb](https://github.com/TahaHachana/FantomasWeb), implemented by [Taha Hachana](https://github.com/TahaHachana), is accessible at http://fantomasweb.apphb.com/.

### Fantomas plugin in Tsunami IDE
Taha also wrote a [blog post](http://fsharp-code.blogspot.dk/2013/04/fantomas-support-in-tsunami-ide.html) on integrating Fantomas into Tsunami IDE.

## Installation
The code base is written in F# 3.0/.NET framework 4.0. 
The solution file can be opened in Visual Studio 2012, Visual Studio 2013 and MonoDevelop/Xamarin Studio.
NuGet is used to manage external packages.
The [test project](src/Fantomas.Tests) depends on FsUnit and NUnit.
However, the [library project](src/Fantomas) and [command line interface](src/Fantomas.Cmd) have no dependency on external packages.

## Testing and validation
We have tried to be careful in testing the project.
There are 209 unit tests and 30 validated test examples, 
but it seems some corner cases of the language haven't been covered.
Feel free to suggests tests if they haven't been handled correctly.

## Why the name "Fantomas"?
There are a few reasons to choose the name as such. 
First, it starts with an "F" just like many other F# projects. 
Second, Fantomas is my favourite character in the literature. 
Finally, Fantomas has the same Greek root as "[phantom](https://en.wiktionary.org/wiki/phantom)"; coincidentally F# ASTs and formatting rules are so *mysterious* to be handled correctly.

## How to contribute
Would like to contribute? Discuss on [issues](../../issues) and send pull requests. You can get started by helping us handle ["You Take It" issues](https://github.com/dungpa/fantomas/issues?labels=You+Take+It&page=1&state=open).

To get an understanding of the code, either:
- start by looking at contribution examples (see [Contribution examples](#contribution-examples) section below);
- or read the few following [architectural notes](#architectural-notes) & tips to help you get started playing with the code.

### Architectural notes
Fantomas' features are basically two commands: *format a document* or *format a selection* in the document.

They both consist in the same two stages:

- parse the code and generate the F# AST (Abstract Syntax Tree). This is provided by the 
by the FSharp.Compiler.Services library (see the `parse` function in
[CodeFormatterImpl.fs](src/Fantomas/CodeFormatterImpl.fs)).
- rewrite the code based on the AST (previous step) and formatting settings.

The following sections describe the modules/function you will most likely be
interested in looking at to get started.

#### The test project: Fantomas.Tests
The organization is really simple. For each F# language feature/constructs,
there is a `[Feature]Test.fs` file. Examples:
 - *StringTests.fs*
 - *UnionTests.fs*
 - ...

Most of the tests are really simple and have this simple algorithm:
_assert that **format \[F# CODE\]** is equal to **\[FORMATTED F# CODE\]**_.

Example (from *UnionTests.fs*):
```fsharp
[<Test>]
let ``discriminated unions declaration``() =
    formatSourceString false "type X = private | A of AParameters | B" config
    |> prepend newline
    |> should equal """
type X = 
    private
    | A of AParameters
    | B
"""
```

#### The `CodePrinter.genParsedInput` function: rewrites formatted code
`CodePrinter.genParsedInput` (see [CodePrinter.fs](src/Fantomas/CodePrinter.fs)): what it
basically does is traversing the AST corresponding to the code to format, and
rewriting it according to the provided formatting options.

#### The `FormatConfig` type: format settings
Settings such as :
 - indent values in spaces
 - reorder open declarations
 - ...

See [CodePrinter.fs](src/Fantomas/CodePrinter.fs).

### How to play with fantomas on F# Interactive
The [CodeFormatter.fsx](src/Fantomas/CodeFormatter.fsx) script file
allows you to test the code formatting behavior. See the function `formatSrc: string -> unit`
that formats the string in input and prints it.

### Contribution examples
The time it took to contribute is sometimes mentioned, as a side note.

#### Fixing code generation
 - Record type formatting generated invalid F# code in some cases ([#197](https://github.com/dungpa/fantomas/pull/197)) - (2h fix)  

## Credits
We would like to gratefully thank the following persons for their contributions.
 - [Eric Taucher](https://github.com/EricGT)
 - [Steffen Forkmann](https://github.com/forki)
 - [Jack Pappas](https://github.com/jack-pappas)
 - [Ivan Towlson](https://github.com/itowlson)
 - [Don Syme](https://github.com/dsyme)
 - [Gustavo Guerra](https://github.com/ovatsus)
 - [Jared Parsons](https://github.com/jaredpar)
 - [Denis Ok](https://github.com/OkayX6)
 - [Enrico Sada](https://github.com/enricosada)

## License
The library and tool are available under Apache 2.0 license. 
For more information see the [License file](LICENSE.md).
