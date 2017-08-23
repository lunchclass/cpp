If we’re working in a mathematical domain, we might find it convenient to have a class representing polynomials. 
A function that compute the root(s) of a polynomial would not modify the polynomial, so it’d be natural to declare it const: 

```
class Polynomial {
public:
  using RootsType =               // data structure holding values
    std::vector<double>;          // where polynomial evals to zero
... 
  RootsType roots() const;
... }; 
```

we certainly don’t want to do it more than once because it can be expensive. We’ll thus cache the root(s) of the polynomial if we have to compute them, and we’ll implement roots to return the cached value. Here’s the basic approach: 

```
   class Polynomial {
   public:
     using RootsType = std::vector<double>;
  RootsType roots() const
  {
    if (!rootsAreValid) {                   // if cache not valid
      ...                                               // compute roots,
                                                        // store them in rootVals
      rootsAreValid = true;        
    }
    return rootVals;
  }
private:
  mutable bool rootsAreValid{ false };
  mutable RootsType rootVals{};
}; 
```

Conceptually, roots doesn’t change the Polynomial object on which it operates, but, as part of its caching activity, it may need to modify rootVals and rootsAreValid. That’s a classic use case for mutable, and that’s why it’s part of the declarations for these data members. 

```
Polynomial p; 
... 
   /*-----  Thread 1  ----- */     /*-------  Thread 2  ------- */
auto rootsOfP = p.roots(); auto valsGivingZero = p.roots(); 
```

roots is a const member function, that means it represents a read operation.
Having multiple threads perform a read opera‐ tion without synchronization is safe. 
In this case, it’s not, because inside roots, one or both of these threads might try to modify the data members rootsAreValid and rootVals. 


what requires rectification is the lack of thread safety. 
The easiest way to address the issue is the usual one: employ a mutex: 

```
class Polynomial {
public:
  using RootsType = std::vector<double>;
  RootsType roots() const
  {
} 
std::lock_guard<std::mutex> g(m);           // lock mutex
if (!rootsAreValid) {                                      // if cache not valid                          
  ...                                                                  // compute/store roots 
  rootsAreValid = true;
}                                                                
return rootVals;
}                                                                     // unlock mutex   

private: 
  mutable std::mutex m;
  mutable bool rootsAreValid{ false };
  mutable RootsType rootVals{};
};
```

In some situations, a mutex is overkill. 
Here’s how you can employ a std::atomic to count calls: 

```
class Point {
public:
... 
  double distanceFromOrigin() const noexcept
  {
    ++callCount;                 // atomic increment
    return std::hypot(x, y);
  }
private: 
     mutable std::atomic<unsigned> callCount{ 0 };
     double x, y;
   };
```

Because operations on std::atomic variables are often less expensive than mutex acquisition and release, you may be tempted to lean on std::atomics more heavily than you should. 

For example, in a class caching an expensive-to-compute int, you might try to use a pair of std::atomic variables instead of a mutex: 

```
class Widget {
public:
... 
  int magicValue() const
  {
    if (cacheValid) return cachedValue;
    else {
        auto val1 = expensiveComputation1();
        auto val2 = expensiveComputation2(); 
        cachedValue = val1 + val2; 
        cacheValid = true; 
        return cachedValue;
    }
} 
private: 

mutable std::atomic<bool> cacheValid{ false }; 
mutable std::atomic<int> cachedValue; 
}; 
```

This will work, but sometimes it will work a lot harder than it should. Consider: 
	• A thread calls Widget::magicValue, sees cacheValid as false, performs the 
two expensive computations, and assigns their sum to cachedValue. 
	• At that point, a second thread calls Widget::magicValue, also sees cacheValid as false, and thus carries out the same expensive computations that the first thread has just finished. (This “second thread” may in fact be several other threads.) 

Such behavior is contrary to the goal of caching. Reversing the order of the assign‐ ments to cachedValue and CacheValid eliminates that problem, but the result is even worse: 

```
class Widget {
public:
... 
  int magicValue() const
  {
    if (cacheValid) return cachedValue;
    else {
        auto val1 = expensiveComputation1(); 
        auto val2 = expensiveComputation2(); 
        cacheValid = true;
        return cachedValue = val1 + val2; 
        } 
    } 
... 
}; 
```

Imagine that cacheValid is false, and then:
• One thread calls Widget::magicValue and executes through the point where 
cacheValid is set to true.
• At that moment, a second thread calls Widget::magicValue and checks cache 
Valid. Seeing it true, the thread returns cachedValue, even though the first thread has not yet made an assignment to it. The returned value is therefore incorrect. 


There’s a lesson here. For a single variable or memory location requiring synchroni‐ zation, use of a std::atomic is adequate, but once you get to two or more variables or memory locations that require manipulation as a unit, you should reach for a mutex. 
