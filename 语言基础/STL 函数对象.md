# STL 函数对象

## 函数对象

重载函数调用操作符的类，其对象通常称为函数对象。当函数对象使用重载的 `()` 时，行为类似函数调用，也叫仿函数。函数对象有以下的特点：

* 函数对象在使用的时候，可以像普通函数那样调用，可以有参数和返回值。

```cpp
// 仿函数，返回两个数的和
// 这里表明了它可以像普通函数一样调用
class MyAdd {
public:
    int operator()(int a, int b) {
        return a + b;
    }  
};

void test() {
    MyAdd myAdd;
    cout << myAdd(10, 20) << endl;
}
```

* 函数对象超出普通函数的概念，函数对象可以有自己的状态。

```cpp
// 这里的 MyPrint 不仅可以打印，同时可以利用成员变量来统计调用的次数
// 体现了它不同于函数的地方，即有自己的状态
class MyPrint {
public:
    MyPrint(){
        this->count = 0;
    }
    void operator()(string s) {
        cout << s << endl;
        this->count++;
    }
private:
    int count;  
};

void test() {
    MyPrint myPrint;
    myPrint("hello world");
}
```

* 函数对象可以作为参数传递。

```cpp
void doPrint(MyPrint& m, string s) {
    m(s);
}

void test() {
    MyPrint myPrint;
    doPrint(myPrint, "hello world");
}
```

&emsp;

## 谓词 (Predicate)

返回 bool 类型的仿函数称为谓词。如果 `()` 接收一个参数的话称为一元谓词，接收两个参数的话称为二元谓词。下面是一个一元谓词的案例：

```cpp
// 查找大于 5 的值
class GreaterFive {
public:
    // 一元谓词
    bool operator()(int val) {
        return val > 5;
    }
};

void test() {
    vector<int> v;
    for (int i = 0; i < 10; i++) {
        v.push_back(i);
    }
    vector<int>::iterator it = find_if(v.begin(), v.end(), GreaterFive());
    if (it == v.end()) {
        cout << "Not found" << endl;
    } else {
        cout << *it << endl;
    }
}
```

下面是一个二元谓词的案例：

```cpp
class MyCompare {
public:
    // 降序排列
    bool operator()(int v1, int v2){
        return v1 > v2;
    }
};

set<int, MyCompare> s;
```

&emsp;

## 内建函数对象

内建函数对象就是 STL 官方提供的一些函数对象，使用的时候需要引入 `#include<functional>` 。  

### 算术仿函数

功能是实现四则运算，除了最后一个取反运算其他都是二元运算。

```cpp
template<typename T> T plus<T>            // 加法
template<typename T> T minus<T>           // 减法
template<typename T> T multiplies<T>      // 乘法
template<typename T> T divides<T>         // 除法
template<typename T> T modules<T>         // 取模
template<typename T> T negate<T>          // 取反
```

&emsp;

### 关系仿函数

实现关系对比

```cpp
template<typename T> bool equal_to<T>            // 等于
template<typename T> bool not_equal_to<T>        // 不等于
template<typename T> bool greater<T>             // 大于
template<typename T> bool greater_equal<T>       // 大于等于
template<typename T> bool less<T>                // 小于
template<typename T> bool less_equal<T>          // 小于等于
```

上述 greater 用的比较多，例如希望一个数组从大到小排序时可以：

```cpp
vector<int> v;
sort(v.begin(), v.end(), greater<int>());
```

&emsp;

### 逻辑仿函数

实现逻辑判断

```cpp
template<typename T> bool logical_and<T>       // 与
template<typename T> bool logical_or<T>        // 或
template<typename T> bool logical_not<T>       // 非
```

&emsp;

## STL 中的常用算法

主要包含在头文件 `<algorithm>`，`<numeric>` 和 `<functional>` 中。

### 遍历算法

**for_each**：实现容器的遍历

```cpp
#include <algorithm>

void print(int val) {
    cout << val << endl;
}

class Printer {
public:
    void operator()(int val) {
        cout << val << endl;
    }  
};

int main() {
    vector<int> v;
    // 利用普通函数实现遍历，最后一个参数是函数名
    for_each(v.begin(), v.end(), print); 
    // 利用仿函数实现遍历，最后一个参数是匿名对象
    for_each(v.begin(), v.end(), Printer()); 
}
```

**transform**：搬运容器到另一个容器

`transform(iterator beg1, iterator end1, iterator beg2, _func)`

它总共有四个参数，前两个是源容器开始迭代器和结束迭代器，第三个是目标容器的开始迭代器，最后一个是函数或者函数对象。需要注意的是目标容器需要提前开辟空间。

```cpp
class Transformer {
public:
    int operator()(int val) {
        return val;
    }  
};

int main() {
    vector<int> v, target;
    target.resize(v.size());    // 这一步不能少
    transform(v.begin(), v.end(), target.begin(), Transformer());
}
```

&emsp;

### 查找算法

**find**：查找元素。

`find(iterator beg, iterator end, value)` 按值查找元素，如果找到返回指定元素的迭代器，如果找不到返回 `end()`。对于自定义的数据类型比较，需要重载 `==` 符号。

```cpp
class Person {
public:
    string name;
    int age;

    Person(string _name, int _age):name(_name), age(_age){}

    bool operator== (const Person& other) {
        if (this->name == other.name && this->age == other.age) {
            return true;
        }
        return false;
    }
};

void test() {
    vector<Person> people;
    people.push_back(Person("Hack", 10));
    people.push_back(Person("Loe", 19));

    auto it = find(people.begin(), people.end(), Person("Loe", 19));
    if (it != people.end()) {
        cout << it->name << endl;
    }
}
```

**find_if**：按照条件查找元素。

`find_if(iterator beg, iterator end, _Pred)` 第三个参数是一个函数或者谓词。按值查找元素，如果找到返回指定元素的迭代器，如果找不到返回 `end()`。

```cpp
// 查找年龄大于 20 的人
class GreaterThan20 {
public:
    bool operator()(Person& p) {
        return p.age > 20;
    }
};

void test() {
    vector<Person> people;
    people.push_back(Person("Hack", 10));
    people.push_back(Person("Loe", 19));

    auto it = find_if(people.begin(), people.end(), GreaterThan20());
    if (it != people.end()) {
        cout << it->name << endl;
    }
}
```

**adjacent_find**：查找相邻重复元素。

`adjacent_find(iterator beg, iterator end)` 如果找到返回相邻元素的第一个的迭代器。

**binary_search**：通过二分法查找指定元素。

`bool binary_search(iterator beg, iterator end)` 必须在有序的序列中使用，返回布尔值代表是否找到。

**count**：统计元素出现的个数。

`count(iterator beg, iterator end, value)` `value` 代表查找的元素。对于自定义的数据类型比较，需要重载 `==` 符号。

**count_if**：按照条件查找元素出现的个数。

`count_if(iterator beg, iterator end, _Pred)` 第三个参数是一个函数或者谓词返回满足条件的元素的个数。

&emsp;

### 排序算法

**sort**：对元素排序。

`sort(iterator beg, iterator end, _Pred)` 第三个参数如果不填默认从小到大排序。

**random_shuffle**：洗牌，指定范围内元素随机调换顺序。

`random_shuffle(iterator beg, iterator end)` 

**merge**：两个容器中的元素合并，存储到另一个元素中。

`merge(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest)` 注意两个容器中的元素必须要是有序的，同时目标容器要提前分配空间。

```cpp
int main() {
    vector<int> v1, v2, target;
    v1.push_back(0);
    v1.push_back(1);
    v1.push_back(2);

    v2.push_back(10);
    v2.push_back(11);
    v2.push_back(12);

    target.resize(v1.size() + v2.size());    // 这一步不能少
    merge(v1.begin(), v1.end(), v2.begin(), v2.end(), target.begin());
}
```

**reverse**：将容器内的元素反转。

`reverse(iterator beg, iterator end)`

&emsp;

### 拷贝和替换算法

**copy**：把容器内指定范围的元素拷贝到另一个容器中。

`copy(iterator beg, iterator end, iterator target)` 第三个参数是目标容器，注意需要提前开辟存储空间。

**replace**：将容器指定范围内的旧元素换成新元素。

`replace(iterator beg, iterator end, old_value, new_value)`

**replace_if**：将容器指定范围内的满足条件的元素都换成新元素。

`replace_if(iterator beg, iterator end, _Pred, new_value)`

```cpp
class GreaterFive {
public:
    bool operator()(int val) {
        return val > 5;
    }
};

int main() {
    vector<int> v;
    v.push_back(10);
    v.push_back(10);
    v.push_back(1);
    v.push_back(12);

    replace_if(v.begin(), v.end(), GreaterFive(), 19);
}
```

**swap**：互换两个容器中的所有元素

`swap(container c1, container c2)` 注意两个容器是同种类型的。

&emsp;

### 算术生成算法

**accumulate**：计算区间内容器元素的和。

`accumulate(iterator beg, iterator end, value)` `value` 是起始累加值，如果不写默认是 0。

**fill**：向容器内填充指定元素。

`fill(iterator beg, iterator end, value)`

&emsp;

### 集合算法

注意所有的源容器必须是有序序列。

**set_intersection**：求两个集合的交集。

**set_union**：求两个集合的并集。

**set_difference**：求两个集合的差集。

`set_xxx(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest)` 最后一个参数目标容器需要提前开辟空间，确保空间足够。
