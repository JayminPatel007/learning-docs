# Open-Closed Principle
Your system should open for extensions but closed for modifications.

## Explanation
```c++
#include <iostream>
#include <vector>
#include <fstream>

using namespace std;

enum class Color { red, green, blue };
enum class Size { small, medium, large };

struct Product
{
    string name;
    Color color;
    Size size;
};

struct ProductFilter
{
    static vector<Product*> by_color(vector<Product*> items, Color color) {
        vector<Product*> result;
        for(auto& i : items) {
            if (i->color == color) {
                result.push_back(i);
            }
        }
        return result;
    }

    static vector<Product*> by_size(vector<Product*> items, Size size) {
        vector<Product*> result;
        for(auto& i : items) {
            if (i->size == size) {
                result.push_back(i);
            }
        }
        return result;
    }

    static vector<Product*> by_size_and_color(vector<Product*> items, Size size, Color color) {
        vector<Product*> result;
        for(auto& i : items) {
            if (i->size == size && i->color == color) {
                result.push_back(i);
            }
        }
        return result;
    }
};

int main() {
    Product apple{"Apple", Color::green, Size::small};
    Product tree{"Tree", Color::green, Size::large};
    Product house{"House", Color::blue, Size::large};

    vector<Product*> items{&apple, &tree, &house};
    ProductFilter::by_color(items, Color::green);

    return 0;
}
```
Above approach just doesn't scale. we have to modify the existing code and let's say we have 8 category types then filer functions will be so many. Better implementation of this is the following...

```c++
#include <iostream>
#include <vector>
#include <fstream>

using namespace std;

enum class Color { red, green, blue };
enum class Size { small, medium, large };

struct Product
{
    string name;
    Color color;
    Size size;
};

template <typename T> struct Specification
{
    virtual bool is_satisfied(T* item) = 0;
};

template <typename T> struct Filter
{
    virtual vector<T*> filter(vector<T*> items, Specification<T>& spec) = 0;
};

struct BetterFilter : Filter<Product>
{
    vector<Product *> filter(vector<Product *> items, Specification<Product>& spec) override {
        vector<Product *> result;
        for (auto& item : items) {
            if (spec.is_satisfied(item)) {
                result.push_back(item);
            }
        }
        return result;
    }
};

struct ColorSpecification : Specification<Product>
{
    Color color;

    ColorSpecification(Color color) : color(color) {}

    bool is_satisfied(Product *item) override {
        return item->color == color;
    }
};

struct SizeSpecification : Specification<Product>
{
    Size size;

    SizeSpecification(Size size) : size(size) {}

    bool is_satisfied(Product *item) override {
        return item->size == size;
    }
};

template <typename T> struct AndSpecification : Specification<T>
{
    Specification<T>& first;
    Specification<T>& second;

    AndSpecification(Specification<T> &first, Specification<T> &second) : first(first), second(second) {}

    bool is_satisfied(T *item) override {
        return first.is_satisfied(item) && second.is_satisfied(item);
    }
};

int main() {
    Product apple{"Apple", Color::green, Size::small};
    Product tree{"Tree", Color::green, Size::large};
    Product house{"House", Color::blue, Size::large};

    vector<Product*> items{&apple, &tree, &house};

    BetterFilter bf;
    ColorSpecification green{Color ::green};
    SizeSpecification medium{Size::medium};

    AndSpecification<Product> green_and_medium{green, medium};

    for (auto& item : bf.filter(items, green_and_medium)) {
        cout << item->name << endl;
    }
    return 0;
}
```