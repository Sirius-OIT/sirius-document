BehaviorTreeについて
================================================================

`ここには概要説明`

BehaviorTree.CPP v4.5 のインストール
--------------------------------

.. code-block:: bash

    $ sudo apt install libzmq3-dev libboost-dev qtbase5-dev libqt5svg5-dev libzmq3-dev libdw-dev
    $ git clone https://github.com/BehaviorTree/BehaviorTree.CPP.git
    $ cd BehaviorTree.CPP
    $ mkdir build
    $ cd build
    $ cmake..
    $ make -j8
    $ sudo make install

これでライブラリをインストールし、読み込めるようになった。

続いて、ROS2から読み込めるようにするために、~/.bashrcに以下を追記する。

.. code-block:: bash

    export export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

Groot2のインストール
------------------

ここからGroot２をダウンロードする。

https://www.behaviortree.dev/groot/


.. code-block:: bash

    $ sudo chmod +x Groot2-v1.5.2-linux-installer.run

chmod +x で実行権限を与える。

.. code-block:: bash

    $ ./Groot2-v1.5.2-linux-installer.run

これでインストールが完了する。


続いて、aliasを使って起動コマンドのショートカットを作成する。

.. code-block:: bash

    $ sudo nano ~/.bash_aliases

alias専用のファイルを作成する。

そのファイルの中に以下を記述する。

.. code-block:: bash

    alias groot2='~/Groot2/bin/groot2'

ファイルを保存し、再読み込みする。

.. code-block:: bash

    $ source ~/.bash_aliases

これで、 ``groot2`` と打つだけでGroot2が起動するようになる。