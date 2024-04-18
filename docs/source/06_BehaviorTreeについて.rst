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


| 続いて、aliasを使って起動コマンドのショートカットを作成する。
| alias専用のファイルを作成する。

.. code-block:: bash

    $ sudo nano ~/.bash_aliases

そのファイルの中に以下を記述する。

.. code-block:: bash

    alias groot2='~/Groot2/bin/groot2'

ファイルを保存し、再読み込みする。

.. code-block:: bash

    $ source ~/.bash_aliases

これで、 ``groot2`` と打つだけでGroot2が起動するようになる。

BehaviorTree.CPPの使い方・実装
--------------------------------

Groot2を使ってビヘイビアツリーを作成する。
--------------------------------

プログラムの実行
--------------------------------

Groot2でモニタリング
--------------------------------

ROSノードをインスタンス化したり、ツリーを作ったりしている関数の中に、以下のコードを追加する。

.. code-block:: cpp

    #include "behaviortree_cpp/loggers/groot2_publisher.h"

.. code-block:: cpp

    const unsigned port = 1667;
    BT::Groot2Publisher publisher(tree, port);

コードの全体例(参考にするだけ)

.. code-block:: cpp

    #include "../include/action_node.hpp"
    #include "ament_index_cpp/get_package_share_directory.hpp"
    #include "behaviortree_cpp/loggers/groot2_publisher.h" // For Groot2Publisher

    using namespace MyActionNodes;
    using namespace BT;

    // メイン関数

    int main(int argc, char* argv[]){
        rclcpp::init(argc, argv);
        ros_node = std::make_shared<BTNode>(); // BTNodeクラスのインスタンスを作成
        BT::BehaviorTreeFactory factory; // ノードの登録を行う
        factory.registerNodeType<Counter>("Counter"); // Counterノードを登録
        factory.registerNodeType<Display>("Display"); // Displayノードを登録
        factory.registerNodeType<GetText>("GetText"); // GetTextノードを登録
        // XMLファイルからBehaviorTreeを作成
        std::string package_path = ament_index_cpp::get_package_share_directory("bt_sample");
        factory.registerBehaviorTreeFromFile(package_path + "/config/main_bt.xml");
        BT::Tree tree = factory.createTree("MainBT");
        printTreeRecursively(tree.rootNode());

        // BehaviorTreeの実行(RUNNINGにする)
        NodeStatus status = NodeStatus::RUNNING;

        const unsigned port = 1667;
        BT::Groot2Publisher publisher(tree, port);

        // ここで、ノードが実行されている
        while(status == NodeStatus::RUNNING && rclcpp::ok()){
            rclcpp::spin_some(ros_node);
            status = tree.tickOnce(); // statusにノードの状態を格納(RUNNING, SUCCESS, FAILURE, IDLE)
            std::this_thread::sleep_for(std::chrono::milliseconds(1000));
        }

        rclcpp::shutdown();
        return 0;
    }