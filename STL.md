# 1. What does STL contain?

Containers — store data
vector, list, stack, queue, deque, set, map, unordered_map, priority_queue

Algorithms — process data
sort(), reverse(), find(), binary_search(), min(), max(), count()

Iterators — navigate through data
begin(), end(), rbegin(), rend() and iterator variables

Utilities
pair, tuple, greater, and custom comparators

## Vector

### Creating a vector

vector<int> v;                      // empty

vector<int> v(5);                   // {0,0,0,0,0}

vector<int> v(5, 7);                // {7,7,7,7,7}

vector<int> v = {1, 2, 3};          // from list

vector<int> w(v);                   // copy of v

vector<int> w(v.begin(), v.begin() + 2); // copy of a range: {1,2}

vector<int> a(arr, arr + n);        // from a plain array

Watch the difference:

vector<int> v(5);      // 5 elements, all 0

vector<int> v{5};      // 1 element, value 5

### Adding, removing, accessing
Function	  	  		    What it does			  	  	  	  	  Time

v.push_back(x)			    add at end		  	  	  	  	  	  	O(1) amortized

v.emplace_back(a, b)	  	construct in place (for pairs/structs)	O(1) amortized

v.pop_back()		  	    remove last (returns nothing)	  	  	O(1)

v[i]		  	  	  	    access, no bounds check	                O(1)

v.at(i)		  	  	  	    access, throws error if out of range	O(1)

v.front() / v.back()	  	first / last element	                O(1)

v.size()	  	  	  	  	number of elements	                    O(1)

v.empty()	  	  	  	  	true if size is 0                   	O(1)

v.clear()	  	  	  	  	remove all (size 0, capacity unchanged)	O(n)

### Size and capacity control
cpp

v.resize(k);        // size becomes k; new slots are 0, extra elements are cut

v.resize(k, x);     // new slots filled with x

v.reserve(k);       // capacity at least k; size does NOT change

v.assign(k, x);     // replace contents with k copies of x

v.shrink_to_fit();  // release unused capacity 


v.reserve(n)

Requests capacity for at least n elements without changing the vector's size.

vector<int> v;

v.reserve(100);

cout << v.size();     // 0

cout << v.capacity();

### Insert and erase (the slow operations)

v.insert(v.begin() + 2, x);              // insert x at index 2

v.insert(v.begin() + 2, 3, x);           // insert 3 copies

v.insert(v.end(), w.begin(), w.end());   // append another vector

v.erase(v.begin() + 2);                  // remove index 2

v.erase(v.begin() + 1, v.begin() + 4);   // remove indices 1, 2, 3 To erase a range, the second iterator is exclusive:

Cost is O(n), because later elements shift (same reasoning as arrays).

If you call erase inside a loop over n elements, you have O(n²). With n = 10⁵ that is TLE.

Alternatives:

Only the end matters → pop_back() is O(1).

Order doesn’t matter → swap with last, then pop_back():

swap(v[i], v.back()); v.pop_back();

## 2D vectors (grids, adjacency lists)

int n = 3, m = 4;
vector<vector<int>> g(n, vector<int>(m, 0));  // n x m grid of zeros
g[1][2] = 5;

### Passing vectors to functions
cpp
void f(vector<int> v)          // COPIES the whole vector: O(n) every call

void f(vector<int> &v)         // reference: no copy, can modify

void f(const vector<int> &v)   // reference: no copy, read-only

Passing by value inside a recursive function or a loop silently adds O(n) per call. This is a classic hidden TLE.

Default habit: const vector<int> &v if you only read, vector<int> &v if you modify.

### Copy, compare, swap
cpp
vector<int> b = a;          // deep copy, O(n)

a == b;                     // element-by-element comparison

a < b;                      // lexicographic comparison

a.swap(b);                  // O(1), swaps internals

### Vector with algorithms (the combination you’ll use constantly)
cpp

sort(v.begin(), v.end());                       // ascending

sort(v.rbegin(), v.rend());                     // descending

reverse(v.begin(), v.end());

int mx = *max_element(v.begin(), v.end());

int mn = *min_element(v.begin(), v.end());

long long s = accumulate(v.begin(), v.end(), 0LL);   // 0LL, not 0

int pos = lower_bound(v.begin(), v.end(), x) - v.begin();  // sorted only

bool has = binary_search(v.begin(), v.end(), x);           // sorted only

iota(v.begin(), v.end(), 0);                    // 0,1,2,3,...

fill(v.begin(), v.end(), -1);

Remove duplicates:

cpp
sort(v.begin(), v.end());

v.erase(unique(v.begin(), v.end()), v.end());

Erase-remove (delete all elements equal to x, or by condition):

cpp
v.erase(remove(v.begin(), v.end(), x), v.end());

v.erase(remove_if(v.begin(), v.end(), [](int a){ return a % 2 == 0; }), v.end());

remove doesn’t actually shrink the vector. It shifts the kept elements to the front and returns the new logical end. erase then cuts the leftover tail. Both steps are needed. Total O(n).


#### 1. accumulate: sum of elements
cpp

long long s = accumulate(v.begin(), v.end(), 0LL);

Header: <numeric>

Syntax: accumulate(begin, end, initial_value)

Starts with initial_value, then adds every element in [begin, end).

Output: 5+2+8+2+9+4 → s = 30

Why 0LL and not 0?

The type of the initial value decides the type of the running sum.

0 is int, so the sum is kept in an int. Overflow if it passes ~2.1×10⁹.

0LL is long long, so the sum is safe.
cpp

vector<int> big = {2000000000, 2000000000};

accumulate(big.begin(), big.end(), 0);    // overflow → garbage (negative)

accumulate(big.begin(), big.end(), 0LL);  // 4000000000  ✅

Start from 100: accumulate(v.begin(), v.end(), 100) → 130.

#### lower_bound: first position with value ≥ x
cpp

int pos = lower_bound(v.begin(), v.end(), x) - v.begin();

Vector must be sorted, otherwise the answer is meaningless.

Does binary search → O(log n).

Returns an iterator; subtracting v.begin() converts it to an index.

Example: v = {10, 20, 20, 40}, x = 20 → pos = 1.

Returns the index of the first element greater than or equal to x. The vector must be sorted in ascending order for this example.

cpp

vector<int> v = {2, 2, 4, 5, 8, 9};   // sorted

x	lower_bound finds	pos

5	first element ≥ 5 → 5	3

2	first element ≥ 2 → first 2	0

6	first element ≥ 6 → 8	4

1	first element ≥ 1 → 2	0

10	none ≥ 10 → returns end()	6 (= size)

If pos == v.size(), no element is ≥ x. Check this before using v[pos].

Related: upper_bound gives the first element > x.

Count of x: upper_bound(...) - lower_bound(...). For x = 2 → 2 - 0 = 2.

#### iota: fill with increasing values
cpp

iota(v.begin(), v.end(), 0);

Header: <numeric>

Syntax: iota(begin, end, start). Fills start, start+1, start+2, ...

cpp

vector<int> v(5);

iota(v.begin(), v.end(), 0);   // 0 1 2 3 4

iota(v.begin(), v.end(), 10);  // 10 11 12 13 14

Mostly used to create an index array (see section 6).

####  erase(remove_if(...)) — remove elements by condition

remove_if() moves the elements for which the condition is false to the front. These are the elements you want to keep.

Suppose you have:

vector<int> v = {1, 2, 3, 4, 5, 6};

You want to remove all even numbers.

v.erase(
    
	remove_if(v.begin(), v.end(),

			  [](int a) { return a % 2 == 0; }),
    v.end()
);

How it works:
1. [](int a) receives each element.

2. a % 2 == 0 returns true for even numbers.

3. remove_if() moves the elements you want to keep to the front and returns the new logical end.

4. erase() deletes everything from that logical end to the actual end.

#### Sorting indices using a comparator
Suppose you want to sort numbers but keep the original vector unchanged.

vector<int> v = {40, 10, 30};

vector<int> idx = {0, 1, 2};

sort(idx.begin(), idx.end(),
     
	 [&](int i, int j) {

		 return v[i] < v[j];
     });


The key idea: sort the indices according to the values they point to, rather than sorting the values themselves.

Initially:

Index (idx)	Value (v[idx])

0	40

1	10

2	30

The comparator:

return v[i] < v[j];


tells sort() that index i should come before index j when its corresponding value is smaller.

After sorting:

Sorted index	Corresponding value

1	10

2	30

0	40

So idx becomes {1, 2, 0}, while v stays {40, 10, 30}.

- [&] allows the lambda to access v from the surrounding scope.

- i and j are indices being compared, not the values themselves.

- < sorts in ascending order. Use v[i] > v[j] for descending order.

When is this useful? When you need sorted order but must preserve the original array—for example, sorting student indices by marks while keeping each student's 
original position.


Use a custom comparator to sort pairs by the second element:

vector<pair<int,int>> v = {{1,30}, {2,10}, {3,20}};

sort(v.begin(), v.end(), [](auto a, auto b) {

	return a.second < b.second;
});


Z

#include <list>

#include <forward_list>

## 1. list (doubly linked list)

Each node stores a value plus pointers to the previous and next node. There is no [i] and no it + n, only ++ and --.

Create

cpp
list<int> l;                    // empty

list<int> l(4);                 // 0 0 0 0

list<int> l(4, 7);              // 7 7 7 7

list<int> l = {3, 1, 2};        // 3 1 2

list<int> m(l.begin(), l.end());// copy of a range

Add / remove at the ends: O(1)

Function	Example	l after

push_back(x)	l={3,1,2}, push_back(4)	3 1 2 4

push_front(x)	push_front(0)	0 3 1 2 4

pop_back()	pop_back()	0 3 1 2

pop_front()	pop_front()	3 1 2

emplace_back(x) / emplace_front(x)	construct in place	same as push

Access / size

Function	Example (l = 3 1 2)	Output
front()	l.front()	3

back()	l.back()	2

size()	l.size()	3

empty()	l.empty()	0 (false)

clear()	l.clear()	l is empty

Insert / erase at an iterator: O(1) (finding the iterator is O(n))

list<int> l = {3, 1, 2};
auto it = next(l.begin());      // points to 1

l.insert(it, 9);                // inserts BEFORE it   → 3 9 1 2

l.insert(it, 2, 7);             // two copies of 7     → 3 9 7 7 1 2

it = l.erase(it);               // erases 1, returns iterator to next (2) → 3 9 7 7 2

l.erase(l.begin(), next(l.begin(), 2)); // erase range [begin, begin+2) → 7 7 2

Member functions that replace <algorithm> ones
(std::sort needs random access, so it does not work on list.)

Function	Example	Result

sort()	{3,1,2}	1 2 3

sort(greater<int>())	{3,1,2}	3 2 1

reverse()	{1,2,3}	3 2 1

unique()	{1,1,2,2,2,3,1}	1 2 3 1 (only adjacent duplicates)

remove(x)	{1,2,3,2}, remove(2)	1 3

remove_if(pred)	{1,2,3,4}, remove_if([](int a){return a%2==0;})	1 3

merge(other)	a={1,3,5}, b={2,4}, a.merge(b)	a = 1 2 3 4 5, b is empty (both must be sorted first)

splice: move nodes without copying, O(1)

cpp
list<int> a = {1, 2}, b = {3, 4, 5};
a.splice(a.end(), b);                 // move all of b to the end of a
// a = 1 2 3 4 5, b = empty

list<int> c = {10, 20, 30};
a.splice(a.begin(), c, next(c.begin())); // move only element 20
// a = 20 1 2 3 4 5, c = 10 30

Others

cpp
l.assign(3, 5);     // 5 5 5
l.resize(5);        // 5 5 5 0 0
l.resize(2);        // 5 5
a.swap(b);          // swap contents, O(1)

## 2. forward_list (singly linked list)

Each node stores only a pointer to the next node. It uses less memory than list, but you can only move forward.

What it does NOT have: push_back, pop_back, back(), size(), insert, erase, --, rbegin.

Create

cpp
forward_list<int> f = {3, 1, 2};
forward_list<int> f(3, 7);      // 7 7 7

Ends

Function	Example (f = 3 1 2)	f after

push_front(0)		0 3 1 2

pop_front()		3 1 2

emplace_front(x)		same as push_front

front()		3

empty()		0

clear()		empty

Size: distance(f.begin(), f.end()) → 3 (O(n)).

Last element: you have to walk to it.

The key idea: _after functions

To insert or erase at a node, you need the previous node, because nodes cannot look backward. So the functions work after a given position.

before_begin() is a special iterator that sits before the first element, so you can insert or erase at the front with it.

cpp
forward_list<int> f = {1, 2, 3};

q	auto it = f.begin();                 // points to 1

f.insert_after(it, 9);               // after 1   → 1 9 2 3

f.insert_after(f.before_begin(), 0); // at front  → 0 1 9 2 3

f.insert_after(it, 2, 5);            // two 5s after 1 → 0 1 5 5 9 2 3

f.erase_after(it);                   // erase the node after 1 → 0 1 5 9 2 3

f.erase_after(f.before_begin());     // erase the first node   → 1 5 9 2 3

f.erase_after(it, next(it, 3));      // erase range (it, it+3) exclusive both → 1 3

erase_after(a, b) erases everything strictly between a and b.

Member functions (same as list)

Function	Example	Result

sort()	{3,1,2}	1 2 3

reverse()	{1,2,3}	3 2 1

unique()	{1,1,2,2,3}	1 2 3

remove(x)	{1,2,3,2}, remove(2)	1 3

remove_if(pred)	{1,2,3,4}, even	1 3

merge(other)	{1,3} + {2,4} (sorted)	1 2 3 4

# Remaining Containers: What’s Different

The shared functions (size, empty, clear, swap, begin/end) work on all of them. Only the differences are listed below.

## 1. Container adapters: stack, queue, priority_queue

They wrap another container and expose only a few functions.

No iterators, no [i], no clear(), and no loops. Pop elements to see them.
	            stack   	queue   	priority_queue
Order      	    LIFO    	FIFO	    largest first

Add	            push(x) 	push(x) 	push(x) O(log n)

Remove pop()    top	        front       top  O(log n)

Peektop()	    front(),    back()	    top() O(1)

Underlying	    deque   	deque   	vector + heap


pop() returns nothing. Read with top()/front() first.

emplace(x) also exists on all three.

Clear: st = stack<int>();

Min-heap: priority_queue<int, vector<int>, greater<int>> pq;

Pair heap (e.g. Dijkstra): priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;

cpp
stack<int> st; st.push(1); st.push(2);

st.top();      // 2

st.pop();      // removes 2, st.top() now 1

queue<int> q; q.push(1); q.push(2);

q.front();     // 1

q.back();      // 2

q.pop();       // removes 1

priority_queue<int> pq; pq.push(3); pq.push(9); pq.push(5);

pq.top();      // 9

pq.pop();      // removes 9, top is now 5


## 2. set / multiset

Sorted, balanced tree. Not indexable, elements are read-only (can’t change a value in place; erase and insert instead).

Function	Example (s = {1,3,5,7})	Output

insert(x)	s.insert(4)	            1 3 4 5 7 (ignored if present)

erase(x)	s.erase(3)	            removes 3 (returns count removed)

erase(it)	s.erase(s.begin())	    removes smallest

find(x)	s.find(5)	                iterator, or end() if missing

count(x)	s.count(5)          	1

lower_bound(x)	s.lower_bound(4)	iterator to 5

upper_bound(x)	s.upper_bound(5)	iterator to 7

*s.begin()		                    1 (min)

*s.rbegin()		                    7 (max)

equal_range(x)		                pair (lower, upper)

All operations are O(log n).

### multiset allows duplicates:
ms.erase(x) removes all copies.

Remove one copy: ms.erase(ms.find(x));

ms.count(x) can be greater than 1.

set<int, greater<int>> s; keeps descending order.

s.contains(x) exists in C++20.

### map / multimap

Like set, but stores key → value pairs, sorted by key.

Function	Example (m = {a:1, b:2})	Output

m[k]	m["c"]	creates c with value 0 and returns it

m[k] = v	m["a"] = 9	sets value

at(k)	m.at("z")	throws error if missing

insert({k,v})	m.insert({"d", 4})	ignored if key exists

emplace(k, v)		same, constructs in place

try_emplace(k, v)		inserts only if missing, doesn’t overwrite

find(k)		iterator or end()

count(k)		0 or 1

erase(k) / erase(it)		removes entry

lower_bound(k), upper_bound(k)		iterator by key

begin()->first		smallest key

rbegin()->first		largest key

cpp

map<string,int> m;

m["apple"]++; m["apple"]++; m["kiwi"] = 5;

for (auto &[k, v] : m) cout << k << " " << v << "\n";

// apple 2

// kiwi 5

m[k] inserts a missing key. To only check, use find or count.

Elements are pair<const Key, Value>: you can change second, never first.

multimap allows duplicate keys, with no [] operator.

Typical use: frequency counting, coordinate mapping, ordered data by key.

### unordered_set / unordered_map

Same function names as set/map, but built on a hash table.

set / map	unordered_set / unordered_map

Insert/find/erase	O(log n)	O(1) average, O(n) worst

Sorted order	yes	no (random order)

lower_bound / upper_bound	yes	no

rbegin, --it	yes	no

Key types	anything with <	anything with hash (pair needs a custom hash)

Extra hash-specific functions:

cpp
um.reserve(n);            // pre-allocate buckets, avoids rehashing

um.max_load_factor(0.25); // fewer collisions, more memory

um.bucket_count();

Can be hacked into worst-case O(n) on Codeforces. Use a custom hash (the one from the earlier STL notes) or map if safety matters.
unordered_multiset / unordered_multimap allow duplicates.


### pair / tuple

pair	tuple

Elements	exactly 2	any number

Access	p.first, p.second	get<0>(t)

Create	{1, 2}, make_pair(1, 2)	make_tuple(1, 2, 3)

Unpack	auto [a, b] = p;	auto [a, b, c] = t;, tie(a, b, c) = t;

Compared lexicographically: first element, then second. So sort on vector<pair<int,int>> works with no comparator.

swap(p1, p2) works.


### string (extra over vector)

Function	Example (s = "hello world")	Output

substr(pos, len)	s.substr(6, 5)	world

find(t)	s.find("lo")	3 (or string::npos)

rfind(t)	s.rfind("o")	7

find_first_of(chars)	s.find_first_of("ol")	2

insert(pos, t)	s.insert(5, ",")	hello, world

erase(pos, len)	s.erase(5, 6)	hello

replace(pos, len, t)	s.replace(0, 5, "HI")	HI world

compare(t)		0 if equal

stoi, stoll, stod	stoi("42")	42

to_string(x)	to_string(42)	"42"

getline(cin, s)		reads whole line

Compare with ==, <, +, += directly.

s.find(t) != string::npos is the existence check.

s += c is O(1); s = s + c is O(n).


###  bitset

Fixed-size array of bits, size set at compile time.


Function	Example (b = 10110)	Output

b[i]	b[1]	1

set() / set(i)	b.set(0)	10111

reset() / reset(i)	b.reset(1)	10100
flip() / flip(i)	b.flip()	01001

test(i)	b.test(2)	1

count()		3

any() / none() / all()		1 / 0 / 0

to_string()		"10110"

to_ulong() / to_ullong()		22

& | ^ ~ << >>	b << 1	01100

Index 0 is the rightmost bit when printed.

Operations run on 64 bits at a time, so about 64× faster than a bool array. Used for subset-sum DP and reachability.


Which container for which job

Need	Use

LIFO (undo, brackets, DFS)	stack

FIFO (BFS, simulation)	queue

Always the max/min (Dijkstra, scheduling, top-k)	priority_queue

Sorted unique + predecessor/successor	set

Sorted with duplicates, remove one copy	multiset

Key → value, sorted by key	map

Fast existence / counting, order irrelevant	unordered_set / unordered_map

Pack data (value + index)	pair / tuple

Huge boolean tables, subset-sum DP	bitset
