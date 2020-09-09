Замыкание (Closure)
-------------------

Замыкание (closure) — функция, которая находится внутри другой функции
и ссылается на переменные объявленные в теле внешней функции (свободные переменные).

Внутренняя функция создается каждый раз во время выполнения внешней.
Каждый раз при вызове внешней функции происходит создание нового 
экземпляра внутренней функции, с новыми ссылками на переменные внешней функции.

Ссылки на переменные внешней функции действительны внутри 
вложенной функции до тех пор, пока работает вложенная функция, даже если внешняя 
функция закончила работу, и переменные вышли из области видимости.

Пример замыкания:

.. code:: python

    def multiply(num1):
        var = 10
        def inner(num2):
            return num1 * num2
        return inner

Тут замыканием является функция inner. Функция inner использует внутри себя
переменную num1 - параметр функции multiply, поэтому переменная num1 будет запомнена,
а вот переменная var не используется и запоминатся не будет.

Использование созданной функции выглядит так:

Сначала делается вызов функции multiply с передачей одного аргумента, значение которого
запишется в переменную num1:

.. code:: python

    In [2]: mult_by_9 = multiply(9)

Переменная mult_by_9 ссылается на внутреннюю функцию inner и при этом внутренняя функция
помнит значение num1 = 9 и поэтому все числа будут умножаться на 9:

.. code:: python

    In [3]: mult_by_9
    Out[3]: <function __main__.multiply.<locals>.inner(num2)>

    In [4]: mult_by_9.__closure__
    Out[4]: (<cell at 0xb0bd5f2c: int object at 0x836bf60>,)

    In [5]: mult_by_9.__closure__[0].cell_contents
    Out[5]: 9

    In [8]: mult_by_9(10)
    Out[8]: 90

    In [9]: mult_by_9(2)
    Out[9]: 18

Еще один пример замыкания с несколькими свободными переменными:

.. code:: python

    def func1():
        a = 1
        b = 'line'
        c = [1, 2, 3]

        def func2():
            return a, b, c

        return func2

    In [11]: call_func = func1()

    In [12]: call_func
    Out[12]: <function __main__.func1.<locals>.func2()>

    In [13]: call_func.__closure__
    Out[13]:
    (<cell at 0xb12170bc: int object at 0x836bee0>,
     <cell at 0xb12172e4: str object at 0xb732d720>,
     <cell at 0xb12177f4: list object at 0xb4e6d66c>)

    In [14]: for item in call_func.__closure__:
        ...:     print(item, item.cell_contents)
        ...:
    <cell at 0xb12170bc: int object at 0x836bee0> 1
    <cell at 0xb12172e4: str object at 0xb732d720> line
    <cell at 0xb12177f4: list object at 0xb4e6d66c> [1, 2, 3]

Изменение свободных переменных
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Для получения значения свободной переменной достаточно обратиться к ней, однако, при изменении
значений есть нюансы. Если переменная ссылается на изменяемый объект, например, список, 
изменение содержимого делается стандартным образом без каких-либо проблем. Однако
если необходимо, к примеру, добавить 1 к числу, мы получим ошибку:

.. code:: python

    In [31]: def func1():
        ...:     a = 1
        ...:     b = 'line'
        ...:     c = [1, 2, 3]
        ...:
        ...:     def func2():
        ...:         c.append(4)
        ...:         a = a + 1
        ...:         return a, b, c
        ...:
        ...:     return func2
        ...:

    In [32]: call_func = func1()

    In [33]: call_func()
    ---------------------------------------------------------------------------
    UnboundLocalError                         Traceback (most recent call last)
    <ipython-input-33-9288e4e0f32f> in <module>
    ----> 1 call_func()

    <ipython-input-31-56414e2c364b> in func2()
          6     def func2():
          7         c.append(4)
    ----> 8         a += 1
          9         return a, b, c
         10

    UnboundLocalError: local variable 'a' referenced before assignment

    In [34]: for item in call_func.__closure__:
        ...:     print(item, item.cell_contents)
        ...:
    <cell at 0xb12174c4: str object at 0xb732d720> line
    <cell at 0xb1217af4: list object at 0xb11e5dac> [1, 2, 3, 4]

Если необходимо присвоить свободной переменной другое значение, необходимо
явно объявить ее как nonlocal:

.. code:: python

    In [40]: def func1():
        ...:     a = 1
        ...:     b = 'line'
        ...:     c = [1, 2, 3]
        ...:
        ...:     def func2():
        ...:         nonlocal a
        ...:         c.append(4)
        ...:         a += 1
        ...:         return a, b, c
        ...:
        ...:     return func2
        ...:

    In [41]: call_func = func1()

    In [42]: call_func()
    Out[42]: (2, 'line', [1, 2, 3, 4])

    In [43]: for item in call_func.__closure__:
        ...:     print(item, item.cell_contents)
        ...:
    <cell at 0xb11fc6bc: int object at 0x836bef0> 2
    <cell at 0xb11fcdac: str object at 0xb732d720> line
    <cell at 0xb11fc56c: list object at 0xb117fe2c> [1, 2, 3, 4]


Использование nonlocal нужно только если необходимо менять свободную переменную
сохраняя измененное значение между вызовами внутренней функции. Для обычного
переприсваивания значения ничего делать не нужно.

Пример использования nonlocal с повторным вызовом внутренней функции:

.. code:: python

    def countdown(n):
        def step():
            nonlocal n
            r = n
            n -= 1
            return r
        return step

    In [49]: do_step = countdown(10)

    In [50]: do_step()
    Out[50]: 10

    In [51]: do_step()
    Out[51]: 9

    In [52]: do_step()
    Out[52]: 8

    In [53]: do_step()
    Out[53]: 7

Примеры использования замыкания
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Так как замыкания позволяют сохранять состояние (значения свободных переменных),
их можно использовать для создания функции, которая отчасти похожа на класс:

.. code:: python

    def func_as_object(a,b):
        def add():
            return a+b
        def sub():
            return a-b
        def mul():
            return a*b
        def replace():
            pass
        replace.add = add
        replace.sub = sub
        replace.mul = mul
        return replace


    In [13]: obj1 = func_as_object(5,2)

    In [14]: obj1.add()
    Out[14]: 7

    In [15]: obj2 = func_as_object(15,2)

    In [16]: obj2.add()
    Out[16]: 17

    In [17]: obj1.add()
    Out[17]: 7

В таких случая обязательно надо делать внутреннюю функцию которой
присваиваются атрибуты и возвращать ее вместо исходной. Если сделать
как в примере ниже, все будет работать корректно только с одним объектом:

.. code:: python

    def func_as_object(a,b):
        def add():
            return a+b
        def sub():
            return a-b
        def mul():
            return a*b
        func_as_object.add = add
        func_as_object.sub = sub
        func_as_object.mul = mul
        return func_as_object

    In [18]: obj1 = func_as_object(5, 2)

    In [19]: obj1.add()
    Out[19]: 7

Как только добавляется второй объект, атрибуты функции подменяются
на другие вложенные функции, которые помнят значения переменных для последнего
объекта и первый объект теперь возвращает неправильные значения:

.. code:: python

    In [9]: obj2 = func_as_object(15,2)

    In [10]: obj2.add()
    Out[10]: 17

    In [11]: obj1.add()
    Out[11]: 17



Пример с подключением SSH:

.. code:: python

    from netmiko import ConnectHandler

    device_params = {
        'device_type': 'cisco_ios',
        'ip': '192.168.100.1',
        'username': 'cisco',
        'password': 'cisco',
        'secret': 'cisco'
    }

    def netmiko_ssh(**params_dict):
            ssh = ConnectHandler(**params_dict)
            ssh.enable()
            def send_show_command(command):
                return ssh.send_command(command)
            netmiko_ssh.send_show_command = send_show_command
            return send_show_command


    In [25]: r1 = netmiko_ssh(**device_params)

    In [26]: r1('sh clock')
    Out[26]: '*15:14:13.240 UTC Wed Oct 2 2019'


