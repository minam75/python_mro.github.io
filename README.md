# الگوی تحلیل ترتیب توابع در پایتون
عبارت python MRO كوتاه شده‌ى عبارت python method resolution order است كه به كمك آن مسير جستجوى كلاس‌ها مشخص مى‌شود. اين روش براى يافتن method درست در كلاس‌هايى كه از multiple-inheritance استفاده می‌كنند به كار مى‌آيد. در پایتون هنگامی که ارث‌بری چندگانه بوجود می‌آید، برای جستجو در کلاس‌ها یک لیست ارث‌بری درست می‌شود که مسیر جستجو را تعیین می‌کند.
براى اين منظور يك روش قديمى و يك روش جديد وجود دارد كه بعد از پایتون 2.2 روش جديد ايجاد شد. در ادامه روش قديمى و روش جديد توضيح داده خواهد شد.

## روش قديمى: 
روش قديمى در پایتون 2.2 به بعد تا قبل از پایتون 3 كار مى‌كند به شرطى كه هيچ كدام از پدران كلاسی که از آن نمونه گرفته‌شده از كلاس object پايتون ارث‌برى نكرده باشند. حال اين روش را به كمك يك مثال نمايش مى‌دهيم.

```python
Class X():
	def printer()
Class Y():
	def printer()
Class A(X, Y):
	def printer()
Class B(Y, X):
	def printer()
Class F(A, B):
	def printer()

inst = F()
inst.printer()
```
در اين حالت چون پدر كلاس A و B كلاس object نيست، از همان روش قديمى استفاده مى‌شود. اين روش به اين شكل است که عمقی طی می‌کند(الگوریتمی شبیه dfs):
<br>
۱.در خود كلاسی که نمونه از آن گرفته‌شده به دنبال متد printer مى‌گردد.
<br>
۲. اگر متد printer را نيافت، به اولین پدرش مى‌رود. حال اگر در پدرش نيز متد printer را نيابد، به پدر پدرش مى‌رود و به همین صورت ادامه می‌یابد تا پدری وجود نداشته‌باشد.
<br>
۳. اما اگر پدر کلاسی در روند بالا وجود نداشت، مى‌رود خود كلاس را مى‌بيند و اگر از كلاس ديگرى ارث‌برى كرده بود، آن كلاس را انتخاب كرده و در آن كلاس به دنبال متد printer مى‌گردد و روند ۲ را دوباره ادامه می‌دهد.
<br>
در مثال فوق اين مسير برابر است با:
<br>
F, A, X
<br>
حال چون متد print در X موجود نبود، می‌رود و کلاس دیگر یعنی Y را می‌گردد.
<br>
F, A, X, Y
<br>
حال دوباره چون متد print در Y موجود نبود، یک کلاس باز می‌گردد و به دنبال کلاس دیگری می‌گردد که دوباره پیدا نمی‌کند و دوباره یک مرحله باز‌می‌گردد که به خود کلاسی که از آن نمونه گرفته شده بود می‌رسیم، حال اگر کلاس دیگری موجود بود، مراحل را برای آن طی می‌کنیم، وگرنه به جستجو پایان می‌دهیم. حال بعد از انجام مراحل برای B نتیجه به شکل زیر می‌شود:
<br>
F, A, X, Y, B, X, Y
<br>
سپس، چون عنصر تکراری مجاز نیست، عناصر تکراری را حذف می‌کنیم و نتیجه نهایی برابر است با:
<br>
F, A, X, Y, B
<br>

## روش جديد:
روش جديد در حالتی که پدران كلاسی که از آن نمونه گرفته‌شده، از كلاس object python ارث‌برى كرده باشند، به كار مى‌رود و چون از پایتون 3 به بعد پدر تمامى كلاس‌ها به صورت پيش‌فرض object است، همواره از روش جديد استفاده مى‌شود.
حال اين روش را بر اساس مثالى كه در بالا گفته شد توضيح مى‌دهيم. در اين روش مسیر به كمك فرمول زير تعيين مى‌شود.
```python
L[C(B1 ... BN)] = C + merge(L[B1] ... L[BN], B1 ... BN)
```
هم چنين الگوريتم merge نيز به صورت زير انجام مى‌شود:
<br>
۱.اولين عنصر ليست L را بر مى‌دارد.
<br>
۲.حال اگر اين عنصر در دنباله‌ى(tail) ليست هاى ديگر نبود، اين عنصر را به ليست خطى C اضافه مى‌كنيم و آن را از ليست در merge حذف مى كنيم.
<br>
۳.اگر شرط بالا برقرار نبود، به ليست بعدى مى‌رويم و دوباره مراحل ١،٢ را انجام مى‌دهيم. 
<br>
۴.اين الگوريتم را تا جايى انجام مى‌دهيم كه همه کلاس‌ها از لیست حذف شده باشند یا آنکه نتوانیم هیچ اولین عنصری(head) با شرایط ۲ بیابیم.
<br>
۵.اگر هيچ عنصر قابل قبولى نيافتيم، اكسپشن رخ مى‌دهد و در پایتون 2.3 از ایجاد کلاس جلوگیری می‌شود.
<br>
حال این الگوریتم را برای مثالی که در بالا ذکر شد انجام می‌دهیم:
<br>
```python
L[A(X, Y)] = A + merge(L[X], L[Y], X, Y) = A, X, Y
L[B(Y, X)] = B + merge(L[Y], L[X], Y, X) = B, Y, X

L[F(A, B)] = F + merge(L[A], L[B], A, B) 
           = F + merge((A, X, Y) + (B, Y, X), A, B)
```
طبق روشی که در بالا گفته شد، اولین عنصر لیست اول A است، و چون دنباله‌ی هیچ لیست دیگری نیست، A را از لیست اولیه حذف می‌کنیم و به لیست خطی F اضافه می‌کنیم.
<br>
```python
L[F(A, B)] = F, A + merge((X, Y) + (B, Y, X), B)
```
اکنون اولین عنصر لیست اول X است که دنباله‌ی لیست دیگری است، پس به سراغ لیست بعدی می‌رویم. اولین عنصر لیست بعدی B است که دنباله‌ی لیست دیگری نیست، پس B را از لیست اولیه حذف می‌کنیم و به لیست خطی F اضافه می‌کنیم.
```python
L[F(A, B)] = F, A, B + merge((X, Y) + (Y, X))
```
در نهایت دوباره اولین عنصر لیست اول X است که دنباله‌ی لیست دیگری است. پس سراغ لیست دوم می‌رویم که اولین عنصر آن Y است. Y نیز دنباله‌ی لیست دیگری است، پس در این حالت هیچ گزینه‌ی دیگری وجود ندارد. هم‌چنین چون امکان خطی‌سازی وجود ندارد، اکسپشن رخ می‌دهد.
<br>
دو روشی که در بالا ذکر شد، برای یافتن مسیر جستجوی کلاس‌ها در ارث‌بری چندگانه استفاده می‌شوند. 
<br>
<br>
نسخه‌ی PDF این متن را می‌توانید از [اینجا](https://github.com/minam75/python_mro.github.io/raw/master/Research%20-%20Python%20MRO.pdf) دانلود کنید.

### منبع:
[Python Tutorial: Understanding Python MRO - Class search path](https://makina-corpus.com/blog/metier/2014/python-tutorial-understanding-python-mro-class-search-path)

