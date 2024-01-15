# 第8章 IO库

## 8.1 IO类

| 头文件 | 类型 |
| --- | --- |
| iostream | istream,wistream从流中读取数据 |
|  | ostream,wostream向流中写入数据 |
|  | iostream,wiostream读写流 |
| fstream | ifstream,wifstream从文件读取数据 |
|  | ofstream,wofstream向文件写入数据 |
|  | fstream,wfstream读写文件 |
| sstream | istringstream,wistringstream从string中读取数据 |
|  | ostringstream,wostringstream向string写入数据 |
|  | stringstream,wstringstream读写stream |

为了支持宽字符的语言，标准库定义了一组类型和对象来操作那个wchar_t类型的数据。宽字符版本的类型和函数的名字以w开始。如：wcin, wcout, wcerr分别对应cin, cout, cerr的宽字符版本。宽字符版本的类型和对象与其对应的普通char版本的类型定义在同一个头文件中。

**IO类型间的关系**：标准库使我们可以忽略不同类型的流之间的差异，通过继承机制实现。利用模板可以使用具有继承关系的类，而不必了解继承机制工作的细节。继承机制使我们可以声明一个特定的类继承自另一个类。通常可以将一个派生类（继承类）对象当作其基类（所继承的类）对象来使用。

类型ifstream, istringstream都继承自istream。因此可以像使用istream对象一样使用ifstream, istringstream对象。类型ostream, ostringstream都继承自ostream。因此如何使用cout，就可以同样的使用这些对象。

### 8.1.1 IO对象无拷贝或赋值

我们不能拷贝或对IO对象赋值：

```jsx
ofstream out1, out2;
out1=out2;  //错误，不能对流对象赋值
ofstream print(ofstream);  //错误，不能初始化ofstream参数
out2=print(out2);  //错误，不能拷贝流对象
```

不能拷贝IO对象，所以不能将形参或返回类型设置为流类型。进行IO操作的函数通常以引用方式传递和返回流。读写IO对象会改变其状态，因此传递和返回的引用不能是const。

### 8.1.2 条件状态

流的条件状态表：

| strm::iostate | iostate是一种机器相关的类型，提供了表达条件状态的完整功能 |
| --- | --- |
| strm::badbit | 用来指出流已崩溃 |
| strm::failbit | 用来指出一个IO操作失败了 |
| strm::eofbit | 用来指出流到达了文件结束 |
| strm::goodbit | 用来指出流未处于错误状态。此值保证为0 |
| s.eof() | 若流s的eofbit置位，则返回true |
| s.fail() | 若流s的failbit或badbit置位，则返回true |
| s.bad() | 若流s的badbit置位，则返回true |
| s.good() | 若流s处于有效状态，则返回true |
| s.clear() | 将流s中的所有条件状态位复位，将流的状态设置为有效。返回void |
| s.clear(flags) | 根据给定的flag标志位，将流s中对应条件状态位复位，flag类型位strm::iostate，返回void |
| s.setstate(flags) | 根据给定的flag标志位，将流中对应的条件状态位置位，flag类型位strm::iostate，返回void |
| s.rdstate() | 返回流s的当前条件状态，返回值类型为strm::iostate |

下列为一个IO错误的例子：

```jsx
int ival;
cin>>ival;
```

若在标准输入上键入boo，读操作将会失败，cin将进入错误状态。一旦流发生错误，其上后续的IO操作都会失败。只有当一个流处于无错误状态时，才可以从它读取数据，或向它写入数据。因此，代码通常应该在使用一个流之前检查它是否处于良好状态。

```jsx
while(cin>>word)
	//...
```

**查询流的状态**：IO库定义了一个与机器无关的iostate类型，提供了表达流状态的完整功能。IO库定义了4个iostate类型的constexpr值表示特定的位模式。这些值用来表示特定类型的IO条件，可以参与位运算符一起使用来一次性检测或设置多个标志位。

- badbit：表示系统级错误，如：不可恢复的读写错误。通常情况下，一旦badbit被置位，流就无法使用；
- failbit：发生可恢复错误后，failbit被置位，如：期望读取数值却读出字符。通常这种问题修正后，流仍然可使用；
- eofbit：到达文件结束位置时被置位，同时failbit也会被置位；
- goodbit：表示流未发生错误，值永远为0，若badbit, failbit, eofbit任一被置位，则检测流状态的条件会失败；

标准库还定义了一组函数来查询这些标志位的状态：good()：所有错误位均未置位时返回true；bad, fail, eof在对应错误位被置位时返回true。badbit被置位时fail也会返回true。所以good或fail是确定流总体状态的正确方法。实际上，我们将流当作条件使用的代码就是等价于!fail()。而eof和bad操作只能表示特定的错误。

**管理条件状态**：

- s.rdstate()：返回一个iostate值，对应流的当前状态；
- s.setstate(flags)：为给定条件位置位，表示发生对应错误；
- s.clear()：清除所有错误标志位。执行clear()后，调用good()会返回true；

```jsx
auto old_state = cin.rdstate();  //记住cin的当前状态
cin.clear();  //使cin有效
process_input(cin);  //使用cin
cin.setstate(old_state);  //将cin置为原有状态
```

- s.clear(flag)：接受iostate值，表示流的新状态。为了复位单一的条件状态位，先用rdstate读出当前条件状态，后用位操作将所需复位来生成新的状态。

```jsx
cin.clear(cin.rdstate() & ~cin.failbit & ~cin.badbit);
//复位failbit和badbit，保持其他标志位不变
```

练习题：

```jsx
编写一个函数接受istream&参数，返回值类型也为istream&。此函数从给定流中读取数据，遇到文件结束标志
时停止。将读取的数据打印在标准输入输出上。返回流之前，进行复位，使其处于有效状态。
代码：
istream& func(istream& is){
    string buf;
    while(getline(is,buf))
        cout<<buf<<endl;
    is.clear();
    return is;
}
int main(){
    func(cin);
    return 0;
}
```

### 8.1.3 管理输出缓冲

每个输出流都管理一个缓冲区，用来保存程序读写的数据。有了缓冲机制，操作系统就可以将程序的多个输出操作组合成单一的系统级写操作。这样可以带来很大的性能提升。导致缓冲区刷新的原因有很多：

- 程序正常结束，作为main函数的return操作的一部分，缓冲区被刷新；
- 缓冲区满时，刷新缓冲，后新的数据才能写入缓冲区；
- 使用操纵符endl显式刷新缓冲区；
- 使用操纵符unitbuf设置流的内部状态，使其在每个输出操作后清空缓冲区。默认情况下，对cerr时设置unitbuf的，所以写到cerr的内容是立即刷新的；
- 当一个流被关联到另一个流时，当读写被关联的流时，关联流的缓冲区会被刷新。如默认情况下，cin和cerr都关联到cout，所以读写cin或cerr都会导致cout的缓冲区被刷新。

**刷新输出缓冲区**：IO库中的flush和ends操纵符可以刷新缓冲区。flush刷新缓冲区，但不输出任何额外字符；ends向缓冲区插入一个空字符，然后刷新缓冲区；

```jsx
cout<<"hi"<<endl;  //输出hi和一个换行符后，刷新缓冲区
cout<<"hi"<<flush;  //输出hi后，刷新缓冲区
cout<<"hi"<<ends;  //输出hi和空字符后，刷新缓冲区
```

**unitbuf操纵符**：使用unitbuf操纵符，使流在之后的每次写操作后都进行一次flush操作；而niunitbuf操纵符重置流，使其恢复正常的系统管理缓冲区刷新机制；

```jsx
cout<<unitbuf;  //所有输出操作后都会立即刷新缓冲区
cout<<nounitbuf;  //回到正常的缓冲方式
```

> 如果程序异常终止，输出缓冲区不会被刷新。当一个程序崩溃后，它所输出的数据可能停留在缓冲区中等待打印。所以调试一个已经崩溃的程序时，需要确认那些已经输出的数据已经刷新了。
> 

**关联输入和输出**：当一个输入流被关联到一个输出流时，任何试图从输入流读取数据的操作都会先刷新关联的输出流。因为标准库将cin和cout关联在一起，所以以下语句将导致cout的缓冲区刷新。

```jsx
cin>>ival;
```

可以将一个istream对象关联到另一个ostream，也可以将一个ostream关联到另一个ostream中：

```jsx
cin.tie(&cout);  //标准库将cin和cout关联到一起

ostream *old_tie = cin.tie(nullptr);  //old_tie指向当前关联到tie的流；cin不再与其他流关联
cin.tie(&cerr);  //读取cin将刷新cerr
cin.tie(old_tie);  //重建cin和cout的关联
```

每个流同时最多关联到一个流，但多个流可以同时关联到同一个ostream。

## 8.2 文件输入输出

头文件fstream定义了三个类型来支持文件IO；

- ifstrean：从一个给定文件读取数据；
- ofstream：向一个给定文件写入数据；
- fstream：读写给定文件；

除了继承自iostream类型的行为外，fstream中定义的类型还增加了一些新的成员来管理与流关联的文件；

fstream特有的操作表：

| fstream fstrm | 创建一个未绑定的文件流。fstream是文件fstream中定义的一个类型 |
| --- | --- |
| fstream fstrm(s) | 创建一个fstream，并打开名为s的文件。sk可以是string类型或C风格字符串的指针。这些构造函数都是explicit的。默认的文件mode依赖于fstream的类型 |
| fstream fstrm(s, mode) | 与前一个构造函数类似，但按指定mode打开文件 |
| fstrm.open(s) | 打开名为s的文件，返回void |
| fstrm.close() | 关闭与fstrm绑定的文件，返回void |
| fstrm.is_open() | 返回一个bool值，指出与fstrm关联的文件是否成功打开且尚且未关闭 |

### 8.2.1 使用文件流对象

当想要读写一个文件时，可以定义一个文件流，并将对象和文件关联起来。每个文件流都定义了一个名为open的成员函数，它完成了一些系统相关的操作，来定位给定的文件，并视情况打开为读或写模式。

创建文件流对象时，可以提供文件名（可选）。当提供了文件名时，open会被自动调用。

```jsx
ifstream in(ifile);  //构造一个ifstream并打开给定文件
ofstream out;  //输出文件流未关联到任何文件
```

**用fstream替代iostream&**：在要求使用基类对象的地方，可以用继承类型的对象来替代。如接受一个iostream类型引用（或指针）参数的函数，可以用一个对应fstream(或sstreeam)类型来调用。

```jsx
ifstream input(argv[1]);  //打开销售记录文件
ofstream output(argv[2]);  //打开输出文件
Sales_data total;  
if(read(input, total)) {  
	Sales_data trans; 
	while(read(input, trans)){
		if(total.isbn() == trans.isbn())
			total.combine(trans);
		else{
			print(output, total) << endl;
			total = trans;
		}
	}	
	print(output,total)<<endl;
}else
	cerr<<"No data?"<<endl;
```

**成员函数open和close**：

```jsx
ifstream in(ifile);  
ofstream out;  //输出文件流未与任何文件相关联
out.open(ifile + ".copy");  //打开指定文件
```

如果调用open失败，failbit会被置位。

```jsx
if(out)  //检查open是否成功，若open成功就可以使用文件了
```

一旦一个文件流已经打开，它就保持和对应文件的关联。对一个已经打开的文件流调用open会失败，并导致failbit被置位。随后试图使用文件流的操作都会失败。所以为了将文件流关联到另一个文件，必须关闭已经关联的文件。

```jsx
in.close();
in.open(iflie, "2");
```

**自动构造和析构**：

```jsx
//使main函数接收一个要处理的文件列表
for(auto p=argv+1; p!=argv+argc;++p){
	ifstream input(*p);  //创建输出流并打开文件
	if(input){  //如果文件打开，就“处理”文件
		process(input);
	}else
		cerr<<"couldn't open:"+string(*p);
} //每个循环input都会离开作用域，因此会被销毁
```

因为input是while循环的局部变量，在每个循环步中都要创建一次，销毁一次。当一个fstream对象离开作用域时，与之关联的文件会自动关闭，下一步循环中，input会被再次创建。

> 当一个fstream对象被销毁时，close会自动被调用
> 

### 8.2.2 文件模式

| in | 以读方式打开 |
| --- | --- |
| out | 以写方式打开 |
| app | 每次写操作前均定位到文件末尾（append） |
| ate | 打开文件后立即定位到文件末尾(at end) |
| trunc | 截断文件 |
| binary | 以二进制方式进行IO |

指定文件模式有如下限制：

- 只可以对ofstream或fstream对象设定out模式；
- 只可以对ifstream或fstream对象设置in模式；
- 只有当out也被设定时才可以设定trunc模式；
- 只要trunc没被设定，就可以设定app模式。在app模式下，即使没有显式指定out模式，文件也总是以out方式打开；
- 默认情况下，即使没有指定trunc，以out模式打开的文件也会被截断。为了保留以out模式打开的文件的内容，必须同时指定app模式，这样只会将数据追加写到文件末尾；或者同时指定in模式，即打开的文件同时进行读写操作；
- ate和binary模式可以用于任何类型的文件流对象，且可以与其他任何文件模式组合使用；

每个文件流都定义了一个默认的文件模式，当未指定文件模式时，就使用默认文件模式。与ifstream关联的文件以in模式打开，与ofstream关联的文件以out模式打开，与fstream关联的文件默认以in和out模式打开。

**以out模式打开文件会丢弃已有数据**：默认情况下，当打开一个ofstream时，文件内容会被丢弃。阻止一个ofstream清空文件内容的方式是同时指定app模式：

```jsx
ofstream out("file1");  //隐含以输出模式打开文件并截断文件
ofstream out("file1", ofstream::out);  //隐含的截断文件
ofstream out("file1", ofstream::out | ofstream::trunc);
 //为保留文件内容，指定app模式
ofstream out("file1", ofstream::app); //隐含为输出模式
ofstream out("file1", ofstream::app | ofstream::out);
```

> 保留被ofstream打开文件中已有数据的唯一方法是显式指定app或In模式
> 

**每次调用open时都会确定文件模式**：对于一个给定流，每次打开文件时都可以改变文件模式：

```jsx
ofstream out;  //未指定文件打开模式
out.open("test");  //模式隐含设置未输出和截断
out.close();  
out.open("test1", ofstream::app); //模式为out和app
out.close();
```

第一个open调用未显式指定输出模式，文件隐式的以out模式打开。通常，out模式意味着同时使用trunc模式。所以目录下test文件的内容将会被清空。

## 8.3 string流

istringstream从string读取数据；ostringstream向string写入数据；stringstream既可以从string读取数据也可以向string写数据。

stringstream特有的操作表

| sstream strm | strm为一个未绑定的stringstream对象，sstream为头文件sstream中定义的一个类型 |
| --- | --- |
| sstream strm(s) | strm是一个sstream对象，保存string s的一个拷贝。构造函数为explicit的 |
| strm.str() | 返回strm所保存的string拷贝 |
| strm.str(s) | 将string s拷贝到strm中。返回void |

### 8.3.1 使用istringstream

处理行内的单个单词时，通常可以使用istringstream;

如输入文件看起来如下：

```jsx
Lily 13312345678 17712345678 
Andy 13812345678
Alex 11112345678 12212345678 18912345678
```

定义一个类来描述输入的数据：

```jsx
struct PersonInfo{
	string name;
	vector<string> phones;
};
```

读取数据的文件，每个循环读取一条记录，提取出一个人名和若干电话号码：

```jsx
string line,word;
vector<PersonInfo> people;
while(getline(cin, line)){
	PersonInfo info;
	istringstream record(line);
	record>>info.name;
	while(record>>word)
		info.phones.push_back(word);
	people.push_back(info);
}
```

### 8.3.2 使用ostringstream

```jsx
//若所有号码都有效，希望输出一个新的文件，包含改变格式后的号码。对于那些无效的号码，将不会输出到
//新文件中，而是打印一条包含人民和无效号码的错误信息。
for(const auto &entry : people){
	ostringstream formatted, badNums;
	for(const auto &nums : entry,phones){
		if(!valid(nums))
			badNums<<" "<<nums; //将数的字符串形式存入badNums
		}else
			formatted<<" "<<format(nums);  //将格式化的字符串写入formatted
	}
	if(badNums.str().empty())  //若badNums为空
		os<<entry.name<<" "
			<<formatted.str()<<endl;
	else
		cerr<<"input error:" <<entry.name <<" invalid number(s) "<<badNums.str()<<endl;
}
//假设valid()完成电话号码的验证，format()完成电话号码格式的改变
```