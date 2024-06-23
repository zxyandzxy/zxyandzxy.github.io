---
layout: post
title: "编写可读代码的艺术"
date: 2024-6-23
tags: [books]
comments: true
author: zxy
---

> 编写可读代码的艺术——阅读笔记

## 第一章：代码应当易于理解

什么是好代码，什么是烂代码？

代码写出来用于运行，但是一个项目的代码不是写好之后就不动了；后续项目升级或者变更都需要进行代码改动。所以代码总是会变，总是需要被阅读的。

现代项目中，一份代码由多个成员进行维护，那么代码的可读性就非常重要了，这关系到团队的工作效率，团队的合作氛围等。

所以好代码：**易于理解的代码就是好代码**

### 可读性基本定理

> 关键思想：代码的写法应该使别人理解它所需的时间最小化

别人能理解自己的代码有一个**判断标准**：别人能改动代码，找出缺陷并且明白你的代码各个部分是如何交互的

这里的别人可以是：同事，老板，几个月后的自己

### 总是越小越好吗

一般来说，解决问题的代码越少越好，因为越多2000行代码所需时间铁定是少于5000行代码的

但是我们将维度放小，只关心一两行代码，这就不一定了，见下栗：

```java
assert((!(bucket = FindBucket(key))) || !bucket->IsOccupied());

bucket = FindBucket(key);
if (bucket != NULL) assert(!bucket->IsOccupied());
上面的例子中，第二种写法由于拆分了逻辑，理解起来更加容易。
这种增加代码可读性的方法就是：简化人的理解内容，将大问题拆分成多个小问题

// Fast version of "hash = (65599 * hash) + c"
hash = (hash << 6) + (hash << 16) - hash + c;
上面的例子中，由于直接给出注释，理解起来更容易
这种增加代码可读性的方法是：直接给出代码的功能，不要人自己去理解了
```

所以增加一定的代码行数，是有助于part代码的理解的

### 理解代码所需时间是否与其他目标冲突

一个项目的代码不可能只关注其可读性的，还有其他目标如：**更有效率 / 架构更加优质 / 容易测试**等等

但是这些目标并不与代码的可读性冲突：

- 一份可读性高的代码是可以存在于代码高度优化的领域的
- 代码可读性好也会带动代码架构变好
- 代码可读性好也会使得代码的测试函数更好书写

### 最难的部分

说了这么多就是一直在强调代码的可读性，那么写出一份可读性很高的代码是非常困难的。

我们平时写代码时如何提高自己代码的可读性呢？

我们要人格分裂，**一个人格做coder，一个人格做reader**。自己想一想自己写完的**一段代码**是不是好理解，如果没有其他背景知识能不能读懂。

这样每从项目的构建开始我们就会一直优化代码的可读性，最终代码将会有更少的缺陷，更高的质量，更高的他人喜爱度。

---

> 第一部分：表面层次的改进

什么叫表面层次：选择好的名字，写一份好的注释，将代码按照更加规范的格式书写。

表面层次是不涉及到代码重构的，通过好的编程习惯和短暂的考虑就能够将代码的可读性变得非常高。

## 第二章：把信息装到名字里

> 关键思想：把信息装进名字里，让名字成为一个小注释

### 选择专业的词

```python
栗子1：
def GetPage(url):
咋一看，这名字还挺好；此函数表示通过url获取页面
但是深入想想呢？是从网络上获取，还是从数据库拿取，还是从缓存中拿取
从网络上：FetchPage()  
从数据库上：GetPageFromDB()
从缓存上：GetPageFromCache()

栗子2：
class BinaryTree {
    int Size();
}
对于一个二叉树而言，希望从Size()中拿到什么呢？树高 / 树的节点 / 树在内存中所占空间
所以问题核心就是Size本身没有占用很多信息
Height()  NumNodes()  MemoryBytes()

栗子3：
class Thread {
    void Stop();
}
如果Stop是为了关闭线程：Kill()
如果只是为了暂时让出CPU资源：Pause()
```

我们区分取名专业与不专业是很直观的，只要一针见血的看出函数或者变量的真实作用，那么取名就非常专业了。

要想取一个专业的名字：

- 平时多积累，多利用见过的专业词语
- 自己写一个函数名/变量名时，多考虑一下有没有歧义，可不可以优化

| 单词  | 更多选择                                                |
| ----- | ------------------------------------------------------- |
| send  | deliver、 dispatch、 announce、 distribute、 route      |
| find  | search、 extract、 locate、 recover                     |
| start | launch、 create、 begin、 open                          |
| make  | create、set up、 build、 generate、 compose、 add、 new |

**但是也要避免过犹不及：清晰和准确比为了取专业名而装可爱要好**

### 避免使用泛泛的名字

```javascript
tmp retval这两个词往往是“想不出好名字”的托词

var euclidean_norm = function (v) {
	var retval = 0.0;
    for (var i = 0; i < v.length; i++) {
        retval += v[i] * v[i];
    }
    return Math.sqrt(retval);
}

这里的retval明明可以有更好的名字，直接让他表示这个值的含义（总和的平方根）
sum_squares就非常好，这样一旦代码写成了： sum_squares += v[i]就一眼看出不合情理
```

#### tmp

> tmp 这个名字只应用于短期存在并且临时性为其主要存在因素的变量

```java
if (left < right) {
	tmp = right;
	right = left;
	left = tmp;
}
这里用tmp就非常合适

tmp_file = tempfile.NamedTemporaryFile()
...
SaveData(tmp_file, ...)
这里让tmp成为某个前缀也非常合适
```

#### 循环迭代器

i，j，iter，it这些名字一般都是用来做索引和循环迭代器的，但是在多重循环嵌套中，如果我们只是用i，j的话，很容易出现索引写错的问题，这时候就可以通过增加前缀解决

```cpp
for (int club_i = 0; club_i < clubs.size(); club_i++) {
	for (int member_j = 0; member_j < clubs[i].members.size(); member_j++) {
		for (int user_k = 0; user_k < users.size(); user_k++) {
			if (clubs[club_i].members[member_j] == users[user_k]) {
				cout << "user[" << user_k << "] is in club[" << club_i << "]" << endl;
			}
		}
	}
}
通过添加club,member,user这样的前缀，我们通过索引访问数组时就不容易写错了
通过首字母缩写来实现也可以：ci, mj, uk
```

不取空泛的名字有一个核心要素：**不要当懒人，多考虑一下会死啊**

### 用具体的名字代替抽象的名字

> 给函数或者变量取名时，描述的具体一点，他用来干什么的就直接翻译

如检测服务是否可以监听某个给定的TCP/IP端口，就可以命名为：CanListenOnPort()

### 使用前缀或者后缀来给名字附带更多信息

> 技巧1：带上单位

如果变量是一个度量的话（时间长度或者字节数），那么名字上带上单位

```javascript
var start_ms = (new Date()).getTime(); // top of the page
...
var elapsed_ms = (new Date()).getTime() - start_ms; // bottom of the page
document_writeLn("Load time was: " + elapsed_ms / 1000 + " seconds");
```



| 函数参数                      | 带单位的参数         |
| ----------------------------- | -------------------- |
| Start(int delay)              | delay ——> delay_secs |
| CreateCache(int size)         | size ——> size_mb     |
| ThrottleDownload(float limit) | limit ——> max_kbs    |
| Rotate(float angle)           | angle ——> degrees_cw |

> 技巧2：附带其他重要属性

| 情形                               | 变量名   | 更好名字           |
| ---------------------------------- | -------- | ------------------ |
| 纯文本格式密码，需要加密后继续使用 | password | plaintext_password |
| 用户提供的注释，需要转义后显示     | comment  | unescaped_comment  |
| 已经转换成utf-8的HTML字节          | html     | html_utf8          |
| 以url方式编码的输入数据            | data     | data_urlenc        |

### 决定名字的长度

名字太长人不想读，名字太短信息太少。我们需要**根据实际场景决定名字长度**

> 小的作用域使用短的名字

```cpp
if (debug) {
	map<string, int> m;
	LookUpNamesNumbers(&m);
	Print(m);
}
```

> 首字母缩略词和缩写

```
BEManager 而不是 BackEndManager 能不能使用缩略词有一个原则：
	团队新成员是否能理解这个名字的含义

一般来说：eval = evaluation   doc = document   str = string
```

> 丢弃没用的词

```
ConvertToString 就不如 ToString 反正意思都能理解
```

### 利用名字的格式来表达含义

> Google开源项目格式规范C++：

```cpp
static const int kMaxOpenFiles = 100;
class LogReader {
    public:
    	void OpenFile(string local_file);
    private:
    	int offset_;
    	DISALLOW_COPY_AND_ASSIGN(LogReader);
};
```

- CamelCase来表示类名
- lower_separated来表示变量名
- 常量的定义使用static const type kConstantName
- 宏使用MACRO_NAME
- 类成员变量和普通变量一样，但是末尾要加下划线

> JavaScript HTML CSS

```javascript
var x = new DatePicker();
var y = pageHeight();
<div id="middle_column" class="main-content">
```

- JavaScript构造函数首字母大写，普通函数首字母小写 
- HTML标记添加id或者class属性时，用下划线分割id中的单词，用连字符区分class中单词

## 第三章：不会误解的名字

> 见名知意：关键点就是多问自己几遍，这个名字别人看了会不会有歧义

### 歧义的例子

```python
Fileter()
	return = Database.all_objects.filter("year <= 2011")
filter作为一个二义性单词，没办法清楚到底是"挑出" 还是 "删去"

Clip(text, length)
	# cuts off the end of the text, and appends "..."
	def Clip(text, length):
Clip可以有两种含义：
	从尾部删除length的长度
	截掉最大长度为length的一段
```

### 推荐用min和max来表示极限

```python
购物车商品最多10件：
	CART_TOO_BIG_LIMIT = 10
	if (shopping_cart.num_items() >= CART_TOO_BIG_LIMIT):
		Error("Too many items in cart.")
CART_TOO_BIG_LIMIT这个名字的二义性很强，到底是"少于"还是"少于且包括"

换成 MAX_ITEMS_IN_CART就清晰很多
```

### 推荐用firt和last来表示包含的范围

```python
print integer_range(start=2, last=4)
这就是打印区间[2, 4]
```

### 推荐用begin和end来表示包含/排除范围

```
print integer_range(begin=2, end=4)
这就是打印区间[2, 4)
```

### 布尔值命名方式

```python
通常来说，加上is，has，can，should这样的词，布尔值就会变得明确
	SpaceLeft()  ->  HasSpaceLeft()   这就更加清晰一点
	
不要用反义名字表示布尔值
	bool disable_ssl = false; 这样子别人读的时候还要绕一个弯，就很烦
	bool use_ssl = false;      
```

### 命名要符合程序员间的潜规则

```java
get*()
	程序员之间习惯认为以get开始的方法当成"轻量级访问器"，也就是代价很小.
	如果下面的函数用get来起名就有问题：
		public class StatisticsCollector {
			public void addSample(double x) {...}
			public double getMean() {
				// Iterate through all samples and return total / num_sample
			}
		}
	这个函数的时间复杂度是O(n)，如果num_sample很大的话，非常容易造成高额时间消耗。
	这与名字get所代表的轻量级是冲突的，更名为computeMean更优质
	
list::size()
	这个方法在老版本的C++中是要通过O(n)实现的，但是调用者如果不知道，疯狂使用size()方法，那么时间复杂度就会很吓人了。
	当然，新版本C++改成O(1)
```

## 第四章：审美

好的源代码应该"养眼"，具体来说三个原则：

- 使用一致的布局，使读者熟悉这种习惯
- 让相似的代码看上去相似
- 把相关的代码行分组，形成代码块

### 一个例子看审美

```cpp
// A class for keeping track of a series of doubles
// and methods for quick statistics about them
class StatsKeeper {
	public:
		void Add(double d);
		double Average();
	private:
		list<double> past_items;
		int count; // how many so far
		
		double minimum;
		double maximum;
};
```

就好的代码，你一看就啥都知道了，理解成本巨低无比。

### 重新安排换行来保持一致和紧凑

```java
public class PerformanceTester {
	// TcpConnectionSimulator(throughput, latency, jitter, packet_loss)
	//                         [Kbps]      [ms]     [ms]    [percent]
	public static final TcpConnectionSimulator wifi = 
		new TcpConnectionSimulator(500, 80, 200, 1);
		
	public static final TcpConnectionSimulator t3)fiber = 
		new TcpConnectionSimulator(45000, 10, 0, 0);
		
	public static final TcpConnectionSimulator cell = 
		new TcpConnectionSimulator(100, 400, 250, 5);
}
```

一句话，规整优雅

### 抽取方法来整理不规则部分代码

```c++
// Turn a partial_name like "Doug Adams" into "Mr. Douglas Adams".
// If not possible, 'error' is filled with an explanation.
string ExpandFullName(DatabaseConnection dc, string partial_name, string* error)
```

乱套的，报看的代码

```cpp
DatabaseConnection database_connection;
string error;
assert(ExpandFullName(database_connection, "Doug Adams", &error)
	== "Mr. Douglas Adams");
assert(error == "");
assert(ExpandFullName(database_connection, " Jake Brown", &error)
	== "Mr. Jacob Brown III");
assert(error == "");
assert(ExpandFullName(database_connection, "No Such Guy", &error) == "");
assert(error == "no match found");
assert(ExpandFullName(database_connection, "John", &error) == "");
assert(error == "more than one result");
```

抽取方法，重新规整代码

```cpp
void CheckFullName(string partial_name,
				 string expected_full_name,
				 string expected_error) {
	// database_connection is now a class member
    string error;
    string full_name = ExpandFullName(database_connection, partial_name, &error);
    assert(error == expected_error);
    assert(full_name == expected_full_name);
}

CheckFullName("Doug Adams", "Mr. Douglas Adams", "");
CheckFullName(" Jake Brown", "Mr. Jacob Brown III", "");
CheckFullName("No Such Guy", "", "no match found");
CheckFullName("John", "", "more than one result");
```

- 消除了原来代码中的大量重复，让代码更加紧凑
- 每个测试用例最重要的部分现在都集中放在一起，没有多余信息
- 添加新的测试更加简单，因为代码结构更好了

### 使用列对齐

> 如果列对齐真的很费功夫，那么也可以不用

```python
# Extract POST parameters to local variables
details  =  request.POST.get('details')
location =  request.POST.get('location')
phone    =  request.POST.get('phone')
email    =  request.POST.get('email')
url      =  request.POST.get('url')
```

### 选择有意义的顺序，并一直使用这个顺序

```python
details  =  request.POST.get('details')
location =  request.POST.get('location')
phone    =  request.POST.get('phone')
email    =  request.POST.get('email')
url      =  request.POST.get('url')
这个顺序理论上是怎么排都可以，但是最好排布的有意义一点
```

- 让变量的顺序对应HTML表单中的<input>字段顺序
- 从最重要到最不重要
- 按照字母顺序排序

### 把声明按照块组织起来

```cpp
class FrontendServer {
	public:	
		FrontendServer();
		~FrontendServer();
	
		// Handlers
		void ViewProfile(HttpRequest* request);
		void SaveProfile(HttpRequest* request);
		void FindProfile(HttpRequest* request);
		
		// Request / Reply Utilities
		string ExtractQueryParam(HttpRequest* request, string param);
		void ReplyOK(HttpRequest* request, string html);
		void ReplyNotFound(HttpRequest* request, string error);

		// Database Helpers
		void OpenDatabase(string location, string user);
		void CloseDatabase(string location);
};
```

### 把代码分成"段落"

- 把相似的想法放在一起并与其他想法分开
- 提供可见的"脚印"
- 便于段落间的导航

```python
def suggest_new_friends(user, email_password):
    # Get the user's friends' email address.
    friends = user.friends()
    friend_emails = set(f.email for f in friends)
    
    # Import all email addresses from this user's email account.
    contacts = import_contacts(user.email, email_password)
    contact_emails = set(c.email for c in contacts)
    
    # Find matching users that they aren't already friends with.
    non_friend_emails = contact_emails - friend_emails
    suggested_friends = User.objects.select(email_in = non_friend_emails)
    
    # Display these lists on the page
    display['user'] = user
    display['friends'] = friends
    display['suggested_friends'] = suggested_friends
    
    return render("suggested_friends.html", display)
```

## 第五章：该写什么样的注释

> 注释的目的是尽量帮助读写了解的和作者一样多

### 什么不需要注释

- 阅读注释也是需要时间的，每条注释也会占用屏幕空间，所以注释应该只写有用的
- 不要为那些从代码本身就能快速推断的事实写注释
- 写好函数名，变量名的优先级是高于写注释的

### 记录你的思想

注释的核心其实就是记录程序员写下这段代码的思想过程

> 技巧1：加上"导演评论"

注释不需要只是写代码的实现原理，代码的运行步骤，

也可以书写一些代码质量相关的评论，代码改进相关的建议

> 技巧2：为代码中的瑕疵写注释

| 标记   | 通常的建议                             |
| ------ | -------------------------------------- |
| TODO:  | 我还没有处理完的事情                   |
| FIXME: | 已知无法运行的代码                     |
| HACK:  | 对一个问题不得不采用比较粗糙的解决方案 |
| XXX:   | 危险，这里有重要问题                   |

> 技巧3：给常量加注释

通过描述常量的值为什么是这个值的原因，来更好的解释常量

```python
NUM_THREADS = 8 # as long as it's >= 2 * num_processors, that's good enough
```

### 站在读者的角度

```cpp
struct Recorder {
	vector<float> data;
	...
	void Clear() {
		// Force vector to relinquish its memory (look up "STL swap trick")
		vector<float>().swap(data);
	}
};
C++程序员看到Clear函数都会想，为毛不直接用data.clear()，反而还要用一个空向量交换
原因是只有这样才能强制使向量真正将内存归还给内存分配器
```

写代码的时候多想想，有没有歧义，会不会被误用。

### "全局观"注释

全局观指的是——类与类之间如何交互，数据如何在整个系统中流动，入口在哪

通过写高阶别的注释信息，很好帮助读者了解系统构建思路

### 程序员如何克服自身写注释的心里障碍

**边写代码边写注释**

- 不管心里想什么，先写下来
- 读一下注释，看一下准不准确，专不专业
- 不断改进注释

## 第六章：写出言简意赅的注释

### 让注释保持紧凑

```cpp
// the int is the CategoryType
// the first float in the inner pair is the 'score'
// the second is the 'weoght'
typedef hash_map<int, pair<float, float> ScoreMap;

完全可以改成
// CategoryType -> (score, weight)
typedef hash_map<int, pair<float, float> ScoreMap;
```

### 避免使用不明确的代词

```cpp
// Insert the data into the cache, but check if it's too big first
it 到达指的是data还是cache呢？

// Insert the data into the cache, but check if the data is too big first
```

### 润色粗糙的句子

```python
# Depending on whether we've already crawled this URL before, give it a different priority

# Give higher priority to URLs we've never crawled before
```

### 精确描述函数行为

```cpp
// Count how many newline bytes ('\n') are in the file
int CountLines(string filename) {...}
```

### 直接用栗子当注释

```cpp
// 正常解释函数行为
// Example: Partition([8 5 9 8 2], 8) might result in [5 2 | 8 9 8] and return 1
int Partition(vector<int>* v, int pivot);
```

对于栗子的设计：

- pivot与向量中的元素相等，解释边界情况
- 在向量中放入重复元素，表名这是可接受输入
- 返回向量没有排好序，如果排好序会造成一定误会
- 返回值是1，不是向量元素，用于区分索引与元素值

### "具名函数参数"的注释

> Python

```python
def Connect(timeout, use_encryption):

# Call the function using named parameters
Connect(timeout = 10, use_encryption = False)
```

> C++，java

```cpp
void Connect(int timeout,bool use_encryption){ }

# Call the function using named parameters
Connect(/*timeout_ms = */ 10, /* use_encryption = */ False)
```

---

> 第二部分：简化循环和逻辑

本部分的核心是让代码中的"思想包袱"最小化，所谓"思想包袱"就是理解代码的逻辑和控制流所需耗费的心神。

当"思想包袱"比较大时，人就没办法持续清晰的理解他书写的程序，从而产生bug

## 第七章：把控制流变得易读

> 把条件、循环以及其他对控制流的改变做得越"自然"越好。最佳效果是使读者不用停下来阅读你的代码

### 条件语句中参数的顺序

| 比较的左侧                                 | 比较的右侧                               |
| ------------------------------------------ | ---------------------------------------- |
| "被问询的"表达式，它的值更加倾向于不断变化 | 用来做比较的表达式，它的值更加倾向于常量 |

```
也就是说
if (bytes_received < bytes_expected) 可读性更好
if (bytes_expected > bytes_received) 可读性就很差 
```

### if/else语句块的顺序

```cpp
这个处理的问题是类似于下面这种：
if (a == b) {
	// case one
} else {
	// case two
}

等价于 
if (a != b) {
	// case two
} else {
	// case one
}
这两种写法，那种好一点的问题 
```

有一些处理原则：

- 首先处理正逻辑而不是负逻辑，例如用 `if(debug)` 而不是 `if(!debug)`
- 先处理掉简单情况，使得if 和 else 都在屏幕的之内可见。
- 先处理"有趣"或者"可疑"的情况

```python
栗子：
if (url.HasQueryParameter("expand_all")) {
	...
}else {

}

if not file:
	# log the error
else:
	# ...
```

这种东西就看程序员自己写下代码时的感受了

### 三目运算符

`time_str += (hour >= 12) ? "pm" : "am"` 这种简单的处理逻辑就实现了易读和紧凑两个目标

`return exponent >= 0 ? mantissa * (1 << exponent) : mantissa / (1 << -exponent)` 这种就将太多处理逻辑塞到一行代码中了，非常难以理解

所以其实使不使用三目运算符的核心还是这个逻辑表达式是不是复杂

默认情况下使用 if / else，其他非常简单的情况下使用 三目运算符

###  避免do / while循环

do / while 循环反直觉的一点就是，循环代码块是否执行要看它后一个条件决定。

一般来说，逻辑条件应该在他们所保护的代码之前。

这就造成了每次模拟do / while时都需要停下来想一想逻辑条件

### 从函数中提前返回

从函数中提前返回是非常受欢迎的一个手法，这能够实现程序员阅读逻辑的简化

在编程语言早期阶段，只有C语言这一种；C语言是偏好只有1个return语句的，因为他需要只有一个出口点来保证 调用函数结尾的清理代码。 

因此，单纯由C语言组成的代码，在函数退出时没有特定机制调用清理代码。

同时大函数有很多清理代码时，提前返回很难做的没有问题（如return语句之后还有堆对象）

但是后来的编程语言提供了更加精细的调用清理代码的逻辑，这样提前return就很受欢迎了

| 语言         | 清理代码    |
| ------------ | ----------- |
| C++          | 析构函数    |
| Java、Python | try finally |
| Python       | with        |
| C#           | using       |

> 不要用goto函数，因为有多个goto目标时，代码会非常杂乱。
>
> goto是可以用if / else 替代的

### 最小化嵌套

> 核心思想就是：不要在if里面再套if

解决思路就是：

- 要么合理安排`if`语句的判断条件，来实现不用嵌套`if`
- 要么实现通过提前`return`

## 第八章：拆分超长的表达式

> 核心思想：将超长的表达式拆分成更容易理解的小块

### 使用解释变量

```python
引入一个额外变量来表示一个小一点的子表达式

if line.split(':')[0].strip == "root":
	...
	
username = line.split(':')[0].strip
if username == "root":
	...
```

### 使用总结变量

```java
用一个短的名字来替代一大块代码，这个名字更加容易管理和思考

if (request.user.id == document.owner_id) {
	// user can edit this document...
}
...
if (request.user.id != document.owner_id) {
	// document is read-only...
}

主要判断逻辑就是  用户是否拥有此文档

final boolean user_owns_document = (request.user.id == document.owner_id);
if (user_owns_document) {
	//  user can edit this document...
}

if (!user_owns_document) {
	// document is read-only...
}
```

### 使用德摩根定理

```cpp
1） not (a or b or c)  <=> (not a) and (not b) and (not c)
2)  not (a and b and c) <=> (not a) or (not b) or (not c)

使用德摩根定理的原因是为了使得布尔表达式更加可读
if (!(file_exists && !is_protected)) Error("Sorry, could not read file.");

改写为：
if (!file_exists || is_protected) Error("Sorry, could not read file.");
```

> 在Python、JavaScript这样的语言中，or操作符会返回其中的一个参数，而不会转化成布尔值。 x = a || b || c，可以用来找出第一个为"真"的值

### 使用反方向简化处理

```cpp
struct Range {
	int begin;
	int end;
	// For example, [0, 5) overlaps with [3, 8)
	bool OverlapsWith(Range other);
};

OverlapsWith是判断两个区间是否有重叠部分：
	如果正向判断，那么就有很多种情况
	如果反向判断，就只有两种：
		另一个范围在这个范围前结束
		另一个范围在这个范围后开始

bool Range::OverlapsWith(Range other) {
	if (other.end <= begin) return false;
	if (other.begin >= end) return false;
	return true;
}
```

### 拆分巨大的语句

如果在一大串代码中，有很多东西都是相似的，那么就可以提取公因式

```javascript
var update_highlight = function (message_num) {
    var thumbs_up = $("thumbs_up" + message_num);
    var thumbs_down = $("thumbs_down" + message_num);
    var vote_value = $("#vote_value" + message_num).html();
    var hi = "hightlighted";
    
    if (vote_value === "Up") {
        thumbs_up.addClass(hi);
        thumbs_down.removeClass(hi);
    }else if (vote_value === "Down") {
        thumbs_up.removeClass(hi);
        thumbs_down.addClass(hi);
    }else {
        thumbs_up.removeClass(hi);
        thumbs_down.removeClass(hi);
    }
}
创建 var hi = "hightlighted"有很多好处
	避免输入错误
    缩短行的宽度，使代码更易阅读
    如果需要修改，改一处就好
```

## 第九章：变量与可读性

变量的草率使用会使得程序难以理解：

- 变量越多，就越难全部跟踪他们的动向
- 变量的作用越大，就越需要跟踪他们的动向越久
- 变量改变的越频繁，就越难跟踪它的当前值

### 减少变量

> 删除没有价值的临时变量

```python
now = datetime.datetime.now()
root_message.last_view_time = now
```

这段代码中，now根本没有必要存在:

- 它没有拆分任何复杂表达式
- 没有做更多澄清，`datetime.datetime.now`够清楚了
- now只用了一次，没有压缩任何冗余代码

> 减少中间结果

```javascript
var remove_one = function (array, value_to_remove) {
    var index_to_remove = null;
    for (var i = 0; i < array.length; i++) {
        if (array[i] === value_to_remove) {
            index_to_remove = i;
            break;
        }
    }	
    if (index_to_remove !== null) {
        array.splice(index_to_remove, 1);
    }
};

index_to_remove只是保存临时结果，为什么不直接处理掉呢？
var remove_one = function (array, value_to_remove) {
    for (var i = 0; i < array.length; i++) {
        if (array[i] === value_to_remove) {
            array.splice(i, 1);
            return;
        }
    }	
};
```

### 缩小变量作用域

"避免使用全局变量"是非常好的建议，因为很难跟踪全局变量在哪使用；而且全局变量会污染命名空间，导致跟局部变量发生冲突。

所以我们应该尽量压缩变量的作用域，让它对越少的代码可见

```cpp
class LargeClass {
    string str_;
    void Method1() {
        str_ = ...;
        Method2();
    }
    void Method2() {
        // use str_
    }
    // Lots of method don't use str_ ...
};

上面的类中，就两个方法用了str_，声明为类成员变量就没必要
  
class LargeClass {
    void Method1() {
        string str_ = ...;
        Method2(str_);
    }
    void Method2(string str_) {
        // uss str_
    } 
};
```

> C++中 if 语句的作用域

```cpp
if (PaymentInfo* info = database.ReadPaymentInfo()) {
	cout << "User paid: " << info->amount << endl;
}
直接将info定义在逻辑表达式中，这样info的作用域就只在if语句内部了
```

### 只写一次的变量更好

变量的值如果一直被更改的话，程序是很难分析与理解的，索性直接定死

`static const int NUM_THREADS = 10`  C++

`static final int NUM_THREADS = 10` Java

---

> 第三部分：重新组织代码

## 第十章：抽取不相关的子问题

> 什么是不相关问题

写代码时，我们的每一个函数都有自己的作用。我们要做的就是将函数的作用最小化，每个作用是一个函数，那么每个函数就是不相关的子逻辑

思考过程如下：

- 看看某个函数 / 代码块，问下自己：这段代码意欲何为？
- 对于每一段代码，问一下自己：它直接为了顶级目标工作吗，它的作用是什么？
- 如果足够的行数在解决不相关的子问题，直接抽取成独立的函数 

### 栗子

```javascript
var spherical_distance = function (lat1, lng1, lat2, lng2) {
    var lat1_rad = radians(lat1);
    var lng1_rad = radians(lng1);
    var lat2_rad = radians(lat2);
    var lng2_rad = radians(lng2);
    
    return Math.acos(Math.sin(lat1_rad) * Math.sin(lat2_rad) +
                     Math.cos(lat1_rad) * Math.cos(lat2_rad) *
                     Math.cos(lng2_rad - lng1_rad));
};

var findClosestLocation = function (lat, lng, array) {
    var = closest;
    var closest_dist = Number.MAX_VALUE;
    for (var i = 0; i < array.length; i++) {
        var dist = spherical_distance(lat, lng, array[i].latitude, array[i].longitude);
        if (dist < closest_dist) {
            closest = array[i];
            closest_dist = dist;
        }
    }
    return closest;
};
```

- 这段代码可读性高的原因在于我们将计算两点距离的复杂数学公式单独抽取成为一个函数，这样主函数的逻辑就非常清晰了。
- 而且`spherical_distance`非常容易测试。
- 我们可以给这样的函数取个单独的名字：纯工具代码。
- 比如C++的<algorithm>算法库，java.util工具包都是放置工具代码的地方
- 工具代码多了，那么代码耦合度就下降了，同时代码结构也被优化了。我们在修改优化代码时要修改的地方同时就剩工具代码

> 自顶向下和自底向上编程

自顶向下编程是一种编程风格：先设计高层次模块和函数，然后根据支持他们所需来实现底层函数

自底向上编程尝试首先预料和解决所有子问题，利用这些代码构建高层次组件

### 根据需求增改接口

本章的核心其实就是抽取函数，解耦合代码。

那么就要考虑一个问题：什么时候抽取函数？

具体来说：

- 我们为本项目专门抽取特定的功能函数
- 编程语言提供的已有工具函数没办法满足本项目需求，那么就改进接口
- 防止过犹不及，不要过渡沉迷于抽取子问题

## 第十一章：一次只做一件事

思考流程：

- 列出代码所做的所有任务，任务可以很小或者很含糊
- 尽量把各个任务拆分到不同函数中，或者至少在不同段落中

## 第十二章：把想法变成代码

把一个想法用自然语言解释是个很有价值的能力，也只有自然语言才能被其他领域的人所理解。用自然语言解释的过程也是帮助自己更加清晰了解这个概念的过程。

思考过程：

- 用自然语言描述代码要做什么
- 注意描述中所用的词语，改进的更加合适
- 写出与自然语言描述相匹配的代码

### 清晰描述逻辑

```cpp
需求：只有拥有文档的用户或者管理员才能看到页面

自然语言描述：
授权有两种方式：
	是管理员
	拥有当前文档

那么代码就直接写：
if (is_admin_request()) {
	// authorized
}else if (document['username'] == session['username']) {
	// authorized
}else {
	return not_authorized();
}
```

> 橡皮鸭技术

- 自己将问题描述出来，往往就能帮自己找到解决问题的办法。
- 因为其实很多时候我们的任务推动不下去是因为我们自己都不知道自己要面对的问题是什么
- 当问题被清晰描述之后，总是可以想出一定的思路进行解决的
- 如果发现自己没办法清晰描述问题，说明缺少了一些东西的定义

## 第十三章：少写代码

- 从项目中消除不必要的功能，不要过度设计
- 重新考虑需求，只要能解决问题，那么设计最简单版本的代码即可
- 经常性地阅读标准库的API文档，保持熟悉程度，这样在我们遇到问题时才有可能想到使用

---

> 第四部分：精选话题

## 第十四章：测试与可读性

### 使测试易于阅读与维护

测试代码应该也是可读的，这样程序员才敢于增加新的测试，修改已有的测试

测试代码可读性的关键是：`对使用者隐去不必要的细节，让他们只关心怎么方便的增加测试，修改测试`

### 栗子

我们要测试的函数为`SortAndFilterDocs`，它的作用是将socre为负的文档移除，然后按照score排序。

**一个比较差的测试代码**

```cpp
void Test1() {
	vector<ScoredDocument> docs;
    docs.resize(5);
    docs[0].url = "http://example.com";
    docs[0].score = -5.0;
    docs[1].url = "http://example.com";
    docs[1].score = 1;
    docs[2].url = "http://example.com";
    docs[2].score = 4;
    docs[3].url = "http://example.com";
    docs[4].score = -99998.7;
    docs[4].url = "http://example.com";
    docs[4].score = 3.0;
    
    SortAndFilterDocs(&docs);      // 真正要测试的函数
    
    assert(docs.size() == 3);
    assert(docs[0].score == 4);
    assert(docs[1].score == 3.0);
    assert(docs[2].score == 1);
}

八宗罪：
    测试代码很长
    新增新的测试很难
    测试失败的提示没啥用
    这个测试想要一次测完所有东西，可读性差
    测试的输入不简单，-99998.7太没意义
    测试的输入没有彻底执行代码，比如没测到输入含0的情况
    没有测试其他极端输入
    测试的名字Test1，可读性不高
```

**改进为可读性好的测试代码**

我们想一想，测试的本质其实就是**给定输入，看经过函数后给出的输出是不是预期的输出**。只要所有的给定输入都给出期望输出，那么测试通过。

```cpp
最为理想的测试代码书写应该是:
CheckScoresBeforeAfter("-5, 1, 4, -99998.7, 3", "4, 3, 1")

void CheckScoresBeforeAfter(string input, string expected_output) {
    vector<ScoredDocument> docs = ScoredDocsFromString(input);
    SortAndFilterDocs(&docs);
    string output = ScoredDocsToString(docs);
    assert(output == expected_output);
}

vector<ScoredDocument> ScoredDocsFromString(string scores) {
    vector<ScoredDocument> docs;
    replace(scores.begin(), scores.end(), ',', ' ');
    // scores现在按照空格分割
    istringstream stream(scores);
    doouble score;
    while (stream >> score) {
        AddScoredDoc(docs, score);
    }
    return docs;
}

string ScoredDocsToString(vector<ScoredDocument> docs) {
    ostringstream stream;
    for (int i = 0; i < docs.size(); i++) {
        if (i > 0) stream << ", ";
        stream << docs[i].score;
    }
    return stream.str();
}

void AddScoredDoc(vector<ScoredDocument> docs, double score) {
    ScoredDocument sd;
    sd.score = scorel
    sd.url = "http://example.com";
    docs.push_back(sd);
}
```

### 让错误消息更可读

我们需要封装assert函数，让错误输出更加具体

```cpp
assert(output == expected_output)打印的错误消息
	Assertion failed: (output == expected_output),
		function CheckScoresBeforeAfter, file test.cc, line 37.

c++ Boost库
BOOST_REQUIRE_EQUAL(output == expected_output)打印的错误消息
	test.cc: fatal error in "CheckScoresBeforeAfter": critical check
		output == expected_output failed ["1, 3, 4" != "4, 3, 1"]

手工打造错误消息
void ShowErrorForCheckScoresBeforeAfter(input, expected_output, output) {
	cout << "CheckScoresBeforeAfter() failed, " << endl;
	cout << "Input:            \"" << input << endl;
	cout << "Expected Output:            \"" << expected_output << endl;
	cout << "Actual Output:            \"" << output << endl;
}

Python是 assert a == b
unittest模块的assertEqual方法：assertEqual(a, b)
```

### 选择一个好的测试输入

> 原则

选择一组最简单的输入，它能完整地使用被测代码。故意写不太可能出现的数据没意义的

`CheckScoresBeforeAfter("1, 2, -1, 3", "3, 2, 1")`

### 测试函数命名

要测试的对象是：

- 被测试的类： Test_<ClassName>
- 被测试的函数： Test_<FunctionName>
- 被测试的情形或bug： Test_<FunctionName>__<Situation>

### 测试驱动开发

测试驱动开发(TDD)是一种编程风格，在写真实代码前就写出测试代码。好处是写的 代码会适合测试，同时代码结构和代码质量都会很好改进

> 可测试性差的代码

| 特征                     | 可测试性问题                           | 设计问题                                               |
| ------------------------ | -------------------------------------- | ------------------------------------------------------ |
| 使用全局变量             | 每个测试都要重置所有的全局状态         | 很难理解哪些函数有什么副作用。没办法独立考虑每个函数。 |
| 对外部组件有大量依赖代码 | 因为要先搭脚手架，再写测试代码，就很烦 | 系统可能会因为某个依赖失败而失败。很难重构类           |
| 代码有不确定行为         | 测试会很古怪，测试结果也不可靠         | 可能有条件竞争或者其他难以重现的bug                    |

> 可测试性好的代码

| 特征                       | 对测试好处           | 对设计好处               |
| -------------------------- | -------------------- | ------------------------ |
| 类中只有很少或没有内部状态 | 很容易写测试         | 类简单，易于理解         |
| 类 / 函数只做一件事        | 只需要很少测试用例   | 耦合度低，模块化程度高   |
| 每个类对别的类依赖很少     | 每个类可以独立测试   | 系统可以并行开发         |
| 函数接口简单，定义明确     | 有明确的行为可以测试 | 重用可能性大，也易于阅读 |