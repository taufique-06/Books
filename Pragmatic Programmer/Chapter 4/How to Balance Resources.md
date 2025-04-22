# How to Balance Resources

We all manage resources whenever we code: memory, transactions, threads, files, timers—all kinds of things with limited availability. Most
of the time, resource usage follows a predictable pattern: you allocate
the resource, use it, and then deallocate it. However, many developers have no consistent plan for dealing with resource allocation and deallocation. So let us suggest a simple tip:

`Finish What You Start`

This tip is easy to apply in most circumstances. It simply means that
the routine or object that allocates a resource should be responsible for
deallocating it. Let’s see how it applies by looking at an example of some
bad code—an application that opens a file, reads customer information
from it, updates a field, and writes the result back. We’ve eliminated
error handling to make the example clearer.

```csharp
void readCustomer(const char *fName, Customer *cRec) {
cFile = fopen(fName, "r+");
fread(cRec, sizeof(*cRec), 1, cFile);
}

void writeCustomer(Customer *cRec) {
rewind(cFile);
fwrite(cRec, sizeof(*cRec), 1, cFile);
fclose(cFile);
}

void updateCustomer(const char *fName, double newBalance) {
Customer cRec;
readCustomer(fName, &cRec);
cRec.balance = newBalance;
writeCustomer(&cRec);
}
```

At first sight, the routine updateCustomer looks pretty good. It seems
to implement the logic we require—reading a record, updating the balance, and writing the record back out. However, this tidiness hides a major problem. The routines readCustomer and writeCustomer are
tightly coupled—they share the global variable cFile. readCustomer opens the file and stores the file pointer in cFile, and writeCustomer
uses that stored pointer to close the file when it finishes. This global variable does not even appear in the updateCustomer routine.

Why is this bad? Let’s consider the unfortunate maintenance programmer who is told that the specification has changed—the balance should
be updated only if the new value is not negative. She goes into the
source and changes updateCustomer:

```csharp
void updateCustomer(const char *fName, double newBalance) {
Customer cRec;
readCustomer(fName, &cRec);
if (newBalance >= 0.0) {
cRec.balance = newBalance;
writeCustomer(&cRec);
}
}
```

All seems fine during testing. However, when the code goes into production, it collapses after several hours, complaining of too many open files.
Because writeCustomer is not getting called in some circumstances,
the file is not getting closed. A very bad solution to this problem would be to deal with the special
case in updateCustomer:

```csharp
void updateCustomer(const char *fName, double newBalance) {
Customer cRec;
readCustomer(fName, &cRec);
if (newBalance >= 0.0) {
cRec.balance = newBalance;
writeCustomer(&cRec);
}
else
fclose(cFile);
}
```

This will fix the problem—the file will now get closed regardless of the
new balance—but the fix now means that three routines are coupled
through the global cFile. We’re falling into a trap, and things are going
to start going downhill rapidly if we continue on this course. The finish what you start tip tells us that, ideally, the routine that allocates a resource should also free it. We can apply it here by refactoring
the code slightly:

```csharp
void readCustomer(FILE *cFile, Customer *cRec) {
fread(cRec, sizeof(*cRec), 1, cFile);
}

void writeCustomer(FILE *cFile, Customer *cRec) {
rewind(cFile);
fwrite(cRec, sizeof(*cRec), 1, cFile);
}

void updateCustomer(const char *fName, double newBalance) {
FILE *cFile;
Customer cRec;
cFile = fopen(fName, "r+"); 
readCustomer(cFile, &cRec);
if (newBalance >= 0.0) {
cRec.balance = newBalance;
writeCustomer(cFile, &cRec); 
}
fclose(cFile);
}
```

Now all the responsibility for the file is in the updateCustomer routine. It opens the file and (finishing what it starts) closes it before exiting. The
routine balances the use of the file: the open and close are in the same place, and it is apparent that for every open there will be a corresponding close. The refactoring also removes an ugly global variable.

## Objects and Exceptions

1. Class as a Resource Container:
   </br>
      In many OOP languages, classes can be thought of as containers for resources. A class can represent some kind of resource, like a file, a database connection, or a network socket. When you create an object of that class, you're essentially acquiring or allocating the resource. 
    </br>
      For example, in a program where you need to interact with a file system, you might have a FileHandler class that manages opening, reading, and closing files. The FileHandler object acts as a resource manager for the file.

2. Constructor and Destructor:
   </br>
- The constructor of a class is responsible for allocating or acquiring resources. It's like when you open a file or connect to a database.
- The destructor (or finalizer in some languages) is responsible for deallocating or releasing resources when the object is no longer needed. It's like when you close the file or release the database connection.
   </br>
```c++
class FileHandler {
private:
    FILE* file;
public:
    // Constructor allocates the resource (opens the file)
    FileHandler(const char* filename) {
        file = fopen(filename, "r");
        if (file == nullptr) {
            throw std::runtime_error("File could not be opened");
        }
    }

    // Destructor deallocates the resource (closes the file)
    ~FileHandler() {
        if (file) {
            fclose(file);
        }
    }
};

```

In this example:

- The constructor opens a file. 
- The destructor ensures that the file is properly closed when the FileHandler object goes out of scope.

3. Challenges with Exceptions
   </br>
   The challenge arises when your code has exceptions. For example, imagine that during the process of allocating or using a resource, an exception occurs (maybe you encounter an error or unexpected input). This can interfere with the proper deallocation of the resource because the object might not get destroyed in the normal way.

```c++
void processFile(const char* filename) {
    FileHandler fileHandler(filename);  // Constructor opens file

    // Code that works with the file...
    
    if (some_error_condition) {
        throw std::runtime_error("Something went wrong!");  // Exception thrown
    }

    // Destructor would typically be called here when fileHandler goes out of scope
}
```

If an exception occurs after the file has been opened and before the fileHandler object goes out of scope, the destructor that would close the file might not get executed. This is because the normal flow of execution is interrupted by the exception, and the object may never get cleaned up properly.