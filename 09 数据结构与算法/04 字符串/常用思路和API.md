# substring() 方法

substring() 方法返回字符串的子字符串。

### 语法

```java
public String substring(int beginIndex)

或

public String substring(int beginIndex, int endIndex)
```

### 参数

- **beginIndex** -- 起始索引（包括）, 索引从 0 开始。
- **endIndex** -- 结束索引（不包括）。

# toCharArray() 方法

toCharArray() 方法将字符串转换为字符数组。

### 语法

```java
public char[] toCharArray()
```

### 参数

- 无

### 返回值

字符数组。

# toLowerCase() 方法

toLowerCase() 方法用于将大写字符转换为小写。

### 语法

```java
char toLowerCase(char ch)
```

### 参数

- **ch** -- 要转换的字符。

### 返回值

返回转换后字符的小写形式，如果有的话；否则返回字符本身。

# toString() 方法

toString() 方法用于返回以一个字符串表示的 Number 对象值。

如果方法使用了原生的数据类型作为参数，返回原生数据类型的 String 对象值。

如果方法有两个参数， 返回用第二个参数指定基数表示的第一个参数的字符串表示形式。

### 语法

以 String 类为例，该方法有以下几种语法格式：

```java
String toString()
static String toString(int i)
```

### 参数

- **i** -- 要转换的整数。

### 返回值

- **toString():** 返回表示 Integer 值的 String 对象。
- **toString(int i):** 返回表示指定 int 的 String 对象。

