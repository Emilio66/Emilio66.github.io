## Usage of Condition Variable in C++11
In concurrent programming, *Monitor* is a synchronization mechanism that allows threads to perform exclusive access, wait for a certain condition to be staisfied and waken up blocked thread. *Condition Variable* is an object that represents a condition for which threads are waiting. Before the condition meets, all the waited threads are blocked or slept. Once the conditon met, they will be notified and ready to continue running.
[Ref: WiKi](https://en.wikipedia.org/wiki/Monitor_(synchronization)#Condition_variables)

In c++11, there is class *thread* that stands for individual threads of execution. It's a encapsulation of thread that is able to execute correctly among all platform. There's no need to use Window API and spare the efforts to use handlers. [For more details, see thread.h](http://www.cplusplus.com/reference/thread/thread/)

> A condition variable is an object able to block the calling thread until notified to resume.
It uses a unique_lock (over a mutex) to lock the thread when one of its wait functions is called. The thread remains blocked until woken up by another thread that calls a notification function on the same condition_variable object.
Objects of type condition_variable always use unique_lock<mutex> to wait: for an alternative that works with any kind of lockable type, see condition_variable_any
[Reference](http://www.cplusplus.com/reference/condition_variable/condition_variable/)

### Basic Usage:

    // condition_variable example

    #include <iostream>           // std::cout
    #include <thread>             // std::thread
    #include <mutex>              // std::mutex, std::unique_lock
    #include <condition_variable> // std::condition_variable

    std::mutex mtx;
    std::condition_variable cv;   //condition varibale object
    bool ready = false;

    void print_id (int id) {
      std::unique_lock<std::mutex> lck(mtx); //acquire lock
      while (!ready) 
        cv.wait(lck); //!!NOTICE: If condition not met, lck will give away the lock until condition is ready, try to regain the lock 
      // do tasks
      std::cout << "thread " << id << '\n';
    }

    void go() {
      std::unique_lock<std::mutex> lck(mtx);
      ready = true;
      cv.notify_all();  //changes happen in main thread and notify waited threads
    }

    int main ()
    {
      std::thread threads[10];
      // spawn 10 threads:
      for (int i=0; i<10; ++i)
        threads[i] = std::thread(print_id,i);

      std::cout << "10 threads ready to race...\n";
      go();    // go!

      for (auto& th : threads)
        th.join();

      return 0;
    }

### Use condition variable to implement classic *Producer-consumer Model*:

    
    #include <iostream>
    #include <condition_variable>
    #include <thread>
    #include <chrono>
    #include <mutex>

    using namespace std;

    //queue mutex
    mutex queue_mutex;

    condition_variable producer_cv;
    condition_variable consumer_cv;

    int size = 0;
    int MAX = 10;
    bool empty() {return size == 0;}
    bool full() {return size == MAX;}

    void produce() {
        //acquire lock
        unique_lock<mutex> full_lock(queue_mutex);
        //check if predicate is true
        while (full()) {
            //wait until queue is not full, wake me up
            producer_cv.wait(full_lock); 
        }
        //produce something
        cout <<"size: " << size << " Hurray, produce produce !!! --from thread "<< this_thread::get_id() << endl;
        ++size;
        consumer_cv.notify_one();//notify a consumer that product is ready 
    }

    void consume() {    
        unique_lock<mutex> empty_lock(queue_mutex);
        while (empty()) {
            consumer_cv.wait(empty_lock); //when not empty, wake me up
        }
        //consume
        cout  <<"size: " << size << "Yummy, consuming is great! --from thread "<< this_thread::get_id() << endl;
        --size;
        producer_cv.notify_one();//notify a produer that repo is available 
    }

    int main() {
        thread pts [10];
        thread cts [10];

        for (int i = 0; i < 10; i++) 
           pts[i] = thread(produce);

        for (int i = 0; i < 10; i++)
           cts[i] = thread(consume);

        for (int i = 0; i < 10; i++) {
            pts[i].join();
            cts[i].join();
        } 
        return 0;
    }

[Recommend code snippet stored in gist](https://gist.github.com/Emilio66)
