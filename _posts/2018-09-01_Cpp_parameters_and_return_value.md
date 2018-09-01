# C++ Parameters and return values

Today I was wondering how the parameters and arguments work in C++ as well as return values. After experimenting, I have gotton a better understanding about these stuff and share them in this post.

## Parameters and Arguments

> Parameter is variable in the declaration of function.   
Argument is the actual value of this variable that gets passed to function.  
[Ref from stackoverflow](https://stackoverflow.com/questions/156767/whats-the-difference-between-an-argument-and-a-parameter)

```
void function(string parameter) { //do something }
int main() {
    function("This is argument");
    return 0;
}
```

Let's figure out how the arguments be passed and how parameters get these passed values.
```
#include <iostream>
#include <string>

using namespace std;

//define a class 
class Student {
public:
    string name;
    Student (string s) {
        name = s;   //COPY instead of pointer replacement
        cout << "param s addr: " << &s << endl;
        cout << "name addr: " << &name << endl;
    }
};

int main()
{
    string s = "bob";
    cout << "arg s addr: " << &s << endl;
    Student st(s);
    cout << "st addr: " << &st << endl;
    return 0;
}
//////// output /////////
/*
arg s addr:   0x7ffd41741fd0
param s addr: 0x7ffd41742000
name addr:    0x7ffd41741fb0
st addr:      0x7ffd41741fb0
*/
```
Basically, an argument's value is copied to the parameter, so we can see the address of parameter 's' is different from the address of argument s. What happens here is a string object copy from argument s to paramenter s.

Another intersting thing to notice is that the address of member 'name' is the same as the address of object st. That indicates the object's starting address is its first member's address.

Having this copy process in mind, we'd better use *pointer* or *reference* when passes the argument to parameter to avoid the copy overhead. 

## Return values
We know that the lifecycle of variables are associated with the code blocks *{...}* they live in. When the code runs out of the block, all the variables inside that block will not be accessible again. So are the return values. When a function returns, if you write a piece of code that intends to get its address, it will cause *compile error*. See the example below:
```
string first(string a) {
    //cout << "a addr: " << &a << endl;
    string b = a.substr(0, 1);
    cout << "before return addr: " << &b << endl;
    return b;
}

int main()
{
    string s = "bob";
    //Compile ERROR: taking address of temporary [-fpermissive]
    cout << "after return addr: " << &first(s) << endl;
    return 0;
}
```
Actually, we need to use another variable to store the temporary value returned by a function. 

From my understanding, the return value will be copied to the new variable to store it and the addresses of the two are supposed to be different. But to my surprise, they are *SAME* !

```
string first(string a) {
    string b = a.substr(0, 1);
    cout << "before return addr: " << &b << endl;
    return b;
}

int main()
{
    string s = "bob";
    string s1 = first(s);   //reuse the temporary address
    cout << "after return addr: " << &s1 << endl;
    return 0;
}
//////// output /////////
/*
before return addr: 0x7ffca71b5ca0
after return addr:  0x7ffca71b5ca0    //same address
*/
```
I think it is the compiler who makes the optimization to save the memory space by not having reallocation overhead.

*But* when I tested it with an already exist variable, the results were different.
```
int main()
{
    string s = "bob";
    
    string s1 = first(s);   //reuse the temporary address
    cout << "after return addr: " << &s1 << endl;
    string s2 = first("abc");
    cout << "after return addr: " << &s2 << endl;  //reuse the temporary address
    s1 = first("abd");      //COPY instead of reusing the temporary address
    cout << "after return addr: " << &s1 << endl; 
    
    return 0;
}
//////// output /////////
/* 
before return addr: 0x7ffe22920d00
after return addr:  0x7ffe22920d00    //same address
before return addr: 0x7ffe22920ce0
after return addr:  0x7ffe22920ce0    //same address
before return addr: 0x7ffe22920de0
after return addr:  0x7ffe22920d00    //DIFFERENT address
*/
```
So when a return value is assiagned to a newly created variable, the compiler reuse the address of temporary location which the return value live, however, when it's assigned to a exist variable, it incurs *copy* process, the returned value is copied to a new place because the exist variable's original place may not have enough space for the new value. Therefore, you can see the newly assigned variable's address not only differ from the return value but also from its original address.

*One more thing* when it comes to *constant* variable, they always have the same address because the constant area has already been allocated before the program running and will remain the same during its lifecycle.

Here is the code:
```
void print(int n) {
    cout << "n addr: " << &n << endl;
}
int main()
{
    int n = 3;
    print(n);  
    print(4);
    print(2147383647);
    return 0;
}
//////// output /////////
/*
n addr: 0x7ffdc26535ec
n addr: 0x7ffdc26535ec
n addr: 0x7ffdc26535ec
*/

```
