# Memory
* Смотреть [[Backend/Golang/Deeper#Memory#Общее]]
# ASP.net
# LINQ
## Разница между select и where
- Select нужен для изменения данных в коллекции, а where для их фильтрации. То есть если в where в лямбду мы пропишем просто условие, которое должно выполняться для итерируемого объекта, то в селект мы напишем что надо сделать с этим объектом
```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
var bigNumbers = numbers.Where(x => x > 3); // Вернёт 4, 5

List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
var doubledNumbers = numbers.Select(x => x * 2); // Вернёт 2, 4, 6, 8, 10
```

# EntityFramework
## Особенности
- Вся работа с реализуется через линк запросы, но если обычные линк запросы зачастую используют под собой Енумерабл, то EntityFramework использует Айквариебл