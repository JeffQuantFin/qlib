
Qlib FAQ
############

Qlib Frequently Asked Questions
================================
.. contents::
    :depth: 1
    :local:
    :backlinks: none

------


1. RuntimeError: An attempt has been made to start a new process before the current process has finished its bootstrapping phase...
------------------------------------------------------------------------------------------------------------------------------------

.. code-block:: console

    RuntimeError:
            An attempt has been made to start a new process before the
            current process has finished its bootstrapping phase.

            This probably means that you are not using fork to start your
            child processes and you have forgotten to use the proper idiom
            in the main module:

                if __name__ == '__main__':
                    freeze_support()
                    ...

            The "freeze_support()" line can be omitted if the program
            is not going to be frozen to produce an executable.

This is caused by the limitation of multiprocessing under windows OS. Please refer to `here <https://stackoverflow.com/a/24374798>`_ for more info.

**Solution**: To select a start method you use the ``D.features`` in the if __name__ == '__main__' clause of the main module. For example:

.. code-block:: python

    import qlib
    from qlib.data import D


    if __name__ == "__main__":
        qlib.init()
        instruments = ["SH600000"]
        fields = ["$close", "$change"]
        df = D.features(instruments, fields, start_time='2010-01-01', end_time='2012-12-31')
        print(df.head())



2. qlib.data.cache.QlibCacheException: It sees the key(...) of the redis lock has existed in your redis db now.
-----------------------------------------------------------------------------------------------------------------

It sees the key of the redis lock has existed in your redis db now. You can use the following command to clear your redis keys and rerun your commands

.. code-block:: console

    $ redis-cli
    > select 1
    > flushdb

If the issue is not resolved, use ``keys *`` to find if multiple keys exist. If so, try using ``flushall`` to clear all the keys.

.. note::

    ``qlib.config.redis_task_db`` defaults is ``1``, users can use ``qlib.init(redis_task_db=<other_db>)`` settings.


Also, feel free to post a new issue in our GitHub repository. We always check each issue carefully and try our best to solve them.

3. ModuleNotFoundError: No module named 'qlib.data._libs.rolling'
------------------------------------------------------------------------------------------------------------------------------------

.. code-block:: python

    #### Do not import qlib package in the repository directory in case of importing qlib from . without compiling #####
    Traceback (most recent call last):
    File "<stdin>", line 1, in <module>
    File "qlib/qlib/__init__.py", line 19, in init
        from .data.cache import H
    File "qlib/qlib/data/__init__.py", line 8, in <module>
        from .data import (
    File "qlib/qlib/data/data.py", line 20, in <module>
        from .cache import H
    File "qlib/qlib/data/cache.py", line 36, in <module>
        from .ops import Operators
    File "qlib/qlib/data/ops.py", line 19, in <module>
        from ._libs.rolling import rolling_slope, rolling_rsquare, rolling_resi
    ModuleNotFoundError: No module named 'qlib.data._libs.rolling'

- If the error occurs when importing ``qlib`` package with ``PyCharm`` IDE, users can execute the following command in the project root folder to compile Cython files and generate executable files:

    .. code-block:: bash

        python setup.py build_ext --inplace

- If the error occurs when importing ``qlib`` package with command ``python`` , users need to change the running directory to ensure that the script does not run in the project directory.


4. BadNamespaceError: / is not a connected namespace
------------------------------------------------------------------------------------------------------------------------------------

.. code-block:: python

      File "qlib_online.py", line 35, in <module>
        cal = D.calendar()
      File "e:\code\python\microsoft\qlib_latest\qlib\qlib\data\data.py", line 973, in calendar
        return Cal.calendar(start_time, end_time, freq, future=future)
      File "e:\code\python\microsoft\qlib_latest\qlib\qlib\data\data.py", line 798, in calendar
        self.conn.send_request(
      File "e:\code\python\microsoft\qlib_latest\qlib\qlib\data\client.py", line 101, in send_request
        self.sio.emit(request_type + "_request", request_content)
      File "G:\apps\miniconda\envs\qlib\lib\site-packages\python_socketio-5.3.0-py3.8.egg\socketio\client.py", line 369, in emit
        raise exceptions.BadNamespaceError(
      BadNamespaceError: / is not a connected namespace.

- The version of ``python-socketio`` in qlib needs to be the same as the version of ``python-socketio`` in qlib-server:

    .. code-block:: bash

        pip install -U python-socketio==<qlib-server python-socketio version>


5. TypeError: send() got an unexpected keyword argument 'binary'
------------------------------------------------------------------------------------------------------------------------------------

.. code-block:: python

      File "qlib_online.py", line 35, in <module>
        cal = D.calendar()
      File "e:\code\python\microsoft\qlib_latest\qlib\qlib\data\data.py", line 973, in calendar
        return Cal.calendar(start_time, end_time, freq, future=future)
      File "e:\code\python\microsoft\qlib_latest\qlib\qlib\data\data.py", line 798, in calendar
        self.conn.send_request(
      File "e:\code\python\microsoft\qlib_latest\qlib\qlib\data\client.py", line 101, in send_request
        self.sio.emit(request_type + "_request", request_content)
      File "G:\apps\miniconda\envs\qlib\lib\site-packages\socketio\client.py", line 263, in emit
        self._send_packet(packet.Packet(packet.EVENT, namespace=namespace,
      File "G:\apps\miniconda\envs\qlib\lib\site-packages\socketio\client.py", line 339, in _send_packet
        self.eio.send(ep, binary=binary)
      TypeError: send() got an unexpected keyword argument 'binary'


- The ``python-engineio`` version needs to be compatible with the ``python-socketio`` version, reference: https://github.com/miguelgrinberg/python-socketio#version-compatibility

    .. code-block:: bash

        pip install -U python-engineio==<compatible python-socketio version>
        # or
        pip install -U python-socketio==3.1.2 python-engineio==3.13.2

6、Install qlib on Mac m1max
------------------------------------------------------------------------------------------------------------------------------------

.. note::

Installing qlib on Mac especially on Mac M1 is a little bit  complicated.  Firstly,  many packages cannot be installed by pip directly, and the gcc compilation of source  need a lot of time to config.   It is suggested to follow steps below to avoid wasting time on installing qlib on Mac M1max.

Mac M1max
python==3.8
Miniforge


1、  Install miniforge  instead of  miniconda or anaconda, which are not compatible with M1 chips well until now.

2、	conda install  numpy pandad cvxpy tornado tqdm xlrd schedule sacred ruamel.yaml hiredis pytables dill filelock fire lightgbm loguru mlflow==1.12.1 joblib==0.17.0 hyperopt==0.0.2 threadpoolctl==3.0.0 lightgbm statsmodels 

3、	install catboost manually, refer to https://github.com/catboost/catboost/issues/1526

4、	pip install --upgrade  cython

5、	pip install CMake python-redis-lock==3.3.1 python-socketio pyyaml==5.3.1 redis==3.0.1 requests==2.18.0 scikit-learn==0.22.0

6、  pip install —upgrade docker

7、  pip uninstall tables
	 conda install pytables==3.6.1

8、  conda uninstall pymongo
     pip install pymongo==3.7.2	

9、  pip install -U mlflow==1.12.1

10、 correct the bug of ecos package installed when installing cvxpy

	1) find the file “ecos-0.0.0.dist-info” in /miniforge3/envs/<env name>/lib/python3.8/site-packages
	modify the file name to “ecos-2.0.8.dist-info”

	2) open  /miniforge3/envs/<env name>/lib/python3.8/site-packages/ecos/version.py
	     modify the blank version to “2.0.8”	

11、 cd <qlib_file_path>

	pip install .
