# Adapter Pattern

## Motivation
- Often we have existing code (a class with a particular interface) that we cannot modify
- We need to use that code but it has an incompatible interface with what our system expects
- The Adapter pattern acts as a "wrapper" or "translator" between two incompatible interfaces
- Converts the interface of a class into another interface that clients expect

&nbsp;
> *A construct which adapts an existing interface X to conform to the required interface Y*

&nbsp;

## Problem: Incompatible Interfaces

Consider a drawing application that works with a `Shape` interface, but you have a third-party library that provides `LegacyRectangle` with a different interface:

```c++
#include <iostream>
using namespace std;

// Target interface our application expects
struct Shape {
    virtual void draw() = 0;
};

// Existing class with incompatible interface (cannot modify)
struct LegacyRectangle {
    void drawRectangle(int x1, int y1, int x2, int y2) {
        cout << "LegacyRectangle: drawing from ("
             << x1 << "," << y1 << ") to ("
             << x2 << "," << y2 << ")" << endl;
    }
};

// Without an adapter, we cannot use LegacyRectangle where Shape is expected
```

&nbsp;

## Object Adapter

The Object Adapter wraps an instance of the adaptee (composition):

```c++
#include <iostream>
using namespace std;

// Target interface
struct Shape {
    virtual void draw() = 0;
    virtual ~Shape() = default;
};

// Adaptee (existing class with incompatible interface)
struct LegacyRectangle {
    int x1, y1, x2, y2;

    LegacyRectangle(int x1, int y1, int x2, int y2)
        : x1(x1), y1(y1), x2(x2), y2(y2) {}

    void drawRectangle() {
        cout << "LegacyRectangle: drawing from ("
             << x1 << "," << y1 << ") to ("
             << x2 << "," << y2 << ")" << endl;
    }
};

// Adapter: wraps LegacyRectangle and implements Shape
struct RectangleAdapter : Shape {
private:
    LegacyRectangle legacy;

public:
    RectangleAdapter(int x, int y, int width, int height)
        : legacy(x, y, x + width, y + height) {}

    void draw() override {
        legacy.drawRectangle();
    }
};

void render(Shape& shape) {
    shape.draw();
}

int main() {
    RectangleAdapter adapter{10, 20, 100, 50};
    render(adapter); // Works seamlessly!
    return 0;
}
```

&nbsp;

## Class Adapter (using multiple inheritance)

In C++, we can also adapt using multiple inheritance — inheriting from both the target interface and the adaptee:

```c++
#include <iostream>
using namespace std;

struct Shape {
    virtual void draw() = 0;
    virtual ~Shape() = default;
};

struct LegacyRectangle {
    int x1, y1, x2, y2;

    LegacyRectangle(int x1, int y1, int x2, int y2)
        : x1(x1), y1(y1), x2(x2), y2(y2) {}

    void drawRectangle() {
        cout << "LegacyRectangle: drawing from ("
             << x1 << "," << y1 << ") to ("
             << x2 << "," << y2 << ")" << endl;
    }
};

// Class Adapter: inherits from both Shape and LegacyRectangle
struct RectangleAdapter : Shape, LegacyRectangle {
    RectangleAdapter(int x, int y, int width, int height)
        : LegacyRectangle(x, y, x + width, y + height) {}

    void draw() override {
        drawRectangle(); // directly calls inherited method
    }
};

int main() {
    RectangleAdapter adapter{10, 20, 100, 50};
    adapter.draw();
    return 0;
}
```

&nbsp;

## Adapter with Caching

When the adaptation involves an expensive transformation, we can cache the result:

```c++
#include <iostream>
#include <string>
#include <map>
#include <vector>
using namespace std;

// Imagine a geometry library using points
struct Point {
    int x, y;
};

// Our rendering system works with pixels (vector of points)
struct PixelRenderer {
    void render(const vector<Point>& pixels) {
        for (const auto& p : pixels) {
            cout << "Pixel at (" << p.x << "," << p.y << ")" << endl;
        }
    }
};

// Legacy geometry: works with lines defined by two endpoints
struct Line {
    Point start, end;
};

struct VectorImage {
    vector<Line> lines;
};

// Adapter: converts VectorImage (lines) to pixels for PixelRenderer
struct VectorToRasterAdapter {
private:
    vector<Point> pixels;

    void generatePoints(const Line& line) {
        // Simple line rasterization (Bresenham's algorithm simplified)
        int x = line.start.x, y = line.start.y;
        int dx = abs(line.end.x - x), dy = abs(line.end.y - y);
        int sx = (line.end.x > x) ? 1 : -1;
        int sy = (line.end.y > y) ? 1 : -1;
        int err = dx - dy;

        while (true) {
            pixels.push_back({x, y});
            if (x == line.end.x && y == line.end.y) break;
            int e2 = 2 * err;
            if (e2 > -dy) { err -= dy; x += sx; }
            if (e2 < dx)  { err += dx; y += sy; }
        }
    }

public:
    VectorToRasterAdapter(const VectorImage& image) {
        for (const auto& line : image.lines) {
            generatePoints(line);
        }
    }

    const vector<Point>& getPixels() const { return pixels; }
};

int main() {
    VectorImage image;
    image.lines.push_back({{0, 0}, {5, 5}});
    image.lines.push_back({{0, 5}, {5, 0}});

    VectorToRasterAdapter adapter{image};

    PixelRenderer renderer;
    renderer.render(adapter.getPixels());

    return 0;
}
```

&nbsp;

## Summary
- Adapter is a structural design pattern that allows objects with incompatible interfaces to collaborate
- Implementing an adapter involves:
  - Determining the target interface your client code expects
  - Wrapping the adaptee (existing class) to implement that interface
- **Object Adapter** uses composition — holds a reference/instance of the adaptee (preferred, more flexible)
- **Class Adapter** uses multiple inheritance — inherits from both target and adaptee (C++ specific)
- The adapter translates calls from the target interface into calls the adaptee understands
- Can introduce caching to avoid repeated expensive conversions
- Follows the Open/Closed Principle: you can introduce adapters without changing existing code
