__dict__
========

**__dict__** - это специальный атрибут объекта, который, как можно догадаться, представляет собой dictionary (всё нутро восстаёт против слова словарь), который призван хранить атрибуты, определённые пользователем. Хранит он их в виде ключ-значение, то есть *имя_атрибута*: *значение_атрибута*. Причём, атрибут может быть определён как для экземпляра (то есть для одного конкретного объекта), так и для класса (то есть для всех объектов, которые являются экземплярами данного класса). 

.. code-block:: python

    class StuffHolder:
        stuff = "class stuff"

    a = StuffHolder()
    b = StuffHolder()
    a.stuff     # "class stuff"
    b.stuff     # "class stuff"
    
    b.b_stuff = "b stuff"
    b.b_stuff   # "b stuff"
    a.b_stuff   # AttributeError

В примере я описываю класс `StuffHolder` с одним атрибутом `stuff`, который, наследуют оба его экземпляра. Затем объекту `b` я добавляю его "персональный" атрибут `b_stuff`, которого у `a` нет (отсюда AttributeError).

Посмотрим на **__dict__** всех действующих лиц:

.. code-block:: python

    StuffHolder.__dict__    # {... 'stuff': 'class stuff' ...} 
    a.__dict__              # {}
    b.__dict__              # {'b_stuff': 'b stuff'}

    a.__class__             # <class '__main__.StuffHolder'>
    b.__class__             # <class '__main__.StuffHolder'>

(У класса StuffHolder в **__dict__** хранится объект класса dict_proxy с кучей разного барахла, на которое пока не нужно обращать внимание)

Ни у `a` ни у `b` в **__dict__** нет атрибута `stuff`, не найдя его там, механизм поиска ищет его в **__dict__** класса (`StuffHolder`), успешно находит и возвращает значение, присвоенное ему в классе. Ссылка на класс хранится в атрибуте **__class__** объекта.


Поиск атрибута происходит во время выполнения, так что даже после создания экземпляров, все изменения в **__dict__** класса отразятся в них:

.. code-block:: python

    a.new_stuff                 # AttributeError
    b.new_stuff                 # AttributeError

    StuffHolder.new_stuff = "new"
    StuffHolder.__dict__        # {... 'stuff': 'class stuff', 'new_stuff': 'new'...}
    a.new_stuff                 # "new"
    b.new_stuff                 # "new"

В случае присваивания значения атрибуту экземпляра, изменяется только **__dict__** экземпляра, то есть значение в **__dict__** класса остаётся неизменным (в случае, если значением атрибута класса не является Data Descriptor):

.. code-block:: python

    StuffHolder.__dict__    # {... 'stuff': 'class stuff' ...}
    c = StuffHolder()
    c.__dict__              # {}

    c.stuff = "more c stuff"
    c.__dict__              # {'stuff': 'more c stuff'}
    StuffHolder.__dict__    # {... 'stuff': 'class stuff' ...}
    

В случае совпадения имён атрибутов в классе и экземпляре, интерпретатор при поиске значения выдаст значение экземпляра (в случае, если значением атрибута класса не является Data Descriptor):


.. code-block:: python

    StuffHolder.__dict__    # {... 'stuff': 'class stuff' ...}
    d = StuffHolder()
    d.stuff                 # "class stuff"

    d.stuff = "d stuff"
    d.stuff                 # "d  stuff"


.. code-block:: python
   :emphasize-lines: 3,5
   :linenos:

   def some_function():
       interesting = False
       print 'This line is highlighted.'
       print 'This one is not...'
       print '...but this one is.'
