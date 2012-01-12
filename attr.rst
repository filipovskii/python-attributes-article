__getattr__(), __setattr__(), __delattr__() и __getattribute__()
================================================================

`__getattr__(self, name)` - специальный метод, который будет вызван в случае, если запрашиваемый атрибут не найден обычным механизмом (в **__dict__** экземпляра, класса и т.д.):

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

`__getattribute__(self, name)` - специальный метод, который вызывается при попытке получить значение атрибута. Если этот метод переопределён, стандартный механизм поиска значения атрибута не будет задействован при попытке получить доступ. Следует иметь ввиду, что вызов специальных методов (например `__len__()`, `__str__()`) через встроенные функции или неявный вызов через синтаксис языка осуществляется в обход `__getattribute__()`.

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

`__setattr__(self, name, value)` - специальный метод, который вызывается при попытке установить значение атрибута экземпляра.:


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

`__delattr__(self, name)` - аналогичен `__setattr__()`, но используется при удалении атрибута.
