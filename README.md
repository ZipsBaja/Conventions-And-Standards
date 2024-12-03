
# University of Akron
## Zips Baja Electrical Subsystem
#### Tutorials, Coding Conventions, and Standards
---
This repository focuses on the conventions used in our code for the vehicle's onboard microprocessors.
All languages used will have separate conventions and all will be covered here.
Examples will be gone over in this markdown file, and more contextual examples are found within the languages respective folder.

---
### C/C++ Workspace
The workspace we use for the Raspberry Pi Pico C/C++ SDK is a special linux Docker container.
The source repo can be found <a href="https://github.com/lukstep/raspberry-pi-pico-docker-sdk">Here</a>.
In order to run this Docker container, make sure to install docker on your system. The getting started page is <a href="https://www.docker.com/get-started/">Here</a>. Docker desktop is a good choice for Windows operating systems.
Now, ensure Docker added its environmental variables to your system. This should be done automatically. Simply open a command terminal and type `docker`.
If all is good, create a volume where you want your code to be stored. Through Docker Desktop, navigate to "Volumes" on the left side, and click create. Name this volume however you'd like. I myself chose "RPiPico" for all the Pico projects.
Now through the command terminal again, copy the command below and run it, but *make sure to replace the volume name with your own!*
`docker run -it --name pico-sdk -v RPiPico:/mnt lukstep/raspberry-pi-pico-sdk:latest`
This will automatically pull the container and run it on your command line. Once installed and initialized, it will respond with a virtual terminal, the username being "root".
Before closing the terminal, run `apt update`, and then `apt upgrade -y`. This will update the packages on the linux virtual machine.
The code editor used is VSCode. Installing it can be found <a href="https://code.visualstudio.com/download">Here</a>.
When setting up VSCode, navigate to the extensions tab on the left, search for Docker, and install the extension. You may need to restart VSCode.
Now, you can access your Docker containers and connect to them through VSCode. Click on the Docker icon on the left-hand panel, right click on the container you ran earlier, and start it. Then when started, right click again and "Attach Visual Studio Code".
The terminals you open will be inside the container, direct access to linux.
Now you can code with the Pico SDK! The default path for the SDK is `/usr/local/picosdk`, which has an environmental variable `$PICO_SDK_PATH`, and the recommended path for your projects is in `/mnt`.
Before coding, the SDK may need to be updated. Change directory to the SDK `cd $PICO_SDK_PATH` and run `git pull origin master`, then `git submodule update --init --remote --recursive`. This will update the SDK and all references.

---
### C/C++ Standards
Most standards in C/C++ follow existing conventions, but let's go over the main features:
<br>

##### C/C++ Classes and Structs
Classes and structs will use "Pascal Case" where all words are capitalized in the name.
```c++
class MyClass;
struct MyStruct;

class ZipsBaja;
struct UniversityOfAkron;
```
When defining classes, the `private`, `public`, and `protected` keywords will remain not indented.
```c++
class ClassDefinition
{
private:
    int num;
public:
    ClassDefinition();
}
```
These keywords are to be avoided when using structs, as structs are more about accessible data storage than encapsulation.

##### C/C++ Functions
Functions part of classes or structs (known as member functions) will also share its Pascal case theme.
```c++
class MyClass
{
    void ClassFunction();
};

struct MyStruct
{
    void StructFunction();
};
```
Here's an example with a vector struct.
```c++
struct Vector2
{
    float x;
    float y;

    float Length();
    float DotProduct();
    Vector2 Normalized();
    Vector2 CrossProduct();
};
```
Note the member variables of the struct are simple letters because of the nature of the usage of vectors. Data structures such as vectors, quarternions, and matrices will be excluded from the standard conventions which will be explained later.
Ordinary functions external to classes or structs should use Pascal case or "snake case" depending on the context of the function. Snake case is all lowercase using underscores as spaces.
`this_is_snake_case`

##### C/C++ Variables and Constants
Variables should be denoted in snake case but is flexible depending on both the writer's preference and also the context of where the variable is created.
 In an ordinary function we can create a variable like so:
```c
void DoSomething()
{
    float my_float;
    int32_t bitflag_for_something = 0x00000001;
}
```
For constants we can follow the same rules if the constant is local to a function. If not, constants stored as a type should follow snake case or for simpler data types with common use. Constants use "screaming snake case" which follows snake case but using all caps.
`THIS_IS_SCREAMING_SNAKE_CASE`
For avoiding linking errors and external definitions many to this day still enjoy defining simple data type constants as macros, which is no problem here. All macros regardless of its functionality should be in screaming snake case. If the macro involves parameters, follow the local variable rules. For constants as values, try to use `constexpr` when you can. Here's a snippet involving all around constants:
```c++
#define MY_MACRO_THAT_DOES_SOMETHING(parameter)
#define PI 3.14159f

constexpr float PiButStoredAsAValue = PI;

void Foo()
{
    // This can be made constexpr, but for
    // example purposes we ignore.
    const size_t planets_in_the_solar_system = 8;
}
```
<br>

##### C/C++ Types
If you've been paying close attention you might have noticed the use of `int32_t` and `size_t`. Because of our cross-compatibility requirements between different architectures and 32-bit or 64-bit machines, we should avoid using the standard integer types when coding because of the differences they have across platforms. A `long` may be 32 bits on one compiler, but 64 bits on another. If we include the header file `<cstdint>` on C++ or `<stdint.h>` on C, we no longer need to worry of the incompatibility if such exists. Heres a list of the common types used:
```c
int64_t integer_64_bit;
int32_t integer_32_bit;
int16_t integer_16_bit;
int8_t  integer_8_bit;

uint64_t unsigned_integer_64_bit;
uint32_t unsigned_integer_32_bit;
uint16_t unsigned_integer_16_bit;
uint8_t  unsigned_integer_8_bit;

// size_t is the same as uint64_t but should
// be used to determine the value represents
// a size of some sort, such as an array length
size_t size_64_bit;
```
In C++ only, these can be found under the `std` namespace but we will refrain from using that. There is one common scenario where using the default defined integer types is fine, and that would be for loops where we know all numbers can be looped through without coming anywhere near the maximum value. It is a good idea to use the type that you are comparing against.
```c
for (int i = 0; i < 10; i++);

int16_t num = 10;
for (int16_t i = 0; i < num; i++);
```
<br>

##### C/C++ Arrays
Defining a simple array in C/C++ can be done like so:
```c
char buff[50];
```
This defines a character buffer capable of storing 50 characters. But know the exception: with character arrays exclusively, if acting as a string, must be terminated with the "null terminiation character", which can be written inside the source as `\0`. If `buff` is used as a string, a maximum of 49 characters can be stored. In C/C++, array sizes need to be known prior. If using the stack to declare an array, the size of the array can be returned using `sizeof(myArray) / sizeof(arrayType)`. Here's an example.
```c
int32_t arr[8] = {...};
size_t arr_size = sizeof(arr) / sizeof(int32_t); 
```
In many SDK's, the `count_of` macro will return the size of a locally allocated array;

##### C++ Namespaces
Naming namespaces should be with all lowercase and kept relatively short. If your code adds a namespace which will be imported with `using namespace`, then it won't be necessary to create the namespace as the use of it is omitted. Never write `using namespace std` or any other large library as is can leak to other header files. Always write `std::` then what you want to use inside the namespace. If you find yourself using one or a few items from a long namespace name, you can create an alias like so:
```c++
using sec = std::chrono::seconds;
```
<br>

##### C/C++ Style
A few syntactical styles are used for the Baja team. As you may have seen, curly braces are typically used on new lines. But, curly braces in terms of object initialization can be used on the same line.
```c++
struct AStruct
{
    int a, b, c, d;
};

void AFunction()
{
    AStruct data = {
        1, 2, 3, 4
    };
    AStruct data2 = {1, 2, 3, 4};
}
```
<br>

##### C++ Templates
Templates are used to define a wild card of a type. It acts as a blank type until used. Naming these template types depends on the use of the template. If the template type works with anything, such as the `std::vector` class, the type name should be denoted with a single capital letter. If only certain types are allowed, or subclasses or superclasses of a defined class are allowed, write the template name accordingly. If using C++20, you can make use of "concepts" which restrict templates to certain conditions. If using an older C++ standard, but also C++11 or newer, `std::enable_if` does the same job.

##### C/C++ Preprocessor
Include statements should always be at the top of the file. There are very few reasons to do so otherwise.
Macros are to use screaming snake case as stated prior, and if such are constants or utility functions, should be below the includes, but above the code. Macro functions should be separate from the other macros. Checking for certain macro conditions may be all over the place especially when making code compatible with different platforms and architecture.
Keep all preprocessor entries unindented unless working with frameworks or for largely nested usage.
When using C++ and in need of a function from the C standard library, use the C++ variants denoted by the C's file name but removing the `.h` and including the letter `c` at the beginning of the file name. For example, `<stdlib.h>` is available in both C and C++ but C++ also has `<cstdlib>` which moves the contents under the `std` namespace, which is useful for tracing where data is used from.

##### C/C++ Main Function
In C/C++, the main function can be written in two ways. Both will be used depending on the need to use the arguments passed. The two functions are `main()` and `main(int argc, char* argv[])`. Use the first main function if it is not necessary to use command line arguments for the compiled executable.

##### C/C++ Casting
Type conversions (known as casting) will be important even if not necessary since the compiler automatically casts in certain scenarios. To avoid writing lengthy comments beside the code, casting with the C-style cast `(type) value` OR the C++ style cast `static_cast<type>(value)` will work. Sometimes one or the other is better, for example, when working with PEMDAS.
```c++
// Note that there is a function dedicated to float
// square roots but this is for example purposes.
// Casting the value inside the square root
// function isn't necessary as we should know
// the compiler already casts it, plus we know,
// the result will be a floating-point type
// because of the nature of irrational constants.
// Without the cast, the compiler automatically
// casts it for us but we might confuse types.
float root5 = (float) std::sqrt(5);
float root2_times5 = static_cast<float>(std::sqrt(2) * 5);

// Now if we know we are square rooting a perfect
// square, or droppng the decimal figures, casting
// to an integer is no problem.
int32_t root25 = (int32_t) std::sqrt(25);
```

Below is an example using all of the conventions used.
```c++
#include <iostream>
#include <cstdlib>
#include <cmath>
#include <vector>

#define TO_FLOAT(str) std::strtof(str)

#define MY_CONSTANT 1.2345

template<typename T>
using list = std::vector<T>;

struct Vec3
{
    float x, y, z;
    inline float GetLength()
    {
        return std::sqrtf(x * x + y * y + z * z);
    }
};

int main(int argc, char* argv[])
{
    if (argc < 4)
        return -1;

    list<float> lengths;
    Vec3 vector_one = {
        TO_FLOAT(argv[1]),
        TO_FLOAT(argv[2]),
        TO_FLOAT(argv[3])
    };
    Vec3 vector_two = {0.f, (float) MY_CONSTANT, 0.f};

    lengths.push_back(vector_one.GetLength());
    lengths.push_back(vector_two.GetLength());

    for (size_t i = 0; i < length.size(); i++)
    {
        std::cout << lengths[i] << '\n';
    }

    return 0;
}
```
