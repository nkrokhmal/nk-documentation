Flask
=====

Definition
^^^^^^^^^^

Route - association between URL and the function

Funtion is called view function

Request parameters and features
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If route is `/user/<int:id>` it would match only URLs that have integer in the id 

Function examples
^^^^^^^^^^^^^^^^^

.. code:: console

        @app.route('/user/<name>')
        def user(name):
          return '<h1>Hello, %s!</h1>' % name
