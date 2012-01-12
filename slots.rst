__slots__
=========

**__slots__** позволяет заранее, на уровне класса, определить набор пользовательских атрибутов, тем самым снимая необходимость создавать **__dict__** экземпляра.


.. code-block:: python

    class Slotter:
        __slots__ = ["a", "b"]

    s = Slotter()
    s.__dict__      # AttributeError
    s.c = 1         # AttributeError
    s.a = 1
    s.a             # 1
    s.b = 1
    s.b             # 1
    dir(s)          # [ ... 'a', 'b'' ... ]
