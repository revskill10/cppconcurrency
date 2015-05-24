# Using Threads to Execute Concurrent Tasks

## Problem

You’re writing a program that is performing poorly, and you’d like to speed up execution by using multiple processors in a system.

## Solution

C++ provides the `thread` type, which can be used to create a native operating system thread. Program threads can be run on more than a single processor and therefore allow you to write programs that can use 
multiple CPUs and CPU cores.

## How It Works

### Detecting the Number of Logical CPU Cores

The C++ thread library provides a feature set that lets programs use all the cores and CPUs available in a  given computer system. The first important function supplied by the C++ threading capabilities that you  should be aware of allows you to query the number of execution units the computer contains.
Listing 1-1 shows the C++ `thread::hardware_concurrency` method.

    #include <iostream>
    #include <thread>
     
    using namespace std;
     
    int main(int argc, char* argv[])
    {
        const unsigned int numberOfProcessors{ thread::hardware_concurrency() };
     
        cout << "This system can run " << numberOfProcessors << " concurrent tasks" << endl;
     
        return 0;
    }
     
     
This code uses the `thread::hardware_concurrency` method to query the number of simultaneous 
threads that can be run on the computer executing the program. 

### Creating Threads

Once you know the system you’re running on might benefit from the use of concurrent execution, you can use the C++ thread class to create tasks to be run on multiple processor cores. The thread class is a portable,  built-in type that allows you to write multithreaded code for any operating system.

**Note** *The `thread` class is a recent addition to the C++ programming language. It was added in the C++11 language spec, so you may need to check the documentation for the StL library you’re using to ensure that it 
supports this feature.*

The `threa`d constructor is simple to use and takes a function to execute on another CPU core. 

Listing 1-2  shows a simple thread that outputs to the console.

    #include <iostream>
    #include <thread>
     
    using namespace std;
     
    void ThreadTask()
    {
        for (unsigned int i{ 0 }; i < 20; ++i)
        {
            cout << "Output from thread" << endl;
        }
    }
     
    int main(int argc, char* argv[])
    {
        const unsigned int numberOfProcessors{ thread::hardware_concurrency() };
     
        cout << "This system can run " << numberOfProcessors << " concurrent tasks" << endl;
     
        if (numberOfProcessors > 1)
        {
            thread myThread{ ThreadTask };
     
            cout << "Output from main" << endl;
     
            myThread.join();
        }
        else
        {
            cout << "CPU does not have multiple cores." << endl;
             }
     
            return 0;
    }
    
Listing 1-2 determines whether to create a thread based on the number of logical cores on the computer executing the program.

**Note** *   Most operating systems allow you to run more threads than there are processors, but you might find  that doing so slows your program due to the overhead of managing multiple threads.*


If the CPU has more than one logical core, the program creates a thread object called `myThread`. The  `myThread` variable is initialized with a pointer to a function.

This function will be executed in the thread 
context and, more likely than not, on a different CPU thread than the main function.


The `ThreadTask` function consists of a for loop that simply outputs to the console multiple times. 


The `main` function also outputs to the console. The intent is to show that both functions are running  concurrently. 

### Cleaning Up After Threads

The main function in Listing 1-2 immediately calls the `join` method on the thread. The `join` method is  used to tell the current thread to wait for the additional thread to end execution before continuing. This is 
important because C++ programs are required to destroy their own threads to prevent leaks from occurring. 

Calling the destructor on a thread object doesn’t destroy the currently executing thread context. Listing 1-3  shows code that has been modified to not call `join` on `myThread`.

*Listing 1-3.  Forgetting to Call join on a thread*

    #include <iostream>
    #include <thread>
     
    using namespace std;
     
    void ThreadTask()
    {
        for (unsigned int i{ 0 }; i < 20; ++i)
        {
            cout << "Output from thread" << endl;
        }
    }
     
    int main(int argc, char* argv[])
    {
        const unsigned int numberOfProcessors{ thread::hardware_concurrency() };
     
        cout << "This system can run " << numberOfProcessors << " concurrent tasks" << endl;
     
        if (numberOfProcessors > 1)
        {
            thread myThread{ ThreadTask };
     
            cout << "Output from main" << endl;
        }
        else
        {
            cout << "CPU does not have multiple cores." << endl;
        }
     return 0;
    }
    
This code causes the `myThread` object to go out of scope before the `ThreadTask` function has completed  execution. This can cause a thread leak in your program that may eventually cause the program or the 
operating system to become unstable.

One approach is to use the `joi`n method to make the program wait for threads to finish before closing  them down. C++ also provides a second option: the `detach` method. Listing 1-4 shows the detach method  in use.

*Listing 11-4.  Using the detach Method*
    #include <iostream>
    #include <thread>
     
    using namespace std;
     
    void ThreadTask()
    {
        for (unsigned int i = 0; i < 20; ++i)
        {
            cout << "Output from thread" << endl;
        }
    }
     
    int main(int argc, char* argv[])
    {
        const unsigned int numberOfProcessors{ thread::hardware_concurrency() };
     
        cout << "This system can run " << numberOfProcessors << " concurrent tasks" << endl;
     
        if (numberOfProcessors > 1)
        {
            thread myThread{ ThreadTask };
     
            cout << "Output from main" << endl;
     
            myThread.detach();
        }
        else
        {
            cout << "CPU does not have multiple cores." << endl;
        }
        return 0;
    }
    
Listing 1-4 shows that the `detach` method can be used in place of `join`. The `join` method causes the  program to wait for a running thread to complete before continuing, but the detach method doesn’t.  

The `detach` method allows you to create threads that outlive the execution of your program. These may be useful for system tasks that need to track time over long periods; however, I’m skeptical about whether many 
day-to-day programs will find a use for this method. There’s also a risk that your program will leak threads  that have been detached and have no way to get those tasks back. Once an execution context in a thread has 
been detached, you can never reattach it.    
