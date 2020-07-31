# [Go编程语言规范](https://golang.org/ref/spec#Size_and_alignment_guarantees)

2020年1月14日版本

[TOC]

## Introduction 简介

```This is a reference manual for the Go programming language. For more information and other documents, see golang.org.```

这是Go语言的参考手册.获取更多信息和其他文档请参考[golang.org](http://golang.org)

```Go is a general-purpose language designed with systems programming in mind. It is strongly typed and garbage-collected and has explicit support for concurrent programming. Programs are constructed from packages, whose properties allow efficient management of dependencies. The existing implementations use a traditional compile/link model to generate executable binaries.```

Go意在设计为一门通用型系统编程语言.它是强类型语言、有垃圾回收机制并且很好的支持并发编程.程序由各种包构建,这些包的特性能有效的管理依赖关系.现有实现是使用传统的编译/链接 模型来产生可执行二进制文件.

```The grammar is compact and regular, allowing for easy analysis by automatic tools such as integrated development environments.```

它的语法是简洁而通用的,能够很容易让自动化工具进行分析,例如:集成开发环境.

## 符号 Notation

The syntax is specified using Extended Backus-Naur Form (EBNF):

```go
Production  = production_name "=" [ Expression ] "." .
Expression  = Alternative { "|" Alternative } .
Alternative = Term { Term } .
Term        = production_name | token [ "…" token ] | Group | Option | Repetition .
Group       = "(" Expression ")" .
Option      = "[" Expression "]" .
Repetition  = "{" Expression "}" .
```

Productions are expressions constructed from terms and the following operators, in increasing precedence:

```go
|   alternation
()  grouping
[]  option (0 or 1 times)
{}  repetition (0 to n times)
```

Lower-case production names are used to identify lexical tokens. Non-terminals are in CamelCase. Lexical tokens are enclosed in double quotes "" or back quotes ``.

The form a … b represents the set of characters from a through b as alternatives. The horizontal ellipsis … is also used elsewhere in the spec to informally denote various enumerations or code snippets that are not further specified. The character … (as opposed to the three characters ...) is not a token of the Go language.

## 源码表示方式 Source code representation

```Source code is Unicode text encoded in UTF-8. The text is not canonicalized, so a single accented code point is distinct from the same character constructed from combining an accent and a letter; those are treated as two code points. For simplicity, this document will use the unqualified term character to refer to a Unicode code point in the source text.```

代码采用Unicode文本UTF-8编码(注：[UTF-8是Unicode的实现方式之一](http://t.cn/hN0xs)).Go沒有实现这个规范化算法(注:应该是Unicode规范),用不同表示形式的同一个字符在这里会被视为两种不同的[码位](http://t.cn/R428a6u).(注:例如é 可以表示成\u00e9和\u0065\u0301 但是在go看来他们是不一样的,参考[Googl Groups上面的提问](http://t.cn/R428J8T)).为简单起见,本篇文档中的源码将使用不规范的字符编码术语指代Unicode的[码位](http://t.cn/R428a6u).

```Each code point is distinct; for instance, upper and lower case letters are different characters.```

每一个码位都是不一样的；例如：大小写字母是不一样的字符

```Implementation restriction: For compatibility with other tools, a compiler may disallow the NUL character (U+0000) in the source text.```

实施限制:为了兼容性其他工具,编译器将不允许NUL字符(U+0000)在源码中出现

```Implementation restriction: For compatibility with other tools, a compiler may ignore a UTF-8-encoded byte order mark (U+FEFF) if it is the first Unicode code point in the source text. A byte order mark may be disallowed anywhere else in the source.```

实施限制:为了兼容性其他工具,编译器将忽略源码中BOM头部.BOM在其他地方也不被允许.

### 字符 Characters

```The following terms are used to denote specific Unicode character classes:```

下面的术语将用来表示特殊的Unicode字符类

```go
newline        = /* the Unicode code point U+000A */ .
unicode_char   = /* an arbitrary Unicode code point except newline */ .
unicode_letter = /* a Unicode code point classified as "Letter" */ .
unicode_digit  = /* a Unicode code point classified as "Decimal Digit" */ .
```

```In [The Unicode Standard 6.3](http://t.cn/zRz14zg), Section 4.5 "General Category" defines a set of character categories. Go treats those characters in category Lu, Ll, Lt, Lm, or Lo as Unicode letters, and those in category Nd as Unicode digits.```

在[Unicode标砖6.3](http://t.cn/zRz14zg),4.5章节 "General Category" 定义为一组字符集.Go将这个字符集里面的Unicode字符当作[Unicode数字](http://t.cn/R42nkpN)

### 字母和数字 Letters and digits

```The underscore character _ (U+005F) is considered a letter.```

下划线字符(U+005F)被认为是一个字母

```go
letter        = unicode_letter | "_" .
decimal_digit = "0" … "9" .
octal_digit   = "0" … "7" .
hex_digit     = "0" … "9" | "A" … "F" | "a" … "f" .
```

## 词汇要素

### 注释 Comments

```bash
Comments serve as program documentation. There are two forms:
```

注释将被当作程序的文档，它有两种格式

```bash
Line comments start with the character sequence // and stop at the end of the line.
General comments start with the character sequence /* and stop with the first subsequent character sequence */.
A comment cannot start inside a rune or string literal, or inside a comment. A general comment containing no newlines acts like a space. Any other comment acts like a newline.
```

单行注释从字符 // 开始到这一行结束的位置

多行注释从字符 /*开始到字符*/ 结束

单行注释前面不能有符号、字符串、内嵌注释。....没看懂

### Tokens 

```bash
Tokens form the vocabulary of the Go language. There are four classes: identifiers, keywords, operators and delimiters, and literals. White space, formed from spaces (U+0020), horizontal tabs (U+0009), carriage returns (U+000D), and newlines (U+000A), is ignored except as it separates tokens that would otherwise combine into a single token. Also, a newline or end of file may trigger the insertion of a semicolon. While breaking the input into tokens, the next token is the longest sequence of characters that form a valid token.
```

Tokens 是Go语言的符号表.分为五类(原文是四类，但写了五类,应该是写错了)：标识符、关键字、操作符、分界符,(http://baike.baidu.com/view/1208327.htm)(注：[字面量是指双引号引住的一系列字符](http://baike.baidu.com/view/1208327.htm),go语言则更丰富)

更多可以看看这篇问文章介绍：[Go语言的标识符、关键字、字面量、类型](https://segmentfault.com/a/1190000002687627)

### Semicolons

```bash
The formal grammar uses semicolons ";" as terminators in a number of productions. Go programs may omit most of these semicolons using the following two rules:
```

在很多编程语言语法使用分号";"作为结束符。Go语言在满足下面两条规则下将省略绝大多数的分号。

```bash
When the input is broken into tokens, a semicolon is automatically inserted into the token stream immediately after a line's final token if that token is
an identifier
an integer, floating-point, imaginary, rune, or string literal
one of the keywords break, continue, fallthrough, or return
one of the operators and delimiters ++, --, ), ], or }
To allow complex statements to occupy a single line, a semicolon may be omitted before a closing ")" or "}".
To reflect idiomatic use, code examples in this document elide semicolons using these rules.
```

### Identifiers 标识符

> Identifiers name program entities such as variables and types. An identifier is a sequence of one or more letters and digits. The first character in an identifier must be a letter.

标志符在编程中可以是变量和类型。一个标志符是由一个或多个字母和符号组成。它的第一个字符必须是字母。

```go
identifier = letter { letter | unicode_digit } .
a
_x9
ThisVariableIsExported
αβ
```

标志符 = 字母 { 字母 | unicode字符 } .

> Some identifiers are predeclared.

有些标志符需要[预定义](08/03.md)

### Keywords 关键字

```The following keywords are reserved and may not be used as identifiers.```

下面的是保留关键字，保留关键字不能作为标志符

```
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

### Operators and Delimiters  操作符和分隔符

> The following character sequences represent operators, delimiters, and other special tokens:

下面的字符序列代表操作符、分隔符、和其他的特殊token

```go
+    &     +=    &=     &&    ==    !=    (    )
-    |     -=    |=     ||    <     <=    [    ]
*    ^     *=    ^=     <-    >     >=    {    }
/    <<    /=    <<=    ++    =     :=    ,    ;
%    >>    %=    >>=    --    !     ...   .    :
     &^          &^=
```

### Integer literals 整型字面量

> An integer literal is a sequence of digits representing an integer constant. An optional prefix sets a non-decimal base: 0 for octal, 0x or 0X for hexadecimal. In hexadecimal literals, letters a-f and A-F represent values 10 through 15.

整型字面量是一个数字序列代表整型常量.非十进制必须以0(八进制)、0x或0X(十六进制)开头。十六进制字面量中字符a-f 和A-F 代表10到15.

下面是示例：

```go
int_lit     = decimal_lit | octal_lit | hex_lit .
decimal_lit = ( "1" … "9" ) { decimal_digit } .
octal_lit   = "0" { octal_digit } .
hex_lit     = "0" ( "x" | "X" ) hex_digit { hex_digit } .
42
0600
0xBadFace
170141183460469231731687303715884105727
```

### Floating-point literals 浮点型字面量

> A floating-point literal is a decimal representation of a floating-point constant. It has an integer part, a decimal point, a fractional part, and an exponent part. The integer and fractional part comprise decimal digits; the exponent part is an e or E followed by an optionally signed decimal exponent. One of the integer part or the fractional part may be elided; one of the decimal point or the exponent may be elided.

浮点数字面量使用小数标示浮点数常数.它有整数部分，小数点、分数部分，指数部分.整数和分数部分组成十进制数字。
指数部分是一个e或E跟在一个可选的符号十进制shi ji d s fa g d d f。
整数部分或小数部分可以被省略(如:.25 或 2. )。小数点或指数可以被省略(如:2 或 1)。

```go
float_lit = decimals "." [ decimals ] [ exponent ] |
            decimals exponent |
            "." decimals [ exponent ] .
decimals  = decimal_digit { decimal_digit } .
exponent  = ( "e" | "E" ) [ "+" | "-" ] decimals .
0.
72.40
072.40  // == 72.40
2.71828
1.e+0
6.67428e-11
1E6
.25
.12345E+5
```

### 虚构文字

虚构的文字表示复数常数的虚部 。它由整数或 浮点文字组成，后跟小写字母i。虚数文字的值是相应整数或浮点文字的值乘以虚数单位i。


```go
imaginary_lit = (decimal_digits | int_lit | float_lit) "i" .
```

为了向后兼容，假想文字的整数部分完全由十进制数字（可能还有下划线）组成，即使它以Lead开头，也被视为十进制整数0。


```go
0i
0123i         // == 123i for backward-compatibility
0o123i        // == 0o123 * 1i == 83i
0xabci        // == 0xabc * 1i == 2748i
0.i
2.71828i
1.e+0i
6.67428e-11i
1E6i
.25i
.12345E+5i
0x1p-2i       // == 0x1p-2 * 1i == 0.25i
```

### 符文文字

符文文字表示符文常量，即标识Unicode代码点的整数值。符文文字表示为一个或多个用单引号引起来的字符，例如'x'或'\n'。在引号内，除换行符和未转义的单引号外，任何字符都可以出现。用单引号引起来的字符表示字符本身的Unicode值，而以反斜杠开头的多字符序列以各种格式编码值。

最简单的形式表示引号内的单个字符；由于Go源文本是用UTF-8编码的Unicode字符，因此多个UTF-8编码的字节可能表示一个整数值。例如，文字'a'包含一个代表文字aUnicode U + 0061，value的字节0x61，而 'ä'包含两个0xc3 0xa4代表文字a-dieresis U + 00E4，value的字节（）0xe4。

多个反斜杠转义符允许将任意值编码为ASCII文本。有四种方法可以将整数值表示为数字常数：\x紧随其后的是两个十六进制数字。\u紧随其后的是四个十六进制数字； \U后跟正好是八个十六进制数字，\再加上一个普通的反斜杠，后跟正好是三个八进制数字。在每种情况下，文字的值都是由相应基数中的数字表示的值。

尽管这些表示均产生整数，但它们具有不同的有效范围。八进制转义符必须表示介于0和255之间（包括0和255）的值。十六进制转义符通过构造满足此条件。转义符 表示Unicode代码点，因此其中的某些值是非法的，尤其是上面的值\u和\U代表0x10FFFF一半的值。

反斜杠后，某些单字符转义符表示特殊值：

```go
\a  U+0007  警报或铃声
\b  U+0008  退格键
\f  U+000C  换页
\n  U+000A  换行符或换行符
\r  U+000D  回车
\t  U+0009  水平制表符
\v  U+000b  垂直制表符
\\  U+005c  反斜杠
\'  U+0027  单引号（仅在符文文字内有效转义）
\"  U+0022  双引号（仅在字符串文字中有效的转义）
```

在符文文字中，所有其他以反斜杠开头的序列都是非法的。

```go
rune_lit         = "'" ( unicode_value | byte_value ) "'" .
unicode_value    = unicode_char | little_u_value | big_u_value | escaped_char .
byte_value       = octal_byte_value | hex_byte_value .
octal_byte_value = `\` octal_digit octal_digit octal_digit .
hex_byte_value   = `\` "x" hex_digit hex_digit .
little_u_value   = `\` "u" hex_digit hex_digit hex_digit hex_digit .
big_u_value      = `\` "U" hex_digit hex_digit hex_digit hex_digit
                           hex_digit hex_digit hex_digit hex_digit .
escaped_char     = `\` ( "a" | "b" | "f" | "n" | "r" | "t" | "v" | `\` | "'" | `"` ) .
```

```go
'a'
'ä'
'本'
'\t'
'\000'
'\007'
'\377'
'\x07'
'\xff'
'\u12e4'
'\U00101234'
'\''         // rune literal containing single quote character
'aa'         // illegal: too many characters
'\xa'        // illegal: too few hexadecimal digits
'\0'         // illegal: too few octal digits
'\uDFFF'     // illegal: surrogate half
'\U00110000' // illegal: invalid Unicode code point
```

### 字符串文字

字符串文字表示 通过连接一系列字符获得的字符串常量。有两种形式：原始字符串文字和解释的字符串文字。

原始字符串文字是反引号之间的字符序列，如中所示 `foo`。在引号内，除反引号外，任何字符都可能出现。原始字符串文字的值是由引号之间未解释（隐式为UTF-8编码）字符组成的字符串。特别是，反斜杠没有特殊含义，并且字符串可能包含换行符。原始字符串文字中的回车符（'\ r'）从原始字符串值中丢弃。

解释的字符串文字是双引号之间的字符序列，如中所示"bar"。在引号内，除换行符和未转义的双引号外，任何字符都可能出现。引号之间的文本构成文字的值，反斜杠转义与符文文字中的解释相同（除非\'是非法且 \"合法），并具有相同的限制。三位八进制（\nnn）和两位十六进制（\xnn）转义符表示所得字符串的各个 字节；所有其他转义代表单个字符的（可能是多字节）UTF-8编码。因此，在字符串文字内\377并\xFF表示值的单个字节0xFF= 255，而ÿ， \u00FF，\U000000FF和\xc3\xbf代表两个字节0xc3 0xbf字符U + 00FF的UTF-8编码的。

```go
string_lit             = raw_string_lit | interpreted_string_lit .
raw_string_lit         = "`" { unicode_char | newline } "`" .
interpreted_string_lit = `"` { unicode_value | byte_value } `"` .
```

```go
`abc`                // same as "abc"
`\n
\n`                  // same as "\\n\n\\n"
"\n"
"\""                 // same as `"`
"Hello, world!\n"
"日本語"
"\u65e5本\U00008a9e"
"\xff\u00FF"
"\uD800"             // illegal: surrogate half
"\U00110000"         // illegal: invalid Unicode code point
```

这些示例都表示相同的字符串：

```go
"日本語"                                 // UTF-8 input text
`日本語`                                 // UTF-8 input text as a raw literal
"\u65e5\u672c\u8a9e"                    // the explicit Unicode code points
"\U000065e5\U0000672c\U00008a9e"        // the explicit Unicode code points
"\xe6\x97\xa5\xe6\x9c\xac\xe8\xaa\x9e"  // the explicit UTF-8 bytes
```

如果源代码将字符表示为两个代码点，例如包含重音符号和字母的组合形式，则如果将其放置在符文文字中（不是单个代码点），结果将为错误，并且显示为如果放在字符串文字中，则两个代码点。

## 常数

有布尔常量， 符文常量， 整数常量， 浮点常量，复数常量和字符串常量。符文，整数，浮点数和复数常量统称为数字常量。

常数值由 符文， 整数， 浮点数， 虚数或 字符串文字表示，标识符表示常数，常数表达式，结果为常数的转换或某些内置结果的值功能，例如 unsafe.Sizeof应用到任何值， cap或len施加到 一些表达式， real并且imag施加到一个复常数和complex施加到数字常数。布尔真值由预先声明的常数 true和表示false。预声明的标识符 iota 表示整数常数。

通常，复数常量是常量表达式的一种形式， 并将在该部分中进行讨论。

数字常数表示任意精度的精确值，并且不会溢出。因此，没有常量表示IEEE-754的负零，无穷大和非数字值。

常量可以是类型化或无类型。字面常数true，false，iota，和某些常量表达式 仅包含无类型常量操作数无类型。

常量可以通过常量声明 或转换来显式指定类型，也可以在变量声明或 赋值中使用，或在表达式中用作操作数时隐 式地给定类型。如果常量值不能表示为相应类型的值，则会出错。

未类型的常量具有默认类型，该默认类型是在需要使用类型值的上下文中（例如在简短的变量声明中， 例如i := 0在没有显式类型的情况下）将常量隐式转换为该类型的类型。一个无类型恒定的默认类型是bool，rune， int，float64，complex128或string 分别，这取决于它是否是一个布尔值，符，整数，浮点，复杂，或字符串常量。

实现限制：尽管数字常量在语言中具有任意精度，但编译器可能会使用内部表示形式来实现它们，而精度有限。也就是说，每个实现都必须：

- 用至少256位表示整数常量。
- 表示浮点常数，包括复数常数的部分，尾数至少为256位，带符号的二进制指数至少为16位。
- 如果无法精确表示整数常数，则给出错误。
- 如果由于溢出而无法表示浮点数或复数常量，则给出错误。
- 如果由于精度限制而无法表示浮点数或复数常数，则四舍五入到最接近的可表示常数。
这些要求既适用于文字常量，也适用于评估常量表达式的结果。

## 变量

变量是用于保存值的存储位置。允许值的集合由变量的类型决定。

甲变量声明 或者，对于功能参数和结果，一个的签名函数声明 或功能字面储备存储名为变量。调用内置函数new 或获取复合文字的地址会 在运行时为变量分配存储空间。此类匿名变量是通过（可能是隐式的） 指针间接指向的。

array，slice和struct类型的结构化变量具有可以单独处理的元素和字段。每个这样的元素就像一个变量。

变量 的静态类型（或只是type）是其声明中给出的类型，new调用或复合文字中提供的类型 或结构化变量的元素的类型。接口类型的变量还具有独特的动态类型，即在运行时分配给变量的值的具体类型（除非该值是预先声明的标识符nil，没有类型）。动态类型在执行期间可能会有所不同，但是存储在接口变量中的值始终可分配 给变量的静态类型。

```go
var x interface {}  // x为nil并且具有静态类型的interface {}
var v * T           // v的值为nil，静态类型* T
x = 42              // x具有值42和动态类型int
x = v               // x具有值（* T）（nil）和动态类型* T
```

通过引用表达式中的变量来检索变量的值 ；它是分配给变量的最新值 。如果尚未为变量分配值，则其值为该类型的 零值。

## 类型

类型确定一组值以及特定于这些值的操作和方法。一个类型可以用一个类型名来表示，如果有一个，或者可以使用类型文字来指定，该类型文字由现有类型组成。

```go
Type      = TypeName | TypeLit | "(" Type ")" .
TypeName  = identifier | QualifiedIdent .
TypeLit   = ArrayType | StructType | PointerType | FunctionType | InterfaceType | SliceType | MapType | ChannelType .
```

该语言预先声明了某些类型名称。其他类型则带有类型声明。 可以使用类型文字来构造复合类型（数组，结构，指针，函数，接口，切片，映射和通道类型）。

每个类型T都有一个基础类型：如果T 是预先声明的布尔，数字或字符串类型之一，或者是类型文字，则相应的基础类型就是T它自己。否则，T其基础类型T是其类型声明中引用 的类型的基础类型。

```go
type (
    A1 = string
    A2 = A1
)

type (
    B1 string
    B2 B1
    B3 []B1
    B4 B3
)
```

基础类型的string，A1，A2，B1，和B2是string。基础类型的[]B1，B3和B4是[]B1。

### 方法集

类型可以具有与之关联的方法集。接口类型的方法集是其接口。任何其他类型的方法集都T包含用接收器类型声明的所有 方法T。对应的指针类型 的方法集 *T 是用 Receiver*T 或声明的所有方法的集T （也就是说，它也包含的方法集T）。进一步的规则适用于包含嵌入式字段的结构，如有关struct type的部分中所述。其他任何类型的方法集都为空。在方法集中，每个方法必须具有 唯一的 非空白 方法名称。

类型的方法集确定该类型实现的接口 以及可以 使用该类型的接收器调用的方法。

### 布尔类型

甲布尔类型表示该组由义的常量来表示布尔真值的true 和false。预声明的布尔类型为bool; 它是定义的类型。

### 数值类型

甲数字类型表示集整数或浮点值的。预先声明的与体系结构无关的数字类型为：

```GO
uint8       所有无符号8位整数（0到255）的集合
uint16      所有无符号16位整数（0到65535）的集合
uint32      所有无符号32位整数（0到4294967295）的集合
uint64      所有无符号64位整数的集合（0到18446744073709551615）

int8        所有有符号8位整数（-128至127）的集合
int16       所有有符号16位整数（-32768至32767）的集合
int32       所有有符号32位整数的集合（-2147483648至2147483647）
int64       所有带符号的64位整数的集合（-9223372036854775808至9223372036854775807）

float32     所有IEEE-754 32位浮点数的集合
float64     所有IEEE-754 64位浮点数的集合

complex64   具有float32实部和虚部的所有复数的集合
complex128  包含float64实部和虚部的所有复数的集合
byte        uint8的别名
rune        int32的别名
```

n位整数 的值是n位宽，并使用二进制补码表示 。

还有一组预先声明的数字类型，具有特定于实现的大小：

```GO
uint    32或64位
int     与uint相同的大小
uintptr 一个无符号整数，其大小足以存储指针值的未解释位
```

为了避免可移植性问题所有数值类型被定义的类型，因此，除了不同 byte，这是一个别名为uint8和 rune，其是用于一个别名int32。在表达式或赋值中混合使用不同的数字类型时，需要进行显式转换。例如，int32而int 不是即使他们可能对特定的架构相同尺寸相同的类型。

### 字符串类型

甲字符串类型表示组字符串值。字符串值是一个字节序列（可能为空）。字节数称为字符串的长度，从不为负。字符串是不可变的：一旦创建，就无法更改字符串的内容。预声明的字符串类型为string; 它是定义的类型。

s可以使用内置函数发现 字符串的长度len。如果字符串是常量，则长度是编译时常量。字符串的字节可以由 0到0 的整数索引访问len(s)-1。引用此类元素的地址是非法的；如果 s[i]为i字符串的第'个字节，&s[i]则无效。

### 数组类型

数组是单一类型元素的编号序列，称为元素类型。元素的数量称为数组的长度，绝不能为负。

```go
ArrayType   = "[" ArrayLength "]" ElementType .
ArrayLength = Expression .
ElementType = Type .
```

长度是数组类型的一部分；它必须求值为一个type可以表示的非负常数 。数组的长度可以使用内置函数发现。可以通过从 0到0 的整数索引来寻址这些元素。数组类型始终是一维的，但可以组成多维类型。 intalenlen(a)-1

```go
[32]byte
[2*N] struct { x, y int32 }
[1000]*float64
[3][5]int
[2][2][2]float64  // same as [2]([2]([2]float64))
```

### 切片类型

切片是基础数组连续段的描述符，并提供对该数组中元素编号序列的访问。切片类型表示其元素类型的所有数组切片的集合。元素的数量称为切片的长度，绝不为负。未初始化的片的值为nil。

```go
SliceType = "[" "]" ElementType .
```

切片的长度s可以通过内置函数发现 len；与数组不同，它可能在执行期间发生变化。可以通过从 0到0 的整数索引来寻址这些元素len(s)-1。给定元素的切片索引可能小于基础数组中相同元素的索引。

切片一旦初始化，便始终与包含其元素的基础数组关联。因此，一个片与其阵列以及同一阵列的其他片共享存储。相反，不同的数组始终代表不同的存储。

切片下面的数组可能会延伸到切片的末尾。的容量是该程度的量度：它是切片并超出片的阵列的长度的长度的总和; 可以通过从原始切片中切片一个新切片来创建最大长度 的切片。a可以使用内置函数发现切片的容量cap(a)。

T使用内置函数创建 给定元素类型的新的初始化切片值，该函数 make采用切片类型和参数来指定长度和容量（可选）。使用创建的切片make始终分配一个新的隐藏数组，返回的切片值将引用该数组。也就是说，执行

```go
make([]T, length, capacity)
```

产生与分配数组和切片相同的切片 ，因此这两个表达式是等效的：

```go
make([]int, 50, 100)
new([100]int)[0:50]
```

像数组一样，切片始终是一维的，但可以组成更高维度的对象。对于数组数组，内部数组通过构造始终具有相同的长度；但是，对于切片的切片（或切片的阵列），内部长度可能会动态变化。此外，内部切片必须单独初始化。

### 结构类型

结构是一系列命名元素（称为字段），每个元素都有一个名称和一个类型。字段名称可以显式指定（IdentifierList）或隐式指定（EmbeddedField）。在结构中，非空白字段名称必须是唯一的。

```go
StructType    = "struct" "{" { FieldDecl ";" } "}" .
FieldDecl     = (IdentifierList Type | EmbeddedField) [ Tag ] .
EmbeddedField = [ "*" ] TypeName .
Tag           = string_lit .
```

```go
//一个空结构。
struct {}

//具有6个字段的结构。
struct {
    x, y int
    u float32
    _ float32  // padding
    A *[]int
    F func()
}
```

用类型声明但没有显式字段名称的字段称为嵌入式字段。必须将嵌入式字段指定为类型名称T或指向非接口类型名称的指针*T，并且T其本身不能为指针类型。非限定类型名称充当字段名称。

```go
//具有四个嵌入式字段的结构，类型为T1，* T2，P.T3和* P.T4
struct {
    T1        // field name is T1
    *T2       // field name is T2
    P.T3      // field name is T3
    *P.T4     // field name is T4
    x, y int  // field names are x and y
}
```

以下声明是非法的，因为字段名称在结构类型中必须是唯一的：

```go
struct {
    T     // conflicts with embedded field *T and *P.T
    *T    // conflicts with embedded field T and *P.T
    *P.T  // conflicts with embedded field T and *T
}
```

如果是表示该字段或方法的合法选择器，则 结构中嵌入字段的 字段或方法 称为提升。 fxx.ff

提升的字段的作用类似于结构的普通字段，只是它们不能用作结构的复合文字中的字段名称 。

给定一个结构类型S和一个定义的类型 T，提升的方法包括在该结构的方法集中，如下所示：

- 如果S包含嵌入字段T，该方法集的S 和*S两者都包括推进与接收器的方法 T。方法集*S还包括带有接收方的提升方法*T。
- 如果S包含一个嵌入式字段*T，则S和的方法集*S都包括带有接收者T或的 升级方法*T。
字段声明后可以跟一个可选的字符串文字标签，该标签成为相应字段声明中所有字段的属性。空标签字符串等同于缺少标签。这些标记通过反射接口可见， 并参与结构的类型标识，但在其他情况下将被忽略。

```go
struct {
    x, y float64 ""  //空标签字符串就像没有标签
    name string  "any string is permitted as a tag"
    _    [4]byte "ceci n'est pas un champ de structure"
}

//与TimeStamp协议缓冲区相对应的结构。
//标签字符串定义协议缓冲区字段号；
//它们遵循反射包概述的约定。
struct {
    microsec  uint64 `protobuf:"1"`
    serverIP6 uint64 `protobuf:"2"`
}
```

### 指针类型

指针类型表示指向给定类型的变量的所有指针的集合，称为指针的基本类型。未初始化的指针的值为nil。

```go
PointerType = "*" BaseType .
BaseType    = Type .
```

```go
*Point
*[4]int
```

### 功能类型

函数类型表示具有相同参数和结果类型的所有函数的集合。函数类型的未初始化变量的值为nil。

```go
FunctionType   = "func" Signature .
Signature      = Parameters [ Result ] .
Result         = Parameters | Type .
Parameters     = "(" [ ParameterList [ "," ] ] ")" .
ParameterList  = ParameterDecl { "," ParameterDecl } .
ParameterDecl  = [ IdentifierList ] [ "..." ] Type .
```

在参数或结果列表中，名称（IdentifierList）必须全部存在或全部不存在。如果存在，则每个名称代表指定类型的一项（参数或结果），并且签名中的所有非空白名称必须唯一。如果不存在，则每种类型代表该类型的一项。参数和结果列表始终带有括号，但如果有一个未命名的结果，则可以将其写为非括号类型。

函数签名中的最终传入参数可以具有以开头的类型...。具有此类参数的函数称为可变参数，可以使用该参数的零个或多个参数来调用。

```go
func()
func(x int) int
func(a, _ int, z float32) bool
func(a, b int, z float32) (bool)
func(prefix string, values ...int)
func(a, b int, z float64, opt ...interface{}) (success bool)
func(int, int, float64) (float64, *[]int)
func(n int) func(p *T)
```

### 接口类型

接口类型指定一个称为其接口的方法集。接口类型的变量可以使用方法集合存储任何类型的值，该方法集是接口的任何超集。据说这种类型 实现了接口。接口类型的未初始化变量的值为。 nil

```go
InterfaceType      = "interface" "{" { ( MethodSpec | InterfaceTypeName ) ";" } "}" .
MethodSpec         = MethodName Signature .
MethodName         = identifier .
InterfaceTypeName  = TypeName .
```

接口类型可以通过方法规范显式指定方法，也可以通过接口类型名称嵌入其他接口的方法。

```go
//一个简单的File界面。
interface {
    Read([]byte) (int, error)
    Write([]byte) (int, error)
    Close() error
}
```

每个显式指定的方法的名称必须唯一 且不能为空。

```go
interface {
    String() string
    String() string  // illegal: String not unique
    _(x int)         // illegal: method must have non-blank name
}
```

可以实现一种以上的接口。例如，如果有两种类型的S1并S2 设置了方法

```go
func (p T) Read(p []byte) (n int, err error)
func (p T) Write(p []byte) (n int, err error)
func (p T) Close() error
```

（其中，T代表任一S1或S2），则File接口由两个来实现S1和 S2，无论什么其它方法 S1和S2可以具有或共享。

类型实现包含其方法的任何子集的任何接口，因此可以实现几个不同的接口。例如，所有类型都实现空接口：

```go
interface{}
```

同样，请考虑此接口规范，该规范出现在类型声明中， 以定义称为的接口Locker：

```go
type Locker interface {
    Lock()
    Unlock()
}
```

如果S1和S2还实现

```go
func (p T) Lock() { … }
func (p T) Unlock() { … }
```

他们既实现Locker接口又实现File接口。

接口T可以使用（可能是合格的）接口类型名称E来代替方法规范。这就是所谓的 嵌入接口E在T。该方法集的T是工会 的方法设置的T的显式声明的方法与 T的嵌入式接口。

```go
type Reader interface {
    Read(p []byte) (n int, err error)
    Close() error
}

type Writer interface {
    Write(p []byte) (n int, err error)
    Close() error
}

// ReadWriter的方法为Read，Write和Close。
type ReadWriter interface {
    Reader //在ReadWriter的方法集中包含Reader的方法
    Writer //在ReadWriter的方法集中包含Writer的方法
}
```

甲联合的方法集包含每个方法集的（出口和非导出）的方法正好一次，并用方法 相同的名称必须具有相同的签名。

```go
type ReadCloser interface {
    Reader   // includes methods of Reader in ReadCloser's method set
    Close()  // illegal: signatures of Reader.Close and Close are different
}
```

接口类型T不能T递归嵌入自己或嵌入的任何接口类型。

//非法：错误无法嵌入自身
type Bad interface {
    Bad
}

//非法：Bad1无法使用Bad2嵌入自身
type Bad1 interface {
    Bad2
}
type Bad2 interface {
    Bad1
}

### 地图类型

映射是一种无序的一组元素，称为元素类型，由一组另一种类型的唯一键（称为键类型）索引。未初始化映射的值为nil。

```go
MapType     = "map" "[" KeyType "]" ElementType .
KeyType     = Type .
```

的比较运算符 ==和!=必须为键类型的操作数被完全定义; 因此，键类型不能为函数，映射或切片。如果键类型是接口类型，则必须为动态键值定义这些比较运算符。失败将导致运行时恐慌。

```go
map[string]int
map[*T]struct{ x, y float64 }
map[string]interface{}
```

地图元素的数量称为其长度。对于地图m，可以使用内置函数来发现它，len 并且在执行过程中可能会更改。可以在执行期间使用赋值添加元素，并使用索引表达式进行检索 ; 可以使用delete内置功能将其删除 。

使用内置函数创建一个新的空地图值make，该函数将地图类型和可选的容量提示作为参数：

```go
make(map[string]int)
make(map[string]int, 100)
```

初始容量不受其大小限制：地图会增长以容纳其中存储的项目数（nil地图除外）。甲nil地图相当于不同之处在于可以添加没有元素的空映射。

## Channel types 频道类型

通道提供了一种机制，用于 通过发送和 接收 指定元素类型的值来 同时执行功能以进行通信 。未初始化的通道的值为。 nil

```go
ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) ElementType .
```

可选的<-运算符指定通道方向，即 发送还是接收。如果没有给出方向，则该信道是 双向的。通道只能通过分配或显式转换来限制发送或仅接收 。

```go
chan T          //可用于发送和接收T类型的值
chan <-float64  //仅可用于发送float64
<-chan int      //仅可用于接收int
```

该<-运营商与带最左边的chan 可能：

```go
chan<- chan int    // same as chan<- (chan int)
chan<- <-chan int  // same as chan<- (<-chan int)
<-chan <-chan int  // same as <-chan (<-chan int)
chan (<-chan int)
```

可以使用内置函数来创建新的初始化通道值make，该函数 将通道类型和可选容量作为参数：

```go
make(chan int, 100)
```

容量（以元素数为单位）设置通道中缓冲区的大小。如果容量为零或不存在，则通道将没有缓冲，并且仅在发送方和接收方都准备就绪时通信才能成功。否则，如果缓冲区未满（发送）或不为空（接收），则通道将被缓冲，并且通信将成功进行而不会阻塞。甲nil信道是从未准备好进行通信。

可以使用内置功能关闭通道 close。接收操作符的多值分配形式 报告在关闭通道之前是否发送了接收值。

单个信道可以在被用于 发送的语句， 接收操作，并调用内置的功能 cap和 len 通过任何数量的够程不经进一步的同步。通道充当先进先出队列。例如，如果一个goroutine在通道上发送值，而第二个goroutine接收到它们，则按发送顺序接收值。

## 类型和值的属性

### 类型识别

两种类型相同或不同。

甲定义的类型是从任何其他类型总是不同的。否则，如果两个类型的基础类型文字在结构上等效，则它们是相同的；否则，两个类型相同。也就是说，它们具有相同的文字结构，而相应的组件具有相同的类型。详细地：

- 如果两个数组类型具有相同的元素类型和相同的数组长度，则它们是相同的。
- 如果两个切片类型具有相同的元素类型，则它们是相同的。
- 如果两个结构类型具有相同的字段序列，并且对应的字段具有相同的名称，相同的类型和相同的标记，则它们是相同的。 来自不同软件包的未导出字段名称始终是不同的。
- 如果两个指针类型具有相同的基本类型，则它们是相同的。
- 如果两个函数类型具有相同数量的参数和结果值，相应的参数和结果类型相同，并且两个函数都是可变参数，或者两个都不相同。参数名和结果名不需要匹配。
- 如果两个接口类型具有相同的名称，功能类型相同的方法集，则它们是相同的。 来自不同程序包的未导出方法名称始终是不同的。方法的顺序无关紧要。
- 如果两个映射类型具有相同的键和元素类型，则它们是相同的。
- 如果两个通道类型具有相同的元素类型和相同的方向，则它们是相同的。
给出声明

```go
type (
    A0 = []string
    A1 = A0
    A2 = struct{ a, b int }
    A3 = int
    A4 = func(A3, float64) *A0
    A5 = func(x int, _ float64) *[]string
)

type (
    B0 A0
    B1 []string
    B2 struct{ a, b int }
    B3 struct{ a, c int }
    B4 func(int, float64) *B0
    B5 func(x int, y float64) *A1
)

type    C0 = B0
```

这些类型是相同的：

```go
A0, A1, and []string
A2 and struct{ a, b int }
A3 and int
A4, func(int, float64) *[]string, and A5

B0 and C0
[]int and []int
struct{ a, b *T5 } and struct{ a, b *T5 }
func(x int, y float64) *[]string, func(int, float64) (result *[]string), and A5
```

B0并且B1之所以不同是因为它们是由不同的类型定义创建的新类型; func(int, float64) *B0并且func(x int, y float64) *[]string 有所不同，因为B0与有所不同[]string。

### 可分配性

的值x是分配给可变型的T （“ x是分配给T”），如果满足下列条件中的一个适用：

- x的类型与相同T。
- x的类型，V并且T具有相同的 基础类型，并且至少是定义的类型之一V 或T不是定义的类型。
- T是接口类型并 x 实现 T。
- x是双向通道值，T是通道类型， x的类型，V并且T具有相同的元素类型，并且至少是定义的类型之一V或T不是定义的类型。
- x是预先声明的标识符，nil并且T 是指针，函数，切片，映射，通道或接口类型。
- x是 可由type值表示的无类型常量 。 T

### 代表性

甲常数 x是可表示 由类型的值T，如果满足下列条件中的一个适用：

- x在设定值的确定通过T。
- T是浮点类型，x可以四舍五入为T的精度而不会溢出。舍入使用IEEE 754舍入到偶数规则，但IEEE负零进一步简化为无符号零。请注意，常数永远不会导致IEEE负零，NaN或无穷大。
- T是一个复杂类型，以及x的 组件， real(x)并且imag(x) 可以通过T的组件类型（float32或 float64）的值表示。

```go
x                   T           x is representable by a value of T because

'a'                 byte        97 is in the set of byte values
97                  rune        rune is an alias for int32, and 97 is in the set of 32-bit integers
"foo"               string      "foo" is in the set of string values
1024                int16       1024 is in the set of 16-bit integers
42.0                byte        42 is in the set of unsigned 8-bit integers
1e10                uint64      10000000000 is in the set of unsigned 64-bit integers
2.718281828459045   float32     2.718281828459045 rounds to 2.7182817 which is in the set of float32 values
-1e-1000            float64     -1e-1000 rounds to IEEE -0.0 which is further simplified to 0.0
0i                  int         0 is an integer value
(42 + 0i)           float32     42.0 (with zero imaginary part) is in the set of float32 values
```

```go
x                   T           x is not representable by a value of T because

0                   bool        0 is not in the set of boolean values
'a'                 string      'a' is a rune, it is not in the set of string values
1024                byte        1024 is not in the set of unsigned 8-bit integers
-1                  uint16      -1 is not in the set of unsigned 16-bit integers
1.1                 int         1.1 is not an integer value
42i                 float32     (0 + 42i) is not in the set of float32 values
1e1000              float64     1e1000 overflows to IEEE +Inf after rounding
```

## Blocks 积木

块是声明和说明的内匹配的括号括号一个可能是空的序列。

```go
Block = "{" StatementList "}" .
StatementList = { Statement ";" } .
```

除了源代码中的显式块，还有隐式块：

1. 在通用块包含所有Go源文本。
2. 每个包都有一个包块，其中包含该包的所有Go源文本。
3. 每个文件都有一个文件块，其中包含该文件中的所有Go源文本。
4. 每个“ if”， “ for”和 “ switch” 语句都被视为在其自己的隐式块中。
5. “ switch” 或“ select”语句中的每个子句都充当隐式块。
阻止嵌套并影响作用域。

## 声明和范围

甲声明结合的非空白标识符与 恒定， 类型， 变量， 函数， 标签，或 封装。程序中的每个标识符都必须声明。不能在同一块中声明两次标识符，也不能在文件和包块中声明任何标识符。

的空白标识符可用于像在声明的任何其它标识符，但不引入结合并因此没有被声明。在package块中，标识符init只能用于 init函数声明，并且像空白标识符一样，它不会引入新的绑定。

声明    = ConstDecl | TypeDecl | VarDecl。
TopLevelDecl   = 声明 | FunctionDecl | MethodDecl。
的范围的声明标识符是其中标识表示指定的常数，类型，变量，函数，标签或包源文本的程度。

Go的词法作用域是使用block：

预定义标识符的范围是Universe块。
表示在顶层（任何函数之外）声明的常量，类型，变量或函数（但不是方法）的标识符范围是package块。
导入的软件包的软件包名称的范围是包含导入声明的文件的文件块。
表示方法接收器，函数参数或结果变量的标识符的范围是函数主体。
在函数内部声明的常量或变量标识符的范围始于ConstSpec或VarSpec的末尾（对于简短变量声明，为ShortVarDecl），并终止于最里面的包含块的末尾。
在函数内部声明的类型标识符的范围从TypeSpec中的标识符开始，到最里面的包含块的结尾结束。
可以在内部块中重新声明在块中声明的标识符。内部声明的标识符在范围内，它表示内部声明声明的实体。

所述包子句是不声明; 程序包名称不会出现在任何范围内。其目的是识别属于同一软件包的文件，并为导入声明指定默认的软件包名称。

标签范围
标签由带标签的语句声明，并在“ break”， “ continue”和 “ goto”语句中使用。定义从未使用过的标签是非法的。与其他标识符相反，标签不是块作用域的，并且不会与不是标签的标识符冲突。标签的范围是在其中声明标签的函数的主体，并且不包括任何嵌套函数的主体。

空白标识符
的空白标识符由下划线表示_。它用作匿名占位符，而不是常规（非空白）标识符，并且在声明，操作数和赋值中具有特殊含义。

预声明的标识符
以下标识符在Universe块中隐式声明 ：

类型：
    布尔字节complex64 complex128错误float32 float64
    int int8 int16 int32 int64符文字符串
    uint uint8 uint16 uint32 uint64 uintptr

常数：
    真假IOTA

零值：
    零

功能：
    追加盖帽关闭复杂副本删除图像
    使新的紧急打印println真正恢复
导出的标识符
标识符可以被导出以允许从另一个包访问它。如果同时满足以下条件，则导出标识符：

标识符名称的第一个字符是Unicode大写字母（Unicode类“ Lu”）；和
标识符在包块中声明， 或者是字段名或 方法名。
所有其他标识符都不会导出。

标识符的唯一性
给定一组标识符，标识符被称为独特的，如果它是 不同从一组每隔。如果两个标识符的拼写不同，或者它们出现在不同的程序包中且未 导出，则它们是不同的。否则，它们是相同的。

常量声明
常量声明将标识符列表（常量名称）绑定到常量表达式列表的值。标识符的数量必须等于表达式的数量，并且左边的第n个标识符绑定到右边的第n个表达式的值。

ConstDecl       =“ const”（ConstSpec |“（” { ConstSpec “;”}“）”）。
ConstSpec       = IdentifierList [[ 类型 ]“ =” ExpressionList ]。

IdentifierList = 标识符 {“，” 标识符 }。
ExpressionList = 表达式 {“，” 表达式 }。
如果存在类型，则所有常量都采用指定的类型，并且表达式必须可分配给该类型。如果省略类型，则常量采用相应表达式的各个类型。如果表达式值是未类型化的常量，则声明的常量将保持未类型化，并且常量标识符表示常量值。例如，如果表达式是浮点常量，则常量标识符表示浮点常量，即使常量的小数部分为零也是如此。

const Pi float64 = 3.14159265358979323846
const zero = 0.0 //无类型的浮点常量
const（
    大小int64 = 1024
    eof = -1 //无类型的整数常量
）
const a，b，c = 3，4，“ foo” // a = 3，b = 4，c =“ foo”，无类型整数和字符串常量
const u，v float32 = 0，3 // u = 0.0，v = 3.0
在带括号的const声明列表中，表达式列表可以从第一个ConstSpec中省略。这样的空列表等效于第一个前面的非空表达式列表及其类型（如果有）的文本替换。因此，省略表达式列表等同于重复前面的列表。标识符的数量必须等于上一个列表中的表达式的数量。与iota常量生成器一起， 此机制允许轻量级声明连续值：

const（
    星期日=艾奥塔
    星期一
    星期二
    星期三
    星期四
    星期五
    聚会日
    numberOfDays //不导出此常量
）
井田
在常量声明中，预声明的标识符 iota表示连续的无类型整数 常量。它的值是该 常量声明中相应ConstSpec的索引，从零开始。它可以用来构造一组相关的常量：

const（
    c0 = iota // c0 == 0
    c1 = iota // c1 == 1
    c2 = iota // c2 == 2
）

const（
    a = 1 << iota // a == 1（iota == 0）
    b = 1 << iota // b == 2（iota == 1）
    c = 3 // c == 3（iota == 2，未使用）
    d = 1 << iota // d == 8（iota == 3）
）

const（
    u = iota * 42 // u == 0（无类型整数常量）
    v float64 = iota * 42 // v == 42.0（float64常数）
    w = iota * 42 // w == 84（无类型整数常量）
）

const x = iota // x == 0
const y = iota // y == 0
根据定义，iota在同一ConstSpec中的多次使用都具有相同的值：

const（
    bit0，mask0 = 1 << iota，1 << iota-1 // bit0 == 1，mask0 == 0（iota == 0）
    bit1，mask1 // // bit1 == 2，mask1 == 1（iota == 1）
    _，_ //（iota == 2，未使用）
    bit3，mask3 // // bit3 == 8，mask3 == 7（iota == 3）
）
最后一个示例利用了 最后一个非空表达式列表的隐式重复。

类型声明
类型声明将标识符（类型名称）绑定到type。类型声明有两种形式：别名声明和类型定义。

TypeDecl = “类型”（类型指定 | “（”{ 类型指定 “;”} “）”）。
类型指定 = AliasDecl | TypeDef。
别名声明
别名声明将标识符绑定到给定的类型。

AliasDecl = 标识符 “ =” 类型。
在标识符的范围内，它用作类型的别名。

类型（
    nodeList = [] * Node // nodeList和[] * Node是相同的类型
    Polar =极地//极和极表示相同的类型
）
类型定义
类型定义使用与给定类型相同的基础类型和操作创建一个新的独特 类型，并将标识符绑定到该类型。

TypeDef = 标识符 Type。
新类型称为定义类型。它不同于任何其他类型，包括创建它的类型。

类型（
    点struct {x，y float64} //点和struct {x，y float64}是不同的类型
    极点//极点和Point表示不同的类型
）

输入TreeNode struct {
    左，右* TreeNode
    值*可比
}

类型Block接口{
    BlockSize（）int
    加密（src，dst [] byte）
    解密（src，dst [] byte）
}
定义的类型可能具有与之关联的方法。它不继承绑定到给定类型的任何方法，但是 接口类型或复合类型的元素的方法集保持不变：

// Mutex是一种具有两种方法的数据类型，即Lock和Unlock。
输入Mutex struct {/ * Mutex字段* /}
func（m * Mutex）Lock（）{/ *锁定实现* /}
func（m * Mutex）Unlock（）{/ *解锁实现* /}

// NewMutex具有与Mutex相同的组成，但其方法集为空。
键入NewMutex Mutex

// PtrMutex的基础类型* Mutex的方法集保持不变，
//，但PtrMutex的方法集为空。
类型PtrMutex * Mutex

// * PrintableMutex的方法集包含方法
//锁定和解锁绑定到其嵌入式字段Mutex。
输入PrintableMutex struct {
    互斥体
}

// MyBlock是一种接口类型，其设置方法与Block相同。
输入MyBlock Block
类型定义可用于定义不同的布尔，数字或字符串类型，并将方法与它们相关联：

键入TimeZone int

const（
    EST时区=-（5 + iota）
    科技委
    MST
    太平洋标准时间
）

func（tz TimeZone）String（）字符串{
    返回fmt.Sprintf（“ GMT％+ dh”，tz）
}
变量声明
变量声明创建一个或多个变量，将相应的标识符绑定到它们，并为每个变量赋予一个类型和一个初始值。

VarDecl      =“ var”（VarSpec |“（” { VarSpec “;”}“）”）。
VarSpec      = IdentifierList（类型 [“ =” ExpressionList ] |“ =” ExpressionList）。
var i int
var U，V，W float64
var k = 0
var x，y float32 = -1，-2
var（
    我诠释
    u，v，s = 2.0，3.0，“ bar”
）
var re，im = complexSqrt（-1）
var _，found = entry [name] //地图查找；只对“发现”感兴趣
如果给出了一个表达式列表，则使用赋值规则中的表达式对变量进行初始化。否则，将每个变量初始化为其零值。

如果存在类型，则为每个变量指定该类型。否则，将为每个变量分配赋值的相应初始化值的类型。如果该值是未类型化的常量，则首先将其隐式 转换为其默认类型；如果它是无类型的布尔值，则首先将其隐式转换为type bool。预声明的值nil不能用于初始化没有显式类型的变量。

var d = math.Sin（0.5）// d为float64
var i = 42 //我是int
var t，ok = x。（T）// t是T，ok是布尔
var n = nil //非法
实现限制：如果从不使用变量，则编译器可能会在函数体内声明变量为非法。

简短的变量声明
一个短变量声明使用语法：

ShortVarDecl = IdentifierList “：=” ExpressionList。
它是 带有初始化表达式但没有类型的常规变量声明的简写：

“ var” IdentifierList = ExpressionList。
i，j：= 0，10
f：= func（）int {return 7}
ch：= make（chan int）
r，w，_：= os.Pipe（）// os.Pipe（）返回一对连接的文件和一个错误（如果有）
_，y，_：= coord（p）// coord（）返回三个值；只对y坐标感兴趣
与常规变量声明不同，短变量声明可以重新声明 变量，前提是它们最初是在相同类型的同一个块中（如果该块是函数体，则在参数列表中）早先声明过，并且具有至少一个非空变量是新的。因此，重新声明只能出现在多变量简短声明中。重新声明不会引入新的变量；它只是为原始值分配一个新值。

field1，偏移量：= nextField（str，0）
field2，offset：= nextField（str，offset）//重新声明偏移量
a，a：= 1，2 //非法：如果a在其他地方声明，则重复声明a或不声明新变量
简短的变量声明只能在函数内部出现。在某些情况下，例如“ if”， “ for”或 “ switch”语句的初始化程序 ，它们可用于声明局部临时变量。

函数声明
函数声明将标识符（函数名称）绑定到函数。

FunctionDecl =“ func” FunctionName  签名 [ FunctionBody ]。
FunctionName = 标识符。
FunctionBody = 块。
如果函数的签名声明了结果参数，则函数主体的语句列表必须以终止语句结尾。

func IndexRune（s string，r rune）int {
    对于i，c：= range s {
        如果c == r {
            还给我
        }
    }
    //无效：缺少return语句
}
函数声明可以省略主体。这样的声明为Go外部实现的功能（例如汇编例程）提供了签名。

func min（x int，y int）int {
    如果x <y {
        返回x
    }
    返回y
}

func flushICache（begin，end uintptr）//外部实现
方法声明
方法是具有接收器的功能。方法声明将标识符，方法名称绑定到方法，并将该方法与接收者的基本类型相关联。

MethodDecl =“ func” 接收者 MethodName  签名 [ FunctionBody ]。
接收器    = 参数。
接收方通过方法名称前面的附加参数部分指定。该参数部分必须声明一个非可变参数，即接收器。它的类型必须是已定义类型T或指向已定义类型的指针T。T被称为接收方 基本类型。接收方基本类型不能是指针或接口类型，并且必须在与方法相同的程序包中定义。据说该方法已绑定到其接收方基本类型，并且该方法的名称仅在type 或的选择器中可见。 T*T

非空白的接收者标识符在方法签名中必须 唯一。如果没有在方法主体内部引用接收方的值，则可以在声明中省略其标识符。通常，这同样适用于功能和方法的参数。

对于基本类型，绑定到它的方法的非空白名称必须是唯一的。如果基本类型是struct type，则非空白方法和字段名称必须不同。

给定定义的类型Point，声明

func（p * Point）Length（）float64 {
    返回math.Sqrt（px * px + py * py）
}

func（p * Point）Scale（factor float64）{
    px * =因素
    py * =因素
}
将方法Length和Scale具有接收者类型的绑定*Point到基本类型Point。

方法的类型是将接收方作为第一个参数的函数的类型。例如，该方法Scale具有类型

func（p * Point，factor float64）
但是，以这种方式声明的函数不是方法。

表达方式
表达式通过将运算符和函数应用于操作数来指定值的计算。

操作数
操作数表示表达式中的基本值。操作数可以是文字，表示常量， 变量或 函数的（可能是限定的）非空白标识符 ，或者是带括号的表达式。

的空白标识符可能显示为仅一个上的左手侧的操作数分配。

操作数      = 文字 | OperandName | “（” 表达式 “）”。
文字      = BasicLit | CompositeLit | FunctionLit。
BasicLit     = int_lit | float_lit | imaginary_lit | rune_lit | string_lit。
OperandName = 标识符 | QualifiedIdent。
合格标识符
合格标识符是具有包名前缀的合格标识符。软件包名称和标识符都不能为 空。

QualifiedIdent = PackageName “。” 标识符。
合格标识符访问另一个包中的标识符，必须将其导入。标识符必须导出并在该包的包块中声明。

math.Sin //表示软件包math中的Sin函数
复合文字
复合文字量为结构，数组，切片和映射构造值，并在每次对其求值时创建一个新值。它们由文字的类型和紧随其后的元素列表组成。每个元素可以可选地在对应的关键字之后。

CompositeLit   = LiteralType  LiteralValue。
LiteralType    = StructType | ArrayType | “ [”“ ...”“]” ElementType |
                SliceType | MapType | TypeName。
LiteralValue   =“ {” [ ElementList [“，”]]“}”。
ElementList    = KeyedElement {“，” KeyedElement }。
KeyedElement   = [ 键 “：”] 元素。
键            = FieldName | 表达| LiteralValue。
FieldName      = 标识符。
元素        = 表达式 | LiteralValue。
LiteralType的基础类型必须是struct，array，slice或map类型（语法强制执行此约束，除非将类型指定为TypeName）。元素和键的类型必须可分配 给文字类型的相应字段，元素和键类型；没有其他转换。键被解释为结构文字的字段名称，数组和切片文字的索引以及映射文字的键。对于地图文字，所有元素都必须具有键。指定具有相同字段名称或常数键值的多个元素是错误的。对于非恒定映射键，请参阅评估顺序部分 。

对于结构文字，以下规则适用：

键必须是在结构类型中声明的字段名称。
不包含任何键的元素列表必须按声明字段的顺序为每个struct字段列出一个元素。
如果任何元素具有键，则每个元素都必须具有键。
包含键的元素列表不需要为每个struct字段都具有一个元素。省略的字段将获得该字段的零值。
文字可能会省略元素列表；这样的文字对其类型求值为零。
为属于不同包的结构的非导出字段指定元素是错误的。
给出声明

输入Point3D struct {x，y，z float64}
输入Line struct {p，q Point3D}
一个人可以写

origin：= Point3D {} // Point3D的值为零
line：= Line {origin，Point3D {y：-4，z：12.3}}} // line.qx的值为零
对于数组和切片文字，以下规则适用：

每个元素都有一个关联的整数索引，用于标记其在数组中的位置。
具有键的元素将键用作其索引。键必须是可以由type值表示的非负常量 int；如果输入，则必须为整数类型。
没有键的元素使用前一个元素的索引加一个。如果第一个元素没有键，则其索引为零。
获取复合文字的地址会生成一个指向使用文字的值初始化的唯一变量的指针。

var指针* Point3D =＆Point3D {y：1000}
请注意，切片或图类型的零值与相同类型的已初始化但空值不同。因此，获取空切片或映射复合文字的地址与使用new分配新切片或映射值的效果不同 。

p1：=＆[] int {} // p1指向一个初始化的空切片，其值为[] int {}，长度为0
p2：= new（[] int）// p2指向值为nil且长度为0的未初始化切片
数组文字的长度是在文字类型中指定的长度。如果文字中提供的元素少于长度，则对于数组元素类型，将缺少的元素设置为零值。为元素提供的索引值超出数组的索引范围是错误的。该符号...指定的数组长度等于最大元素索引加一。

缓冲区：= [10] string {} // len（buffer）== 10
intSet：= [6] int {1、2、3、5} // len（intSet）== 6
days：= [...] string {“ Sat”，“ Sun”} // len（days）== 2
切片文字描述了整个基础数组文字。因此，切片文字的长度和容量为最大元素索引加一。切片文字的格式为

[] T {x1，x2，…xn}
并且是应用于数组的切片操作的简写：

tmp：= [n] T {x1，x2，…xn}
tmp [0：n]
在数组，切片或映射类型的复合文字中T，如果元素或映射键本身与复合文字相同，则元素或映射键可以忽略相应的文字类型（如果它与的元素或键类型相同）T。同样，&T当元素或键的类型为时，作为组合文字的地址的元素或键可能会消失*T。

[...] Point {{1.5，-3.5}，{0，0}} //与[... Point {Point {1.5，-3.5}，Point {0，0}}相同
[] [] int {{1，2，3}，{4，5}} //与[] [] int {[] int {1，2，3}，[] int {4，5}}相同
[] [] Point {{{{0，1}，{1，2}}}} //与[] [] Point {[] Point {Point {0，1}，Point {1，2}}}相同
map [string] Point {“ orig”：{0，0}} //与map [string] Point {“ orig”：Point {0，0}}相同
map [Point] string {{0，0}：“ orig”} //与map [Point] string {Point {0，0}：“ orig”}相同

输入PPoint * Point
[2] * Point {{1.5，-3.5}，{}} //与[2] * Point {＆Point {1.5，-3.5}，＆Point {}}相同
[2] PPoint {{1.5，-3.5}，{}} //与[2] PPoint {PPoint（＆Point {1.5，-3.5}），PPoint（＆Point {}）}相同
当使用LiteralType的TypeName形式的复合文字作为关键字和“ if”，“ for”或“ switch”语句的块的开头括号之间的操作数出现时，解析歧义就会出现。 不能用括号，方括号或花括号括起来。在这种罕见的情况下，字面量的开头括号被错误地解析为引入语句块的括号。要解决歧义，复合文字必须出现在括号内。

如果x ==（T {a，b，c} [i]）{…}
如果（x == T {a，b，c} [i]）{…}
有效数组，切片和映射文字的示例：

//质数列表
素数：= [] int {2，3，5，7，9，2147483647}

//如果ch是元音，则元音[ch]为真
元音：= [128] bool {'a'：true，'e'：true，'i'：true，'o'：true，'u'：true，'y'：true}

//数组[10] float32 {-1，0，0，0，-0.1，-0.1，0，0，0，-1}
过滤器：= [10] float32 {-1，4：-0.1，-0.1，9：-1}

//等温刻度的频率（Hz）（A4 = 440Hz）
noteFrequency：= map [string] float32 {
    “ C0”：16.35，“ D0”：18.35，“ E0”：20.60，“ F0”：21.83，
    “ G0”：24.50，“ A0”：27.50，“ B0”：30.87，
}
函数文字
函数文字代表一个匿名函数。

FunctionLit =“ func” 签名 FunctionBody。
func（a，b int，z float64）bool {return a * b <int（z）}
可以将函数文字分配给变量或直接调用。

f：= func（x，y int）int {return x + y}
func（ch chan int）{ch <-ACK}（replyChan）
函数文字是闭包的：它们可以引用在周围函数中定义的变量。然后，这些变量在周围的函数和函数文字之间共享，并且只要可以访问它们就可以保留。

主要表达
主表达式是一元和二进制表达式的操作数。

PrimaryExpr =
     操作数 |
    转换 |
    MethodExpr |
    PrimaryExpr  选择器 |
    PrimaryExpr  指数 |
    PrimaryExpr  切片 |
    PrimaryExpr  TypeAssertion |
    PrimaryExpr  参数。

选择器        =“。标识符。
索引           =“ [” 表达式 “]”。
Slice           =“ [” [ 表达式 ]“：” [ 表达式 ]“]” |
                 “ [” [ 表达式 ]“：” 表达式 “：” 表达式 “]”。
TypeAssertion   =“。” “（” 类型 “）”。
参数       =“（” [（（ExpressionList | 类型 [“，” ExpressionList ]）[“ ...”] [“，”]]“）”。
X
2
（s +“ .txt”）
f（3.1415，正确）
点{1，2}
m [“ foo”]
s [i：j + 1]
obj.color
fp [i] .x（）
选择器
对于一次式 x ，是不是一个包的名称中， 选择表达

f
表示场或方法f的值的x （或有时*x;见下文）。标识符f称为（字段或方法）选择器；它不能是空白标识符。选择器表达式的类型是的类型f。如果x是软件包名称，请参见有关合格标识符的部分 。

选择器f可以表示一个字段或方法f的类型的T，或者它可以指字段或方法f嵌套的 嵌入式领域的T。走过来达到嵌入式领域的数f称为它的深度在T。f 声明的字段或方法的深度T为零。f嵌入字段Ain中声明的字段或方法T的深度为fin 的深度A加一。

以下规则适用于选择器：

对于值x类型的T或*T 其中T不是指针或接口类型， x.f表示在最浅深度域或方法T，其中有这样一个f。如果没有一个f 深度最浅的选择器，则选择器表达式是非法的。
对于其中 接口类型x为type 的值，表示动态方法名称为的实际方法 。如果没有与名称的方法，在 设置方法中，选择表达式是非法的。 IIx.ffxfI
作为例外，如果类型x是已定义的 指针类型，并且(*x).f是表示字段（但不是方法）的有效选择器表达式，x.f则是的简写(*x).f。
在所有其他情况下，x.f都是非法的。
如果xis是指针类型并具有值 nil并x.f表示struct字段，则分配或评估将x.f 导致运行时恐慌。
如果x是接口类型并具有值 nil，则调用或 评估该方法将x.f 导致运行时恐慌。
例如，给定声明：

输入T0 struct {
    x int
}

函数（* T0）M0（）

T1型结构{
    y int
}

函数（T1）M1（）

T2型结构{
    z int
    T1
    * T0
}

函数（* T2）M2（）

Q * T2型

var t T2 // // t.T0！= nil
var p * T2 // // p！= nil和（* p）.T0！= nil
var q Q = p
一个人可以这样写：

tz // tz
ty // t.T1.y
tx //（* t.T0）.x

pz //（* p）.z
py //（* p）.T1.y
px //（*（* p）.T0）.x

qx //（*（* q）.T0）.x（* q）.x是有效的字段选择器

p.M0（）//（（**）。T0）.M0（）M0需要* T0接收者
p.M1（）//（（（* p）.T1）.M1（）M1需要T1接收器
p.M2（）// p.M2（）M2需要* T2接收器
t.M2（）//（＆t）.M2（）M2需要* T2接收器，请参阅“通话”部分
但是以下无效：

q.M0（）//（* q）.M0有效但不是字段选择器
方法表达式
如果M在type 的方法集中T， T.M则是可以作为常规函数调用的函数，该函数具有与作为M该方法的接收者的附加参数作为前缀的相同参数。

MethodExpr     = 接收器类型 “。” 方法名。
ReceiverType   = 类型。
考虑T具有两种方法 的struct类型Mv，其接收者的类型为T， Mp接收者的类型为*T。

类型T struct {
    一个整数
}
func（tv T）Mv（int）int {return 0} //值接收者
func（tp * T）Mp（f float32）float32 {return 1} //指针接收者

变数
表达方式

电视
产生一个Mv与第一个参数等效但带有显式接收器的函数；它有签名

func（tv，int）int
通常可以使用显式接收器调用该函数，因此这五个调用是等效的：

吨MV（7）
Mv（t，7）
（T）.Mv（t，7）
f1：= T.Mv; f1（t，7）
f2：=（T）.Mv; f2（t，7）
同样，表达式

（* T）.Mp
产生一个Mp以签名 表示的函数值

func（tp * T，f float32）float32
对于具有值接收器的方法，可以使用显式指针接收器派生一个函数，因此

（* T）.Mv
产生一个Mv以签名 表示的函数值

func（tv * T，int）int
这样的函数通过接收器间接创建一个值，该值作为接收器传递给基础方法。该方法不会覆盖在函数调用中传递其地址的值。

最后一种情况是指针接收器方法的值接收器函数，是非法的，因为指针接收器方法不在值类型的方法集中。

从方法派生的函数值使用函数调用语法进行调用；提供接收方作为呼叫的第一个参数。也就是说，给定f := T.Mv，f被调用为f(t, 7)没有t.f(7)。要构造绑定接收者的函数，请使用 函数文字或 方法value。

从接口类型的方法派生函数值是合法的。结果函数采用该接口类型的显式接收器。

方法值
如果表达式x具有静态类型T并且 M在方法的类型集中T， x.M则称为方法值。方法值x.M是可以使用与方法调用相同的参数调用的函数值x.M。在x计算方法值期间将对表达式进行求值并保存；然后，保存的副本将在任何调用中用作接收者，稍后可能会执行。

该类型T可以是接口或非接口类型。

正如在讨论方法表达式以上，考虑一种结构类型T与两个方法， Mv，其接收器的类型为T，和 Mp，其接收器的类型的*T。

类型T struct {
    一个整数
}
func（tv T）Mv（int）int {return 0} //值接收者
func（tp * T）Mp（f float32）float32 {return 1} //指针接收者

变数
var pt * T
func makeT（）T
表达方式

吨
产生一个类型的函数值

func（int）int
这两个调用是等效的：

吨MV（7）
f：= t.Mv; f（7）
同样，表达式

磅
产生一个类型的函数值

func（float32）float32
与选择器一样，使用指针通过值接收器对非接口方法的引用将自动取消引用该指针：pt.Mv等效于(*pt).Mv。

与方法调用一样，使用指针指针接收器使用可寻址的值对非接口方法的引用将自动采用该值的地址：t.Mp等效于(&t).Mp。

f：= t.Mv; f（7）//就像t.Mv（7）
f：= pt.Mp; f（7）//就像pt.Mp（7）
f：= pt.Mv; f（7）//喜欢（* pt）.Mv（7）
f：= t.Mp; f（7）//像（＆t）.Mp（7）
f：= makeT（）。Mp //无效：makeT（）的结果不可寻址
尽管上面的示例使用了非接口类型，但是从接口类型的值创建方法值也是合法的。

var i interface {M（int）} = myVal
f：= iM; f（7）//就像iM（7）
索引表达式
形式的主要表达

斧头]
表示数组的元素，指向a由索引的数组，切片，字符串或映射的指针x。该值分别x称为index或map键。适用以下规则：

如果a不是地图：

索引x必须是整数类型或无类型常量
常数索引必须是非负数，并且可以 由type的值表示int
没有类型的常量索引被赋予类型 int
该指数x是在范围内，如果0 <= x < len(a)，否则它是超出范围
为a的数组类型 A：

一个恒定索引必须是在范围
如果x在运行时超出范围，则会发生运行时恐慌
a[x]是索引处的数组元素，x类型 a[x]是的元素类型A
对于数组类型a的指针：

a[x] 是的简写 (*a)[x]
对于a的片类型 S：

如果x在运行时超出范围，则会发生运行时恐慌
a[x]是索引处的slice元素，x类型 a[x]是的元素类型S
对于a的字符串类型：

一个恒定索引必须是在范围内，如果该字符串a也是恒定
如果x在运行时超出范围，则会发生运行时恐慌
a[x]是索引处的非恒定字节值，x且类型 a[x]为byte
a[x] 可能未分配给
对于a的地图类型 M：

x的类型必须可 分配 给的键类型M
如果地图包含具有key的条目x， a[x]则是具有key 的map元素，x 类型a[x]为的元素类型M
如果映射nil包含或不包含此类条目， a[x]则为 的元素类型的零值M
否则a[x]是非法的。

a类型 图上的索引表达式，map[K]V 用于特殊形式的赋值或初始化

v，确定= a [x]
v，好的：= a [x]
var v，确定= a [x]
产生另一个无类型的布尔值。的值ok是 true键x是否存在于地图中， false否则为。

分配给nil地图元素会导致 运行时恐慌。

切片表达式
切片表达式根据字符串，数组，指向数组或切片的指针构造子字符串或切片。有两种变体：一种简单的形式，它指定一个下限和一个上限；一个完整的形式，它还指定一个容量的界限。

简单的切片表达式
对于字符串，数组，指向数组的指针或slice a，主表达式

a [低：高]
构造一个子字符串或切片。的索引 low和 high选择的哪些元素的操作数a出现在结果中。结果的索引从0开始，长度等于 high -  low。切片后a

a：= [5] int {1、2、3、4、5}
s：= a [1：4]
切片s具有type []int，长度3，容量4和元素

s [0] == 2
s [1] == 3
s [2] == 4
为了方便起见，可以省略任何索引。缺失low 索引默认为零；缺少的high索引默认为切片操作数的长度：

a [2：] //与a [2：len（a）]相同
a [：3] //与a [0：3]相同
a [：] //与a [0：len（a）]相同
如果a是指向数组的指针，a[low : high]则是的简写 (*a)[low : high]。

对于数组或字符串，如果 <= <= <= 则索引在范围内，否则它们超出范围。对于切片，索引的上限是切片容量而不是长度。甲恒定索引必须为非负和 可表示由类型的值 ; 对于数组或常量字符串，常量索引也必须在范围内。如果两个索引都恒定，则它们必须满足。如果索引在运行时超出范围，则会发生运行时恐慌。 0lowhighlen(a)cap(a)intlow <= high

除未类型化的字符串外，如果切片的操作数是字符串或切片，则切片操作的结果是与该操作数相同类型的非常数值。对于无类型的字符串操作数，结果是type的非恒定值string。如果切片的操作数是一个数组，则它必须是可寻址 的，并且切片操作的结果是与数组具有相同元素类型的切片。

如果有效切片表达式的切片操作数是nil切片，则结果是nil切片。否则，如果结果是切片，则它将与操作数共享其基础数组。

var a [10] int
s1：= a [3：7] // s1的基础数组是数组a; ＆s1 [2] ==＆a [5]
s2：= s1 [1：4] // s2的基础数组是s1的基础数组，即数组a；＆s2 [1] ==＆a [5]
s2 [1] = 42 // s2 [1] == s1 [2] == a [5] == 42; 它们都引用相同的基础数组元素
完整切片表达式
对于数组，指向数组或切片a（而不是字符串）的指针，主要表达式

a [低：高：最大]
构造与简单slice表达式相同类型，长度和元素相同的slice a[low : high]。此外，它通过将其设置为来控制所得切片的容量max - low。仅第一个索引可以省略；它的默认值为0。切片数组后a

a：= [5] int {1、2、3、4、5}
t：= a [1：3：5]
切片t具有类型[]int，长度2，容量4和元素

t [0] == 2
t [1] == 3
对于简单的切片表达式，如果a是指向数组的指针， a[low : high : max]则是的简写形式(*a)[low : high : max]。如果切片的操作数是一个数组，则它必须是可寻址的。

该指数是在范围内，如果0 <= low <= high <= max <= cap(a)，否则他们是超出范围。甲恒定索引必须为非负和 可表示由类型的值 int; 对于数组，常量索引也必须在范围内。如果多个索引为常数，则存在的常数必须在相对范围内。如果索引在运行时超出范围，则会发生运行时恐慌。

类型断言
对于表达x的接口类型 和类型T，初级表达

吨
断言x不是，nil 并且存储的值x是type T。该符号x.(T)称为类型声明。

更精确地，如果T不是一个接口类型，x.(T)断言，动态型的x是相同 的类型T。在这种情况下，T必须实现的（接口）类型x；否则类型断言无效，因为无法x 存储type的值T。如果T为接口类型，x.(T)则断言动态类型的x实现接口T。

如果类型断言成立，则表达式的值为存储在其中的值x，其类型为T。如果类型断言为假，则会发生运行时恐慌。换句话说，即使动态类型x 只能在运行时是已知的类型，x.(T)被称为是T在一个正确的程序。

var x interface {} = 7 // x具有动态类型int，值7
i：= x。（int）//我的类型为int且值为7

类型I接口{m（）}

func f（y I）{
    s：= y。（string）//非法：string无法实现I（缺少方法m）
    r：= y。（io.Reader）// r具有io.Reader类型，并且y的动态类型必须同时实现I和io.Reader
    …
}
在特殊形式 的赋值或初始化中 使用的类型断言

v，ok = x。（T）
v，好的：= x。（T）
var v，ok = x。（T）
var v，OK T1 = x。（T）
产生另一个无类型的布尔值。 如果断言成立，ok则值为true。否则，它是false和值v是零值的类型T。在这种情况下，不会发生运行时恐慌。

来电
给定一个f函数类型 的表达式F，

f（a1，a2，…an）
f带参数的 调用a1, a2, … an。除一种特殊情况外，参数必须是可分配给的参数类型的单值表达式 ， F并且必须在调用函数之前对其求值。表达式的类型是的结果类型F。方法调用类似，但是方法本身根据方法的接收器类型的值指定为选择器。

math.Atan2（x，y）//函数调用
var pt *点
pt.Scale（3.5）//使用接收方pt进行方法调用
在函数调用中，函数值和参数按通常的顺序求值 。在对它们进行评估之后，调用的参数将按值传递给函数，并且被调用函数开始执行。当函数返回时，该函数的返回参数按值传递回调用函数。

调用nil函数值会导致运行时恐慌。

作为一种特殊情况，如果一个函数或方法的返回值 g在数量上相等，并且可以分别分配给另一个函数或方法的参数 f，则该调用 将在按顺序将的返回值绑定到参数之后 调用。的调用除的调用外不得包含任何参数，并且必须至少具有一个返回值。如果具有最终参数，则分配常规参数后保留的返回值。 f(g(parameters_of_g))fgffggf...g

func Split（s string，pos int）（string，string）{
    返回s [0：pos]，s [pos：]
}

func Join（s，t string）字符串{
    返回s + t
}

如果Join（Split（value，len（value）/ 2））！= value {
    log.Panic（“测试失败”）
}
x.m()如果方法集 （的类型）x包含m并且参数列表可以分配给的参数列表，则该 方法调用有效m。如果x是可寻址且&x方法集包含m，x.m()则为(&x).m()：

变点
p。规模（3.5）
没有独特的方法类型，也没有方法文字。

将...参数传递给参数
如果f为带有最终类型为type的可变参数，则 类型内等于type 。如果在没有实际参数的情况下调用，则传递给的值为。否则，传递的值是带有新的基础数组的类型的新切片，该基础数组的连续元素是实际参数，所有参数都必须可分配 给。因此，切片的长度和容量是绑定到每个调用站点的参数的数量，并且对于每个调用站点而言，可能会有所不同。 p...Tfp[]Tfppnil[]TTp

给定功能和调用

func Greeting（前缀字符串，who ... string）
问候（“没人”）
问候语（“ hello：”，“ Joe”，“ Anna”，“ Eileen”）
在中Greeting，who将nil在第一个调用中具有值，在第二个调用中 具有值 []string{"Joe", "Anna", "Eileen"}。

如果最终参数可分配给切片类型[]T，...T则在参数后跟时将其不变地作为参数值传递...。在这种情况下，不会创建新的切片。

给定切片s和调用

s：= [] string {“ James”，“ Jasmine”}
问候语（“再见：”，s ...）
中的Greeting，who将具有s 与相同基础数组相同的值。

经营者
运算符将操作数组合成表达式。

表达式 = UnaryExpr | 表达式 binary_op  表达式。
UnaryExpr   = PrimaryExpr | unary_op  UnaryExpr。

binary_op   =“ ||” | “ &&” | rel_op | add_op | mul_op。
rel_op      =“ ==” | “！=” | “ <” | “ <=” | “>” | “> =”。
add_op      =“ +” | “-” | “ |” | “ ^”。
mul_op      =“ *” | “ /” | “％” | “ <<” | “ >>” | “＆” | “＆^”。

unary_op    =“ +” | “-” | “！” | “ ^” | “ *” | “＆” | “ <-”。
比较在其他地方讨论。对于其他二进制运算符， 除非该操作涉及shifts或未类型化的常量，否则操作数类型必须相同。对于仅涉及常量的操作，请参见关于常量表达式的部分 。

除移位操作外，如果一个操作数是未类型化的常量 ，而另一个操作数不是，则将该常量隐式转换 为另一操作数的类型。

移位表达式中的右操作数必须具有整数类型，或者是可以由type值表示的无类型常量uint。如果非恒定移位表达式的左操作数是未类型化的常量，则首先将其隐式转换为如果仅将移位表达式替换为其左操作数时将假定的类型。

var s uint = 33
var i = 1 << ss // 1的类型为int
var j int32 = 1 << s // 1的类型为int32; j == 0
var k = uint64（1 << s）// 1的类型为uint64; k == 1 << 33
var m int = 1.0 << s // 1.0的类型为int; 如果int的大小为32位，则m == 0
var n = 1.0 << s == j // 1.0的类型为int32; n ==真
var o = 1 << s == 2 << s // 1和2的类型为int；o ==如果ints的大小为32bits，则为true
var p = 1 << s == 1 << 33 //如果int的大小为32bits是非法的：1的类型为int，但1 << 33溢出int
var u = 1.0 << s //非法：1.0的类型为float64，无法移位
var u1 = 1.0 << s！= 0 //非法：1.0的类型为float64，无法移位
var u2 = 1 << s！= 1.0 //非法：1的类型为float64，无法移位
var v float32 = 1 << s //非法：1的类型为float32，无法移位
var w int64 = 1.0 << 33 // 1.0 << 33是一个常数移位表达式
var x = a [1.0 << s] // 1.0的类型为int；x == a [0]如果int的大小为32bits
var a = make（[] byte，1.0 << s）// 1.0的类型为int；如果int大小为32位，则len（a）== 0
运算符优先级
一元运算符的优先级最高。由于 ++and --运算符形成语句而不是表达式，因此它们不在运算符层次结构之内。因此，陈述*p++与相同(*p)++。

二进制运算符有五个优先级。乘法运算符的绑定最强，其次是加法运算符，比较运算符，&&（逻辑与）和最后||（逻辑或）：

优先运算符
    5 * /％<< >>＆＆^
    4 +-| ^
    3 ==！= <<=>> =
    2 &&
    1 ||
具有相同优先级的二元运算符从左到右关联。例如，x / y * z与相同(x / y) * z。

+ x
23 + 3 * x [i]
x <= f（）
^ a >> b
f（）|| G（）
x == y + 1 && <-chanPtr> 0
算术运算符
算术运算符适用于数值，并产生与第一个操作数相同类型的结果。四个标准的算术运算符（+， -，*，/）适用于整数，浮点，和复杂类型; +也适用于字符串。按位逻辑和移位运算符仅适用于整数。

+将整数，浮点数，复数值，字符串相加
-差整数，浮点数，复数值
*产品整数，浮点数，复数值
/商整数，浮点数，复数值
％剩余整数

＆按位与整数
| 按位或整数
^按位XOR整数
＆^位清除（AND NOT）整数

<<左移整数<<无符号整数
>>右移整数>>无符号整数
整数运算符
对于两个整数值x和y，整数商 q = x / y和余数r = x % y满足以下关系：

x = q * y + r和| r | <| y |
与x / y向零截断（“截短的分裂”）。

 xyx / yx％y
 5 3 1 2
-5 3 -1 -2
 5 -3 -1 2
-5 -3 1 -2
该规则的一个例外是，如果被除数x是int类型的最大负值x，则由于二进制补码整数溢出，商 q = x / -1等于x（和r = 0）：

             设
整数8 -128
整数16 -32768
整数32 -2147483648
整数64 -9223372036854775808
如果除数是一个常数，则它不能为零。如果除数在运行时为零，则会发生运行时恐慌。如果被除数是非负数，并且除数是2的恒定乘方，则除法可以用向右移位代替，计算余数可以用按位与运算代替：

 xx / 4 x％4 x >> 2 x和3
 11 2 3 2 3
-11 -2 -3 -3 1
移位运算符将左操作数移位由右操作数指定的移位计数，该移位计数必须为非负数。如果运行时班次计数为负，则会发生运行时恐慌。如果左操作数是有符号整数，则移位运算符将执行算术移位；如果是无符号整数，则将进行逻辑移位。班次计数没有上限。移位的行为就好像左操作数被移位n1倍（移位计数为）n。结果，x << 1与x*2 和x >> 1相同， x/2但被截断为负无穷大。

对于整数操作数，一元运算符 +，-以及^被定义为如下：

+ x是0 + x
-x取反为0-x
^ x按位补码为m ^ x，其中m =“无符号x的所有位都设置为1”
                                      对于带符号的x，m = -1
整数溢出
为无符号整数值，所述操作+， -，*，和<<被计算模2 Ñ，其中Ñ为的比特宽度的无符号整数的类型。松散地说，这些无符号整数运算在溢出时会丢弃高位，并且程序可能依赖于“环绕”。

对于有符号整数，操作+， -，*，/，和<<可以合法溢出并且将所得值存在并且确定地由带符号整数，操作，和它的操作数定义。溢出不会引起运行时恐慌。在不发生溢出的假设下，编译器可能不会优化代码。例如，它可能不会假设x < x + 1始终为真。

浮点运算符
对于浮点数和复数， +x与相同x，而-x对的取反x。超出IEEE-754标准时，未指定浮点或复数除以零的结果。运行时是否 发生恐慌是特定于实现的。

一个实现可以将多个浮点操作组合为单个融合操作（可能跨语句），并产生与通过单独执行和舍入指令所获得的值不同的结果。显式浮点类型转换将舍入到目标类型的精度，从而防止融合会舍弃该舍入。

例如，某些体系结构提供了“融合乘法和加法”（FMA）指令，该指令的计算x*y + z无需舍入中间结果x*y。这些示例说明Go实现何时可以使用该指令：

// FMA允许计算r，因为x * y没有显式舍入：
r = x * y + z
r = z; r + = x * y
t = x * y; r = t + z
* p = x * y; r = * p + z
r = x * y + float64（z）

// FMA不允许计算r，因为它将省略x * y的舍入：
r = float64（x * y）+ z
r = z; r + = float64（x * y）
t = float64（x * y）; r = t + z
字符串串联
可以使用+运算符或+=赋值运算符来连接字符串：

s：=“ hi” +字串（c）
s + =“和再见”
字符串加法通过连接操作数来创建新的字符串。

比较运算符
比较运算符比较两个操作数，并产生无类型的布尔值。

==等于
！=不等于
<少于
<=小于或等于
>更大
> =大于或等于
在任何比较中，第一个操作数必须可分配 给第二个操作数的类型，反之亦然。

等于运算符==和!=适用于可比较的操作数。排序运营商<，<=，>和>= 适用于那些操作数排序。这些术语和比较结果定义如下：

布尔值是可比较的。两个布尔值相等true或相等 false。
整数值是按常规方式可比较且有序的。
浮点值是可比较的且有序，如IEEE-754标准所定义。
复数值是可比较的。如果和 u，v则两个复数值和相等。 real(u) == real(v)imag(u) == imag(v)
字符串值是可比较的且按字节顺序排序。
指针值是可比较的。如果两个指针值指向同一个变量或都具有value，则它们相等nil。指向不同零大小变量的指针可以相等或可以不相等。
通道值是可比较的。如果两个通道值是通过相同的调用创建的，make 或者两个通道都具有value，则它们是相等的 nil。
接口值是可比较的。如果两个接口值具有相同的动态类型和相等的动态值，或者两个接口值都具有value，则它们是相等的nil。
当xtype X的值是可比较的并且实现时，non-interface type 的值t和interface type T的值X是可比较的 。如果动态类型等于， 并且动态值等于，则它们相等。 XTtXtx
如果结构的所有字段都是可比较的，则它们的值是可比较的。如果两个结构值对应的非空白字段相等，则它们相等。
如果数组元素类型的值可比较，则数组值可比较。如果两个数组的对应元素相等，则它们相等。
如果动态类型相同的两个接口值不具有可比性，则将它们进行比较会导致运行时恐慌。此行为不仅适用于直接接口值比较，而且适用于将接口值或结构的数组与接口值字段进行比较。

切片，贴图和函数值不可比较。但是，在特殊情况下，可以将切片，映射或函数值与预先声明的标识符进行比较nil。指针，通道和接口值to的nil 比较也是允许的，并且遵循上面的一般规则。

const c = 3 <4 // c是无类型的布尔常量true

输入MyBool bool
var x，y int
var（
    //比较的结果是一个无类型的布尔值。
    //通常的分配规则适用。
    b3 = x == y // b3具有布尔型
    b4 bool = x == y // b4的类型为bool
    b5 MyBool = x == y // b5具有类型MyBool
）
逻辑运算符
逻辑运算符适用于布尔值，并产生与操作数相同类型的结果。正确的操作数是有条件的。

&&条件AND p && q为“如果p则q否则为false”
|| 有条件OR p || q是“如果p，则为true，否则为q”
！NOT！p是“ not p”
地址运算符
对于x类型的操作数T，地址操作 &x生成类型*T为的指针x。操作数必须是可寻址的，即变量，指针间接寻址或切片索引操作；或可寻址结构操作数的字段选择器；或可寻址数组的数组索引操作。除可寻址性要求之外，x还可以是（可能带有括号的） 复合文字。如果对的评估x会导致运行时恐慌，那么对的评估&x也会这样做。

一个操作数x指针类型*T，指针间接*x表示所述可变类型的T指向x。如果x是nil，则尝试评估*x 会导致运行时恐慌。

＆X
＆a [f（2）]
＆Point {2，3}
* p
* pf（x）

var x * int = nil
* x //导致运行时恐慌
＆* x //导致运行时恐慌
接收运算符
对于通道类型的操作数ch，接收操作的值为从通道接收的值。通道方向必须允许接收操作，并且接收操作的类型是通道的元素类型。表达式将阻塞直到有一个值可用为止。从频道接收到的内容将永远被阻止。封闭通道上的接收操作始终可以立即进行， 在接收到任何先前发送的值之后，得出元素类型的零值。 <-chchnil

v1：= <-ch
v2 = <-ch
f（<-ch）
<-strobe //等待直到时钟脉冲并丢弃接收到的值
特殊形式 的赋值或初始化中 使用的接收表达式

x，确定= <-ch
x，好的：= <-ch
var x，ok = <-ch
var x，OK T = <-ch
产生另一个无类型的布尔结果，报告通信是否成功。的值为ok是true 接收到的值是通过成功的发送操作传递给通道的，还是false由于通道关闭且为空而生成的零值。

转换次数
A转换改变类型的表达式的要由所述转换所指定的类型。转换可能从字面上出现在源中，或者可能 由表达式出现的上下文隐含。

一个显式转换为形式的表达T(x) ，其中T是一种类型并且x是可转换为类型的表达式T。

转换 = 类型 “（” 表达式 [“，”]“）”。
如果类型以运算符*或开头<-，或者类型以关键字开头func 且没有结果列表，则在必要时必须将其括起来，以避免产生歧义：

* Point（p）//与*（Point（p））相同
（* Point）（p）// p转换为* Point
<-chan int（c）//与<-（chan int（c））相同
（<-chan int）（c）//将c转换为<-chan int
func（）（x）//函数签名func（）x
（func（））（x）// x转换为func（）
（func（）int）（x）// x转换为func（）int
func（）int（x）// x转换为func（）int（明确）
甲恒定值x可以被转换为类型T是否x是可表示 通过的值T。作为一种特殊情况，x可以使用 与 non-constant 相同的规则将整数常量显式转换为 字符串类型。 x

转换常数会产生一个类型化的常数。

uint（iota）// uint类型的iota值
float32（2.718281828）// float32类型的2.718281828
complex128（1）//类型为complex128的1.0 + 0.0i
float32（0.49999999）//类型为float32的0.5
float64（-1e-1000）//类型为float64的0.0
string（'x'）//字符串类型的“ x”
string（0x266c）//字符串类型的“♬”
MyString（“ foo” +“ bar”）//类型为MyString的“ foobar”
string（[] byte {'a'}）//不是常量：[] byte {'a'}不是常量
（* int）（nil）//不是常量：nil不是常量，* int不是布尔，数字或字符串类型
int（1.2）//非法：1.2不能表示为int
string（65.0）//非法：65.0不是整数常量
在以下任何一种情况下，都x可以将 非恒定值转换为类型T：

x是分配 给T。
忽略struct标签（见下文） x的类型，并T具有相同的 基础类型。
忽略struct标记（请参见下文）， x的类型和T是未定义类型的指针类型，并且它们的指针基类型具有相同的基础类型。
x的类型T均为整数或浮点类型。
x的类型和T都是复杂类型。
x是整数或字节或符文的片段，并且T是字符串类型。
x是一个字符串，T是字节或符文的一部分。
在比较结构类型的标识以进行转换时，将忽略结构标签：

输入Person struct {
    名称字符串
    地址* struct {
        街串
        城市字符串
    }
}

var data * struct {
    名称字符串`json：“ name”`
    地址* struct {
        街道字符串`json：“ street”`
        城市字符串`json：“ city”`
    }`json：“ address”`
}

var person =（* Person）（data）//忽略标签，底层类型相同
特定规则适用于数字类型之间或字符串类型之间的（非恒定）转换。这些转换可能会更改的表示形式x 并产生运行时成本。所有其他转换只会更改类型，而不会更改的表示形式x。

没有语言机制可以在指针和整数之间进行转换。该程序包unsafe 在受限的情况下实现此功能。

数字类型之间的转换
对于非恒定数值的转换，适用以下规则：

在整数类型之间进行转换时，如果值是有符号整数，则将其符号扩展为隐式无限精度；否则为零扩展。然后将其截断以适合结果类型的大小。例如，如果v := uint16(0x10F0)，则uint32(int8(v)) == 0xFFFFFFF0。转换总是产生一个有效值。没有溢出迹象。
将浮点数转换为整数时，将舍弃小数（截断为零）。
将整数或浮点数转换为浮点类型，或将复数转换为另一种复数类型时，结果值将舍入为目标类型指定的精度。例如，x类型变量的值float32 可以使用IEEE-754 32位数字之外的其他精度来存储，但是float32（x）表示将x值四舍五入为32位精度的结果。同样，x + 0.1可能会使用超过32位的精度，但float32(x + 0.1)不会。
在所有涉及浮点或复杂值的非恒定转换中，如果结果类型不能表示该值，则转换成功，但结果值取决于实现。

字符串类型之间的转换
将有符号或无符号整数值转换为字符串类型会产生一个包含整数的UTF-8表示形式的字符串。有效Unicode代码点范围之外的值将转换为"\uFFFD"。
string（'a'）//“ a”
string（-1）//“ \ ufffd” ==“ \ xef \ xbf \ xbd”
string（0xf8）//“ \ u00f8” ==“ø” ==“ \ xc3 \ xb8”
输入MyString字符串
MyString（0x65e5）//“ \ u65e5” ==“日” ==“ \ xe6 \ x97 \ xa5”
将一个字节的片段转换为字符串类型会产生一个字符串，其连续字节是该片段的元素。
string（[] byte {'h'，'e'，'l'，'l'，'\ xc3'，'\ xb8'}）//“hellø”
string（[] byte {}）//“”
string（[] byte（nil））//“”

输入MyBytes [] byte
string（MyBytes {'h'，'e'，'l'，'l'，'\ xc3'，'\ xb8'}）//“hellø”
将一片符文转换为字符串类型会产生一个字符串，该字符串是转换为字符串的各个符文值的串联。
string（[] rune {0x767d，0x9d6c，0x7fd4}）//“ \ u767d \ u9d6c \ u7fd4” ==“白鹏翔”
string（[] rune {}）//“”
string（[] rune（nil））//“”

输入MyRunes [] rune
string（MyRunes {0x767d，0x9d6c，0x7fd4}）//“ \ u767d \ u9d6c \ u7fd4” ==“白鹏翔”
将字符串类型的值转换为字节类型的切片会产生一个切片，其连续元素是字符串的字节。
[] byte（“hellø”）// [] byte {'h'，'e'，'l'，'l'，'\ xc3'，'\ xb8'}
[] byte（“”）// [] byte {}

MyBytes（“hellø”）// [] byte {'h'，'e'，'l'，'l'，'\ xc3'，'\ xb8'}
将字符串类型的值转换为符文类型的切片会产生一个切片，其中包含字符串的各个Unicode代码点。
[] rune（MyString（“白鹏翔”））// //] rune {0x767d，0x9d6c，0x7fd4}
[] rune（“”）// [] rune {}

MyRunes（“白鹏翔”）// [] rune {0x767d，0x9d6c，0x7fd4}
常数表达式
常量表达式只能包含常量 操作数，并在编译时求值。

在合法使用布尔，数字或字符串类型的操作数的任何地方，都可以将未类型化的布尔，数字和字符串常量用作操作数。

常数比较始终会产生无类型的布尔常数。如果常量移位表达式的左操作数 是未类型化的常量，则结果是整数常量；否则，返回整数。否则，它是与左操作数相同类型的常量，该常量必须是 整数类型。

对无类型常量的任何其他操作都会导致相同类型的无类型常量。即布尔，整数，浮点数，复数或字符串常量。如果二进制操作（移位除外）的未类型化操作数是不同类型的，则结果是此操作数类型的结果，该类型将出现在此列表的后面：整数，符文，浮点数，复数。例如，将一个未类型化的整数常量除以一个未类型化的复数常量会得出一个未类型化的复数常量。

const a = 2 + 3.0 // a == 5.0（无类型浮点常量）
const b = 15/4 // b == 3（无类型整数常量）
const c = 15 / 4.0 // c == 3.75（无类型浮点常量）
constΘfloat64 = 3/2 //Θ== 1.0（类型float64，3/2是整数除法）
const float float64 = 3/2。//Π== 1.5（类型float64，3/2。是浮点除法）
const d = 1 << 3.0 // d == 8（无类型整数常量）
const e = 1.0 << 3 // e == 8（无类型整数常量）
const f = int32（1）<< 33 //非法（常数8589934592溢出int32）
const g = float64（2）>> 1 //非法（float64（2）是类型化的浮点常量）
const h =“ foo”>“ bar” // h == true（无类型的布尔常量）
const j = true // j == true（无类型的布尔常量）
const k ='w'+ 1 // k =='x'（无类型符文常数）
const l =“ hi” // l ==“ hi”（无类型的字符串常量）
const m = string（k）// m ==“ x”（字符串类型）
constΣ= 1-0.707i //（无类型复数常数）
constΔ=Σ+ 2.0e-4 //（无类型的复数常数）
constΦ= iota * 1i-1 / 1i //（无类型复数常数）
将内置函数complex应用于无类型的整数，符文或浮点常量会产生无类型的复数常量。

const ic = complex（0，c）// ic == 3.75i（无类型复数常数）
constiΘ= complex（0，Θ）//iΘ== 1i（类型complex128）
常量表达式总是精确地求值；中间值和常量本身可能需要比该语言中任何预声明类型支持的精度大得多的精度。以下是法律声明：

const Huge = 1 << 100 // Huge == 1267650600228229401496703205376（无类型整数常量）
const四个int8 =巨大>> 98 //四个== 4（类型int8）
常数除法或余数运算的除数不得为零：

3.14 / 0.0 //非法：被零除
类型化常量 的值必须始终可以由常量类型的值准确 表示。以下常量表达式是非法的：

uint（-1）// -1不能表示为uint
int（3.14）// 3.14不能表示为int
int64（Huge）// 1267650600228229401496703205376无法表示为int64
四* 300 //操作数300不能表示为int8（四类型）
四* 100 //产品400不能表示为int8（四类型）
一元按位补码运算符使用的掩码^与非常量规则匹配：对于无符号常量，掩码全为1，对于有符号和无类型常量，掩码全为-1。

^ 1 //无类型整数常量，等于-2
uint8（^ 1）//非法：与uint8（-2）相同，-2不能表示为uint8
^ uint8（1）//键入uint8常量，与0xFF相同^ uint8（1）= uint8（0xFE）
int8（^ 1）//与int8（-2）相同
^ int8（1）//与-1 ^ int8（1）= -2
实现限制：编译器在计算无类型浮点数或复杂常量表达式时可能会使用舍入；请参见常量部分中的实现限制。这种舍入可能会导致浮点常量表达式在整数上下文中无效，即使使用无限精度进行计算时它是整数，反之亦然。

评估顺序
在程序包级别，初始化依赖关系 确定变量声明中各个初始化表达式的求值顺序 。否则，在评估表达式，赋值或 return语句的操作数时，所有函数调用，方法调用和通信操作均按从左到右的词汇顺序进行评估。

例如，在（本地函数）分配中

y [f（）]，确定= g（h（），i（）+ x [j（）]，<-c），k（）
函数调用和通信发生的顺序 f()，h()，i()，j()， <-c，g()，和k()。但是，未指定这些事件与的评估，索引x和评估相比的顺序y。

a：= 1
f：= func（）int {a ++; 返回}
x：= [] int {a，f（）} // x可能是[1，2]或[2，2]：未指定a和f（）之间的评估顺序
m：= map [int] int {a：1，a：2} // m可能为{2：1}或{2：2}：未指定两个地图分配之间的评估顺序
n：= map [int] int {a：f（）} // n可以为{2：3}或{3：3}：未指定键和值之间的评估顺序
在程序包级别，初始化相关性会覆盖单个初始化表达式的从左到右规则，但不会覆盖每个表达式中的操作数：

var a，b，c = f（）+ v（），g（），sqr（u（））+ v（）

func f（）int {return c}
func g（）int {返回}
func sqr（x int）int {return x * x}

//函数u和v独立于所有其他变量和函数
该函数调用发生的顺序 u()，sqr()，v()， f()，v()，和g()。

根据运算符的关联性评估单个表达式内的浮点运算。显式括号会通过覆盖默认关联性来影响评估。在表达式中x + (y + z)，加法y + z 在加法之前执行x。

陈述
语句控制执行。

声明 =
     声明 | 标签为stmt | SimpleStmt |
    GoStmt | ReturnStmt | BreakStmt | ContinueStmt | GotoStmt |
    FallthroughStmt | 块 | IfStmt | SwitchStmt | SelectStmt | ForStmt |
    DeferStmt。

SimpleStmt = EmptyStmt | ExpressionStmt | SendStmt | IncDecStmt | 作业 | ShortVarDecl。
终止声明
一个终止声明之后，它词汇出现在相同的所有语句的执行防止块。以下语句终止：

一个“返回”或 “GOTO”语句。
调用内置函数 panic。
一个块中的语句列表中终止语句结束。
一个“如果”的声明，其中：
“ else”分支存在，并且
两个分支都是终止语句。
一个“为”的声明，其中：
没有引用“ for”语句的“ break”语句，并且
循环条件不存在。
一个 “ switch”语句，其中：
没有引用“ switch”语句的“ break”语句，
有默认情况，并且
该语句在每种情况下（包括默认值）都以终止语句或可能标记为“ fallthrough”的语句结尾。
一个“选择”的声明，其中：
没有引用“ select”语句的“ break”语句，并且
每种情况下的语句列表（包括缺省值，如果有的话）以终止语句结尾。
带标签的语句标记终止语句。
所有其他语句不终止。

一个语句列表中终止语句结束，如果列表不为空，其最终的非空语句被终止。

空语句
空语句不执行任何操作。

EmptyStmt =。
带标签的声明
带标签的语句可能是一个目标goto， break或continue声明。

LabeledStmt = 标签 “：” 语句。
标签        = 标识符。
错误：log.Panic（“遇到错误”）
表达陈述
除特定的内置函数外，函数和方法调用以及 接收操作 可以出现在语句上下文中。此类声明可以用括号括起来。

ExpressionStmt = 表达式。
语句上下文中不允许使用以下内置函数：

附加帽复杂imag len使新的真实
unsafe.Alignof不安全.Offsetof不安全.Sizeof
h（x + y）
f。关闭（）
<-ch
（<-ch）
len（“ foo”）//如果len是内置函数则非法
发送陈述
send语句在通道上发送值。通道表达式必须是通道类型，通道方向必须允许发送操作，并且要发送的值的类型必须可分配 给通道的元素类型。

SendStmt = 通道 “ <-” 表达式。
Channel   = 表达式。
在通信开始之前，将对通道和值表达式进行评估。通信将阻塞，直到发送可以继续。如果接收器准备就绪，则可以在无缓冲通道上进行发送。如果缓冲区中有空间，则可以在缓冲通道上进行发送。在封闭通道上进行发送会引起运行时恐慌。一个发送nil通道永远受阻。

ch <-3 //将值3发送到通道ch
IncDec语句
“ ++”和“-”语句通过未类型化的常量 增加或减少其操作数1。与赋值一样，操作数必须是可寻址的 或映射索引表达式。

IncDecStmt = 表达式（“ ++” |“-”）。
以下赋值语句在语义上是等效的：

IncDec语句赋值
x ++ x + = 1
x-- x-= 1
作业
分配 = ExpressionList  Assign_op  ExpressionList。

Assign_op = [ add_op | mul_op ]“ =”。
每个左侧操作数都必须是可寻址的，映射索引表达式或（=仅用于赋值） 空白标识符。操作数可以用括号括起来。

x = 1
* p = f（）
a [i] = 23
（k）= <-ch //与：k = <-ch
一个赋值操作 x 运算= y，其中OP是二进制算术运算符 相当于x = x 运算 (y)，但求x 唯一的一次。的运算=构建体是单令牌。在赋值操作中，左手表达式列表和右手表达式列表都必须恰好包含一个单值表达式，并且左手表达式不能为空标识符。

a [i] << = 2
i＆^ = 1 << n
元组分配将多值操作的各个元素分配给变量列表。有两种形式。首先，右侧操作数是单个多值表达式，例如函数调用，通道或 映射操作或类型断言。左侧的操作数数量必须与值的数量匹配。例如，如果 f是一个返回两个值的函数，

x，y = f（）
将第一个值分配给x，将第二个值分配给y。在第二种形式中，左边的操作数的数量必须等于右边的表达式的数量，每个表达式都必须是单值，并且 右边的第n个表达式被分配给左边的第n个操作数：

一，二，三='一'，'二'，'三'
所述坯件标识符提供了一种在分配忽略右侧的值：

_ = x //计算x但忽略它
x，_ = f（）//计算f（）但忽略第二个结果值
作业分两个阶段进行。首先，左侧的索引表达式 和指针间接操作 （包括选择器中的隐式指针间接操作）和右侧的表达式的操作数均按 通常的顺序求值。其次，分配是从左到右执行的。

a，b = b，a //交换a和b

x：= [] int {1、2、3}
i：= 0
i，x [i] = 1，2 //设置i = 1，x [0] = 2

我= 0
x [i]，i = 2，1 //设置x [0] = 2，i = 1

x [0]，x [0] = 1、2 //设置x [0] = 1，然后x [0] = 2（所以x [0] == 2结束）

x [1]，x [3] = 4，5 //设置x [1] = 4，然后紧急设置x [3] = 5。

输入Point struct {x，y int}
var p *点
x [2]，px = 6，7 //设置x [2] = 6，然后紧急设置px = 7

我= 2
x = [] int {3，5，7}
对于i，x [i] =范围x {//设置i，x [2] = 0，x [0]
    打破
}
//在此循环之后，i == 0和x == [] int {3，5，3}
在分配中，每个值都必须可分配 给为其分配了操作数的类型，并具有以下特殊情况：

任何键入的值都可以分配给空白标识符。
如果将未类型化的常量分配给接口类型或空白标识符的变量，则该常量首先隐式转换为其 默认类型。
如果将未类型化的布尔值分配给接口类型的变量或空标识符，则首先将其隐式转换为type bool。
如果陈述
“ If”语句根据布尔表达式的值指定两个分支的条件执行。如果表达式的计算结果为true，则执行“ if”分支，否则，如果存在，则执行“ else”分支。

IfStmt =“ if” [ SimpleStmt “;” ] 表达式 块 [“ else”（IfStmt | Block）]。
如果x> max {
    x =最大
}
该表达式之前可能有一个简单的语句，该语句在对表达式求值之前执行。

如果x：= f（）; x <y {
    返回x
}如果x> z {
    返回z
}其他{
    返回y
}
切换语句
“ Switch”语句提供多路执行。将表达式或类型说明符与“ switch”内的“ case”进行比较，以确定要执行的分支。

SwitchStmt = ExprSwitchStmt | TypeSwitchStmt。
有两种形式：表达式开关和类型开关。在表达式开关中，个案包含与开关表达式的值进行比较的表达式。在类型开关中，案例包含与专门注释的开关表达式的类型进行比较的类型。switch表达式在switch语句中仅计算一次。

表达开关
在表达式开关中，对开关表达式进行求值，而不需要为常量的case表达式则从左至右和从上至下进行求值；第一个等于switch表达式的触发器触发相关案例的语句的执行；其他情况将被跳过。如果没有大小写匹配并且存在“默认”大小写，则执行其语句。默认情况下最多可以有一种情况，它可能出现在“ switch”语句中的任何位置。缺少的switch表达式等效于boolean值 true。

ExprSwitchStmt =“开关” [ SimpleStmt “;” ] [ 表达式 ]“ {” { ExprCaseClause }“}”。
ExprCaseClause = ExprSwitchCase “：” StatementList。
ExprSwitchCase = “案例” 表达式列表 | “默认”。
如果switch表达式的计算结果为无类型的常量，则首先将其隐式 转换为其默认类型；如果它是无类型的布尔值，则首先将其隐式转换为type bool。预声明的无类型值nil不能用作开关表达式。

如果case表达式是无类型的，则首先将其隐式转换 为switch表达式的类型。对于每个（可能是转换的）case表达式x和t switch表达式的值，x == t必须是有效的比较。

换句话说，将switch表达式视为用于声明和初始化t没有显式类型的临时变量。它是t针对每个case表达式x测试是否相等的值。

在case或default子句中，最后一个非空语句可以是一个（可能标记为） “ fallthrough”语句，以指示控制权应从该子句的末尾流到下一个子句的第一个语句。否则，控制流到“ switch”语句的末尾。除了表达式开关的最后一个子句外，“ fallthrough”语句可能作为所有其他语句的最后一个语句出现。

开关表达式之前可以有一个简单的语句，该语句在计算表达式之前执行。

切换标签{
默认值：s3（）
情况0、1、2、3：s1（）
情况4、5、6、7：s2（）
}

切换x：= f（）; {//缺少的开关表达式表示“ true”
情况x <0：返回-x
默认值：返回x
}

切换{
例x <y：f1（）
例x <z：f2（）
案例x == 4：f3（）
}
实现限制：编译器可能不允许多个case表达式求值相同的常量。例如，当前的编译器在case表达式中不允许重复的整数，浮点数或字符串常量。

类型开关
类型开关比较类型而不是值。否则它类似于表达式开关。它由一个特殊的开关表达式标记，该表达式具有 使用保留字而不是实际类型的类型断言的形式type：

切换x。（类型）{
//案例
}
然后，案例将实际类型T与表达式的动态类型进行匹配x。与类型断言一样，x必须是 接口类型，并且T案例中列出的每个非接口类型都 必须实现的类型x。在类型开关的情况下列出的类型必须全部 不同。

TypeSwitchStmt   =“开关” [ SimpleStmt “;” ] TypeSwitchGuard “ {” { TypeCaseClause }“}”。
TypeSwitchGuard = [ 标识符 “：=”] PrimaryExpr “。“（”“ type”“）”。
TypeCaseClause   = TypeSwitchCase “：” StatementList。
TypeSwitchCase   =“ case”类型列表 | “默认”。
TypeList         = 类型 {“，” 类型 }。
TypeSwitchGuard可以包含一个 简短的变量声明。使用该格式时，该变量在每个子句的隐式块的TypeSwitchCase的末尾声明。在大小写正好列出一种类型的子句中，变量具有该类型。否则，该变量具有TypeSwitchGuard中表达式的类型。

案例可以使用预先声明的标识符来代替类型 nil；当TypeSwitchGuard中的表达式是nil接口值时，将选择这种情况。最多可能有一种nil情况。

给定xtype 的表达式interface{}，以下类型切换：

开关i：= x。（type）{
零案：
    printString（“ x is nil”）// i的类型是x的类型（interface {}）
case int：
    printInt（i）// i的类型为int
案例float64：
    printFloat64（i）//我的类型是float64
case func（int）float64：
    printFunction（i）//我的类型是func（int）float64
大小写布尔值，字符串：
    printString（“ type is bool or string”）// i的类型为x的类型（interface {}）
默认：
    printString（“不知道类型”）// i的类型是x的类型（interface {}）
}
可以改写为：

v：= x // x仅被评估一次
如果v == nil {
    i：= v // i的类型是x的类型（interface {}）
    printString（“ x为零”）
}否则，如果i，isInt：= v。（int）; isInt {
    printInt（i）// i的类型为int
}否则，如果i，isFloat64：= v。（float64）; isFloat64 {
    printFloat64（i）//我的类型是float64
}否则，如果i，isFunc：= v。（func（int）float64）; isFunc {
    printFunction（i）//我的类型是func（int）float64
}其他{
    _，isBool：= v。（布尔）
    _，isString：= v。（字符串）
    如果isBool || isString {
        i：= v // i的类型是x的类型（interface {}）
        printString（“类型为布尔值或字符串”）
    }其他{
        i：= v // i的类型是x的类型（interface {}）
        printString（“不知道类型”）
    }
}
类型切换防护的前面可以有一个简单的语句，该语句在评估防护之前执行。

类型切换中不允许使用“ fallthrough”语句。

对于声明
“ for”语句指定重复执行一个块。有三种形式：迭代可以由单个条件，“ for”子句或“ range”子句控制。

ForStmt =“ for” [ 条件 | 条款 | RangeClause ] 块。
条件 = 表达式。
对于单一条件的语句
最简单的形式是，“ for”语句指定重复执行一个块，只要布尔条件的计算结果为true。在每次迭代之前评估条件。如果不存在该条件，则等于布尔值 true。

对于<b {
    * = 2
}
对于带有for子句的语句
带有ForClause的“ for”语句也受其条件控制，但是另外，它可以指定init 和post语句，例如赋值，递增或递减语句。init语句可以是 简短的变量声明，但post语句不能。由init语句声明的变量在每次迭代中都会重复使用。

ForClause = [ InitStmt ]“;” [ 条件 ]“;” [ PostStmt ]。
InitStmt = SimpleStmt。
PostStmt = SimpleStmt。
对于我：= 0; 我<10; 我++ {
    （
}
如果为非空，则在评估第一次迭代的条件之前执行一次init语句；每次执行该块后（仅在执行该块时）才执行post语句。ForClause的任何元素都可以为空，但是 除非只有一个条件，否则分号是必需的。如果不存在该条件，则等于布尔值 true。

for cond {S（）}与for相同；条件 ; {S（）}
{S（）}与true {S（）}相同
对于带有range子句的语句
带有“ range”子句的“ for”语句遍历数组，切片，字符串或映射的所有条目，或通道上接收到的值。对于每个条目，它将迭代值分配 给相应的迭代变量（如果存在），然后执行该块。

RangeClause = [ ExpressionList “ =” | IdentifierList “：=”]“范围” Expression。
“范围”子句右侧的表达式称为范围表达式，它可以是数组，指向数组的指针，切片，字符串，映射或允许接收操作的通道 。与赋值一样，如果存在赋值，则左侧的操作数必须是 可寻址的或映射索引表达式。它们表示迭代变量。如果范围表达式是一个通道，则最多允许一个迭代变量，否则最多可以有两个。如果最后一个迭代变量是空白标识符，则range子句等效于没有该标识符的同一子句。

范围表达式x在开始循环之前先评估一次，但有一个例外：如果最多存在一个迭代变量且该变量 len(x)是常量，则不评估范围表达式。

左侧的函数调用每次迭代评估一次。对于每个迭代，如果存在各自的迭代变量，则会按以下方式生成迭代值：

范围表达式1值2值

数组或切片[n] E，* [n] E或[] E索引i int a [i] E
字符串s字符串类型索引i int参见下面的符文
map m map [K] V键k K m [k] V
通道c通道E，<-通道E元素e E
对于数组，指向数组的指针或切片值a，索引迭代值从元素索引0开始以递增顺序生成。如果存在最多一个迭代变量，则范围循环会生成从0到的迭代值，len(a)-1而不会索引到数组或切片本身。对于nil切片，迭代次数为0。
对于字符串值，“ range”子句从字节索引0开始在字符串中的Unicode代码点上进行迭代。在连续迭代中，索引值将是UTF-8编码的连续代码点中第一个字节的索引。字符串，第二个类型rune的值将是相应代码点的值。如果迭代遇到无效的UTF-8序列，则第二个值将是0xFFFDUnicode替换字符，而下一个迭代将在字符串中前进单个字节。
未指定地图的迭代顺序，并且不能保证每次迭代之间都相同。如果在迭代过程中删除了尚未到达的映射条目，则不会生成相应的迭代值。如果在迭代过程中创建了映射条目，则该条目可能在迭代过程中产生或可以被跳过。对于创建的每个条目以及从一个迭代到下一个迭代，选择可能有所不同。如果映射为nil，则迭代次数为0。
对于通道，所产生的迭代值是在通道上发送的连续值，直到通道关闭为止。如果channel为nil，则范围表达式将永远阻塞。
如赋值语句中所述，将迭代值分配给各个迭代变量。

迭代变量可以使用简短变量声明 （:=）的形式由“ range”子句 声明。在这种情况下，将它们的类型设置为各个迭代值的类型，并且它们的范围是“ for”语句的块；它们在每次迭代中都会重复使用。如果迭代变量在“ for”语句之外声明，则执行后它们的值将是上一次迭代的值。

var testdata * struct {
    * [7] int
}
对于我，_：= range testdata.a {
    //永远不会评估testdata.a；len（testdata.a）是常数
    //我的范围是0到6
    （
}

var [10]字符串
对于i，s：=范围a {
    //我的类型是int
    //的类型是字符串
    // s == a [i]
    g（i，s）
}

var键字串
var val interface {} // m的元素类型可分配给val
m：= map [string] int {“ mon”：0，“ tue”：1，“ wed”：2，“ thu”：3，“ fri”：4，“ sat”：5，“ sun”：6 }
对于密钥，val =范围m {
    h（键，val）
}
// key ==迭代中遇到的最后一个映射键
// val == map [key]

var ch chan Work = producer（）
对于w：=范围ch {
    doWork（w）
}

//清空频道
对于范围ch {}
转到语句
“ go”语句作为一个独立的并发控制线程（或goroutine）在同一地址空间内开始执行函数调用。

GoStmt =“ go” 表达式。
表达式必须是函数或方法调用；不能用括号括起来。内置函数的调用与表达式语句一样受到限制 。

函数值和参数是 在调用goroutine中照常计算的，但是与常规调用不同，程序执行不等待调用的函数完成。相反，该函数开始在新的goroutine中独立执行。当函数终止时，其goroutine也终止。如果函数具有任何返回值，则在函数完成时将其丢弃。

去Server（）
go func（ch chan <-bool）{for {sleep（10）; ch <-true}}（c）
选择语句
“选择”语句选择一组可能的发送或 接收 操作中的哪一个 进行。它看起来类似于 “ switch”语句，但所有情况均涉及通信操作。

SelectStmt =“选择”“ {” { CommClause }“}”。
CommClause = CommCase “：” StatementList。
CommCase    =“ case”（SendStmt | RecvStmt）| “默认”。
RecvStmt    = [ ExpressionList “ =” | IdentifierList “：=”] RecvExpr。
RecvExpr    = 表达式。
带有RecvStmt的案例可以将RecvExpr的结果分配给一个或两个变量，可以使用 短变量声明来声明。RecvExpr必须是（可能带有括号）接收操作。最多可以有一个默认案例，它可能出现在案例列表中的任何位置。

“ select”语句的执行分几个步骤进行：

对于语句中的所有情况，输入“ select”语句后，接收操作的通道操作数以及send语句的通道表达式和右侧表达式将按源顺序进行一次精确评估。结果是一组要从中接收或发送到的通道，以及要发送的相应值。不管选择进行哪个通信操作，都会发生该评估中的任何副作用。带有简短变量声明或赋值的RecvStmt左侧的表达式尚未评估。
如果可以进行一种或多种通信，则可以通过统一的伪随机选择来选择可以进行的单个通信。否则，如果存在默认情况，则选择该情况。如果没有默认情况，则“ select”语句将阻塞，直到可以进行至少一种通信为止。
除非所选情况是默认情况，否则将执行相应的通信操作。
如果所选案例是带有简短变量声明或赋值的RecvStmt，则将评估左侧表达式并分配接收的值（或多个值）。
执行所选案例的语句列表。
由于nil通道上的通信永远无法进行，因此仅包含nil通道且没有默认情况的选择将永远被阻止。

var a [] int
var c，c1，c2，c3，c4 chan int
var i1，i2 int
选择 {
情况i1 = <-c1：
    print（“收到”，i1，“来自c1 \ n”）
情况c2 <-i2：
    print（“ sent”，i2，“ to c2 \ n”）
情况i3，好的：=（<-c3）：//与：i3，好的：= <-c3
    如果可以，{
        print（“ received”，i3，“来自c3 \ n”）
    }其他{
        打印（“ C3已关闭\ n”）
    }
情况a [f（）] = <-c4：
    // 和...一样：
    //情况t：= <-c4
    // a [f（）] = t
默认：
    打印（“无通信\ n”）
}

对于{//将随机位序列发送给c
    选择 {
    case c <-0：//注意：没有声明，没有失败，没有案例折叠
    情况c <-1：
    }
}

选择{} //永远阻止
退货声明
函数中的“ return”语句F终止的执行F，并可选地提供一个或多个结果值。任何功能递延由F 之前执行F返回到它的调用者。

ReturnStmt =“ return” [ ExpressionList ]。
在没有结果类型的函数中，“ return”语句不能指定任何结果值。

func noResult（）{
    返回
}
有三种方法可以从具有结果类型的函数返回值：

一个或多个返回值可以在“ return”语句中明确列出。每个表达式必须是单值的，并且可以分配 给函数结果类型的相应元素。
func simpleF（）int {
    返回2
}

func complexF1（）（re float64，im float64）{
    返回-7.0，-4.0
}
“ return”语句中的表达式列表可以是对多值函数的单个调用。效果好像是将从该函数返回的每个值都分配给具有相应值类型的临时变量，然后是列出这些变量的“ return”语句，此时适用前一种情况的规则。
func complexF2（）（re float64，im float64）{
    返回complexF1（）
}
如果函数的结果类型为其结果参数指定名称，则表达式列表可能为空。结果参数充当普通的局部变量，并且函数可以根据需要为其分配值。“ return”语句返回这些变量的值。
func complexF3（）（re float64，im float64）{
    重新= 7.0
    即时通讯= 4.0
    返回
}

func（devnull）Write（p [] byte）（n int，_ error）{
    n = len（p）
    返回
}
无论如何声明它们，所有结果值在输入函数时都将初始化为其类型的零值。指定结果的“ return”语句在执行任何延迟函数之前会设置结果参数。

实现限制：如果与返回参数同名的其他实体（常量，类型或变量）在范围之内，则编译器可能不允许在“返回”语句中使用空表达式列表 。

func f（n int）（res int，err error）{
    如果_，err：= f（n-1）; err！= nil {
        return //无效的return语句：err被遮盖
    }
    返回
}
中断声明
“ break”语句终止同一函数内最里面的 “ for”， “ switch”或 “ select”语句的执行。

BreakStmt =“ break” [ 标签 ]。
如果有标签，则必须是封闭的“ for”，“ switch”或“ select”语句的标签，并且该标签的执行终止。

外环：
    当我= 0; 我<n; 我++ {
        当j = 0; j <m; j ++ {
            切换a [i] [j] {
            零案：
                状态=错误
                打破外环
            案例：
                状态=找到
                打破外环
            }
        }
    }
继续陈述
“ continue”语句在其post语句处开始最里面的“ for”循环的下一次迭代。“ for”循环必须在同一函数内。

ContinueStmt =“继续” [ Label ]。
如果有一个标签，则必须是一个封闭的“ for”语句的标签，并且该标签将执行。

行循环：
    对于y，行：=范围行{
        对于x，数据：=范围行{
            如果数据== endOfRow {
                继续RowLoop
            }
            行[x] =数据+偏差（x，y）
        }
    }
转到语句
“ goto”语句将控制权转移到同一函数中带有相应标签的语句。

GotoStmt =“ goto” 标签。
转到错误
执行“ goto”语句一定不能使任何变量在goto时尚未进入 范围。例如，此示例：

    转到L //错误
    v：= 3
L：
是错误的，因为跳转到标签会L跳过的创建v。

块 外的“ goto”语句不能跳转到该块内的标签。例如，此示例：

如果n％2 == 1 {
    转到L1
}
对于n> 0 {
    F（）
    n--
L1：
    F（）
    n--
}
是错误的，因为标签L1位于“ for”语句的块内，而标签goto不在。

失败陈述
“ fallthrough”语句将控制权转移到表达式“ switch”语句中下一个case子句的第一个语句。它只能用作该子句中的最终非空语句。

FallthroughStmt =“ fallthrough ”。
推迟陈述
“ defer”语句调用一个函数，该函数的执行被推迟到周围函数返回的那一刻，这是因为周围函数执行了一个return语句，到达了函数体的末尾，或者是因为相应的goroutine正在惊慌。

DeferStmt =“ defer” 表达式。
表达式必须是函数或方法调用；不能用括号括起来。内置函数的调用与表达式语句一样受到限制 。

每次执行“ defer”语句时，都会照常评估调用的函数值和参数 并重新保存，但不会调用实际函数。而是，在周围的函数返回之前，立即以延迟的相反顺序调用延迟的函数。也就是说，如果周围的函数通过显式的return语句返回，则在该return语句设置任何结果参数之后但在函数返回其调用者之前，将执行延迟的函数。如果将延迟函数的值计算为nil， 则在调用该函数时（而不是在执行“ defer”语句时），执行会出现紧急情况。

例如，如果延迟函数是函数文字，并且周围函数已命名结果参数在文字范围内，则延迟函数可以在返回结果参数之前对其进行访问和修改。如果延迟函数具有任何返回值，则在函数完成时将其丢弃。（另请参阅“ 处理恐慌 ”部分。）

锁（l）
defer unlock（l）//解锁发生在周围的函数返回之前

//在周围函数返回之前打印3 2 1 0
对于我：= 0; 我<= 3; 我++ {
    延迟fmt.Print（i）
}

// f返回42
func f（）（结果int）{
    延迟func（）{
        //通过return语句将结果设置为6后访问结果
        结果* = 7
    }（）
    返回6
}
内建功能
内置函数是 预先声明的。它们像任何其他函数一样被调用，但是其中一些接受类型而不是表达式作为第一个参数。

内置函数没有标准的Go类型，因此它们只能出现在调用表达式中；它们不能用作函数值。

关
对于通道c，内置函数close(c) 记录该通道将不再发送任何值。如果c是仅接收通道，则错误。发送到或关闭已关闭的通道会导致运行时恐慌。关闭nil通道也会引起运行时恐慌。调用之后close，并且在接收到任何先前发送的值之后，接收操作将返回通道类型的零值而不会阻塞。多值接收操作 返回接收值以及通道是否关闭的指示。

长度和容量
内置函数len和cap接受各种类型的参数并返回type的结果int。该实现可确保结果始终适合int。

调用参数类型结果

len字符串类型字符串长度（以字节为单位）
          [n] T，* [n] T数组长度（== n）
          [] T片长
          map [K] T映射长度（已定义键的数量）
          chan T在通道缓冲区中排队的元素数

上限[n] T，* [n] T数组长度（== n）
          [] T切片容量
          通道通道缓冲容量
切片的容量是在基础数组中为其分配了空间的元素数。任何时候都保持以下关系：

0 <=镜头<=上限
nil切片，地图或通道 的长度为0。nil切片或通道的容量为0。

表达len(s)是常数，如果 s是字符串常量。如果和的类型是数组或指向数组的指针，并且表达式不包含 通道接收或（非恒定） 函数调用，则表达式len(s)和 cap(s)是常量。在这种情况下不进行评估。否则，调用和不恒定，并进行评估。 ssslencaps

const（
    c1 = imag（2i）// imag（2i）= 2.0是一个常数
    c2 = len（[10] float64 {2}）// [10] float64 {2}不包含任何函数调用
    c3 = len（[10] float64 {c1}）// [10] float64 {c1}不包含任何函数调用
    c4 = len（[10] float64 {imag（2i）}）// imag（2i）是一个常量，不会发出任何函数调用
    c5 = len（[10] float64 {imag（z）}）//无效：imag（z）是（非恒定）函数调用
）
var z complex128
分配
内置函数new采用类型T，在运行时为该类型的变量分配存储，并返回*T 指向该类型的值。变量按照有关初始值的部分中的说明进行 初始化。

新（T）
例如

类型S struct {a int; b float64}
新闻）
为类型的变量分配存储空间S，对其进行初始化（a=0，b=0.0），然后返回*S包含位置地址的类型的值。

制作切片，地图和通道
内置函数make采用type T，它必须是切片，映射或通道类型，并可以选择后面跟特定类型的表达式列表。它返回类型的值T（不是*T）。存储器按照有关初始值的部分所述进行 初始化。

通话类型T结果

长度为n且容量为n的T类型的make（T，n）切片
长度为n且容量为m的T类型的make（T，n，m）切片

T型的make（T）映射图
make（T，n）类型T的映射图，其初始空间约为n个元素

T类型的make（T）通道无缓冲通道
make（T，n）类型为T的通道缓冲通道，缓冲区大小n
每个的大小参数n和m必须是整数类型或无类型的常数。大小不变的参数必须是非负数，并且可以 由type的值表示int；如果它是未类型的常量，则指定为type int。如果n和和m都提供且为常数，则 n其不得大于m。如果n为负或大于m运行时，则会发生运行时恐慌。

s：= make（[] int，10，100）//镜头的len（s）== 10，cap（s）== 100
s：= make（[] int，1e3）//带有len的切片== cap（s）== 1000
s：= make（[] int，1 << 63）//非法：len（s）无法用int类型的值表示
s：= make（[] int，10，0）//非法：len（s）> cap（s）
c：= make（chan int，10）//缓冲区大小为10的通道
m：= make（map [string] int，100）//以大约100个元素的初始空间进行映射
make使用地图类型和大小提示进行 调用n将创建一个具有初始空间的地图，以容纳n地图元素。精确的行为取决于实现。

附加并复制切片
内置功能append并copy协助常见的分片操作。对于这两个函数，结果与参数所引用的内存是否重叠无关。

所述可变参数函数append 追加零个或多个值x ，以s类型的S，它必须是一个切片类型，并返回型也所得到的切片，S。值x将传递到类型为的参数，...T 其中T是的元素类型， S并且适用各自的 参数传递规则。作为一种特殊情况，append还接受可分配给type []byte的第一个参数，其后是字符串类型的第二个参数...。这种形式附加了字符串的字节。

append（s S，x ... T）S // T是S的元素类型
如果的容量s不足以容纳附加值，则append分配一个新的，足够大的基础数组，使其既适合现有slice元素又适合附加值。否则，请append重新使用基础数组。

s0：= [] int {0，0}
s1：= append（s0，2）//附加单个元素s1 == [] int {0，0，2}
s2：= append（s1，3，5，7）//附加多个元素s2 == [] int {0，0，2，3，5，7}
s3：= append（s2，s0 ...）//附加切片s3 == [] int {0，0，2，3，5，7，0，0}
s4：= append（s3 [3：6]，s3 [2：] ...）//追加重叠的切片s4 == [] int {3，5，7，2，2，3，5，7，0，0 }

var t [] interface {}
t = append（t，42，3.1415，“ foo”）// t == [] interface {} {42，3.1415，“ foo”}

var b [] byte
b = append（b，“ bar” ...）//追加字符串内容b == [] byte {'b'，'a'，'r'}
该函数将copy切片元素从源复制src到目标，dst并返回复制的元素数。这两个参数必须具有相同的元素类型，T并且必须可 分配给type的切片[]T。复制的元素的数目是最小 len(src)和len(dst)。作为一种特殊情况，copy还接受可分配给type []byte并带有字符串类型source参数的destination参数。这种形式将字节从字符串复制到字节片中。

复制（dst，src [] T）int
复制（dst [] byte，src字符串）int
例子：

var a = [... int] {0，1，2，3，4，5，6，7}
var s = make（[] int，6）
var b = make（[] byte，5）
n1：= copy（s，a [0：]）// n1 == 6，s == [] int {0，1，2，3，4，5}
n2：= copy（s，s [2：]）// n2 == 4，s == [] int {2，3，4，4，5，4，5}
n3：= copy（b，“ Hello，World！”）// n3 == 5，b == [] byte（“ Hello”）
删除地图元素
内置函数从地图中delete删除带有键的元素 。的类型必须可分配 给的密钥类型。 k mkm

delete（m，k）//从映射m中删除元素m [k]
如果地图m是nil或元素m[k] 不存在，delete则为no-op。

处理复数
三个函数组合和分解复数。内置的功能complex构建从浮点实部和虚部的复数值，而 real与imag 提取的复数值的实部和虚部。

complex（realPart，imaginaryPart floatT）complexT
实数（complexT）floatT
imag（complexT）floatT
参数的类型和返回值相对应。对于complex，两个参数必须具有相同的浮点类型，并且返回类型是具有相应浮点组成的复杂类型： complex64用于float32参数， complex128用于float64参数。如果其中一个参数求值为无类型常量，则首先将其隐式 转换为另一个参数的类型。如果两个参数都求值为无类型的常量，则它们必须为非复数或虚部必须为零，并且函数的返回值为无类型的复数常量。

对于real和imag，参数必须是复杂类型，并且返回类型是对应的浮点类型：float32对于complex64参数， float64对于complex128参数。如果参数的计算结果为无类型常量，则该参数必须为数字，并且函数的返回值为无类型浮点常量。

的real和imag功能一起形成的逆 complex，所以对一个值z的复杂类型的Z， z == Z(complex(real(z), imag(z)))。

如果这些函数的操作数均为常数，则返回值为常数。

var a = complex（2，-2）// complex128
const b = complex（1.0，-1.4）//无类型的复数常数1-1.4i
x：= float32（math.Cos（math.Pi / 2））// float32
var c64 = complex（5，-x）// complex64
var s int = complex（1，0）//无类型的复数常量1 + 0i可以转换为int
_ = complex（1，2 << s）//非法：2假定为浮点类型，不能移位
var rl = real（c64）// float32
var im = imag（a）// float64
const c = imag（b）//无类型常量-1.4
_ = imag（3 << s）//非法：3假定为复杂类型，无法移位
处理恐慌
有两个内置函数panic和recover，它们有助于报告和处理运行时的紧急情况 和程序定义的错误情况。

func panic（interface {}）
func restore（）接口{}
执行函数时F，对的显式调用panic或运行时恐慌会 终止的执行F。 然后，照常执行任何延迟的功能F。接下来，将运行由F's调用者运行的所有延迟函数，依此类推，直到被执行的goroutine中顶级函数所延迟的一切。此时，程序将终止并报告错误情况，包括to的参数值panic。此终止序列称为panicking。

惊慌（42）
恐慌（“无法到达”）
恐慌（错误（“无法解析”））
该recover功能允许程序管理恐慌性goroutine的行为。假设一个函数G推迟了D调用的函数， recover并且在执行该例程的同一个goroutine上的函数中发生了恐慌G 。当递延函数的运行达到时D，对D的调用的返回值recover将是传递给对的调用的值panic。如果D正常返回而没有开始新的 panic，则恐慌序列停止。在这种情况下，在之间G调用和调用的函数的状态将panic 被丢弃，并且恢复正常执行。然后，G之前D运行的所有函数都将运行，并且G通过返回其调用方来终止其执行。

如果满足以下任一条件 ，recover则返回值为nil：

panic的观点是nil;
goroutine不会惊慌；
recover 不是由延迟函数直接调用的。
的protect在下面调用示例性函数的参数函数g并保护通过升高运行时恐慌呼叫者g。

函数保护（g func（））{
    延迟func（）{
        log.Println（“ done”）//即使发生紧急情况，Println也可以正常执行
        如果x：= recovery（）; x！= nil {
            log.Printf（“运行时恐慌：％v”，x）
        }
    }（）
    log.Println（“开始”）
    G（）
}
自举
当前的实现提供了一些自举过程中有用的内置函数。这些功能已记录完整，但不能保证始终保留该语言。他们不返回结果。

功能行为

print打印所有参数；参数的格式是特定于实现的
类似于println的println，但在参数和换行符的最后打印空格
实施限制：print而println不用接受任意参数类型，但布尔，数字和字符串的打印 类型必须得到支持。

配套
Go程序是通过将程序包链接在一起而构造的。一个包又由一个或多个源文件构成，它们一起声明了属于该包的常量，类型，变量和函数，并且可以在同一包的所有文件中访问它们。这些元素可以 导出并在另一个包中使用。

源文件组织
每个源文件都由一个package子句组成，该子句定义了它所属的package，其后是一组可能为空的导入声明，这些声明声明了要使用其内容的包，然后是一组可能为空的函数，类型，变量，和常数。

SourceFile        = PackageClause “;” { ImportDecl “;” } { TopLevelDecl “;” }。
包装条款
package子句从每个源文件开始，并定义该文件所属的软件包。

PackageClause   =“ package” PackageName。
PackageName     = 标识符。
PackageName不能为空白标识符。

套餐数学
共享相同PackageName的一组文件构成了一个包的实现。一个实现可能要求包的所有源文件都驻留在同一目录中。

进口报关单
导入声明声明包含该声明的源文件取决于导入包的功能（§Program初始化和执行），并允许访问该包的导出标识符。导入命名用于访问的标识符（PackageName）和指定要导入的软件包的ImportPath。

ImportDecl        =“ import”（ImportSpec |“（” { ImportSpec “;”}“）”）。
ImportSpec        = [“。| PackageName ] ImportPath。
ImportPath        = string_lit。
PackageName用于合格标识符中， 以访问导入源文件中包的导出标识符。它在文件块中声明。如果省略PackageName，则默认为导入包的package子句中指定的标识符 。如果出现一个明确的句点（.）而不是名称，则在该软件包的package块中声明的所有软件包导出标识符 将在导入源文件的file块中声明，并且必须在不使用限定符的情况下进行访问。

ImportPath的解释取决于实现，但是它通常是已编译软件包的完整文件名的子字符串，并且可能与已安装软件包的存储库有关。

实现限制：编译器可以仅使用属于Unicode的 L，M，N，P和S常规类别的字符（无空格的图形字符）将ImportPaths限制为非空字符串 ，并且还可以排除字符 !"#$%&'()*,:;<=>?[\]^`{|} 和Unicode替换字符U + FFFD。

假设我们已经编译了一个包含package子句的程序包 package math，该子程序导出了功能Sin，并将编译后的程序包安装在标识的文件中 "lib/math"。下表说明了Sin在各种类型的导入声明之后导入软件包的文件中的访问方式。

进口申报单罪名

导入“ lib / math” math.Sin
导入m“ lib / math” m.Sin
进口。“ lib / math”罪
导入声明声明了导入和导入包之间的依赖关系。包直接或间接导入自身，或不引用其任何导出标识符直接导入包是非法的。要仅出于副作用（初始化）导入软件包，请使用空白 标识符作为显式软件包名称：

导入_“ lib / math”
一个示例包
这是一个完整的Go包，它实现了并发的主筛。

包主

导入“ fmt”

//将序列2、3、4，…发送到频道“ ch”。
func generate（ch chan <-int）{
    对于我：= 2; ; 我++ {
        ch <-i //向频道“ ch”发送“ i”。
    }
}

//将值从通道“ src”复制到通道“ dst”，
//删除可被“素数”整除的那些。
func filter（src <-chan int，dst chan <-int，prime int）{
    对于i：= range src {//循环从'src'接收的值。
        如果i％prime！= 0 {
            dst <-i //将“ i”发送到频道“ dst”。
        }
    }
}

//主筛：菊花链过滤器一起处理。
func sieve（）{
    ch：= make（chan int）//创建一个新频道。
    go generate（ch）//将generate（）作为子进程启动。
    为{
        素数：= <-ch
        fmt.Print（prime，“ \ n”）
        ch1：= make（chan int）
        去过滤器（ch，ch1，prime）
        ch = ch1
    }
}

func main（）{
    筛（）
}
程序初始化与执行
零值
当通过声明或调用来为变量分配存储空间new时，或者通过复合文字或调用来创建新值时make，并且未提供显式初始化时，将为变量或值提供默认值值。此类变量或值的每个元素都将其类型设置为零值：false布尔值， 0数字类型，"" 字符串以及nil指针，函数，接口，切片，通道和映射。该初始化是递归完成的，因此，例如，如果未指定任何值，则结构数组的每个元素的字段都将为零。

这两个简单的声明是等效的：

var i int
var i int = 0
后

输入T struct {i int; f float64; 下一个* T}
t：= new（T）
以下内容成立：

ti == 0
tf == 0.0
t.next ==零
之后也是如此

变数
包初始化
在包中，包级变量的初始化是逐步进行的，每个步骤都按照声明顺序最早选择变量，而该变量 与未初始化的变量无关。

更确切地说，一个包级变量被认为是准备好初始化，如果它尚未初始化，要么没有初始化表达式或初始化表达式没有依赖于未初始化的变量。通过重复初始化声明顺序中最早的下一个包级变量并准备初始化来进行初始化，直到没有变量准备初始化为止。

如果在此过程结束时仍未初始化任何变量，则这些变量是一个或多个初始化周期的一部分，并且该程序无效。

由右侧的单个（多值）表达式初始化的变量声明的左侧的多个变量将一起初始化：如果初始化左侧的任何变量，则所有这些变量都将初始化在同一步骤中。

var x = a
var a，b = f（）//在初始化x之前将a和b一起初始化
出于程序包初始化的目的，空白 变量与声明中的其他任何变量一样对待。

在多个文件中声明的变量的声明顺序由向编译器提供文件的顺序确定：在第一个文件中声明的变量先声明在第二个文件中声明的变量之前，依此类推。

依赖性分析不依赖于变量的实际值，而仅依赖于源中对它们的词法引用，并进行了传递性分析。例如，如果变量x的初始化表达式引用的函数的主体引用变量，y则x依赖于y。特别：

对变量或函数的引用是表示该变量或函数的标识符。
到的方法的参考m是一个 方法值或 方法表达的形式的 t.m，其中，（静态）类型的t不是一个接口类型，并且该方法m是在 方法集的t。是否t.m调用结果函数值 并不重要。
变量，函数，或方法x取决于变量 y如果x的初始化表达或体（对于函数和方法）包含到参考y 或到取决于函数或方法y。
例如，给定声明

var（
    a = c + b // == 9
    b = f（）// == 4
    c = f（）// == 5
    d = 3 // == 5初始化完成后
）

func f（）int {
    d ++
    返回d
}
初始化顺序是d，b，c，a。请注意，初始化表达式中子表达式的顺序无关紧要： 在本示例中a = c + b，a = b + c结果也相同。

对每个程序包执行依赖性分析；仅考虑引用在当前包中声明的变量，函数和（非接口）方法的引用。如果变量之间存在其他隐藏的数据依赖性，则未指定这些变量之间的初始化顺序。

例如，给定声明

var x = I（T {}）。ab（）// x对a和b具有未被检测到的隐藏依赖性
var _ = sideEffect（）//与x，a或b无关
var a = b
var b = 42

类型I接口{ab（）[] int}
输入T struct {}
func（T）ab（）[] int {return [] int {a，b}}
变量a将后进行初始化b，但是否x被初始化之前b，之间 b和a，或后a，并因此也°的瞬间sideEffect()是所谓的（之前或之后x被初始化）没有被指定。

也可以使用init 在程序包块中声明的命名函数（没有参数也没有结果参数）来初始化变量。

func init（）{…}
甚至可以在单个源文件中为每个包定义多个此类功能。在package块中，init标识符只能用于声明init函数，而标识符本身未声明。因此， init不能从程序中的任何位置引用功能。

通过将初始值分配给其所有程序包级变量来初始化不导入的程序包，然后init 按照它们在源代码中出现的顺序（可能在多个文件中）的顺序调用所有函数，以呈现给编译器。如果程序包已导入，则在初始化程序包本身之前初始化导入的程序包。如果有多个软件包导入一个软件包，则导入的软件包将仅初始化一次。通过构造的方式导入程序包可确保没有循环初始化依赖项。

程序包初始化-变量初始化和init函数调用 -一次在单个goroutine中顺序出现，一个程序包一次。一个init功能可能启动其他够程，可与初始化代码同时运行。但是，初始化总是对init函数进行排序：在前一个函数返回之前，它不会调用下一个函数。

为了确保可重现的初始化行为，鼓励构建系统以词法文件名的顺序向编译器提供属于同一软件包的多个文件。

程序执行
通过将单个未导入的程序包（称为主程序包）与该程序包所导入的所有程序包链接起来，可以创建一个完整的程序。主程序包必须具有程序包名称，main并声明一个main不带参数也不返回值的函数。

func main（）{…}
程序执行首先初始化主程序包，然后调用函数main。当该函数调用返回时，程序退出。它不等待其他（非main）goroutine完成。

失误
预声明的类型error定义为

类型错误界面{
    Error（）字符串
}
它是用于表示错误状态的常规接口，其中nil值表示没有错误。例如，可能定义了从文件读取数据的功能：

func Read（f * File，b [] byte）（n int，err错误）
运行时恐慌
执行错误（例如尝试对数组进行索引超出范围）会触发运行时恐慌，等效于使用panic 实现定义的接口类型的值调用内置函数runtime.Error。该类型满足预先声明的接口类型 error。未指定代表不同的运行时错误条件的确切错误值。

包运行时

键入错误接口{
    错误
    //以及其他方法
}
系统注意事项
包 unsafe
unsafe编译器已知并可以通过导入路径 访问 的内置包"unsafe"为低级编程提供了便利，其中包括违反类型系统的操作。使用的包装unsafe 必须经过人工审核，以确保类型安全，并且不可携带。该软件包提供以下接口：

包装不安全

type ArbitraryType int //任意Go类型的简写；这不是真正的类型
类型指针* ArbitraryType

func Alignof（variable ArbitraryType）uintptr
func Offsetof（selector ArbitraryType）uintptr
func Sizeof（variable ArbitraryType）uintptr
A Pointer是指针类型，但不能取消引用Pointer 值。基础类型的任何指针或值都可以转换为基础类型的类型，反之亦然。在和之间转换的效果是实现定义的。 uintptrPointerPointeruintptr

var f float64
位= *（* uint64）（不安全的Pointer（＆f））

输入ptr unsafe.Pointer
位= *（* uint64）（ptr（＆f））

var p ptr = nil
函数Alignof和Sizeof接受x 任何类型的表达式，并分别返回假设变量的对齐方式或大小，v 就像v通过声明了一样var v = x。

该函数Offsetof采用（可能带有括号的）选择器 s.f，表示f由s 或表示的结构*s的字段，并以字节为单位返回相对于结构地址的字段偏移量。如果f是一个嵌入式字段，则该字段必须是可访问的，而无需通过结构的字段进行指针间接调用。对于s带有字段的结构f：

uintptr（unsafe.Pointer（＆s））+ unsafe.Offsetof（sf）== uintptr（unsafe.Pointer（＆s.f））
计算机体系结构可能需要对齐内存地址; 也就是说，如果变量的地址是一个因子的倍数，则该变量的类型为alignment。该函数Alignof 采用表示任何类型变量的表达式，并以字节为单位返回（变量的类型）对齐方式。对于变量 x：

uintptr（unsafe.Pointer（＆x））％unsafe.Alignof（x）== 0
调用Alignof，Offsetof以及 Sizeof有型的编译时间常数表达式uintptr。

尺寸和对齐保证
对于数字类型，可以保证以下大小：

类型大小（以字节为单位）

字节，uint8，int8 1
uint16，int16 2
uint32，int32，float32 4
uint64，int64，float64，complex64 8
复杂128 16
保证以下最小对齐方式属性：

对于x任何类型的变量：unsafe.Alignof(x)至少为1。
对于x结构类型的变量：unsafe.Alignof(x)是的unsafe.Alignof(x.f)每个字段f的所有值中的最大值x，但至少为1。
对于x数组类型unsafe.Alignof(x)的变量：与数组元素类型的变量的对齐方式相同。
如果结构或数组类型不包含大小大于零的字段（或元素），则其大小为零。两个不同的零大小变量在内存中可能具有相同的地址。