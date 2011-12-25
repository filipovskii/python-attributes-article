__getattr__, __setattr__, __delattr__ и __getattribute__
========================================================

`__getattr__()` - специальный метод, который будет вызван в случае, если запрашиваемый атрибут не найден обычным механизмом (в **__dict__** экземпляра, класса и т.д.):

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

`__setattr__()` - специальный метод, который вызывается при попытке установить значение атрибута экземпляра.:


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
