# Compiling

## Compiling
* Transformation from source code (human readable) into machine code (computer executable)
* Compiler is a program
* Input is a program written in high level language
* Output is machine language

## Brief program life cycle
* write
* compile
* link
* load
* execute

## Stages in C compiling

1. **Preprocessing**
	* Macro expansion
	* header file expansion
	* conditional compilation instructions
2. **Compilation**
	* Generates assembler source code
3. **Assembly**
	* takes the assembly source code an assembly listing with offsets
	* assembler output is stored in an object file
4. **Linking**
	* takes one or more object files/libraries as input
	* combines them to produce a single executable file
	* resolves references to external symbols, final addresses of procedures/functions and variables
	* relocation - revise code and data to reflect new addresses

## Static linking vs Dynamic linking
### What is linking?
It is the process of bringing external programs together for successful execution of a given program.

| Static linking | Dynamic linking |
| --- | --- |
| External libraries/programs are copied and tied together with the given programs code | External libraries/programs are referenced while there object code remains in memory and can be shared by many programs |
| Last step of compilation | Performed at runtime by OS |
| Larger in sizes | Size of executable is small |
| If anything is changed, everything needs to be relinked | Individual library can be updated without affecting others |
| Program execution is faster | Slower |
| Less prone to compatibility issues | Prone to compatibility issues |
