# Factory Pattern
- Factory Pattern
- Abstract Factory Pattern

## Motivation
- Object creation logic becomes too convoluted.
- Constructor is not descriptive.
  - Name mandate by name of containing type
  - Cannot overload with same sets of arguments with different names
  - Can turn into a *'Optional parameter hell'*
- Object creation (non-piecewise, unlike Builder) can be outsourced to
  - A separate function (Factory Method)
  - That may exist in a separate class (Factory)
  - Can create hierarchy of factory with Abstract Factory.

&nbsp;
> *A component responsible solely for the wholesale (not piecewise) creation of Objects.*

&nbsp;

## Point example
```c++
using namespace std;

struct Point {
    float x, y;

    Point(float x, float y) : x(x), y(y) {}
    
    // How to construct a point with Radius and theta ?
    // as we can not overload Constructor with same argument type
};

int main() {
    return 0;
}
```
```c++
#include <cmath>

using namespace std;

enum class PointSystem
{
    cartesian,
    polar
};

struct Point {
    float x, y;

    // No documentation on what is a and b for different PointSystems?
    Point(float a, float b, PointSystem type = PointSystem::cartesian) {
        if (type == PointSystem::cartesian) {
            x = a;
            y = b;
        } else {
            x = a * cos(b);
            y = a * sin(b);
        }
    }
};

int main() {
    return 0;
}
```
&nbsp;
## Factory Method Pattern
Creation of object using methods in the same class and making its constructor private.
```c++
#include <iostream>
#include <utility>
#include <cmath>

using namespace std;

class Point {
    Point(float x, float y) : x(x), y(y) {}
public:
    float x, y;

    static Point NewCartesian(float x, float y) {
        return {x, y};
    }

    static Point NewPolar(float radius, float theta) {
        return {radius * cos(theta), radius * sin(theta)};
    }

    friend ostream &operator<<(ostream &os, const Point &point) {
        os << "x: " << point.x << " y: " << point.y;
        return os;
    }
};

int main() {
    Point p = Point::NewPolar(5, M_PI_4);
    cout << p << endl;
    return 0;
}
```
&nbsp;
## Factory Pattern
In this pattern we create a separate Factory class which have static methods which creates a Point object. But sometime it violates open-closed principle as we have to make either Constructor public or have to add friend class inside Point class.
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

class Point {
    friend class PointFactory;
    Point(float x, float y) : x(x), y(y) {}
public:
    float x, y;

    friend ostream &operator<<(ostream &os, const Point &point) {
        os << "x: " << point.x << " y: " << point.y;
        return os;
    }
};

struct PointFactory {
    static Point NewCartesian(float x, float y) {
        return {x, y};
    }
    
    static Point NewPolar(float radius, float theta) {
        return {radius * cos(theta), radius * sin(theta)};
    }
};

int main() {
    Point p = PointFactory::NewPolar(5, M_PI_4);
    cout << p << endl;
    return 0;
}
```
&nbsp;
## Inner Factory pattern
We can create Factory as inner class of a class, this solves two problems, when a consumer sees constructors as Private they may get confused that how to create instance of that class, by making factory class as inner class we are eliminating that confusion. And also if we create factory class at a time ae are writing base class then it won't brake open-closed principle.
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

class Point {
    Point(float x, float y) : x(x), y(y) {}
private:
    struct PointFactory {
        Point NewCartesian(float x, float y) {
            return {x, y};
        }

        Point NewPolar(float radius, float theta) {
            return {radius * cos(theta), radius * sin(theta)};
        }
    };
public:
    float x, y;

    friend ostream &operator<<(ostream &os, const Point &point) {
        os << "x: " << point.x << " y: " << point.y;
        return os;
    }

    static PointFactory Factory;

};

int main() {
    Point p = Point::Factory.NewPolar(5, M_PI_4);
    cout << p << endl;
    return 0;
}
```
&nbsp;
## Abstract Factory Pattern
When we have to create objects at run-time, then this is used. For example we have a 2 Hot drinks Coffee and Tea and we want to create its objects based on user input, then abstract factory patter is used. Its implementation example is show below.

1. HotDrink.hpp
```c++
#ifndef UNTITLED_HOTDRINK_HPP
#define UNTITLED_HOTDRINK_HPP

#include <iostream>
#include <memory>
using namespace std;

struct HotDrink
{
    virtual ~HotDrink() = default;
    virtual void perpare(int volume) = 0;
};

struct Tea: HotDrink
{
    void perpare(int volume) override {
        cout << "Take a tea bag, boil water, pour " << volume
             << "ml, add some lemon\n";
    }
};

struct Coffee : HotDrink
{
    void perpare(int volume) override {
        cout << "Grind some beans, boil water, pour " << volume
             << "ml, add some cream, enjoy!\n";
    }
};

#endif //UNTITLED_HOTDRINK_HPP
```
2. HotDrinkFactory.hpp
```c++
#ifndef UNTITLED_HOTDRINKFACTORY_HPP
#define UNTITLED_HOTDRINKFACTORY_HPP

#include "HotDrink.hpp"

struct HotDrinkFactory
{
    virtual unique_ptr<HotDrink> make() = 0;
};

struct TeaFactory : HotDrinkFactory
{
    unique_ptr<HotDrink> make() override {
        return unique_ptr<Tea>();
    }
};

struct CoffeeFactory : HotDrinkFactory
{
    unique_ptr<HotDrink> make() override {
        return unique_ptr<Coffee>();
    }
};

#endif //UNTITLED_HOTDRINKFACTORY_HPP
```
3. DrinkFactory.hpp
```c++
#ifndef UNTITLED_DRINKFACTORY_HPP
#define UNTITLED_DRINKFACTORY_HPP

#include "HotDrink.hpp"
#include "HotDrinkFactory.hpp"
#include <map>
using namespace std;

class DrinkFactory
{
    map<string, unique_ptr<HotDrinkFactory>> hot_factories;
public:
    DrinkFactory() {
        hot_factories["coffee"] = make_unique<CoffeeFactory>();
        hot_factories["tea"] = make_unique<TeaFactory>();
    }

    unique_ptr<HotDrink> make_drink(const string& name) {
        auto drink = hot_factories[name]->make();
        drink->perpare(200);
        return drink;
    }
};

#endif //UNTITLED_DRINKFACTORY_HPP
```
4. main.cpp
```c++
#include "DrinkFactory.hpp"

using namespace std;

int main() {
    DrinkFactory df;
    auto c = df.make_drink("coffee");
    return 0;
}
```
&nbsp;
## Functional Factory
1. HotDrink.hpp
```c++
#ifndef UNTITLED_HOTDRINK_HPP
#define UNTITLED_HOTDRINK_HPP

#include <iostream>
#include <memory>
using namespace std;

struct HotDrink
{
    virtual ~HotDrink() = default;
    virtual void perpare(int volume) = 0;
};

struct Tea: HotDrink
{
    void perpare(int volume) override {
        cout << "Take a tea bag, boil water, pour " << volume
             << "ml, add some lemon\n";
    }
};

struct Coffee : HotDrink
{
    void perpare(int volume) override {
        cout << "Grind some beans, boil water, pour " << volume
             << "ml, add some cream, enjoy!\n";
    }
};

#endif //UNTITLED_HOTDRINK_HPP
```
2. DrinkFactory.hpp
```c++
#ifndef UNTITLED_DRINKFACTORY_HPP
#define UNTITLED_DRINKFACTORY_HPP

#include "HotDrink.hpp"
#include <map>
#include <functional>
using namespace std;

class DrinkFactory
{
    map<string, function<unique_ptr<HotDrink>()>> factories;

public:
    DrinkFactory() {
        factories["tea"]=[] {
            auto tea =make_unique<Tea>();
            tea->perpare(200);
            return tea;
        };
        factories["coffee"]=[] {
            auto coffee =make_unique<Coffee>();
            coffee->perpare(200);
            return coffee;
        };
    }

    unique_ptr<HotDrink> prepare(const string& name) {
        return factories[name]();
    }
};

#endif //UNTITLED_DRINKFACTORY_HPP
```
3. main.cp
```c++
#include "DrinkFactory.hpp"

using namespace std;

int main() {
    DrinkFactory df;
    auto c = df.prepare("coffee");
```
&nbsp;
## Summary
- factory method is static that creates objects.
- A factory can take care of object creation.
- A factory can be external or reside inside the object as an inner class.
- Hierarchy of factories can be used to create related objects.