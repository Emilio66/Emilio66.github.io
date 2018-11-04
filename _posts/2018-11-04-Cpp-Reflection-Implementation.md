# Implement Reflection in C++
Since C++ hasn't support *Reflection* mechanism as Java, We need to figure out some smart way to implement it for daily use.
Here I tried 3 ways to implement it:
- Function Pointer
- Class Builder Factory
- Macro and Template

## Function Pointer
The core requirement of reflection in my project is "**reflect** an object from a **string**" so that we can build class from configuration file.

Therefore, the basic thing is to build a bridge between a string with Class builder. So we need a Map to bridge the two, and in order to create an object, we need a function which could **new** a required object.

Here we go:
```
#include <iostream>
#include <map>

using namespace std;

//MAP: class name => class builder function
map<string, void* (*)()> m;

class A {
public:
    A() {
        cout << "A built" << endl;
    }
    ~A() {
        cout << "A destoryed" << endl;
    }
    void hi() {
        cout << "Hi from A" << endl;
    }
};

class B {
public:
    B() {
        cout << "B built" << endl;
    }
    ~B() {
        cout << "B destoryed" << endl;
    }
    void hi() {
        cout << "Hi from B" << endl;
    }
};

// Actual class builder function
A* build_A() {
    return new A;
}
B* build_B() {
    return new B;
}

// building function register
void register_A(string& name) {
    if (m.find(name) == m.end())
        m[name] = (void* (*)())build_A;
}
void register_B(string& name) {
    if (m.find(name) == m.end())
        m[name] = (void* (*)())build_B;
}

// retrieve class instance from map
void* new_instance (string& name) {
    if (m.find(name) != m.end()) {
        auto fp = (void* (*)())m[name];
        return fp();
    }
    return NULL;
}
int main()
{
   cout << "Hello Template" << endl;
   string name = "A";
   register_A(name);
   A* a = (A*)new_instance(name);
   a->hi();
   delete a;
   a = NULL;
   name = "B";
   register_B(name);
   B* b = (B*)new_instance(name);
   b->hi();
   delete b;
   b = NULL;
   return 0;
}
```

This solution does bridge class builder and class name, however, the drawbacks are also obvious: 
- everytime we have a new class, we need to create a class builder function and a register function
- void* is not straightforward and user friendly, we need type conversion(if we know the exact type, why don't we just create it directly?)

## Class Builder Factory
Solution 1 ignores the fact that the object we reflect are mostly the **"changing part"** i.e. the derived class and the **base class** is usually the interface between different parties. 

Therefore we could utilize the base class and use **"Factory Design Pattern"**.
```
#include <iostream>
#include <map>
#include <vector>

using namespace std;

class Base {
public:
    void hi() {
        cout << "Hi from Base" << endl;
    }
};

class A : public Base {
public:
    A() {
        cout << "A built" << endl;
    }
    ~A() {
        cout << "A destoryed" << endl;
    }
    void hi() {
        cout << "Hi from A" << endl;
    }
};

class B : public Base {
public:
    B() {
        cout << "B built" << endl;
    }
    ~B() {
        cout << "B destoryed" << endl;
    }
    void hi() {
        cout << "Hi from B" << endl;
    }
};

// Base Builder
class Builder {
public:
    virtual Base* build() {
        cout << "Base Builder " << endl;
        return NULL;
    }
};

// MAP: class name => class builder
map<string, Builder*> m;

//MACRO: eliminate duplicate code
// 1. create builder for each class derived from Base class
// 2. register the class builder with class name in Map m
// here we used constructor to do the 2nd step because it will not get the expected result.//TODO figure out the reason
#define REGISTER_CLASS(clazz) {\
    class Builder##clazz : public Builder {\
        public:\
            Builder##clazz() {\
                cout << " => "<< #clazz << endl;\
            }\
            clazz* build() {\
                cout << "Derived building" << endl;\
                return new clazz;\
            }\
    };\
    class Register##clazz {\
    public: \
        Register##clazz() {\
            if (m.find(#clazz) == m.end()) {\
                m[#clazz] = new Builder##clazz;\
                cout << #clazz << " was built " << endl;\
            }\
        }\
    };\
    Register##clazz rc;\
}

// Get class instance, macro cannot return value so I used template
template <typename T>
T* new_instance(string& name) {
    cout << "map size: " << m.size() << endl;
    if (m.find(name) != m.end()) {
        auto fp = m[name];
        return fp->build();
    }
    return NULL;
}

int main()
{
   REGISTER_CLASS(A);
   string name = "A";
   Base* a = new_instance<Base>(name);
   if (NULL == a) {
       cout << "ERROR in building" << endl;
   } else {
       a->hi();
       delete a;
       a = NULL;
   }
   REGISTER_CLASS(B);
   name = "B";
   Base* b = new_instance<Base>(name);
   b->hi();
   delete b;
   b = NULL;
   return 0;
}
```
This solution works well. Can we do better?
When I used **template**, it gave me a hint: *template can pass Type into function*, so we could utilize this trait to get the **Type** information instead of using MACRO concatation. So here I got the solution 3:

## Final Version of Reflection 

```
/*
Usage of 2 Macro:
REFLECT_NEW(class_name) //create object from string
REFLECT_REGISTER(base_class_name, derived_class_name) //register a reflection for derived class
*/
#include <iostream>
#include <map>

namespace common {
namespace reflect {
    using std::string;
    using std::cout;
    using std::cerr;
    using std::endl;

template <typename TBase>
class Reflection {
public:
    typedef TBase* (*TFunc)();    //define object's builder function type
    typedef std::map<string, TFunc> register_map;   //bridge map

    // the actual action of putting builder function into map
    static int register_class(string name, TFunc func) {
        register_map& m = _get_map();
        if(m.find(name) == m.end()) {
            m[name] = func;
            cout << name << " registered successfully! " << endl;
            return 0;
        } else {
            cerr << name << " has already been registered! " << endl;
            return 1;
        }
    }

    // the actual action of retrieving builder function from map and then invoking it 
    static TBase* new_instance(string name) {
        register_map& m = _get_map();
        if(m.find(name) != m.end()) {
            return m[name]();
        }else {
            cerr << " Can not find class " << name << endl;
            return NULL;
        }
    }

private:
    // singleton for register map
    static register_map& _get_map() {
        static register_map _map;
        return _map;
    }

    // Instantiation helper
    class Instantiation {
    public:
        Instantiation() {
            cout << "Instantiation instantiated" << endl;
            Reflection<TBase>::_get_map();
        }
        static void instantiate() {
            cout << "called instantiate" << endl;
        }
    };
    // static variable declaration
    static Instantiation _instant;
    
    //hide constructor & destructor
    Reflection() {}
    ~Reflection() {}
};

// static variable definition
template <typename TBase>
typename Reflection<TBase>::Instantiation Reflection<TBase>::_instant;

// reflector is the entry for outside users
template <typename TBase, typename TDerived>
class Registor {
public:
    static TBase* new_func() {
        return new TDerived();
    }
    static void register_for(const char name[]) {
        Reflection<TBase>::register_class(name, new_func);
    }
};
}
}

// register class builder macro
#define REFLECT_REGISTER(base_classname, derived_classname) {\
    using common::reflect::Registor;\
    Registor<base_classname, derived_classname>::register_for(#derived_classname);\
}

// new class macro
#define REFLECT_NEW(base_classname, derived_classname) \
    ::common::reflect::Reflection<base_classname>::new_instance(derived_classname);

///// UNIT TESTING //////
using namespace std;

class Base {
public:
    virtual void hi() {
        cout << "Hi from Base" << endl;
    }
};

class A : public Base {
public:
    A() {
        cout << "A built" << endl;
    }
    ~A() {
        cout << "A destoryed" << endl;
    }
    void hi() {
        cout << "Hi from A" << endl;
    }
};

class B : public Base {
public:
    B() {
        cout << "B built" << endl;
    }
    ~B() {
        cout << "B destoryed" << endl;
    }
    void hi() {
        cout << "Hi from B" << endl;
    }
};

// Simple Usage
int main()
{
    REFLECT_REGISTER(Base, A);
    Base* b = REFLECT_NEW(Base, "A");
    if (NULL != b)
        b->hi();
    else
        cout << "Null found!" << endl;
    return 0;
}
```

This solution requires some knowledge about *static storage*, *template instantiation* and *macro*. 
Please don't heasitate to leave a message if you have any question, thanks:)



