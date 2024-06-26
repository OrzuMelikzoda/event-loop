# Event loop

### Как уже было описано в одной из предыдущих статьей, при работе с асинхронными функциями, которые используют Future, Dart использует событийный цикл Event Loop. Когда запускается приложение, единственный поток приложения инициализирует две очереди - MicroTask Queue и Event Queue, которые будут содержать задачи Future. Далее поток запускает функцию main() и прежде всего выполняет в нем все синхронные задачи. Синхронные задачи в основном потоке всегда выполняются немедленно. Если Dart встречает вызовы асинхронных функций, которые возвращают Future, то они помещается в очередь событий Event Queue или MicroTask Queue.

### Когда синхронные задачи и задачи из Microtask Queue завершили выполнение, цикл событий начинает выбирать задачи из очереди Event Queue и помещает их в основной поток, где они выполняются синхронно.

### Если в очередь Microtask Queue поступит новая микрозадача, то цикл событий выполняет ее до любой последующей задачи из очереди Event Queue.

### Этот процесс продолжается до тех пор, пока очереди не станут пустыми.

### Рассотрим на примере, как работает событийний цикл Event Loop в приложении Dart. Так, определим следуюшую программу:

```js
void main() {
  print("1 synchronous");
  Future(() => print("2 event queue")).then(
    (value) => print("3 synchronous"),
  );
  Future.microtask(() => print("4 microtask queue"));
  Future.microtask(() => print("5 microtask queue"));
  Future.delayed( Duration(seconds: 1), () => print("6 event queue"));
  Future(() => print("7 event queue")).then((value) => Future(() => print("8 event queue")));
  Future(() => print("9 event queue")).then((value) => Future.microtask(() => print("10 microtask queue")));
  print("11 synchronous");
}
```

### Посмотрим, какой будет консольный вывод:

```
1 synchronous
11 synchronous
4 microtask queue
5 microtask queue
2 event queue
3 synchronous
7 event queue
9 event queue
10 microtask queue
8 event queue
6 event queue
```

### Вначале выполняются все синхронные операции, соответственно это первая и последняя строки функции main:


### А все задачи, который создаются с помощью конструктора Future.microtask (то есть микрозадачи), помещаются в очередь MicroTask Queue. Остальные задачи, определяемые с помощью других конструкторов Future, помещаются в Event Queue

### Микрозадачи имеют приоритет перед обычными задачами, поэтому очередь MicroTask Queue начинает выполняться первой. То есть выполняются задачи


```js
Future.microtask(() => print("4 microtask queue"));
Future.microtask(() => print("5 microtask queue"));
```

### На консоль выводится

```
4 microtask queue
5 microtask queue
```

### Далее выполняются задачи из Event Queue:


```js
Future(() => print("2 event queue")).then((value) => print("3 synchronous"));
 
Future.delayed( Duration(seconds: 1), () => print("6 event queue"));
Future(() => print("7 event queue")).then((value) => Future(() => print("8 event queue")));
Future(() => print("9 event queue")).then((value) => Future.microtask(() => print("10 microtask queue")));
```

### Поскольку очередь представляет структуру FIFO, то те задачи, которые первыми были добавлены, первыми же и выполняются. Добавлениие производится в порядке определения вызова задач. Например, после добавления 7-й задачи добавляется 9-я задачи. 8-я задача добавляется только после 9-й, поскольку 8-я задача определяется только после выполнения 7-й задачи.


### Также обращает на себя внимание добавление новой микрозадачи в результате выполнения 9-й задачи. Поскольку микрозадачи имеют приоритет, то они выполняются до любых других задач. Поэтому последняя микрозадача будет выполняться до 8-й и 6-й задач.

```js
10 microtask queue
8 event queue
6 event queue
```

### 6-я задача завершается последней в силу задержки в 1 секунду, поскольку за это время все остальные задачи успеют отработать.






