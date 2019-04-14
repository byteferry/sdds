# Sdds Specification 1.0 beta
## Sdds规范1.0 beta
    
Copyright： Sdds project 2018
    
License： GNU GPL
    
### 概要：
    
#### 什么是sdds？
sdds是stream data dynamic structure的缩写，即流数据动态结构。sdds是一种dsl(domain-specific language)。使用程序员可读且程序可理解的方式用来描述二进制流数据的结构，从而实现与规范程序对二进制流数据的读写。本文讲述sdds的结构，以及sdds引擎程序需要支持的功能。
    
#### 为什么要用sdds？
在基于数据流的通讯中，或者其它所有涉及二进制数据流的处理中，到目前为止，我们没有一个有效的处理方式来进行数据的读写，通常情况下，我们要针对不同的二进制数据流的处理来编写不同的程序。
    
sdds的目标则是，使用统一的sdds基础编码与解码引擎程序，只要通过极少的部分扩展，就能快速实现二进制数据流编码与解码。
    
sdds专门针对二进制数据流的应用层网络通讯协议的编码与解码，力求简化通讯开发。
sdds的目标是：简单，易用、快速、高效。

### sdds文档定义格式：
sdds的定义，我们称其为sdds schema。sdds schema是数据流的编码与解码指令的文件。这些指令通过sdds引擎来解析数据。也就是说，它是基于通讯协议或文档数据格式的动态数据结构。
    
sdds的schema定义使用json格式，以方便在编写schema时可借助于相关IDE进行语法检查。
    
sdds schema中关键字使用完全小写，并且，单词间连接采用下划线。
   
### sdds的数据类型
    
#### 基本数据类型：
sdds目前支持以下基本数据类型：
    
   (1)整数类型：
       
int8：整数类型，1个字节（8位）长度的整数。
int16：整数类型，2个字节（16位）长度的整数。
int24：整数类型，3个字节（24位）长度的整数。
int32：整数类型，4个字节（32位）长度的整数。
int64：整数类型，8个字节（64位）长度的整数。
    
所有以上类型均支持有符号(signed)与无符号(unsigned)。且默认是有符号(signed)，如果无符号，则 unsigned属性（attribute）要指定为true.
    
   (2)整数类型别名 
      
bool：int8保存的布尔类型，在字节字段中是1个字节，在位字段中是1位。数值为0和1。
char：int8类型的别名。
byte：int8类型的别名。
short：int16类型的别名。
word：int16类型的别名。
dword： int32类型的别名。
int： int32类型的别名。
long： int64类型的别名。
    
    
   (3)浮点数类型：
    
float32： 符点数，4个字节（32位）长度的符点数。
float64： 符点数，8个字节（64位）长度的符点数。

   (4)浮点数类型别名：
   
single： float32类型的别名。
double： float64类型的别名。

   (5)其它基本类型：
   
bytes：字节类型，读取指定长度字节的字节
bit：位类型，
bits：读取指定长度的位。
string： 读取指定长度，指定charset的字符串。
bcd8421：基于8421算法的bcd码。
hex： 16进制字符串类型
    
注：所有数值类型均有endianness区分。可通过options节点指定。
     
sdds引擎不仅要支持读(read)，同时也要支持写(write),插入(insert),替换(replace）。
    
sdds节点中，通过以下两个attributes指定数据类型：data_type和unsinged。
    
endianness则在options节点指定。如果有个别节点与实际的endianness不一致，则需要通过自定义函数实现。

#### 数据类型的扩展
    
除了上述基本数据类型以外，对于不支持的数据类型，sdds引擎支持数据类型的扩展。扩展方式有：通过基本数据类型组合的自定义类型，参见Attribute byte_fields和bit_fields.
    
也可以通过基本类型bytes读出字节，然后再通过format,formula,after_change等属性指定对应的的扩展函数实现。

### sdds schema节点结构。

#### Schema结构
说明，任一sdds schema 均包括以上三个分支节点：即格式如下：
```
{
"meta"：{},
"options"：{},
"sdds"：{}
}
```
其中：
meta：用来说明本sdds schema相关信息的元数据。sdds引擎并不处理此节点。
options：为sdds引擎配置的全局程序运行参数
sdds：sdds schema主体。

#### meta节点结构
     
由于meta节点仅仅是一个信息(information)节点，程序不会用它来处理数据，所以，meta节点的字段可以随意增加或减少。所以，以下仅为推荐关键字。
    
```
    "meta":{
        "comment"："注释，程序注释",
        "version"："版本",
        "author"："作者",
        "copyright"："版权信息",
        "summary"："概要信息",
        "key_words"："关键字",
        "license"："授权信息",
        "support"："技术支持信息",
        "keywords"："关键字",
        "contact"："联系方式"
    }
```
当然，以上信息均是可以选填的，但若要开源，则以上有些信息是必不可少的。
    
#### options节点结构
    
options节点的基本内容如下：
    
```
   "options"：{
        "endianness"：2,
        "min_length"：15,
        "debug"：true,
        "top_node"："message"
   }
```
    
options配置节点提供给程序进行编码与解码的全局程序运行参数。参数说明如下:
    
endianness：integer,字节序，机器字节序：0，小端字节序：1，大端字节序：2
    
min_length：integer,数据包最小长度。用于写数据时的内存分配，因为节点读写是在内存中完成的。这样，减少动态分配内存的性能开销。
    
debug：bool，指定是否为调试状态。当此属性关闭，则不再输出调试信息。
    
top_node：string，指定程序从哪个节点开始解析。通常，如果未指定，程序会从decode或encode下的document,或message这两个节点。找到后，就以此开始。
    
#### sdds节点结构
    
```
    "sdds": {
        "decode"：{
            "message"{}
        },
        "encode"：{
            "message"{}
        }
    }
```
    
通常sdds有两个分支节点：
decode：json，指定此节点为解码用的节点列表的分支；但如果是纯解码或编码，即只有单一功能，也可以不要这一层分支。
encode：json，指定此节点为编码用的节点列表的分支；但如果是纯解码或编码，即只有单一功能，也可以不要这一层分支。
下文中，我们默认decode、encode分支存在。
本质上，decode、encode分支中的子节点才是真正的SDDS读写指令。
在decode、encode分支中，通常有一个节点是顶层节点。即此节点是描述整个数据包结构的节点。上例中message为顶层节点。默认情况下，message或document为程序开始解析的第一个节点。也就是最大的节点。当然，你可以通过在options节点中的top_node等指定为document，或其它名称的节点。推荐使用message或document。
    
#### decode、encode分支中子节点的结构
    
（1）基本数据结点结构
    
基本数据结点的结构如下：
    
```
    "node_name"：{
        "name"："var_name",
        "id"："node_id",
        "type"："int32",
        "unsigned"：false,
        "length"：4,
        "position"：4,
        "value"："",
        "required"：false,
        "default"："",
    }
```
   
以上节点实际上是定义了数据流中某一段与基本数据类型的对应关系。其中的属性相关说明如下：
    
"node_name"：节点的名称。
    
"name"：变量名称，如果要读，要写时，此属性不可缺少。name在重复节点中是可以同名的。
    
"id"：节点的ID，必须在整个schema中是唯一的才可以。
    
"type"：数据类型，除上述基本类型外，也可以是任意名称，但这一名称必须要在sdds分支中有对应的节点，这即是自定义类型。
    
"unsigned"：false,变量是否为无符号类型，此属性。仅对整数类型有效，且默认为有符号，即"unsigned"：false,可以不写。
    
"length"：数据的长度，这里是字节的长度。
    
"position"：读写数据流的位置。以字节为单位的偏移量。
    
"value"：数值，读出时，存入此属性，写入时在此属生中先读出。
    
"required"：是否必须。也就是不可以为空，仅对发送数据进行校验。
    
"default"：默认值。仅用来在value未被指定时，作为默认发送的数值
    
（2）自定义类型的节点。
    
自定义类型的节点与基本数据类型节点一样，只是type中的类型，是sdds中某个节点的名称。这就是说，sdds使用节点实现自定义类型。例如：
    
```    
"node_name"：{
	"name"："var_name",
	"id"："node_id",
	"type"："message_header",
	"unsigned"：false,
	"length"：4,
	"position"：4,
	"value"："",
	"required"：false,
	"default"："",
}
```
    
则在sdds或sdds的decode、与encode分支节点中一定有一个节点名为message_header的节点。
    
（3）带有分支的节点：
    
带有分支的节点，如同SDDS节点一样，带有分支。这样的分支只有两类，分类是字节字段分支和位字段分支。
分别如下：
    
```    
"node_name"：{
	"name"："var_name",
	"id"："node_id",
	"length"：4,
	"position"：4,
	"byte_fields"：{
	}
}
"node_name"：{
	"name"："var_name",
	"id"："node_id",
	"length"：4,
	"position"：4,
	"bit_fields"：{
	}
}
```
与基本数据类型节点不同，节点中许多属性是不需要的。同时，byte_fields和bit_fields中也必须要有分支节点。并且byte_fields分支均可以是任一种类型的节点。而bit_fields中的分支，只能是基本数据类型节点。
    
通常情况下，顶层节点必须是带有byte_fields分支的节点。因为，sdds引擎不会按顺序主动遍历sdds的分支，而是由顶层节点指定流程进行。
    
例如，一个消息节点的结构可能会是这样的：
    
```    
"message"：{
	"name"："message",
	"id"："node_id",
	"length"：4,
	"position"：4,
	"byte_fields"：{
		"header"：{},
		"body"：{},
		"footer"：{}
	}
}
```
    
当然，上述数据包的结构不包含数据包的开始与结束标识。如果这样，你需要在调用解码前先要去除，而对于发送，则要在编码完成后加上。如果不希望额外写这些代码，你也可以直接写在分支中。结构如下：
    
```    
"message"：{
	"name"："message",
	"id"："node_id",
	"length"：4,
	"position"：4,
	"byte_fields"：{
		"stx"：{},
		"header"：{},
		"body"：{},
		"footer"：{}
		"etx"：{},
	}
}
```
(4)单选节点
    
单选节点，是根据某一条件，而选择某一个节点的结构。单选结点的结构如下：
    
```    
"node_name"： {
        "comment"： "",
        "one_of"： {
	          "key"： {
		            "comment"： "",
		            "name"： "message_key",
		            "value"： "#message_id",
		            "format"： "0x%04X",
		            "id"： "message_key"
	          },
	          "list"： {
			    "0x0201"：{},
			    "0x0302"：{}
		  }
}
```
    
从上例可以看出：单选节点必须要有一个名为one_of的分支。同时，one_of分支中，必须要有key和list分支，而key则是指明，用哪个节点的值来作为选择的条件。同时，有format将key格式化。格式化的结果，一定是list分支中的一个节点名。
    
上例中，value是用的选择器，即是指明，读取ID为message_id节点的值。
    
(5)重复节点
    
重复节点是指它有多个同类的子节点。重复节点的结构如下：
    
```    
"node_name"： {
        "comment"： "",
        "position"： 0,
        "length"： -2,
        "repeat"： true,
        "count"： 4,
        "until"： "",
        "type"： "repeat_node_name",
        "id"： "node_id",
        "name"： "some_name"
} 
```
    
从上例可以看出，重复节点内没有分支，而是用自定义类型名称，指向到另一个节点。重复节点可以通过指定repeat属性为true，告诉程序，此为重复节点，同时，如果知道重复的数量，可以指定到count属性中。如果数量不知，则可以不指定，也可以通过until属性指向自定义的扩展函数确定重复的条件。
    
(6)多选节点
    
多选节点则是一种组合节点。外层是重复节点，内层则是单选节点。通过重复实现多选。示例（略）

### 参数传递
    
sdds schema中采用以下三种方式进行参数传递。 
    
(1)数值传递选择器
    
"#" id选择器，指定读取对应id的值。注意，id在同一sdds文件中应当唯一。如果不唯一，则应当使用其它的选择器。示例："value"："#body_length",则是指其值是读取id为body_length的值。当前节点一定是body_length后的节点。因此，一定有另外一个节点，属性中有："id"："body_length"。
    
"@"： path选择器,指定通过路径读取某节点的值。注意，路径是先向上搜索，搜索到相同节点后，再向下搜索。所以，路径只要指定到分叉节点即可，不一定要指定到根节点。如果无法清楚path,可以通过调试属性指定，以查看实际的path。示例："value"："&body.message_body.body_length",则是指其值是按路径查找并读取body_length的值。当前节点一定是body_length后的节点。使用路径的好处是，你所要操作的节点，无需进行任何指定。你只要在要操作的地方直接给出路径选择器即可。
但是，如果要读取的节点是repeat节点中的分支，那么，节点的name是重复的，同时，即使你指定id，但因是repeat，则id也会出现重复。同样，最终生成的节点，节点名是以节点的index来命名的，所以，path选择器一样也不可用。这时，需要通过索引（index）选择器来获取这一类节点。。
    
"$" 索引（index）选择器，被操作的节点需要通过selector指定为index.例如：,"selector"："index"，而另一节点中，则是"value"："$item_length"，此例则是指定操作相同index子节点中的item_length子节点。这一子节点中肯定有"selector"："index"这一指定。
    
"*" 函数选择器，调用在扩展（继承类）中实现的自定义函数。
任一属性值中，如果第一个字符是以上的选择器字符，sdds将会调用选择器程序读取数据。所以，如果开头是以上选择器字符，则要使用转义。比如："\#","\$"。一旦使用转义，则需要在扩展程序中做相关处理。sdds不会自动清除转义。
    
(2)数值传递属性
    
数值传递属性即是指selector属性。即为节点指定一种选择器以使后续可以对它操作。selector有两个值："id"和"index"，如果使用id,则必须保证在decode或encode分支下此id是唯一的。对于repeat的子节点。由于不能指定id,则必须指定selector为index。


### sdds关键字：

sdds包括以下关键字，功能说明如下：

(1)标准通用关键字：
    
comment：string 注释，用于方便文档阅读与程序调试时使用。程序不处理此attribute。
    
(2)节点类型关键字：
    
meta：json,元数据节点，为文档提供注释与说明的节点，不进行数据解析。
    
options：json,指定一个配置节点。sdds从中读取配置数据。
    
sdds：json,指定数据结构的节点列表。
 
(3)节点属性关键字：
	
(3.1)数据结构类：
    
此类是两种分支节点：byte_fields和bit_fields。
    
byte_fields：json 指定节点包括的子节点，即字段，其数据类型是以字节（byte）读写的基本类型的子节点。
    
bit_fields： json 指定节点包括的子节点，即字段，其数据类型是以位(bit)读写的基本类型的子节点。
    
(3.2)数据类型属性关键字
    
type：string  数据类型，可以是上述的基本数据类型，如果基本数据类型，则会使用自定义名称，并且此名称一定会在sdds中有对应定义的节点。
    
position：int 字节或位的开始定位。如果是当前到倒数第几位的位置，则填与负数。此属性不是必须的，程序通常不会处理它。但它可以帮助我们在编写Schema时校对。
    
length：int 数据长度。无法得知其长度，可以是null，如果是当前到倒数第几位的位置，则填与负数。
    
unsigned：bool 是否无符号。基本数据类型，默认为false。无符号要指定为true。
    
(3.3)数值相关属性。
    
value：变量的值。schema配置中，一般不会出现此关键字，但是，像stx,etx的value是固定的，则会写入value。value可以通过选择器读取其它节点的值（参见数值传递选择器）此属性，是程序在解码时写入数值，编码时从中读入数值。
    
default：变量的默认值，编码时要从value中读入数值，如果value为空，则取default值。
    
required： bool 进行数据写入时的校验。当此值为空时，且default也为空时会抛出异常。
    
(3.4)变量相关属性关键字：
    
name：string 变量名，用于程序中使用，或数据库中的字段对应。解码时，程序不会返回无name的数据。即：name告诉程序，我们要读出什么。通常，解码返回的结果是keyValue结构。Key使用的是name。repeat为true的子节点中，注意：name是相同的。id也会重复。
    
(3.5)程序行为属性关键字：
    
repeat：bool 用以声明，此类型数据是否有连续多项。
    
one_of：json 用于通过key选择某个子类型。
    
key：json 用于one_of指定的检索条件。
    
list：json 用于one_of的列表定义

(3.6)预处理与后处理属性关键字：
    
before_change：json,用以指明，当前节点数据处理前要调用的用户自定义函数。调用发生的时间是：函数解码时，在初始化以后，读取数据之前，编码时，在初始化以后，写入数据之前。
    
after_change：json 用以指明，当前节点数据处理后要调用的用户自定义函数。调用发生的时间是：函数解码时，在初始化以后，读取数据之后，编码时，在初始化以后，写入数据之后。
    
(3.6)数值传递关键字：
    
selector：选择器设置，指明程序后续查找此节点的方式，数值有：
    
* "id"：通过id找到此节点，此时，该 节点的id属性不能为空，且ID必须是全schema中唯一的。读取时写 "#{node_id}"({}表示要换成实际的名称）
	    
* "index"：此节点必须是repeat的中某一项的子项，通过此项的index与子项的name查找到此节点。读取时写 "#{node_name}"({}表示要换成实际的名称）
    
id: 用于后续通过此id获取数值的id，任一属性，如果其值以#开头，则是要读取对应id节点的值。此时：find_by属性值是："id"
    
(3.7)数据格式处理关键字：
    
format：string 数据格式指定（仅用于one_of中使用的搜索关键字key的定义中）
    
formula：string 编码、解码公式，在decode分支，即是解码公式，而encode分支中，则是解码公式。公式中的变量，是节点中的Value，变量名为"A"
     
(3.8)调试关键字：
    
debug：bool,当为true时，会在当前节点中断，并打印出当前读出/写入的数据。
    
trace： string 可以录入需要调试查看的当前节点的属性，用逗号分隔，会由程序指定的方式输出，options中debug为true时，会通过web或控制台方式输出。否则，写入到日志文件。
    
ignore_errors：指定此节点忽略错误。默认为false。为true时，异常将不会抛出。除非必须，尽量少做这类指定，因为，这会给调试带来麻烦。
    
### 常见问题：
    
1、sdds引擎是否可以做数据包有效性验证？
    
sdds引擎不做数据包的有效性验证。这一部分必须在sdds引擎之外完成。这样可以适合不同的通讯协议。但若需要sdds引擎在验证前读出相关数据结构，则可以在sdds引擎中定义validate节点进行读取。
    
2、sdds引擎未读出数据，或未写入数据。但无任何错误或异常。
    
这是因为，缺少name属性。添加name属性，即可解决。
    
3、如何实现repeat多个one_of节点？
    
只要schema中，定义父节点是repeat，子节点使用one_of即可。
    
4、attribute和property区别是什么？attribute和property都可称为属性。但是，attribute是所有可以在schema中定义的字段。而property则是sdds引擎中使用的属性。我们一样要了解property，因为程序需要对它进行操作。本规范约定：所有property均以"_"开头。
    
5、sdds中没有if条件，如何处理？
    
sdds中不需要if条件，这样的需求可以通过one_of节点结构来实现。
    
6、选择器是不是只能搜索本节点前的已加载的节点？未加载的如何处理？
    
是的，选择器只能搜索已存在的节点。要处理未加载的节点，不要在前面的节点中处理后面的节点，而是在后面的节点中读写前面的节点。
      
### sdds attributes索引
   
index of attributes of sdds node
  
| 属性名	| 数据类型 |  说明 |
|---------------|-----------|----------|   
| after_change | string | 定义读写操作后调用的自定义函数，逗号分隔 
| before_change | string | 定义读写操作前调用的自定义函数，逗号分隔
| bit_fields | json | 定义位读写的分支节点
| byte_fields | json |定义字节读写的分支节点
| comment | string |  节点或schema的文字注解。
| count | integer |  定义重复写入的项数	
| debug | bool | options节点用来定义schema是否为调试状态，节点中用来指定此节点是否显示调试信息。
| default | mixed | 定义要写入的默认值
| format | string |用来定义获取数值后的数据格式化处理
| formula | string |用来定义解码读取后或编码写入前所用的公式。
| id | string |  定义节点的唯一ID，方便后续访问
| ignore_errors | bool | 定义此节点是否忽略错误。
| key | string | 定义选择节点中所用的key。
| length | integer | 定义当前节点的数值在Stream中的长度
| list | json | 定义选择节点中候选节点列表
| name | string | 定义节点的变量名
| one_of | json | 节点one_of属性中有子节点时，节点为选择节点。
| position | string | 定义数值读写时在Stream中的Offset.	
| repeat | bool | 指明当前节点是否为重复节点。
| required | bool |指明要写入的值是否为必须。
| selector | string | 定义当前节点的选择器。即通过id还是index可以访问。
| trace | string |  定义当前节点的调试跟踪，可以用逗号分隔要trace的sttribut。
| type | string | 定义当前节点的数据类型。如果是自定义类形，则在sdds的decode或encode下必须有要同名节点。
| unsigned | string | 定义当前节点的数据是否为无符号。
| until | string | 定义重复节点的重复条件函数。
| value | mixed | 当前节点的值，可以是读取的，也可以是写入的。
 
