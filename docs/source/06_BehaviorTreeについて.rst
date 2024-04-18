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

| ROS2から読み込めるようにする。
| ~/.bashrcに以下を追記する。

.. code-block:: bash

    export export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
