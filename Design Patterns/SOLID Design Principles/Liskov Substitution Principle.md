# Liskov Substitution Principle
This principle states that Subtype should be immediately substitutable for their base type.

## Explanation
```c++
#include <iostream>
#include <vector>
#include <fstream>

using namespace std;

class Rectangle
{
protected:
    int width, height;
public:
    Rectangle(int width, int height) : width(width), height(height) {}

    int getWidth() const {
        return width;
    }

    virtual void setWidth(int width) {
        Rectangle::width = width;
    }

    int getHeight() const {
        return height;
    }

    virtual void setHeight(int height) {
        Rectangle::height = height;
    }

    int area() const { return width * height; }
};

class Square : public Rectangle
{
public:
    Square(int size): Rectangle(size, size) {}

    void setWidth(int width) override {
        this->width = this->height = width;
    }

    void setHeight(int height) override {
        this->width = this->height = height;
    }
};

void process(Rectangle& r) {
    int w = r.getWidth();
    r.setHeight(10);

    cout << "expected output = " << (w * 10) << ", got " << r.area() << endl;
}

int main() {
    Rectangle r{3,4};
    process(r);
    
    Square sq{5};
    process(sq);
    return 0;
}
```

In the example above, process function takes reference of Rectangle object, and in the example above square is a derived class of Rectangle then according to this principle if we pass reference of square then also it should work fine, but it doesn't. Better approach would be to create a factory which will create appropriate Rectangles.

```c++
#include <iostream>
#include <vector>
#include <fstream>

using namespace std;

class Rectangle
{
protected:
    int width, height;
public:
    Rectangle(int width, int height) : width(width), height(height) {}

    int getWidth() const {
        return width;
    }

    void setWidth(int width) {
        Rectangle::width = width;
    }

    int getHeight() const {
        return height;
    }

    void setHeight(int height) {
        Rectangle::height = height;
    }

    int area() const { return width * height; }
};

struct RectangleFactory
{
    static Rectangle create_rectangle(int w, int h) {
        return Rectangle{w,h};
    }
    static Rectangle create_square(int size) {
        return Rectangle{size, size};
    }
};

void process(Rectangle& r) {
    int w = r.getWidth();
    r.setHeight(10);

    cout << "expected output = " << (w * 10) << ", got " << r.area() << endl;
}

int main() {
    Rectangle r = RectangleFactory::create_rectangle(3,4);
    process(r);

    Rectangle sq = RectangleFactory::create_square(5);
    process(sq);
    return 0;
}

```