

bieset类使创建和操作位集合更容易，相当于数组。



| 方法       | 含义                                                         |
| ---------- | ------------------------------------------------------------ |
| operator[] | 返回容器第n位中的引用                                        |
| count      | 返回容器中被设置位on的位数                                   |
| size       | 返回容器中元素的数量                                         |
| test       | 进行一次越界检查。如果没有越界，则返回对应位的on/off状态。如果越界，则抛出out_of_range异常。 |
| any        | 若任意一位位on，则返回true。                                 |
| none       | 若没有一位是on，则返回true。                                 |
| all        | 若所有位都是on，则返回true。                                 |
| set        | 将对应的位设置位true。                                       |
| reset      | 将对应的位设置位off。                                        |
| flip       | 将对应位置的状态反转。                                       |
| to_string  | 转换成一个字符串                                             |
| to_ulong   | 转换成一个无符号长整数                                       |
| to_ullong  | 转换成一个无符号长长整数                                     |
| at         | 进行一次越界检查。如果没有越界，则返回对应位的on/off状态。如果越界，则抛出一个out_of_range异常。 |

#### 1. bitset::operator[]

```cpp
     bool operator[] (size_t pos) const;
reference operator[] (size_t pos);
```

**Example**

```cpp
// bitset::operator[]
#include <iostream>       // std::cout
#include <bitset>         // std::bitset

int main ()
{
  std::bitset<4> foo;
  foo[1]=1;             // 0010
  foo[2]=foo[1];        // 0110
  std::cout << "foo: " << foo << '\n';
  return 0;
}
```

#### 2. bitset::count

**Example**

```c++
// bitset::count
#include <iostream>       // std::cout
#include <string>         // std::string
#include <bitset>         // std::bitset

int main ()
{
  std::bitset<8> foo (std::string("10110011"));

  std::cout << foo << " has ";
  std::cout << foo.count() << " ones and ";
  std::cout << (foo.size()-foo.count()) << " zeros.\n";

  return 0;
}
```



#### 3. bitset::size

**Example**

```c++
// bitset::count
#include <iostream>       // std::cout
#include <string>         // std::string
#include <bitset>         // std::bitset

int main ()
{
  std::bitset<8> foo (std::string("10110011"));

  std::cout << foo << " has ";
  std::cout << foo.count() << " ones and ";
  std::cout << (foo.size()-foo.count()) << " zeros.\n";

  return 0;
}
```

Output

```
foo.size() is 8
bar.size() is 4
```

#### 4. bitset::test

**Example**

```cpp
// bitset::test
#include <iostream>       // std::cout
#include <string>         // std::string
#include <cstddef>        // std::size_t
#include <bitset>         // std::bitset

int main ()
{
  std::bitset<5> foo (std::string("01011"));

  std::cout << "foo contains:\n";
  std::cout << std::boolalpha;
  for (std::size_t i=0; i<foo.size(); ++i)
    std::cout << foo.test(i) << '\n';

  return 0;
}
```

Output

```
foo contains:
true
true
false
true
false
```

#### 5. bitset::any

**Example**

```c++
// bitset::any
#include <iostream>       // std::cin, std::cout
#include <bitset>         // std::bitset

int main ()
{
  std::bitset<16> foo;

  std::cout << "Please, enter a binary number: ";
  std::cin >> foo;

  if (foo.any())
    std::cout << foo << " has " << foo.count() << " bits set.\n";
  else
    std::cout << foo << " has no bits set.\n";

  return 0;
}
```

Output

```
Please, enter a binary number: 10110
0000000000010110 has 3 bits set.
```

#### 6. bitset::none

**Example**

```c++
// bitset::none
#include <iostream>       // std::cin, std::cout
#include <bitset>         // std::bitset

int main ()
{
  std::bitset<16> foo;

  std::cout << "Please, enter a binary number: ";
  std::cin >> foo;

  if (foo.none())
    std::cout << foo << " has no bits set.\n";
  else
    std::cout << foo << " has " << foo.count() << " bits set.\n";

  return 0;
}
```

Output

```
Please, enter a binary number: 11010111
0000000011010111 has 6 bits set.
```

#### 7. bitset::all

**Example**

```c++
// bitset::all
#include <iostream>       // std::cin, std::cout, std::boolalpha
#include <bitset>         // std::bitset

int main ()
{
  std::bitset<8> foo;

  std::cout << "Please, enter an 8-bit binary number: ";
  std::cin >> foo;

  std::cout << std::boolalpha;
  std::cout << "all: " << foo.all() << '\n';
  std::cout << "any: " << foo.any() << '\n';
  std::cout << "none: " << foo.none() << '\n';

  return 0;
}
```

Output

```
Please, enter an 8-bit binary number: 11111111
all: true
any: true
none: false
```

#### 8. bitset::set

**Example**

```c++
// bitset::set
#include <iostream>       // std::cout
#include <bitset>         // std::bitset

int main ()
{
  std::bitset<4> foo;

  std::cout << foo.set() << '\n';       // 1111
  std::cout << foo.set(2,0) << '\n';    // 1011
  std::cout << foo.set(2) << '\n';      // 1111

  return 0;
}
```

Output

```
1111
1011
1111
```

#### 9. bitset::reset

**Example**

```
// bitset::reset
#include <iostream>       // std::cout
#include <string>         // std::string
#include <bitset>         // std::bitset

int main ()
{
  std::bitset<4> foo (std::string("1011"));

  std::cout << foo.reset(1) << '\n';    // 1001
  std::cout << foo.reset() << '\n';     // 0000

  return 0;
}
```

Output

```
1001
0000
```

#### 10. bitset::flip

**Example**

```
// bitset::flip
#include <iostream>       // std::cout
#include <string>         // std::string
#include <bitset>         // std::bitset

int main ()
{
  std::bitset<4> foo (std::string("0001"));

  std::cout << foo.flip(2) << '\n';     // 0101
  std::cout << foo.flip() << '\n';      // 1010

  return 0;
}
```

Output

```
0101
1010
```

#### 11. bitset::to_string

**Example**

```
// bitset::to_string
#include <iostream>       // std::cout
#include <string>         // std::string
#include <bitset>         // std::bitset

int main ()
{
  std::bitset<4> mybits;     // mybits: 0000
  mybits.set();              // mybits: 1111

  std::string mystring =
    mybits.to_string<char,std::string::traits_type,std::string::allocator_type>();

  std::cout << "mystring: " << mystring << '\n';

  return 0;
}
```

Output

```
mystring: 1111
```

#### 12. bitset::to_ulong    bitset::to_uulong

**Example**

```
// bitset::to_ulong
#include <iostream>       // std::cout
#include <bitset>         // std::bitset

int main ()
{
  std::bitset<4> foo;     // foo: 0000
  foo.set();              // foo: 1111

  std::cout << foo << " as an integer is: " << foo.to_ulong() << '\n';
  std::cout << foo << " as an integer is: " << foo.to_ullong() << '\n';
  return 0;
}
```

Output

```
1111 as an integer is: 15
1111 as an integer is: 15
```

