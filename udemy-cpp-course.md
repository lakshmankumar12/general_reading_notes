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
