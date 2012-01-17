Дескрипторы
===========

функцийпростыми типами в качестве значений атрибутов пока всё ясно. Посмотрим, как ведёт себя функция в тех же условиях:


.. code-block:: python

    class FuncHolder:
        def func(self):
            pass
    fh = FuncHolder()

    FuncHolder.func     # <function func at 0x8f806ac>
    FuncHolder.__dict__ # {...'func': <function func at 0x8f806ac>...}
    fh.func             # <bound method FuncHolder.func of <__main__.FuncHolder object at 0x900f08c>>

WTF!? Спросите вы ... возможно. Я бы спросил. Чем функция в этом случае отличается от того, что мы уже видели? Ответ прост: методом `__get__`

.. code-block:: python

    FuncHolder.func.__class__.__get__   # <slot wrapper '__get__' of 'function' objects>

Этот метод переопределяет механизм получения значения атрибута `func` экземпляра `fh`, а объект, который реализует этот метод непереводимо называется **non-data descriptor**.

Из `howto <http://docs.python.org/howto/descriptor.html>`_:
    Дескриптор - это объект, доступ к которому через атрибут переопределён методами в *дескриптор протоколе*:

.. code-block:: python

    descr.__get__(self, obj, type=None) --> value   (переопределяет способ получения значения атрибута)

    descr.__set__(self, obj, value) --> None        (переопределяет способ присваивания значения атрибуту)

    descr.__delete__(self, obj) --> None            (переопределяет способ удаления атрибута)

Дескрипторы бывают двух видов:

1. Data Descriptor (дескриптор данных) - объект, который реализует метод `__get__()` и `__set__()`
2. Non-data Descriptor (дескриптор не данных?) - объект, который реализует метод `__get__()`

Отличаются они своим поведением по отношению к записям в `__dict__` экземпляра. Если в `__dict__` есть запись с тем же именем, что у дескриптора данных, у дескриптора преимущество. Если имя записи совпадает с именем "дескриптора не данных", приоритет записи в `__dict__` выше.

Дескрипторы данных
------------------

Рассмотрим повнимательней дескриптор данных:

.. code-block:: python

    class DataDesc:

        def __get__(self, obj, cls):
            print("Trying to access from {0} class {1}".format(obj, cls))

        def __set__(self, obj, val):
            print("Trying to set {0} for {1}".format(val, obj))

        def __delete__(self, obj):
            print("Trying to delete from {0}".format(obj))

    class DataHolder:
	    data = DataDesc()
    d = DataHolder()

    DataHolder.data     # Trying to access from None class <class '__main__.DataHolder'>
    d.data              # Trying to access from <__main__.DataHolder object at 0x8feb40c> class <class '__main__.DataHolder'>
    d.data = 1          # Trying to set 1 for <__main__.DataHolder object at 0x8feb40c>
    del(d.data)         # Trying to delete from <__main__.DataHolder object at 0x8feb40c>

Стоит обратить внимание, что вызов `DataHolder.data` передаёт в метод `__get__` *None* вместо экземпляра класса.

Проверим утверждение о том, что у дата дескрипторов преимущество перед записями в `__dict__` экземпляра:

.. code-block:: python
    
    d.__dict__["data"] = "override!"
    d.__dict__  # {'data': 'override!'}
    d.data      # Trying to access from <__main__.DataHolder object at 0x9008a4c> class <class '__main__.DataHolder'>

Так и есть, запись в `__dict__` экземпляра игнорируется, если в `__dict__` класса экземпляра (или его базового класса) существует запись с тем же именем и значением - дескриптором данных.

Ещё один важный момент. Если изменить значение атрибута с дескриптором через класс, никаких методов дескриптора вызвано не будет, значение изменится в `__dict__` класса как если бы это был обычный атрибут:


.. code-block:: python

    DataHolder.__dict__ # {...'data': <__main__.DataDesc object at 0x900236c>...}
    DataHolder.data = "kick descriptor out"
    DataHolder.__dict__ # {...'data': 'kick descriptor out'...}
    DataHolder.data     # "kick descriptor out"


Дескрипторы не данных
---------------------

Пример дескриптора не данных:

.. code-block:: python

    class NonDataDesc:<class '__main__.NonDa

        def __get__(self, obj, cls):
            print("Trying to access from {0} class {1}".format(obj, cls)) 
            
    class NonDataHolder:
        non_data = NonDataDesc()
    n = NonDataHolder()

    NonDataHolder.non_data  # Trying to access from None class <class '__main__.NonDataHolder'>
    n.non_data              # Trying to access from <__main__.NonDataHolder object at 0x9c7be4c> class <class '__main__.NonDataHolder'>
    n.non_data = 1
    n.non_data              # 1
    n.__dict__              # {'non_data': 1}

Его поведение слегка отличается от того, что вытворял дата-дескриптор. При попытке присвоить значение атрибуту `non_data`, оно записалось в `__dict__` экземпляра, скрыв таким образом дескриптор, который хранится в `__dict__` класса.

Примеры использования
---------------------

Дескрипторы это мощный инструмент, позволяющий контролировать доступ к атрибутам экземпляра класса. Один из примеров их использования - функции, при вызове через экземпляр они становятся методами (см. пример выше). Также распространённый способ применения дескрипторов - создание *свойства* (*property*). Под свойством я подразумеваю некое значение, характеризующее состояние объекта, доступ к которому управляется с помощью специальных методов (геттеров, сеттеров). Создать свойство просто с помощью дескриптора:

.. code-block:: python
    
    class Descriptor:
        def __get__(self, obj, type):
            print("getter used")
        def __set__(self, obj, val):
            print("setter used")
        def __delete__(self, obj):
            print("deleter used")

    class MyClass:
        prop = Descriptor()

Или можно воспользоваться встроенным классом **property**, он представляет собой дескриптор данных. Код, представленный выше можно переписать следующим образом:

.. code-block:: python

    class MyClass:

        def __init__(self):
            self._prop = None
        def _getter(self):
            print("getter used")
        def _setter(self, val):
            print("setter used")
        def _deleter(self):
            print("deleter used")

        prop = property(_getter, _setter, _deleter, "doc string")

В обоих случаях мы получим одинаковое поведение:

.. code-block:: python

    m = MyClass()
    m.prop          # getter used 
    m.prop = 1      # setter used
    del(m.prop)     # deleter used

Важно знать, что **property** всегда является дескриптором данных. Если в его конструктор не передать какую либо из функций (геттер, сеттер или делитер), при попытке выполнить над атрибутом соответствующее действие - выкинется **AttributeError**.

.. code-block:: python

    class MySecondClass:
        prop = property()

    m2 = MySecondClass()
    m2.prop     # AttributeError: unreadable attribute
    m2.prop = 1 # AttributeError: can't set attribute
    del(m2)     # AttributeError: can't delete attribute

К встроенным дескрипторам также относятся:
    * **staticmethod** - тоже что функция вне класса, в неё **не передаётся** экземпляр в качестве первого аргумента.
    * **classmethod** - тоже что метод класса, только в качестве первого аргумента передаётся класс экземпляра.

.. code-block:: python

    class StaticAndClassMethodHolder:
        
        def _method(*args):
            print("_method called with ", args) 
        static = staticmethod(_method)
        cls = classmethod(_method)

    s = StaticAndClassMethodHolder()
    s._method()     # _method called with (<__main__.StaticAndClassMethodHolder object at 0x9c855cc>,)
    s.static()      # _method called with ()
    s.cls()         # _method called with (<class '__main__.StaticAndClassMethodHolder'>,)
