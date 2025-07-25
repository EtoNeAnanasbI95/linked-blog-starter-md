# 🔧 Переменные и константы
```csharp
// --- Явное объявление переменных ---

// целые числа
sbyte a = -1;
byte b = 2;
short c = -4;
ushort d = 5;
int e = -100;
uint f = 200;
long g = -999999;
ulong h = 999999;

// числа с плавающей точкой
float i = 3.14f;
double j = 1.618;
decimal k = 9999.99m; // для финансов

// символы и строки
char letter = 'A';
string text = "Hello NotPineapple";

// логические значения
bool isOn = true;

// --- Неявное объявление ---
var name = "Pineapple"; // string
var num = 123;          // int
var logic = false;      // bool

// --- Константы ---
const string Nickname = "NotPineapple";
const double Pi = 3.1415;

// --- Nullable типы ---
int? maybeNumber = null;
bool? maybeFlag = null;
```

# ⚙️ Условные конструкции, циклы, switch, коллекции, ошибки
```csharp
// if-else
if (isOn)
{
    Console.WriteLine("Включено");
}
else
{
    Console.WriteLine("Выключено");
}

// тернарный оператор
string status = isOn ? "On" : "Off";

// while
int counter = 0;
while (counter < 5)
{
    Console.WriteLine("counter: " + counter);
    counter++;
}

// do-while
do
{
    Console.WriteLine("выполняется хотя бы раз");
} while (false);

// for
for (int i = 0; i < 3; i++)
{
    Console.WriteLine("i = " + i);
}

// foreach
int[] numbers = { 1, 2, 3, 4 };
foreach (var n in numbers)
{
    Console.WriteLine("n = " + n);
}

// switch
string day = "вторник";
switch (day)
{
    case "понедельник":
        Console.WriteLine("Начало недели");
        break;
    case "вторник":
        Console.WriteLine("Рабочий день");
        break;
    case "суббота":
    case "воскресенье":
        Console.WriteLine("Выходной");
        break;
    default:
        Console.WriteLine("Неизвестный день");
        break;
}
```
# 📚 Массивы, списки, словари
`// массив int[] arr = new int[] { 1, 2, 3 };  // список List<string> list = new List<string> { "a", "b", "c" }; list.Add("d");  // словарь Dictionary<string, int> dict = new Dictionary<string, int> {     { "apple", 5 },     { "banana", 2 } };  Console.WriteLine(dict["apple"]); // 5`

# ❗ Ошибки, try-catch, throw, finally

```csharp
// деление с обработкой ошибки
int Divide(int a, int b)
{
    if (b == 0)
        throw new DivideByZeroException("Нельзя делить на ноль");

    return a / b;
}

try
{
    int result = Divide(10, 0);
    Console.WriteLine(result);
}
catch (DivideByZeroException ex)
{
    Console.WriteLine("Ошибка: " + ex.Message);
}
catch (Exception ex)
{
    Console.WriteLine("Общая ошибка: " + ex.Message);
}
finally
{
    Console.WriteLine("Этот блок выполнится всегда");
}

```

# 🧰 Разное: строки, ввод, вывод, приведение типов
```csharp
// строка как массив символов
string hello = "hello";
Console.WriteLine(hello[0]); // h

// конкатенация
string who = "NotPineapple";
string greet = "Hi, " + who;

// интерполяция
string formatted = $"Hello, {who}, you are {2025 - 2000} years old";

// ввод с консоли
// string input = Console.ReadLine();

// преобразования
int num = int.Parse("123");
bool success = int.TryParse("abc", out int result); // false

// кастинг
double d = 4.9;
int casted = (int)d; // 4

```