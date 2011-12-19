Дескрипторы
===========

С простыми типами в качестве значений атрибутов пока всё ясно. Посмотрим, как ведёт себя функция в тех же условиях:


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

Этот метод переопределяет механизм получения значения атрибута `fh.func`, а объект, который реализует этот метод непереводимо называется **non-data descriptor**.

Из `howto <http://docs.python.org/howto/descriptor.html>`_:
    Дескриптор - это объект, доступ к которому через атрибут переопределён методами в *дескриптор протоколе*.

*Дескриптор протокол* это набор методов:

.. code-block:: python

    descr.__get__(self, obj, type=None) --> value   (переопределяет способ получения значения атрибута)

    descr.__set__(self, obj, value) --> None        (переопределяет способ присваивания значения атрибуту)

    descr.__delete__(self, obj) --> None            (переопределяет способ удаления атрибута)

Дескрипторы бывают двух видов:

1. Data Descriptor (дескриптор данных) - объект, который реализует метод `__get__` и `__set__`
2. Non-data Descriptor (дескриптор не данных?) - объект, который реализует метод `__get__`

Отличаются они своим поведением по отношению к записям в **__dict__** экземпляра. Если в нём есть запись с тем же именем, что у дескриптора данных, у дескриптора преимущество. Если имя записи совпадает с именем "дескриптора не данных", приоритет записи выше.

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

Стоит проверить утверждение о том, что у дата дескрипторов преимущество перед записями в **__dict__** экземпляра:

.. code-block:: python
    
    d.__dict__["data"] = "override!"
    d.__dict__  # {'data': 'override!'}
    d.data      # Trying to access from <__main__.DataHolder object at 0x9008a4c> class <class '__main__.DataHolder'>

Так и есть, запись в **__dict__** экземпляра игнорируется, если в **__dict__** класса экземпляра (или его базового класса) существует запись с тем же именем, а в качестве значения там дескриптор данных.

Ещё один важный момент. Если изменить значение атрибута с дескриптором через класс, никаких методов дескриптора вызвано не будет, значение изменится в **__dict__** класса как если бы это был обычный атрибут:


.. code-block:: python

    DataHolder.__dict__ # {...'data': <__main__.DataDesc object at 0x900236c>...}
    DataHolder.data = "kick descriptor out"
    DataHolder.__dict__ # {...'data': 'kick descriptor out'...}
    DataHolder.data     # "kick descriptor out"


Дескрипторы не данных
---------------------
