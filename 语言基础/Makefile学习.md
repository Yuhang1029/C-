# Makefile 学习

第一种常见的格式

```makefile
hello: main.cpp printHello.cpp factorial.cpp
    g++ -o hello main.cpp printHello.cpp factorial.cpp
```

第一行代表 hello 这个可执行文件依赖三个 `.cpp` 文件。它会通过文件生成时间的比较去判断是否需要重新编译。



第二种格式

```makefile
CXX = g++
TARGET = hello
OBJ = main.o printHello.o factorial.o

$(TARGET): $(OBJ)
    $(CXX) -o $(TARGET) $(OBJ)

main.o: main.cpp
    $(CXX) -c main.cpp

printHello.o: printHello.cpp
    $(CXX) -c printHello.cpp

factorial.o: factorial.cpp
    $(CXX) -c factorial.cpp
```

这样写的好处是编译的时候只会重新去编译修改过的文件，达到节省时间的目的。



第三种格式

```makefile
CXX = g++
TARGET = hello
OBJ = main.o printHello.o factorial.o

CXXFLAGS = -c -Wall

$(TARGET): $(OBJ)
    $(CXX) -o $@ $^

%.o: %.cpp
    $(CXX) $(CXXFLAGS) $< -o $@

.PHONY: clean
clean:
    rm -f *.o $(TARGET)
```

通过一些代指符号避免了对每一个文件去定义。



第四种格式

```makefile
CXX = g++
TARGET = hello
SRC = $(wildcard *.cpp)        # 当前文件夹下的所有 .cpp 文件
OBJ = $(patsubst %.cpp, %.o, $(SRC))    # 当前路径下 .cpp 文件替换成 .o 文件名

CXXFLAGS = -c -Wall

$(TARGET): $(OBJ)
    $(CXX) -o $@ $^

%.o: %.cpp
    $(CXX) $(CXXFLAGS) $< -o $@

.PHONY: clean
clean:
    rm -f *.o $(TARGET)
```

即使增加新的 `.cpp` 文件都不需要修改 `Makefile`。


