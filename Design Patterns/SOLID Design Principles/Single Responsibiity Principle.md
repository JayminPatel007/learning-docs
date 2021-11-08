# Single Responsibility Principle
A class should have a single reason to change, in other words a class should have a primary responsibility and it should not take up other responsibility.

## Explanation
```c++
#include <iostream>
#include <cstdio>
#include <string>
#include <vector>
#include <fstream>

using namespace std;

struct Journal
{
    string title;
    vector<string> entries;

    Journal(const string &title) : title(title) {}

    void add_entry(const string& entry) {
        static int count = 1;
        entries.push_back(entry);
    }

    void save(const string& filename) {
        ofstream ofs(filename);
        for (auto& e : entries) {
            ofs << e << endl;
        }
    }
};

int main() {
    Journal journal{"Dear Diary"};
    journal.add_entry("I at a panipuri");
    journal.add_entry("I went to rishivan today");
    return 0;
}
```

in above example Journal class has 2 responsibilities, it is keeping track of journal entries and also it is responsible for storing it to the file. Let's say there are many classes which needs persistance then we need to add save method to all the classes and if later on there is a change required in save method then we have to make change at all the places. Better solution would be the following

```c++
#include <iostream>
#include <vector>
#include <fstream>

using namespace std;

struct Journal
{
    string title;
    vector<string> entries;

    Journal(const string &title) : title(title) {}

    void add_entry(const string& entry) {
        static int count = 1;
        entries.push_back(entry);
    }
};

struct PersistanceManager {
    static void save(Journal j, const string &filename) {
        ofstream ofs(filename);
        for (auto &e : j.entries) {
            ofs << e << endl;
        };
    }
};

int main() {
    Journal journal{"Dear Diary"};
    journal.add_entry("I at a panipuri");
    journal.add_entry("I went to rishivan today");
    PersistanceManager::save(journal, "test");
    return 0;
}
```