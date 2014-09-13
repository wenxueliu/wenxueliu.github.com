##input/output

cerr : standard error , typically used to generate warning and error messages 
clog : general information

The iostream library defines versions of the input and output operators that accept all of the
built-in types.

##Basic 

###Essentially all languages:
* Built-in data types
* Expressions and statements
* Variables
* Control structures
* Functions 

Integral Types: integer,characters and boolen

In C++ it is perfectly legal to assign a **negative number** to an object with **unsigned type**.

prefer int or long:
	run-time cost of doing arithmetic with long s can be considerably greater than 
	doing the same calculation using a 32-bit int 

prefer double to float:
	cost of double precision calculations versus single precision is negligible. In fact, on some machines,
	double precision is faster thansingle

keywords: 63 

the different between definitions and declarations:
	extern int i; //declares but does not define i
	int i;        //declares and defines i
	extern int i = 2; //definition
	extern int i = 2; //redefinition and behavior

scope : local scope, global scope,statement scope,class scope and namespace scope 

char str[] = {'a','b','c'} //not c-style string, strcpy() strlen() maybe disaster

The order of operand evaluation matters if one subexpression changes the value of an operand
used in another subexpression:

		// oops! language does not define order of evaluation
		if (ia[index++] < ia[index])

standard library
	cstdlib : EXIT_SUCCESS EXIT_FAILURE
	cstddef : size_t ptrdiff_t
	cstdlib : NULL


varing parameter
	void foo(parm_list,...)
	void foo(...)

a void function uses a return to cause premature termination of the function. 





Whenever an inline function is added to or changed in a header
file, every source file that uses that header must be recompiled.

overload

	int test(const int )
	int test(int a)      //redeclaration

	int test(Account *const a)
	int test(Account *a)		//redeclaration 

	int test(const Account *a)
	int test(Account *a)		//overload
	
	int test(const Account &a)
	int test(Account &a)		//overload
	
overload and Scopeq
	string init();
	void fun()
	{
		int init = 0;
		string s = init();
	}


	void print(const string &);
	void print(double);
	// overloads the print function
	void fooBar(int ival)
	{
		void print(int);// new scope: hides previous instances of print
		print("Value: "); // error: print(const string &) is hidden
		print(ival); // ok: print(int) is visible
		print(3.14); // ok: calls print(int); print(double) is hidden
	}
The declaration of print(int) in the function fooBar hides the other declarations of print .

###type conversion
implicit type conversions
	arthmetic conversions: integral promotions
Explicit conversions: static_cast const_cast reinterpret_cast dynamic_cast





###pointer


	char arr[0];// error: cannot define zero-length array
	char *cp = new char[0]; // ok: but cp can't be dereferenced

When we use new to allocate an array of zero size, new returns a valid, nonzero pointer. This
pointer will be distinct from any other pointer returned by new . The pointer cannot be
dereferencedafter all, it points to no element. The pointer can be compared and so can be used
in a loop such as the preceeding one. It is also legal to add (or subtract) zero to such a pointer
and to subtract the pointer from itself, yielding zero.

	size_t n = get_size(); // get_size returns number of elements needed
	int* p = new int[n];
	for (int* q = p; q != p + n; ++q)
	/* process the array */ ;

if the call to get_size returned 0, then the call to new would still
succeed. However, p would not address any element; the array is empty. Because n is zero, the
for loop effectively compares q to p . These pointers are equal; q was initialized to p , so the
condition in the for fails and the loop body is not executed.

	string s_str("abcde");
	const char c_str = s_str.c_str(); //OK
	char c_str = s_str(); //error invalid conversion from ‘const char*’ to ‘char*’

function pointer
	typedef bool (*pf)(string lstr, string rstr);//
	void func(string lstr, string rstr, bool (*)(string lstr, string rstr));//parameter as function pointer
	void func(string lstr, string rstr, bool(string lstr, string rstr));//parameter as function pointer

	void (*fun1(int))(string lstr, string rstr);// return function pointer
	void (*pf)(string lstr, string rstr)
	pf = fun1(int)
	func fun2(int) //error, a return type of function type
	func *fun3(int) // Ok, return a pointer to function type

Pointers to Overloaded Functions
	
	extern void ff(vector<double>);
	extern void ff(unsigned int);
	// which function does pf1 refer to?
	void (*pf1)(unsigned int) = &ff; // ff(unsigned)

If no function matches exactly, the initialization or assignment results in a compile-time error:

	// error: no match: invalid parameter list
	void (*pf2)(int) = &ff;
	// error: no match: invalid return type
	double (*pf3)(vector<double>);
	pf3 = &ff;


###iostream
No Copy or Assign for IO Objects 
	ofstream out1, out2;
	out1 = out2;
	// error: cannot assign stream objects
	// print function: parameter is copied
	ofstream print(ofstream);
	out2 = print(out2); // error: cannot copy stream objects

we cannot have a parameter or return type that is one of the stream types. If we need to pass or 
return an IO object, it must be passed or returned as a pointer or reference:

	ofstream &print(ofstream&); // ok: takes a reference, no copy
	while (print(out2)) { /* ... */ } // ok: pass reference to out2


	if (cin) // ok to use cin, it is in a valid state
	while (cin >> word) // ok: read operation successful ...


flush : endl  flush ends unitbuf





###array

parameter is a reference to an array; size of array is fixed

	void printValues(int (&arr)[10]) {}
	int main(){
		int i = 0;
		int j[2] = {0,1};
		int k[10] = {0,1,2,3,4};
		printValue(i); //error
		printValue(j); //error
		printValue(k); //OK 
		return 0;
	}

because the parameter is a reference, it is safe to rely on the 	size in the body of the function

arry as function arguments
1. the array has sign of end;
2. standard library
3. function(array, array_size)

###const :
const is only used in current file unless use extern const
		
	const for type switch:

		int i = 42;
		const int &a = i;
		const double &b = i; 
		std::cout << "a=" << a << std::endl;//42
		std::cout << "b=" >> b << std::endl;//42
		i = 43;
		std::cout << "a=" << a << std::endl;//43
		std::cout << "b=" >> b << std::endl;//42 

	reason:
		double tmp = i;
		const double b = tmp;

	append: 
		if you change the double to char, you will find something more.

constant varable: pg 59
* define macro
* const 
* Enumerations: provide an alternative method of not only defining but also grouping sets of integral constants.

	int i_val = 0;
	const int ci_val = 0;
	int *ip = i_val; //error
	int *ip = ci_val;//OK ci_val is const with complie-time value of 0.
	int *ip = 0; //Ok
	int *ip = NULL;


	typedef string *pstring;
	const pstring cstr1; // string* const cstr1;
	pstring const cstr2; // string* const cstr2;

const int p_val = new const int[10]   //error
const int p_val = new const int[10]() //OK, but it's meaningless.
const int p_str = new const string()  //OK


###void
	double d_val = 1.0;
	int i_val = 1;
	char *c_val = "abc";
	viod pd_val = &d_val;
	void pi_val = &i_val;
	void pc_val = c_val;

We can compare it to another pointer, we can pass or return it from a function, and we can assign it to another
void* pointer. We cannot use the pointer to operate on the object it addresses.



###string
When mixing string s and string literals, at least one operand to each + operator must be of string type
	string str("abc");
	string str1 = str + "cde" + "\n";//OK
	string str2 = "abc" + "cde" + str;//error

string:size_type
	size() // return 
	[index]// index


###vector
vector<T> vtr;
* if T has default constructor, the initialize vtr with the default constructor.
* if T has constructor(without default constructor), initialize the element of vtr with constructor.
* if T hasn't constructor, the library still creates a value-initialized object.which do so by value-
initializing each member of that object. 

An element must exist in order to subscript it; elements are not added when we assign through a subscript.

const vector<T>::iterator : an iterator whose value cannot change
vector<T>::const_iterator : an iterator that cannot write elements




##file struct 
###head file 
Based on headers are for declarations, not definitions
* class definition
* extern variable declarations
* function declarations (exception)
* definition of inline function (exception)
* const objects which is initialized by a constant expression(whose value is known at compile time) (exception)

Avoiding multiple inclusions
* define



###source file



##Concept
strong type check ?
statically typed : check the the operation at compile time, so we can find bug in our programs earlier




##best practices
We recommend that every object of built-in type be initialized. It is not always necessary to initialize such
variables, but it is easier and safer to do so until you can be certain it is safe to omit an initializer.

initilize a empty vector then  add element gradually and dynamiclly 

Because there are no guarantees for how the sign bit is handled, we strongly recommend using an unsigned type
when using an integral value with the bitwise operators.
