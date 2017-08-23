Prefer const_iterators to iterators
---
(The content is an excerpt and summary from Effective Modern C++ for the purpose of this study.)

const_iterator: points to values that's not modified. The standard practice of using const *whenever* possible encourages us to use const_iterator when possible.

In C++98, it wasn't easy to create them and had limited use.

```c++
// Using iterators in C++98
std::vector<int> values;

...

std::vector<int>::iterator it = std::find(values.begin(), values.end(), 1983);
values.insert(it, 1998);

```

iterator in the above snippet isn't the best choice as we don't mutate what the iterator points to. So we want to use const_iterator!

```c++
// Trying to use const_iterators in C++98. Not working!
typedef std::vector<int>::iterator IterT;
typedef std::vector<int>::const_iterator ConstIterT;

std::vector<int> values;

...

ConstIterT ci = std::find(static_cast<ConstIterT>(values.begin()), static_cast<ConstIterT>(values.end()), 1983);
values.insert(static_cast<IterT>(ci), 1998) // may not compile
```

The above code cast because in C++98 there was no way to get a const_iterator from a non-const container directly. We may bind values to a reference-to-const variable and get a const_iterator from it. But we're putting additional work anyway.

Another issue even after having gotten the const_iterator is locations for insertions can be specified only by iterators. Passing a const_iterator to insert wouldn't compile.

The insert call above wouldn't compile anyway indeed as there's no conversion from a const_iterator to an iterator can be done. (even reinterpret_cast won't do it.)

So, the point of the author is const_iterators were so much trouble in C++98. Devs didn't use them when they aren't very practical.

C++11 changes that! It's now easy to get and easy to use.

- The container member functions (cbegin, cend) return const_iterators even for non-const containers.
- STL member functions that use iterators to indicate the positions (e.g. insert, erase) use const_iterators.

```c++
// C++11 does it!
std::vector<int> values;

...

auto it = std::find(values.cbegin(), values.cend(), 1983);
values.insert(it, 1998);
```

When C++11's const_iterator support comes a bit short is to write maximally generic lib.

```c++
template <typename C, typename V>
void findAndInsert(C& container, const V& targetVal, const V& insertVal)
{
  using std::cbegin;
  using std::cend;

  auto it = std::find(cbegin(container), cend(container), targetVal);
  container.insert(it, insertVal);
}
```

Works in C++14. Not in C++11. C++11 added begin and end but failed to add cbegin, cend, rbegin, crbegin, and crend.

