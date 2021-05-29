# Section-3 Exception

* exceptions should be caught by reference
* Note that
    ```
    function()
    {
        throw string("abc") <-- creates a new object there whose scope is ?? (i guess, till the catch clause)
    }

    ..
    caller()
    {
        try {
            function();
        }
        catch (string &e) { <-- better be &e

        }
    }
    ```

# Section-4 Standard Exception

std::exception is the base class for exception
std::bad_alloc is the exception for out of memory

```
virtual const char* what() const throw();
```
is availabe in std::exception

# Section-5 Custom Exception

void some_function_defn() throw();  // throw() means it can't throw an exception

* Notes on what is throw() specification. Updated in cpp_notes

# section-6 Exception catching order

Derived first.

# section-7 writing text files

* These are file-stream classes
    ofstream
    fstream
* .open(filename) method.
    * if you use plan fstrea, use ios::out as a 2nd arg to .open
    * ios::binary is a extra flag to OR to specify mode
* .is_open() method to check if its open
* .close() method

# section-8 Reading text file

* similar - fstream,ios::in or ifstream

* getline(filestreamvar, line);
    * this is not a member. take the file-stream as arg
* !filestreamvar.eof()
* filestream has operator bool() overloaded with eof() meaning.

# section-9 Parsing text files

* getline(filestreamvar, retstring, delimiterChar);
* .get()
    * get just one char.
* std::ws is a way to get all whitespace
    `filestream >> std::ws`

# section -10 Structs and Padding

```
#pragma pack(push, 1)
#pragma pack(pop, 1)
```

# section 11 Reading and Writing binary files

# section 12 vectors

* vector has no size-check. Its undefined like array!

# section 13 vectors and memory

vector.size()
vector.capacity()
vector.resize()
vector.reserve()

# section 14 : two-dim-vectors

note that vector's constructor first arg is initial size, second is the object that will be copy constructed

# section 15: list

```
std::list<int> numbers;
numbers.insert(it, 100); /* noe that the it stays same. THe elem is inserte before */

newit = numbers.erase(it); /* it is gone and shouldn't be used */
                           /* it return the it after the one that was erased */
```

# section 16: maps

```
make_pair(varA,varB); /* tries to make the right pair<TypeA,TypeB> depending on varA,varB types
```

# section 17: Using custom values in maps

* You should have a default constructor to participate in maps
    * very likely because `map[non_exist_key]` needs a default-constructor.
* You also very liekly want a copy-constructor as mind-you the stl's happily copy by value.

# section 18: Using custom values in keys

* the key , the `it->first` in maps is always const
* signature of less-than
    ```
    bool operator<(const ClassName &other) const {}
    ```

# section 19: Multimap

* Get the start and end (that is out of range) to the key-value given
    ```
    pair<iteratortype,iteratortype> it = multimapVar.equal_range(key_value);
    ```

# section 20: sets

* setVar.count(keyval) will return 1/0 if its found/not-found.
* multi-set will give the count of elements.
* set doesn't need a default constructor

# section 21: stacks and queues

* stack has a .top()  which just peeks
* stack has a .pop()  which deletes off.
* we cant iterator through a stack
* queue has a .front() , and a .pop() . Has a push()
* queue has a .back()  

# section 22: Vectors, sorting, deque and friend

* sort(firstIterator, lastIterator, optionalComparatorFunction)
* friend function-declaration.
* deque has both push_front/push_back and pop_front/pop_back

# section 33: Complex stl types

# section 24: operator overloading

test2 = test1;
is same as
test2.operator=(test1);

In copy construtor, you can do *this=other;
to leverate the assignment operator semantics

signature:

# sectoin 25: overloading left-shift operator

* NOte that bit-shift has left to right associativity

    ```
    int x = a + b + c;  is same as  x = a + ( b + c );

    int x = a << b << c;  is same as x = (x << b ) << c;
    ```
* friend functions can be defined inside clas as well. They aren't methods.

* signature
    ```
    ostream& operator<<(ostream &out, const UserClass &obj);
    ```

# sectio 26 . compelx class

# section 27. overloading +

* operator+() is best a non-member so that (regular_int + your_class) works.
* :e

# section 32 : template functions

You can use `template <class T>` or `template<typename T>`

# section 33: Template functions and type inference

```
template<typename T>
void print(T n)
{
    cout << n << endl;
}

void print(int n)
{
    cout << n << endl;
}

..
print<string>("hello");   // explicitly choose the template version and its type as string
print("hello");           // automatically chooses the template version and uses string
print<int>(5);            // explicitly choose the template version and its type as int.
print(5);                 // automatically chooses the non-template version and teh int-overload
print<>(5);               // explicilty chooses the template version and int type is auto-picked.
```

* Siutation where expclitly choosing template type is mandatory

```
template<typename T>
void showdefault()
{
    cout << T() << endl;
}

showdefault();       // error .. compiler can't inter what typename to use
showdefault<>();     // error .. compiler can't inter what typename to use
showdefault<int>();  // okay.
```
# section 35: Using function pointers

```
count_if(begin_it, end_it, fn_ptr);
```

# section 36: Object slicing and polymorphism

* Note the absence of reference or pointer. This will create a child-object, but only use the parent's part of the child-object to copy-contruct a parent.
```
Parent p = Child()
```

# section 38: Functors

* The operator() is the only operator that takes a variable list of operators. Its return type can be anything
* This is also referred function-call operator
* There is another type-converstion operator which is say foreg `int operator int();`. This return the specific type you want to covert to. You can omit the return type for this operator.

# section 39: typeid, decltype, name mangling

* we should `#include <typeinfo>` to include `typeid`
* Usage: `typeid(object).name()` will give you a const char* of the name, from the typeinfo object
* `decltype(varname)` gets the type of the varname and you can instantiate a second variable now

# section 40: auto

* auto can be used for variable declarations that have an initializer. The type of the variable will be deduced from the initializer.
    * auto can't be used for return types.
        * Note that that function can only be delcared and hence compiler can't know the declaration fully if you put auto.
* for return type you can use the auto + trailing-return-type (which is using a arrow) syntax
    * `auto identifier ( argument-declarations... ) -> return_type`
    * the thing is in return_type, you can use decltype(expressions)

# section 41:

New for loop:

```
auto texts = { "one", "two", "three" }; // note auto

for (auto t : texts) {  // t is the data-type (not a iterator type)
cout << t << endl;
}
```

# section 42: Nested Templates

* Nested regular class
    * Just put the class declaration inside the top class
    * Then define the `topclass::nested_class` later.

* Template nested class needn't have a template type of its own. It can take the top-class's template arg

    ```
    template <typename T>
    class ring
    {
        public:
            class iterator;
    }

    template <typename T>
    class ring<T>::iterator      /* Note the type-name arg only for top-class */
    {
        public:
            ...
    }
    ```

# section 44: Making classes iterator

* if the following works

    ```
    for (myclass::iterator it = myclassobj.begin() ; it != myclassobj.end() ; ++it) {

    }
    ```
* then the following will work
    ```
    for (type_of_iterator_dereferenced t = myclassobj) {

    }
    ```

* Note the prefix/postfix operator syntaxes.
    * postfix has a unused int arg
    * postfix member function should return a new object, while prefix should return the same obj reference.
* T& operator*()

# section 45 : Initialization in c++98

* Note that = equal sign at initialization in c++98 will invoke the copy constructor initialization.
* array initialization with curly braces
* struct initialization with curly braces (one per member)
    * we cannot do that after the variable's initialization.
* we can't initialize vector or other custom classes with curly braces

# section 46 : Initialization in c++11

* Use curly brace for int, strings, pointers, arrays
* for new-array-allocation, you should give size, but the intializer list works still.
* `nullptr` is a new keyword
* empty braces invoke default contructor (on all elements if its array)
* class/struct members can have a `= value;` in them in class/struct defintion for default values!

* vector<int> a{1,2,3}; works!

# section 47: intializer_list for our class

```
#include <initializer_list>     // Include this

class Test {
public:
    Test (initializer_list<string> texts) {    // Should it be referece? Looks like no.
        for (auto value : texts) {
            // value is now a string

        }
    }

    void print(initializer_list<string> texts) { // we can pass initializer_list to functions too.
    }
}
```

# section 48: object initialization, default and delete

* A member can have the `= value` initialization or the `type member{initialize-value}` also.
    * Default constructor will use these values.
    * Constructors can replace some values, and leave others to their member-defined values.
* A default contructor can be simply defined as
   `ClassName() = default;`
* A Copy contructor can be simply defined as
   `ClassName(const ClassName &other) = default;`
   `ClassName& operator=(const ClassName &other) = default;`
* `= delete;` will not allow the generated default

# section 49: lambda expression

* simplest lambda expression - does nothing ofcourse
    `[](){};`
* Valid lambda expression, with function-code, but again does nothing.
    * Its like you defined an anonymous function, but no invocation.
    `[](){ cout << "some execution" << endl; };`
* Save this to a name
    `auto func = [](){ cout << "some execution" << endl; };`
    * Now invoke it
        `func()`
    * you can also invoke it at defintion itself
    `auto func = [](){ cout << "some execution" << endl; }();`
* lambda function is compatible with a function-poiter type, that takes the same args and return type.
    ```
    void some_function(void (*pfn)(void))
    {
        pfn();
    }
    int main()
    {
        auto func = [](){ cout << "no-arg-not-return-lambda" << endl; };

        some_function(func);

        /* directly pass lambda literal as arg! */
        some_function([]() { cout << "direct-lambda-litaral"; });

    }
    ```
* return types of lambda are auto-inteferred if its not ambiguous, or can be explicitly mentioned
  using the trailing-return-type syntax (the arrow thingy)
    ```
    auto pDivide = [](double a, double b) -> double { if(!b) {return 0;} return a/b;};
    ```

# section 51: capturing in lambda

* The square bracket capture local variables by value.
* Using a = sign captures all local variables
    ```
    int main()
    {
        int a;
        int b;
        auto l = [a,b]() { cout << a << b << endl; }
        auto m = [=, &b]() { cout << a << b << endl; }

        /* capture all by referece! */
        auto m = [&]() { cout << a << b << endl; }

        /* capture all by referece, and b by value */
        auto m = [&, b]() { cout << a << b << endl; }
    }
    ```
* Value-captured variables are like const-members - not mutable

# section 52: capturing this  in methods

* We can't capture member variables in a method. However, just capture `this`
* `this` is always captured by reference. It can appear anywhere
* Once `this` is captured, all members are visible inside by default!
* `[&, this](){}` is okay. But not `[=, this](){}:`
    * this itself seems to come in by value (and hence value of this can't be altered)
    * however all members come in as reference, so you can access and modify them

# section 53: The standard function type

* function-type enables to point to any type of callable
    * plain old function pointer
    * functor class
    * Lambda
* Basically it gives the return-type, and arg-types.

    ```
    #include <functional>

    function<string(int, int)> function_type = function_ptr;
    function<string(int, int)> function_type = functor_object;
    function<string(int, int)> function_type = [](int,int) -> string {return string("abc");};
    ```

# section 54: Mutable lambda

* mutable makes the capture by value mutable.

    ```
    int main()
    {
        int abc=5;

        [abc]() mutable { abc=7; cout<<abc; }();  // Mutable and hence you can edit the captured value
    }
    ```

# section 55 : Delegating Constructor

* One constructor of the same class can call another constructor.
    * Simply call the common-constructor from the initializer-list of the other constructor
    * c++98, you would do a init() method and have all constructors call that.
* No protection agianst cyclic calls. Dont do it.

# seciotn 56 : Ellison and optimzation

`-fno-elide-constructors`
    * turns off return-value optimizations.
        * All return temporaries are destructed
        * return values themselves are destructed
