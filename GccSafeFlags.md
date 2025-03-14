# Флаги компилятора gcc повышающие безопасность программы

Включить вывод всех предупреждений:

```
  -Wall -Wextra
```

Обязательные флаги:

```
  -Werror=return-type -Werror=float-equal -Werror=unused-variable -Werror=switch -Werror=implicit-fallthrough -Werror=free-nonheap-object -Werror=sequence-point -Werror=return-local-addr -Werror=format=2 -Werror=shadow -Werror=duplicated-cond -Werror=duplicated-branches -Werror=type-limits -Werror=logical-op -Werror=parentheses -Werror=char-subscripts -Werror=address -Werror=memset-elt-size -Werror=memset-transposed-args
```

Рекомендуемые флаги:

```
  -Werror=uninitialized -Werror=init-self -Werror=conversion -Werror=old-style-cast
```

Флаги применяемые <b>только в отладочной сборке</b>. Существенно, в разы или даже десятки раз замедляют скорость работы программы, но позволяют найти серьезные ошибки во время выполнения программы. Работают только под Linux. Под Windows либо не работают либо работают частично (по состоянию на 2025 год). Одновременно использовать эти флаги нельзя, только по отдельности. Сначала первый, затем второй, или делать 2 отдельные сборки:

<table>
  <tr><td>-fsanitize=undefined</td><td>Находит UB(undefined behaviour)</td></tr>
  <tr><td>-fsanitize=address</td><td>Находит некорректное обращение к памяти кучи</td></tr>
  <tr><td>-fsanitize=thread</td><td>Находит "гонку данных"</td></tr>
</table>

<b>-Werror=return-type</b><br>
Выводит ошибку если функция должна вернуть значение, но return отсутствует. Пример:
```C++
int test()
{
// Не скомпилируется, так как забыт return
}
```

<b>-Werror=float-equal</b><br>
Выводит ошибку если числа типов float и double сравниваются напрямую. Это делать нельзя, так как из-за особого представления этих чисел в компьютере сравнение может приводить к неправильному результату. Пример:

```C++
float a = 100, b = 103;

// ... код работающий с a и/или b
if( a == b ) // не скомпилируется, см. EPSILON и IEEE754
{
    printf("Equals\r\n");
}
```

<b>-Werror=uninitialized</b><br>
Выводит ошибку если какая-либо переменная неинициализирована. Если переменная используется только для вычисления значения, которое само по себе не используется, то ошибки может и не быть, поскольку такие вычисления могут быть удалены анализом потока данных до того, как предупреждения будут выведены.


<b>-Werror=init-self</b><br>
Выводит ошибку если не инициализированная переменная инициализируется самой собой. То есть остается в не инициализированном состоянии. Работает только вместе с флагом -Werror=uninitialized Пример:

```C++
  int i = i;
```

<b>-Werror=switch  
-Werror=implicit-fallthrough</b><br>

Первый флаг(switch) вызовет ошибку если в switch передается перечисление и какой-то из элементов перечисления остается необработанным.
Второй флаг выдает ошибку если забыт break. Пример:
```C++
enum COLORS
{
    RED,
    GREEN,
    BLUE
};
COLORS color = BLUE;
switch(color)
{
    case RED   : std::cout << "RED\n" ;  break; 
    case GREEN : std::cout << "GREEN\n"; break;

    // Забыта обработка одного из элементов перечисления
    //  -Werror=switch не даст скомпилироваться данному коду

}

switch(color)
{
    case RED   : std::cout << "RED\n";
    case GREEN : std::cout << "GREEN\n"; break;
    case BLUE  : std::cout << "BLUE\n"; break;

    // Забыт break у case RED, -Werror=implicit-fallthrough
    //  не даст скомпилироваться данному коду  
}
```

<b>-Werror=free-nonheap-object</b><br>

Вызов функции free с адресом переменной не из кучи.
```C++
int a = 10;

{
    int b = 10;

    int* c = &b;    

    free(&a); // не скомпилируется

    free(&b); // не скомпилируется

    free(&c); // не скомпилируется

    delete &a; // скомпилируется, проверяется только
               // free !!!
}
```
<b>-Werror=sequence-point</b><br>

Не компилировать код который, вероятно, может иметь неопределенный порядок выполнения.
```C++

   a = a++; // не скомпилируется
    
   a[n] = b[n++]; // не скомпилируется
    
   a[i++] = i; // не скомпилируется
```

<b>-Werror=return-local-addr</b><br>

Попытка возврата адреса, который будет не актуален после выхода из функции.
```C++
int* test()
{
   int a;
   return &a; // Не скомпилируется
}
```
<b>-Werror=format=2</b><br>

У вызовов функции printf and scanf и подобных проверят соответствие строки формата типам переданных переменных
```C++

float myvalue;

printf(“Test value: %d”, myvalue); // не скомпилируется

```
<b>-Werror=conversion</b><br>

Возникает при неявных преобразованиях, которые могут изменить значение. Сюда входят преобразования между действительными и целыми числами, например abs (x), когда x имеет тип double; преобразования между знаковыми и беззнаковыми, например unsigned ui = -1; и преобразования в меньшие типы, например sqrtf (M_PI).
```C++
long long int a = 10;

short int b = 0;
a = b; // Не скомпилируется, нужно явно преобразовать типы


```
<b>-Werror=shadow</b><br>

Программа не скомпилируется, если локальная переменная «затеняет» глобальную переменную или переменную объявленную во внешнем блоке.
```C++
int a = 10;

void myfunc()
{
    int a = 20; // не скомпилируется
}

{
    for( int i = 0 ; i < 10; i++ )
    {
        for(int i = 0 ; i < 10 ; i++ ) // не скомпилируется
        {
        }
    }
}
```
<b>-Werror=duplicated-cond</b><br>

Повторная проверка одного и того-же условия.
```C++
int a = 10;

if( a == 10 )
{
}
else if( a == 10 ) // не скомпилируется
{
}
```

<b>-Werror=duplicated-branches</b><br>

Содержимое блоков if и else одинаково.
```C++
if (true)
    return 0;
else
    return 0;
```
<b>-Werror=type-limits</b><br>
Сравнение всегда истинно или всегда ложно из-за ограниченного диапазона типа данных. Не сработает для константных выражений.
```C++
unsigned int a = 10;

if( a < 0 ) // не скомпилируется. Данное сравнение
{           // всегда будет ложно
}
```

<b>-Werror=logical-op</b><br>

Подозрительное использование логических операторов в выражениях. Это включает использование логических операторов в контекстах, где, скорее всего ожидается побитовый оператор. Также предупреждает, когда операнды логического оператора одинаковы.

```C++
int b = 10;

if (b < 0 && b < 0)
{
}
```

<b>-Werror=parentheses</b><br>

Пропущены скобки в определенном контексте, либо если появляется сравнение типа x<=y<=z; это эквивалентно (x<=y ? 1 : 0) <= z, что является интерпретацией, отличной от обычной математической записи.

```C++

```

<b>-Werror=char-subscripts</b><br>

Индекс массива имеет тип char. Это частая причина ошибки, так как программисты часто забывают, что этот тип является знаковым на некоторых платформах.
```C++
char val = -1;

int arr[10] = {0};
arr[val] = 1; // Не скомпилируется
```
<b>-Werror=address</b><br>

Подозрительное использование адресных выражений. К ним относятся сравнение адреса функции или объявленного объекта с константой нулевого указателя.
```C++
void f (void);
void g (void)
{
  if (!f)   // Выражение всегда будет ложно
    abort ();
}

void f (const char *x)
{
  if (x == "abc")   // Выражение всегда будет ложно
    puts ("equal");
}
```

Также может быть проверка результатов сложения или вычитания указателя на равенство нулю.
```C++
void f (const int *p, int i)
{
  return p + i == NULL;
}
```

<b>-Werror=memset-elt-size</b><br>

 Вероятно есть ошибка в 3 аргументу функции memset. В 3 аргументе быть количество байт а не количество элементов. 

```C++
#define SIZE 20

int a[SIZE];
std::memset(a, 0, SIZE);
```
<b>-Werror=memset-transposed-args</b><br>

Скорее всего перепутан порядок аргументов в функции memset.

```C++
#define SIZE 20

int a[SIZE];
std::memset(a, SIZE, 0);
```
<b>-Werror=old-style-cast</b><br>
Приведение типов в стиле языка C. В С++ нужно использовать одно из следующих, более безопасных, приведений типов: static_cast, dynamic_cast, reinterpret_cast и const_cast.
```C++
int *a;

short int b;

a = (int*) &b; // не скомпилируется
```
Ссылки:
1. https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html
2. https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html
