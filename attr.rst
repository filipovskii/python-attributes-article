__getattr__(), __setattr__(), __delattr__() и __getattribute__()
================================================================

Если нужно определить поведение какого-либо объекта **как атрибута**, следует использовать дескрипторы (например **property**). Тоже справедливо для семейства объектов (например **функций**). Ещё один способ повлиять на доступ к атрибутам: методы `__getattr__()`, `__setattr__()`, `__delattr__()` и `__getattribute__()`. В отличие от дескрипторов их следует определять для объекта, **содержащего атрибуты** и вызываются они при доступе **к любому** атрибуту этого объекта.

`__getattr__(self, name)` будет вызван в случае, если запрашиваемый атрибут не найден обычным механизмом (в `__dict__` экземпляра, класса и т.д.):

.. code-block:: python
    
    class SmartyPants:
        def __getattr__(self, attr):
            print("Yep, I know", attr)
        tellme = "It's a secret"

    smarty = SmartyPants()
    smarty.name = "Smartinius Smart"

    smarty.quicksort    # Yep, I know quicksort
    smarty.python       # Yep, I know python
    smarty.tellme       # "It's a secret"
    smarty.name         # "Smartinius Smart"

`__getattribute__(self, name)` будет вызван при попытке получить значение атрибута. Если этот метод переопределён, стандартный механизм поиска значения атрибута не будет задействован. Следует иметь ввиду, что вызов специальных методов (например `__len__()`, `__str__()`) через встроенные функции или неявный вызов через синтаксис языка осуществляется в обход `__getattribute__()`.

.. code-block:: python

    class Optimist:
        attr = "class attribute"
        
        def __getattribute__(self, name):
            print("{0} is great!".format(name))
        
        def __len__(self):
            print("__len__ is special")
            return 0
        
    o = Optimist()
    o.instance_attr = "instance"

    o.attr          # attr is great!
    o.dark_beer     # dark_beer is great!
    o.instance_attr # instance_attr is great!
    o.__len__       # __len__ is great!
    len(o)          # __len__ is special\n 0

`__setattr__(self, name, value)` будет вызван при попытке установить значение атрибута экземпляра. Аналогично `__getattribute__()`, если этот метод переопределён, стандартный механизм установки значения не будет задействован:


.. code-block:: python
    
    class NoSetters:
        attr = "class attribute"
        def __setattr__(self, name, val):
            print("not setting {0}={1}".format(name,val))

    no_setters = NoSetters()
    no_setters.a = 1            # not setting a=1
    no_setters.attr = 1         # not setting attr=1
    no_setters.__dict__         # {}
    no_setters.attr             # "class attribute"
    no_setters.a                # AttributeError

`__delattr__(self, name)` - аналогичен `__setattr__()`, но используется при удалении атрибута.

При переопределении `__getattribute__()`, `__setattr__()` и `__delattr__()` следует иметь ввиду, что стандартный способ получения доступа к атрибутам можно вызвать через **object**:

.. code-block:: python

    class GentleGuy:
        def __getattribute__(self, name):
            if name.endswith("_please"):
                return object.__getattribute__(self, name.replace("_please", ""))
            raise AttributeError("And the magic word!?")

    gentle = GentleGuy()

    gentle.coffee = "some coffee"
    gentle.coffee           # AttributeError
    gentle.coffee_please    # "some coffee"
