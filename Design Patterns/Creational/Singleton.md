# Singleton
- much hated design pattern
&nbsp;

> WHen discussing which pattern to drop, we found that we still love them all. (Not really -- I'm still in favour of dropping singleton. Its use is almost always a design smell.) - Eric Gamma

&nbsp;
## Motivation
- For some component it only make sense to have one instance in the system.
  - Database Repository
  - Object Factory
- E.g., Constructor call is expensive
  - We only do it once
  - We provide everyone with the same instance.
- Want to prevent anyone creating additional copies.
- Need to take care of lazy initialization and thread safety.

&nbsp;
> A component which is instantiated only once.

&nbsp;

## Singleton implementation
```c++
#define _USE_MATH_DEFINES
#include <iostream>
#include <cstdio>
#include <string>
#include <vector>
#include <fstream>
#include <tuple>
#include <sstream>
#include <memory>
#include <cmath>
#include <map>
#include <boost/lexical_cast.hpp>
using namespace boost;

using namespace std;

class SingletonDatabase {
private:
    SingletonDatabase()
    {
        cout << "Initializing the database\n" << endl;
        ifstream ifs("capital.txt");
        string s, s1;
        while (getline(ifs, s)) {
            getline(ifs, s1);
            int pop = lexical_case<int>(s1);
            capitals[s] = pop;
        }
    }
    map<string, int> capitals;

public:
    SingletonDatabase(SingletonDatabase const&) = delete;
    void operator=(SingletonDatabase const&) = delete;

    static SingletonDatabase& get() {
        static SingletonDatabase db;
        return db;
    }

    int getPopulation(const string& name) {
        return capitals[name];
    }

};

int main() {
    string city = "Tokyo";
    cout << city << " has population " <<
        SingletonDatabase::get().getPopulation(city) << endl;
    return 0;
}
```

## Testability Issue
```c++
#define _USE_MATH_DEFINES
#include <iostream>
#include <cstdio>
#include <string>
#include <vector>
#include <fstream>
#include <tuple>
#include <sstream>
#include <memory>
#include <cmath>
#include <map>
#include <boost/lexical_cast.hpp>
using namespace boost;

using namespace std;

class SingletonDatabase {
private:
    SingletonDatabase()
    {
        cout << "Initializing the database\n" << endl;
        ifstream ifs("capital.txt");
        string s, s1;
        while (getline(ifs, s)) {
            getline(ifs, s1);
            int pop = lexical_case<int>(s1);
            capitals[s] = pop;
        }
    }
    map<string, int> capitals;

public:
    SingletonDatabase(SingletonDatabase const&) = delete;
    void operator=(SingletonDatabase const&) = delete;

    static SingletonDatabase& get() {
        static SingletonDatabase db;
        return db;
    }

    int getPopulation(const string& name) {
        return capitals[name];
    }

};

struct SingletonRecordFinder
{
    int total_population(vector<string> names) {
        int result{0};
        for (auto& name: names) {
            result += SingletonDatabase::get().getPopulation(name);
        }
        return result;
    }
};
TEST(RecordFinderTests, SingletonTotalPopulationTest)
{
    SingletonRecordFinder rf;
    vector<string> names{"Seoul", "Mexico City"};
    int tp = rf.total_population(names);
    EXPECT_EQ(17500000 + 17400000, tp);
}



int main(int ac, char* av[]) {
    testing::InitGooleTest(&ac, av);
    string city = "Tokyo";
    cout << city << " has population " <<
        SingletonDatabase::get().getPopulation(city) << endl;
    return 0;
}
```
Testing of SingletonRecordFinder has became difficult, as it is tightly coupled with production data. This means Unit test is dependent on Production Data. This is not only bad dependency wise but also if production data changes in future then Unit test will fail in that case even if implementation is correct.

## Singleton in Dependency Injection
```c++
#define _USE_MATH_DEFINES
#include <iostream>
#include <cstdio>
#include <string>
#include <vector>
#include <fstream>
#include <tuple>
#include <sstream>
#include <memory>
#include <cmath>
#include <map>
#include <boost/lexical_cast.hpp>
using namespace boost;

using namespace std;

class Database {
public:
    virtual int getPopulation(const string& name) = 0;
};

class SingletonDatabase : public Database {
private:
    SingletonDatabase()
    {
        cout << "Initializing the database\n" << endl;
        ifstream ifs("capital.txt");
        string s, s1;
        while (getline(ifs, s)) {
            getline(ifs, s1);
            int pop = lexical_case<int>(s1);
            capitals[s] = pop;
        }
    }
    map<string, int> capitals;

public:
    SingletonDatabase(SingletonDatabase const&) = delete;
    void operator=(SingletonDatabase const&) = delete;

    static SingletonDatabase& get() {
        static SingletonDatabase db;
        return db;
    }

    int getPopulation(const string& name) {
        return capitals[name];
    }

};

class DummyDatabase : public Database {
    map<string, int> capitals;
public:
    DummyDatabase() {
        capitals["alpha"] = 1;
        capitals["beta"] = 2;
        capitals["gamma"] = 3;
    }

    int getPopulation(const string &name) override {
        return capitals[name];
    }
};

struct SingletonRecordFinder
{
    int total_population(vector<string> names) {
        int result{0};
        for (auto& name: names) {
            result += SingletonDatabase::get().getPopulation(name);
        }
        return result;
    }
};

struct ConfigurableRecordFinder
{
    Database& db;

    ConfigurableRecordFinder(Database &db) : db(db) {}

    int total_population(vector<string> names) {
        int result{0};
        for (auto& name: names) {
            result += db.getPopulation(name);
        }
        return result;
    }
};
TEST(RecordFinderTests, SingletonTotalPopulationTest)
{
    SingletonRecordFinder rf;
    vector<string> names{"Seoul", "Mexico City"};
    int tp = rf.total_population(names);
    EXPECT_EQ(17500000 + 17400000, tp);
}

TEST(RecordFinderTests, DependentTotalPopulationTest)
{
    DummyDatabase db;
    ConfigurableRecordFinder crf{db};
    EXPECT_EQ(4, crf.total_population(vector<string>{"alpha", "gamma"}))
}

int main(int ac, char* av[]) {
    testing::InitGooleTest(&ac, av);
    string city = "Tokyo";
    cout << city << " has population " <<
        SingletonDatabase::get().getPopulation(city) << endl;
    return 0;
}
```

## Monostate
```c++
#define _USE_MATH_DEFINES
#include <iostream>
#include <cstdio>
#include <string>
#include <vector>
#include <fstream>
#include <tuple>
#include <sstream>
#include <memory>
#include <cmath>
#include <map>

using namespace std;

class Printer {
    static int id;
public:
    int get_id() const { return id; }
    int set_id(int value) { id = value; }
};

int main(int ac, char* av[]) {
    Printer p;
    int id = p.get_id();
    return 0;
}
```

Above example is of Monostate pattern. In this pattern data members are stored in static data members. This design pattern is bad as you can not use inheritance as static members are not inherited. And also if client makes more than one copy and if one instance updates to one value and second instance updates to 2nd value. and client calls get_value method the it will get 2nd value and this might confuse him that he hasn't updates value from 1st instance as client doesn't know that value he is getting/setting is from static type.

## Multiton
```c++
#include <map>
#include <memory>
#include <iostream>
using namespace std;

enum class Importance
{
    primary,
    secondary,
    tertiary
};

template <typename T, typename Key = std::string>
class Multiton
{
public:
    static shared_ptr<T> get(const Key& key)
    {
        if (const auto it = instances.find(key);
        it != instances.end()) {
            return it;
        }
        auto instance = make_shared<T>();
        instances[key] = instance;
        return instance;
    }
protected:
    Multiton() = default;
    virtual ~Multiton() = default;
private:
    static map<Key, shared_ptr<T>> instances;
};

class Printer
{
public:
    Printer()
    {
        ++Printer::totalInstanceCount;
        cout << "A total of " << Printer::totalInstanceCount
             << " instances created so far\n";
    }

private:
    static int totalInstanceCount;
};
int:: Printer::totalInstanceCount = 0;

int main() {
    typedef Multiton<Printer, Importance> mt;

    auto main = mt::get(Importance::primary);
    auto aux = mt::get(Importance::secondary);
    auto aux2 = mt::get(Importance::secondary);
    return 0;
}
```
In multiton unique instance is created per unique key. But is same key is passed then it will return previously created instance for that key. In above example different instances of printer are created for primary and secondary, but for 2nd time secondary new instance is not created, it will return old instance created for second key.

&nbsp;
## Summary
- Making a *safe* singleton is easy
  - Hide  or delete the type's constructor, copy constructor and copy assignment operators.
  - Create a static method that returns a reference to  static member.
- Type with hard dependencies on singletons are difficult to test.
  - Cannot decouple the singleton and supply a fake object.
- Instead of directly using a singleton, consider depending on abstraction (e.g., an interface)
- Consider defining singleton lifetime in DI container