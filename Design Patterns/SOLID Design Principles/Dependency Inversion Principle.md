# Dependency Inversion Principle
-  High level module should not depend on low level modules. Both should depend on abstractions 
-  Abstractions should not depend on details. Details should depend on abstractions.

## Explanation
```c++
#include <iostream>
#include <cstdio>
#include <string>
#include <vector>
#include <fstream>
#include <tuple>

using namespace std;

enum class Relationship
{
    parent,
    child,
    sibling,
};

struct Person
{
    string name;
};

struct Relationships  // Low-level
{
    vector<tuple<Person, Relationship, Person>> relations;

    void add_parent_and_child(const Person& parent, const Person& child) {
        relations.push_back({parent, Relationship ::parent, child});
        relations.push_back({child, Relationship ::child, parent});
    }
};

struct Research // high-level
{
    Research(Relationships& relationships) {
        auto& relations = relationships.relations;
        for (auto&& [first, rel, second] : relations )
        {
            if (first.name == "John" && rel == Relationship::parent) {
                cout << "John has a child called " << second.name << endl;
            }
        }
    }
};

int main() {
    Person parent{"John"};
    Person child1{"Chris"}, child2{"Matt"};
    Relationships relationships;
    relationships.add_parent_and_child(parent, child1);
    relationships.add_parent_and_child(parent, child2);

    Research _(relationships);
    return 0;
}
```
In the above example, Research class is directly using Relations which is low-level module. also it it using details of the Relations implementations that it will store relationships in vector of tuple form. This is bad, as if later we want to makes relations variable private then, we will get problems in Research class. Better implementation would be following...

```c++
#include <iostream>
#include <cstdio>
#include <string>
#include <vector>
#include <fstream>
#include <tuple>

using namespace std;

enum class Relationship
{
    parent,
    child,
    sibling,
};

struct Person
{
    string name;
};

struct RelationshipBrowser {
    virtual vector<Person> find_all_children_of(const string& name) = 0;
};

struct Relationships: RelationshipBrowser  // Low-level
{
    vector<tuple<Person, Relationship, Person>> relations;

    void add_parent_and_child(const Person& parent, const Person& child) {
        relations.push_back({parent, Relationship ::parent, child});
        relations.push_back({child, Relationship ::child, parent});
    }

    vector<Person> find_all_children_of(const string &name) override {
        vector<Person> result;
        for (auto&& [first, rel, second]: relations) {
            if (first.name == name && rel == Relationship::parent) {
                result.push_back(second);
            }
        }
        return result;
    }
};

struct Research // high-level
{
    Research(RelationshipBrowser& browser) {
        for (auto& child :browser.find_all_children_of("John")) {
            cout << "John has a child called " << child.name << endl;
        }
    }
};

int main() {
    Person parent{"John"};
    Person child1{"Chris"}, child2{"Matt"};
    Relationships relationships;
    relationships.add_parent_and_child(parent, child1);
    relationships.add_parent_and_child(parent, child2);

    Research _(relationships);
    return 0;
}
```