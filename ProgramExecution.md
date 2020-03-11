# Program execution
* The OS loads the code and data part in the memory
* Loads PC register with first instruction of `main`
* Declaration of variables does not populate the stack, it is only populated when the variable is initialised
* Dynamically allocated memory is provided through heap
* On a function call - 
	* push a frame on stack
	* the returning address is pushed in the frame so that after completion of function execution, the PC is loaded with return address
	* the function arguments, local variables and results are pushed in the function's frame
	* when return is executed, the expression to be returned is preserved through registers and then assigned later on