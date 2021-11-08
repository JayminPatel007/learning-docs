# Interface Segregation Principle
Idea of the interface segregation principle is basically to get you to not to create interfaces which are too large and require implementors to implement too much.

```c++
#include <iostream>
#include <vector>
#include <fstream>

using namespace std;

struct Document;

struct IMachine
{
    virtual void print(Document& doc) = 0;
    virtual void scan(Document& doc) = 0;
    virtual void fax(Document& doc) = 0;
};

struct MFP : IMachine
{
    void print(Document& doc) override {
        // OK
    }

    void scan(Document& doc) override {
        // OK
    }

    void fax(Document& doc) override {
        // OK
    }
};

struct Printer: IMachine
{
    void print(Document &doc) override {
        // OK
    }

    void scan(Document &doc) override {
        // ?
    }

    void fax(Document &doc) override {
        // ?
    }
};

int main() {
    return 0;
}
```

In above example problem is that let's say we have now to implement just a printer or just a scanner. The better implementation would be the following...

```c++
#include <iostream>
#include <vector>
#include <fstream>

using namespace std;

struct Document;

struct IPrinter
{
    virtual void print(Document& doc) = 0;
};

struct IScanner
{
    virtual void scan(Document& doc) = 0;
};

struct IFax
{
    virtual void fax(Document& doc) = 0;
};

struct Printer : IPrinter
{
    void print(Document &doc) override {
        // OK
    }
};

struct IMachine : IPrinter, IScanner, IFax {};

struct MFP: IMachine
{
    IPrinter& printer;
    IScanner& scanner;
    IFax& faxMachine;

    MFP(IPrinter &printer, IScanner &scanner, IFax &faxMachine) : printer(printer), scanner(scanner),
                                                                  faxMachine(faxMachine) {}

    void print(Document &doc) override {
        printer.print(doc);
    }

    void scan(Document &doc) override {
        scanner.scan(doc);
    }

    void fax(Document &doc) override {
        faxMachine.fax(doc);
    }
};

int main() {
    return 0;
}
```