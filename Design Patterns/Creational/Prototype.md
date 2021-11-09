# Prototype
- Complicated objects aren't designed from scratch.
  - They reiterate existing design
- An existing (partially or fully constructed) design is a Prototype.
- We make a copy of (clone) of the Prototype and customize it.
  - Require **Deep copy** support.

&nbsp;

> *A partially or fully initialized object that you (clone) and make use of.*

&nbps;

## Life without Prototype
```c++
#include <vector>
#include <string>
#include <tuple>
#include <cstdio>
#include <fstream>
#include <memory>
#include <sstream>
#include <iostream>
#include <utility>
#include <cmath>

using namespace std;

struct Address {
    string street, city;
    int suite;

    Address(const string &street, const string &city, int suite)
    : street(street), city(city), suite(suite) {}

    friend ostream &operator<<(ostream &os, const Address &address) {
        os << "street: " << address.street << " city: " << address.city << " suite: " << address.suite;
        return os;
    }

};

struct Contact
{
    string name;
    Address* address;

    Contact(const string &name, Address *address) : name(name), address(address) {}

    ~Contact() {
        delete address; 
    }

    friend ostream &operator<<(ostream &os, const Contact &contact) {
        os << "name: " << contact.name << " address: " << *contact.address;
        return os;
    }
};

int main() {
    Contact john{"John Doe", new Address{"123 East Dr", "London", 123}};
    Contact jane{"Jane Smith", new Address{"123 East Dr", "London", 103}};
    return 0;
}
```
In this example we are creating whole new object for jane. In the above example it is okay because object creation is easy. But if it is complicated then we want to copy the old object we created and then modify it. Below is the example but without Prototype pattern...
```c++
#include <vector>
#include <string>
#include <tuple>
#include <cstdio>
#include <fstream>
#include <memory>
#include <sstream>
#include <iostream>
#include <utility>
#include <cmath>

using namespace std;

struct Address {
    string street, city;
    int suite;

    Address(const string &street, const string &city, int suite)
    : street(street), city(city), suite(suite) {}

    friend ostream &operator<<(ostream &os, const Address &address) {
        os << "street: " << address.street << " city: " << address.city << " suite: " << address.suite;
        return os;
    }

};

struct Contact
{
    string name;
    Address* address;

    Contact(const string &name, Address *address) : name(name), address(address) {}

    ~Contact() {
        delete address; 
    }

    friend ostream &operator<<(ostream &os, const Contact &contact) {
        os << "name: " << contact.name << " address: " << *contact.address;
        return os;
    }
};

int main() {
    Contact john{"John Doe", new Address{"123 East Dr", "London", 123}};
    Contact jane = john; // shallow copy
    jane.name = "Jane Smith";
    jane.address->suite = 103;
    cout << john << endl << jane << endl;
    return 0;
}
```
In the above example, we are copying the object john to jane and changing the name and suite for jane. But address is the reference type so change suit for jane is also changing the value of suite for John.

$nbsp;

## Prototype Patten
```c++
#include <vector>
#include <string>
#include <tuple>
#include <cstdio>
#include <fstream>
#include <memory>
#include <sstream>
#include <iostream>
#include <utility>
#include <cmath>

using namespace std;

struct Address {
    string street, city;
    int suite;

    Address(const string &street, const string &city, int suite)
    : street(street), city(city), suite(suite) {}

    Address(const Address& otherAddress)
    : street(otherAddress.street), city(otherAddress.city), suite(otherAddress.suite) {}

    friend ostream &operator<<(ostream &os, const Address &address) {
        os << "street: " << address.street << " city: " << address.city << " suite: " << address.suite;
        return os;
    }

};

struct Contact
{
    string name;
    Address* address;

    Contact(const string &name, Address *address) : name(name), address(address) {}

    Contact(const Contact& other) :
    name(other.name), address(new Address{*other.address}) {}

    ~Contact() {
        delete address; 
    }

    friend ostream &operator<<(ostream &os, const Contact &contact) {
        os << "name: " << contact.name << " address: " << *contact.address;
        return os;
    }
};

int main() {
    Contact john{"John Doe", new Address{"123 East Dr", "London", 123}};
    Contact jane{john};
    jane.name = "Jane Smith";
    jane.address->suite = 103;
    cout << john << endl << jane << endl;
    return 0;
}
```
&nbsp;
## Prototype Factory
```c++
#include <vector>
#include <string>
#include <tuple>
#include <cstdio>
#include <fstream>
#include <memory>
#include <sstream>
#include <iostream>
#include <utility>
#include <cmath>

using namespace std;

struct Address {
    string street, city;
    int suite;

    Address(const string &street, const string &city, int suite)
    : street(street), city(city), suite(suite) {}

    Address(const Address& otherAddress)
    : street(otherAddress.street), city(otherAddress.city), suite(otherAddress.suite) {}

    friend ostream &operator<<(ostream &os, const Address &address) {
        os << "street: " << address.street << " city: " << address.city << " suite: " << address.suite;
        return os;
    }

};

struct Contact
{
    string name;
    Address* address;

    Contact(const string &name, Address *address) : name(name), address(address) {}

    Contact(const Contact& other) :
    name(other.name), address(new Address{*other.address}) {}

    ~Contact() {
        delete address; 
    }

    friend ostream &operator<<(ostream &os, const Contact &contact) {
        os << "name: " << contact.name << " address: " << *contact.address;
        return os;
    }
};

struct EmployeeFactory
{
    static unique_ptr<Contact> new_main_office_employee(const string& name, const int suite) {
        static Contact p{ "", new Address{"123 East Drive", "London", 0}};
        return new_employee(name, suite, p);
    }
private:
    static unique_ptr<Contact> new_employee(const string& name, const int suite, const Contact& prototype) {
        auto result = make_unique<Contact>(prototype);
        result->name = name;
        result->address->suite = suite;
        return result;
    }

};

int main() {
    auto john = EmployeeFactory::new_main_office_employee("john", 123);
    cout << *john << endl;
    return 0;
}
```
&nbsp;
## Summary
- To implement a prototype, partially construct n object and store it somewhere.
- Clone the prototype
  - Implement your own deep copy functionality; or ...
  - Serialize and deserialize.
- Customize the resulting instance.