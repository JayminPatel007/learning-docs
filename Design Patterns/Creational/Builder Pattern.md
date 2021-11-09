# Builder Pattern

## Motivation
- Some objects are simple and can be created in a single constructor call
- Other objects require a lot of ceremony to create
- Having an object with 10 constructor argument is not productive
- Instead, opt for piecewise construction
- Builder provides a API for constructing an object step-by-step
  

&nbsp;
> When piecewise object construction is complicated, provide an API for doing it succinctly

&nbsp;

## Life without Builder:
```c++
#include <iostream>
#include <tuple>
#include <sstream>

using namespace std;

int main() {
    auto text = "hello";
    string output;
    output += "<p>";
    output += text;
    output += "</p>";
    cout << output << endl;

    string words[] = { "hello", "world" };
    ostringstream oss;
    oss << "<ul>";
    for (auto w : words) {
        oss << "<li>" << w << "</li>";
    }
    oss << "</ul>";
    cout << oss.str() << endl;
    return 0;
}
```
&nbsp;
## With builder pattern:
```c++
#include <iostream>
#include <sstream>
#include <utility>
#include <vector>

using namespace std;

struct HTMLElement {
    string name, text;
    vector<HTMLElement> elements;
    const size_t indent_size = 2;

    HTMLElement() {}

    HTMLElement(string name, string &text) : name(std::move(name)), text(text) {}

    string str(int indent = 0) const {
        ostringstream oss;
        string i(indent_size*indent, ' ');
        oss << i << "<" << name << ">" << endl;
        if (text.size() > 0) {
            oss << string(indent_size*(indent + 1), ' ') << text <<endl;
        }
        for (const auto& e : elements) {
            oss << e.str(indent + 1);
        }
        oss << i << "</" << name << ">" << endl;
        return oss.str();
    }
};

struct HTMLBuilder
{
    HTMLElement root;

    HTMLBuilder(string root_name) {
        root.name = root_name;
    }

    void add_child(string child_name, string child_text) {
        HTMLElement e{child_name, child_text};
        root.elements.emplace_back(e);
    }

    string str() const { return root.str(); }
};

int main() {
    HTMLBuilder builder{"ul"};
    builder.add_child("li", "Hello");
    builder.add_child("li", "world");
    cout<< builder.str() << endl;
    return 0;
}
```
&nbsp;
## Fluent builder
add child returns a reference of this to chain method calls.
```c++
#include <iostream>
#include <sstream>
#include <utility>
#include <vector>
#include <string>

using namespace std;

class HTMLElement {
    friend class HTMLBuilder;
    string name, text;
    vector<HTMLElement> elements;
    const size_t indent_size = 2;

    HTMLElement() {}

    HTMLElement(string name, string &text) : name(std::move(name)), text(text) {}

public:
    string str(int indent = 0) const {
        ostringstream oss;
        string i(indent_size*indent, ' ');
        oss << i << "<" << name << ">" << endl;
        if (text.size() > 0) {
            oss << string(indent_size*(indent + 1), ' ') << text <<endl;
        }
        for (const auto& e : elements) {
            oss << e.str(indent + 1);
        }
        oss << i << "</" << name << ">" << endl;
        return oss.str();
    }
};

class HTMLBuilder
{
    HTMLElement root;

public:
    HTMLBuilder(string root_name) {
        root.name = root_name;
    }

    HTMLBuilder& add_child(string child_name, string child_text) {
        HTMLElement e{child_name, child_text};
        root.elements.emplace_back(e);
        return *this;
    }

    string str() const { return root.str(); }

    HTMLElement build() { return root; }

    operator HTMLElement() const { return root; }
};

int main() {
    HTMLBuilder builder{"ul"};
    builder.add_child("li", "Hello").add_child("li", "world");
    cout<< builder.str() << endl;

    HTMLElement elem = HTMLBuilder("ul").add_child("li", "Hello").add_child("li", "world").build();

    cout << elem.str() << endl;
    return 0;
}
```
&nbsp;
## Groovy-Style builder
```c++
#include <iostream>
#include <sstream>
#include <utility>
#include <vector>
#include <string>
#include <tuple>
#include <cstdio>
#include <fstream>
#include <memory>

using namespace std;

struct Tag
{
    string name, text;
    vector <Tag> children;
    vector <pair<string, string>> attributes;

    friend std::ostream& operator<<(std::ostream& os, const Tag& tag)
    {
        os << "<" << tag.name;
        for (const auto& att : tag.attributes) {
            os << " " << att.first << "=\"" << att.second <<"\"";
        }
        if (tag.children.size() == 0 && tag.text.length() == 0) {
            os << "/>" << endl;
        }
        else {
            os << ">" << endl;

            if (tag.text.length()) {
                os << tag.text << endl;
            }
            for (const auto& child : tag.children) {
                os << child;
            }
            os << "</" << tag.name << ">" << endl;
        }
    }
protected:
public:
    Tag(const string &name, const string &text) : name(name), text(text) {}

    Tag(const string &name, const vector<Tag> &children) : name(name), children(children) {}
};

struct IMG: Tag {
    explicit IMG(const string& url): Tag{"img", ""} {
        attributes.emplace_back(make_pair("src", url))
    }
};

struct P : Tag {
    P(const string &text) : Tag("p", text) {}
    P(initializer_list<Tag> children) : Tag{"p", children} {}
};

int main() {
    cout <<
        P {
            IMG{ "http://pokemon.com/pikachu.png" }
        } << endl;
    return 0;
}
```
&nbsp;
## Builder Facets
In this type, we use multiple Builders to build the object. In below example we have Person class with address and employment info in it.

1. Person.hpp
```c++
#ifndef UNTITLED_PERSON_HPP
#define UNTITLED_PERSON_HPP


#include <string>

class PersonBuilder;

class Person {
    // address
    std::string street_address, post_code, city;

    //employment
    std::string company_name, position;
    int annual_income{0};
public:
    static PersonBuilder create();

    friend class PersonBuilder;
    friend class PersonJobBuilder;
    friend class PersonAddressBuilder;
};
```
2. Person.cpp
```c++
#include "Person.hpp"
#include "PersonBuilder.hpp"

PersonBuilder Person::create() {
    return PersonBuilder();
}
```
3. PersonBuilder.hpp
```c++
#ifndef UNTITLED_PERSONBUILDER_HPP
#define UNTITLED_PERSONBUILDER_HPP

#include "Person.hpp"

class PersonJobBuilder;
class PersonAddressBuilder;

class PersonBuilderBase
{
protected:
    Person& person;
public:
    PersonBuilderBase(Person &p);

    explicit operator Person() const {
        return person;
    }

    PersonJobBuilder works() const;
    PersonAddressBuilder lives() const;
};

class PersonBuilder : public PersonBuilderBase {
private:
    Person p;
public:
    PersonBuilder();
};

#endif //UNTITLED_PERSONBUILDER_HPP
```
4. PersonBuilder.cpp
```c++
#include "PersonBuilder.hpp"
#include "PersonAddressBuilder.hpp"
#include "PersonJobBuilder.hpp"

PersonBuilderBase::PersonBuilderBase(Person &p) : person(p) {}

PersonJobBuilder PersonBuilderBase::works() const {
    return PersonJobBuilder{person};
}

PersonAddressBuilder PersonBuilderBase::lives() const {
    return PersonAddressBuilder{person};
}

PersonBuilder::PersonBuilder() : PersonBuilderBase(p) {}
```
5. PersonAddressBuilder.hpp
```c++
#ifndef UNTITLED_PERSONADDRESSBUILDER_HPP
#define UNTITLED_PERSONADDRESSBUILDER_HPP

#include "PersonBuilder.hpp"
#include <string>

class PersonAddressBuilder : public PersonBuilderBase {
    typedef PersonAddressBuilder Self;
public:
    PersonAddressBuilder(Person &person);

    Self& at(std::string street_address) {
        person.street_address = street_address;
        return *this;
    }

    Self& with_postcode(std::string post_code) {
        person.post_code = post_code;
        return *this;
    }

    Self& in(std::string city) {
        person.city = city;
        return *this;
    }
};

#endif //UNTITLED_PERSONADDRESSBUILDER_HPP
```
6. PersonAddressBuilder.cpp
```c++
#include "PersonAddressBuilder.hpp"

PersonAddressBuilder::PersonAddressBuilder(Person &person) : PersonBuilderBase(person) {}
```
7. PersonJobBuilder.hpp
```c++
#ifndef UNTITLED_PERSONJOBBUILDER_HPP
#define UNTITLED_PERSONJOBBUILDER_HPP

#include "PersonBuilder.hpp"
#include <string>

class PersonJobBuilder : PersonBuilderBase {
    typedef PersonJobBuilder Self;
public:
    PersonJobBuilder(Person &person);

    Self& at(std::string company_name) {
        person.company_name = company_name;
        return *this;
    }

    Self& as_a(std::string position) {
        person.position = position;
        return *this;
    }

    Self& earning(int annual_income) {
        person.annual_income = annual_income;
        return *this;
    }
};

#endif //UNTITLED_PERSONJOBBUILDER_HPP
```
8. PersonJobBuilder.cpp
```c++
#include "PersonJobBuilder.hpp"

PersonJobBuilder::PersonJobBuilder(Person &person) : PersonBuilderBase(person) {}
```
9. main.cpp
```c++
#include <iostream>
#include <sstream>
#include <utility>
#include <vector>
#include <string>
#include <tuple>
#include <cstdio>
#include <fstream>
#include <memory>

#include "Person.hpp"
#include "PersonBuilder.hpp"
#include "PersonAddressBuilder.hpp"
#include "PersonJobBuilder.hpp"

using namespace std;

int main() {
    Person p = Person::create()
            .lives().at("123 London Road").with_postcode("SW1 1GB").in("London")
            .works().at("Pragmasoft").as_a("Consultant").earning(10e6);
    return 0;
}
```
&nbsp;
## Summary
- A builder is a separate component for building an object
- Can either give builder a constructor or return it via a static function
- To make builder a fluent, return this
- Different facets of an object can be built with different working in tandem via a base class.