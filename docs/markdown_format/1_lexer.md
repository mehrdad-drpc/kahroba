# مقدمه

کاری که میخواهیم انجام دهیم پردازش یک رشته متنی و اجرای دستورات درون آن است زبان برنامه نویسی ما هیچ درکی از اینکه این رشته های پشت سر هم چه معنی ای میدهد ندارد. پس باید بصورتی که قابل درک باشد برای کامپیوتر آن را تفسیر کنیم.

قدم اول این است که متن سورس کد را به حداقل کاراکترهای معنادار جدا کنیم. با این حداقل کاراکتر های معنا دار توکن گفته میشود.

 برای مثال دستور زیر را در نظر بگیرید

```go
if 10 > 5 {
 print("Hello")
}
```

حداقل کاراکترهای معنی دار در این مثال توکن های زیر هستند.

```go
if
10
>
5
{
print
"Hello"
}
```

همانطور که میبنید ما کلمه
`f ,i و if`
را به دو حرف تقسیم نمیکنیم چون به تنهایی معنایی ندارند.

ولی برای مثال توکن
`}`
به ما نشان میدهد که به شروع یک بلاک کد رسیده ایم.
ما در این مرحله کاراکتر به کاراکتر رشته را میخوانیم و چک میکنیم که آیا با کاراکتر بعدی یا قبلی توکن معناداری را شکل میدهند یا نه. اگر ترکیب شناخته شده ای پیدا شد آن را به لیست توکن ها اضافه میکنیم. در غیر اینصورت خود آن کاراکتر را به لیست توکن ها اضافه میکنیم. به این کار
`Lexical Analysis`
گفته میشود و کدی که مینویسیم
`Lexer`
یا
`Tokenizer`
نام دارد.

```go
package main

import (
 "bufio"
 "io"
 "strings"
 "unicode"
)
```

نوع داده جدیدی برای تشخیص نوع توکن تعریف میکنیم تا کد ما خوانایی بیشتری داشته باشد.

```go
type Type string
```

این استراکت نوع و مقدار توکن را در خود نگه میدارد.

برای مثال رشته های زیر توکن های متناظر را تولید میکنند.

```go
+ => {Type:PLUS,Value:"+"}
if => {Type:IF,Value:"if"}
*/
type Token struct {
 Type  Type
 Value string
}
```

در اینجا لیستی از تمام توکن های معتیر ایجاد میکنیم. میتوانیم نوع توکن ها را عدد و از نوع
`iota`
در نظر بگیریم ولی تعریف توکن ها بصورت رشته در زمان دیباگ و چاپ در خروجی به فهمیدن نوع توکن کمک بیشتری میکند.

یک توکن به نام
`EOF`
تعریف کردیم که نشان دهنده رسیدن به پایان فایل و اتمام توکن ها می‌باشد هر برنامه که که پردازش می‌کنیم حداقل این توکن را دارد حتی رشته خالی
.

```go
const (
 EOF       Type = "EOF"
 LPARENT        = "("
 RPARENT        = ")"
 LCURLY         = "{"
 RCURLY         = "}"
 LBRACKET       = "["
 RBRACKET       = "]"
 COMMA          = ","
 DOT            = "."
 PLUS           = "+"
 MINUS          = "-"
 STAR           = "*"
 SLASH          = "/"
 COLON          = ":"
 SEMICOLON      = ";"
 NOT            = "!"
 QUESTION       = "?"
 EQ             = "="
 EQEQ           = "=="
 NEQ            = "!="
 GREATER        = ">"
 LESSER         = "<"
 GEQ            = ">="
 LEQ            = "<="
 STRING         = "STRING"
 INT            = "INT"
 FLOAT          = "FLOAT"
 IDENT          = "IDENT"
 ERR            = "ERROR"
 TRUE           = "TRUE"
 FALSE          = "FALSE"
 IF             = "IF"
 ELSE           = "ELSE"
 FN             = "FN"
 PRINT          = "PRINT"
 PRINTLN        = "PRINTLN"
 RETURN         = "RETURN"
 FOR            = "FOR"
 DOTDOT         = ".."
 SWAP           = "SWAP"
 INPUT          = "INPUT"
 LEN            = "LEN"
 IMPORT         = "IMPORT"
 OR             = "OR"
 AND            = "AND"
)
```

لیستی از توکن هایی که برای زبان کهربا کلمه کلیدی محسوب میشود را درون یک مپ قرار میدهیم تا به وسیله آن راحت تر بتوانیم نوع کلمه را تشخیص بدهیم.

برای مثال اگر به کلمه
`hello`
برخورد کردیم باید بدانیم این یک متغیر است یا یک کلمه کلیدی.

```go
var keywords = map[string]Type{
 "fn":      FN,
 "print":   PRINT,
 "println": PRINTLN,
 "return":  RETURN,
 "true":    TRUE,
 "false":   FALSE,
 "if":      IF,
 "else":    ELSE,
 "for":     FOR,
 "swap":    SWAP,
 "input":   INPUT,
 "len":     LEN,
 "import":  IMPORT,
 "or":      OR,
 "and":     AND,
}
```

در زیر هم یک مپ از تمام کاراکترهای تکی یا جفتی که برای برنامه ما شناخته شده است
را لیست میکنیم تا اگر به هرکدام از این کاراکتر ها برخورد کردیم بتوانیم آن را تشخیص دهیم.

برای مثال کاراکتر
`@`
جزو کاراکترهای معنی دار در زبان کهربا نیست پس در این لیست جایی ندارد.

```go
var symobls = map[string]Type{
 "(":  LPARENT,
 ")":  RPARENT,
 "{":  LCURLY,
 "}":  RCURLY,
 "[":  LBRACKET,
 "]":  RBRACKET,
 ",":  COMMA,
 "..": DOTDOT,
 ".":  DOT,
 "+":  PLUS,
 "-":  MINUS,
 "*":  STAR,
 "/":  SLASH,
 ":":  COLON,
 "!":  NOT,
 "?":  QUESTION,
 "=":  EQ,
 "==": EQEQ,
 "!=": NEQ,
 ">":  GREATER,
 ">=": GEQ,
 "<":  LESSER,
 "<=": LEQ,
}
```

برای ایجاد لکسر یک استراکت ایجاد میکنیم که دو متغیر دارد.

یک چنل از نوع توکن که توکن هایی که پیدا کردیم را درون آن قرار میدهیم
و یک ریدر از نوع
`bufio.Reader`

ما میتوانستیم توکن های اسکن شده را درون یک اسلایس قرار دهیم ولی در اینجا میتوانیم از قابلیت زبان گو استفاده کنیم و به محض پیدا کردن یک توکن آن را درون چنل قرار دهیم تا پارسر آن را از چنل بردارد و مرحله بعدی کار را پیش ببرد.

در فایل بعدی با پارسر آشنا میشویم ولی فقط در این حد بدانید که خروجی لکسر توسط پارسر پردازش میشود.

پس اگر توکن ها را درون آرایه قرار دهیم تا پایان پردازش رشته پارسر نمیتواند آن را پردازش کند. بخاطر همین از چنل برای افزایش سرعت پردازش همزمان استفاده میکنیم.

```go
type lexer struct {
 tokens chan Token
 reader*bufio.Reader
}
```

در این قسمت نکته خاصی وجود ندارد فقط یک سازنده است که استراکت ما را مقدار دهی اولیه میکند.

تنها نکته این قسمت این است که متد
`run`
بصورت یک گوروتین اجرا میشود که تا پایان پردازش فایل کد ما بلاک نشود و
پارسر بتواند همزمان کار خود را شروع کند.

```go
func NewLexer(s string) *lexer {
 reader := bufio.NewReader(strings.NewReader(s))
 l := &lexer{
  reader: reader,
  tokens: make(chan Token),
 }
 go l.lex()
 return l
}
```

اصلی ترین لاجیک لکسر در این متد صورت میگیرد.
یک کاراکتر را میخوانیم و چک میکنیم این کاراکتر شروع چه توکنی میتواند باشد.

مثلا اگر کاراکتری که خوانده ایم از نوع عددی باشد پس وارد عملیات پردازش عدد میشویم.

اگر به کاراکتر کوتیشن برخورد کردیم پس وارد عملیات پردازش رشته میشویم.

اگر کاراکتر از نوع حرف بود و با کوتیشن شروع نمیشد پس رشته نیست و یک
`Identifier`
است.
توکن آیدنتیفایر یک کلمه را مشخص میکند که میتواند یک متغیر، ثابت، نام فانکشنی که یوزر تعریف کرده و چیزهایی از این دست باشد.

اگر کاراکتری که اسکن کردیم هیچکدام از موارد بالا نبود پس یک کاراکتر از نوع کاراکترهای شناسایی شده لیست قبلیست.

مثل
`>`
یا
`)`
اگر هم به انتهای فایل رسیدیم یک توکن از نوع
`EOF`
ایجاد میکنیم.

**نکته:**
در اینجا به کاراکتر اشاره کردیم ولی ما رشته را به جای کاراکتر بوسیله
`rune`
اسکن میکنیم.

رون به معنی یک کاراکتر یونیکد است.

کاراکترهای اسکی یک بایت است اما کاراکترهای یونیکد ممکن است
تا ۴ بایت هم طول داشته باشند. پس ما به جای خواندن یک بایت یک بایت رشته مان را یک رون یک رون میخوانیم.

این کار برای خواندن رشته های فارسی مثل کامنت ها بسیار کارآمد است.

```go
func (l *lexer) lex() {
 r, _, err := l.reader.ReadRune()
 if err == io.EOF {
  l.emit(EOF, "")
  close(l.tokens)
  return
 }
 switch {
 case r == '"':
  l.lexString()
 case unicode.IsDigit(r):
  l.lexNumber(r)
 case unicode.IsLetter(r):
  l.lexIdent(r)
 default:
  l.lexSymbol(r)
 }
 l.lex()
}
```

این متد وظیفه ساخت یک توکن با نوع و مقدار مورد نظر و اضافه کردن به چنل توکن ها را به عهده دارد.

```go
func (l *lexer) emit(t Type, v string) {
 tt := Token{t, v}
 l.tokens <- tt
}
```

این ساده ترین متد ماست.

اگر به یک کوتیشن برخورد کردیم این متد اجرا میشود و ما تا به یک کوتیشن دیگر برسیم به خواندن ادامه میدهیم سپس کوتیشن ها را اول و آخر رشته حذف میکنیم و یک توکن از نوع استرینگ به چنل اضافه میکنیم.

```go
func (l *lexer) lexString() {
 str, _ := l.reader.ReadString('"')
 str = strings.TrimRight(str, `"`)
 l.emit(STRING, str)
}
```

به محض برخورد با یک کاراکتر از نوع حرف این متد اجرا میشود.

تا وقتی به یک کاراکتر غیر حرف برخورد نکرده ایم به خواندن ادامه میدهیم.

اگر به کاراکتر غیر از حرف برخورد کردیم یک کاراکتر به عقب برمیگردیم و رشته بدست آمده را
درون مپ کلمات کلیدی جستجو میکنیم، اگر از نوع کلمه کلیدی بود نوع خودش را برایش در نظر میگیریم.

مثلا توکن
`PRINT`
در غیر اینصورت توکن را از نوع
`IDENT`
ثبت میکنیم.

```go
func (l *lexer) lexIdent(r rune) {
 var str string
 for {
  if unicode.IsLetter(r) || unicode.IsDigit(r) {
   str += string(r)
  } else {
   l.reader.UnreadRune()
   break
  }
  r, *,* = l.reader.ReadRune()
 }

 if t, ok := keywords[str]; ok {
  l.emit(t, str)
 } else {
  l.emit(IDENT, str)
 }
}
```

اگر کاراکتری که به آن برخورد کردیم از نوع عدد بود این متد اجرا میشود.

ما کاراکترهای عدد پشت هم را به عنوان یک توکن واحد در نظر میگیریم..

مثلا
`123456`
یک توکن است.

فقط کاراکترهای عددی در اینجا معتبر است، به استثنای یک کاراکتر دیگر و آن هم کاراکتر نقطه است.

کاراکتر نقطه بعد از یک عدد نشان دهنده یک عدد اعشاریست، پس اگر به نقطه برخورد کردیم نوع را اعشاری در نظر میگیریم در غیر اینصورت عددمان از نوع صحیح یا اینت است.

اما ممکن است به جای یک نقطه به دو نقطه پشت هم برخورد کنیم.

برای مثال
**1..20**
این یک دستور معتبر در زبان کهرباست یعنی اعداد
**1**
تا
**20**
اما ظاهر این عدد بسیار شبیه به عدد اعشاری
**1.20**
است.

پس در اینجا دقت میکنیم که دو کاراکتر نقطه پشت هم نشانه یک توکن جدید است و هر عددی تا اینجا بدست آمده به عنوان یک توکن عدد صحیح در نظر گرفته میشود و توکن بعدی یک توکن از نوع رنج یا دونقطه است.

```go
func (l *lexer) lexNumber(r rune) {
 var t Type = INT
 var str string
 for {
  if r == '.' { // نقطه اول
   r2, *,* := l.reader.ReadRune() // نقطه دوم
   if r2 == '.' {
    l.emit(INT, str)
    l.emit(DOTDOT, "..")
    return
   } else {
    l.reader.UnreadRune()
   }
   t = FLOAT
  }
  if unicode.IsDigit(r) || r == '.' {
   str += string(r)
  } else {
   l.reader.UnreadRune()
   break
  }
  r, *,* = l.reader.ReadRune()
 }
 l.emit(t, str)
}
```

اگر شروع توکن ما هیچکدام از موارد بالا نبود پس ما این متد را اجرا میکنیم.
این متد علائمی مثل
`} . / = !=*`
را درون کد پیدا میکند. بعضی این علائم تک کاراکتری هستند و بعضی دیگر نه.

برای مثال ما هم دنبال کاراکتر
`=`
هستیم هم علامت
`==`

اولی به معنی مقدار دهی و دومی به معنی تساوی به کار میرود.

اگر به محض برخورد با اولین کاراکتر آن را یک توکن در نظر بگیریم در مواجه علامت تساوی دو توکن مقدار دهی تشخیص میدهیم که قطعا برنامه ما را دچار خطا میکند.
.
پس ما در اینجا دو کاراکتر را میخوانیم و اگر علائم دو کاراکتری را پیدا کردیم پس اولویت با آن علائم است.

اما اگر علائم دو کاراکتری را پیدا نکردیم یک کاراکتر به عقب برمیگردیم و همان یک کاراکتر را به عنوان یک توکن در نظر میگیریم.

در میان تمام علائم دو کاراکتری که آنها را به عنوان یک توکن در نظر میگیریم یک استثنا وجود دارد و آن هم دو اسلش پشت هم به نشانه شروع یک کامنت است.

اگر به این علامت رسیدیم بدون اینکه توکنی بسازیم فقط تا آخر خط کاراکتر ها را میخوانیم.
در واقع این خط را نادیده میگیریم و برایش توکنی ثبت نمی کنیم.

```go
func (l*lexer) lexSymbol(r rune) {
 singleCharSymbol := string(r)

 r, *,* = l.reader.ReadRune()
 doubleCharSymbol := singleCharSymbol + string(r)
 if t, ok := symobls[doubleCharSymbol]; ok {
  l.emit(t, doubleCharSymbol)
  return
 }
 if doubleCharSymbol == "//" {
  l.reader.ReadLine()
  return
 }
 if t, ok := symobls[singleCharSymbol]; ok {
  l.reader.UnreadRune()
  l.emit(t, singleCharSymbol)
  return
 }
 l.reader.UnreadRune()
}
```

در انتهای این فایل توانستیم به سادگی یک لکسر بسازیم که میتواند رشته ای از کاراکترهای بی معنی را تبدیل به تعدادی توکن با معنی کند..

قدم بعدی این است که توکن های با معنی را تبدیل به یک دستور با معنی کنیم.

تبریک میگم یک سوم مسیر را طی کردید :)
