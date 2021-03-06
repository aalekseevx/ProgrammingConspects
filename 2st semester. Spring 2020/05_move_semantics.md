# Lecture 5. Move semantics

## 10.7 Stream iterators

Proxies for basic streams to use as iterators
```cpp
template<typename T>

// Impementation is similar to this:

struct istream_iterator
{
private:
	std::basic_istream* in;
	T cur;
public:

	T& operator*() {
		return cur;
	}
	istream_iterator& operator++() {
		*in >> cur;
	}
};

template<typename T>
struct ostream_iterator
{
private:
	std::basic_ostream* out;
public:
	// constructs from stream and delimeter
	ostream_iterator& operator*() {}
	ostream_iterator& operator++() {}
	ostream_iterator& operator=(const T& value) {
		out << value;
	}
};
```

## 10.8 Iterator invalidation

```cpp
it = v.begin();
// some operators, which change size of the vector
*it // UB

// also UB
int& it = v[0];
v.push_back();
*it
```

Containers, which do rebuild themselves (invalidate iterators)
- vector (deque)
- umap

Do not invalidate
- map/set
- list/forward_list

# Unit 11. Move semantics.

## 11.1 Motivation

```cpp
// bad swap, uses a lot of copy operators
template<typename T>
void swap(T& x, T& y) {
	T t = x;
	x = y;
	y = t;
}

//bad construct with tmo objects
void construct(T* p, const Args&... args) {
	new(p) T(args);
}

// also emplace_back is trying to fix the problem
```

We would like to copy temporary objects by stealing data from it. The object will be then destroyed.

## 11.2 Magic `std::move` used in calls, where object needs to be moved.

```cpp
// fast swap
void swap(T& x, T& y) {
	T t = std::move(y);
	x = std::move(y);
	y = std::move(t);
}
```

## 11.3 Move constructor and move assignment

Do describe resource stealing process, we define move constructor and move assignment operator along with simple ones.

```cpp
// T&& is an rvalue-reference, result of std::move()
vector(vector<T>&& other) {
	alloc = std::move(other.alloc);
	sz = other.sz;
	cp = other.cp;
	arr = other.arr;
	other.app = nullptr;
}
```

Rvalue reference is a new type of objects.

Canonical move constructor (generates automatically)

- Copies pointers, makes pointers nullptr
- `moves` all other data.

Use `= default` to explicitly generate a method, `= delete` to remove autogenerated method.

Move assignment (can be generated automatically)
```cpp
vector<T>& operator=(Vector<T>&&);
```

Rule of 5 is rule of 3 + (2 above)

`Vector<T>&&` -> casts to `const Vector<T>&`, **But not vice versa**.

Rvalue prefers move assignement

Definition connected to assignment operator doesn't work:
- Constant objects are lvalue, but can't be assigned to
- If operator= is badly overloaded, there can be lvalue at the end. // x + y = 5, what?

## 11.4 Lvalue and rvalue

Lvalue and Rvalue are types of expressions.

Lvalue:

- identifier, string literal
- function call with lvalue-ref return type
- builtin operators:
	* prefix++
	* =
	* op=
	* unary operators

Rvalue:

- literal (except string litetal)
- function call non-lvalue-ref return type
- builtin operators:
	* arithmetic, +, -, &, |, ==, !=, ~

Different results : operator "?", ","

`std::move` returns `static_cast<typename std::remove_reference<T>::type&&>(t)` and makes object rvalue-like.
Called rvalue-references. So it now prefers move constructors and assignment.

## 11.5 Rvalue references

```cpp
int x = 5;
int&& y = x; // prohibited cast
int&& y = std::move(x); // correct
int&& z = y; // also prohibited, y is an lvalue as identifier
int& z = y; // y is another name for x, so OK
const int& t = std::move(y); // also ok
const int&& u = std::move(t); // also ok
```

Rvalue references are almost the same to lvalue references after the declaration.