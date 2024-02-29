kobukiについて
================================================================

見るべき関数
----------------------------------------------------------------

kobukiがspinしてるときに呼ばれる関数(メインで動いてる関数)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
- `void Kobuki::spin()` で `sendBaseControlCommand()` が呼ばれる

マイコンに送信するデータの型を作成するやつ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
``kobuki_core/src/driver/command.cpp``

.. code-block:: c++

    bool Command::serialise(ecl::PushAndPop<unsigned char> & byteStream)
    {
        // need to be sure we don't pass through an emum to the Trans'd buildBytes functions.
        unsigned char cmd = static_cast<unsigned char>(data.command);
        unsigned char length = 0;
        switch (data.command)
        {
            case BaseControl: // cmd == 1
            buildBytes(cmd, byteStream); // 1
            buildBytes(length=4, byteStream); // データの長さ
            buildBytes(data.speed, byteStream); // 速度 [mm/s]
            buildBytes(data.radius, byteStream); // 回転半径 [mm]
            break;
    ...


マイコンにシリアル通信経由でデータを送信する関数
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
``kobuki_core/src/driver/kobuki.cpp``

.. code-block:: c++

    void Kobuki::sendBaseControlCommand()
    {
        std::vector<double> velocity_commands_received; // 受信した速度指令を格納する変数
        if( acceleration_limiter.isEnabled() ) { // 加速度制限が有効なら
            velocity_commands_received=acceleration_limiter.limit(diff_drive.pointVelocity()); // 加速度制限をかける
        } else {
            velocity_commands_received=diff_drive.pointVelocity(); // 加速度制限が無効ならそのまま
        }
        diff_drive.velocityCommands(velocity_commands_received); // 速度指令をセット
        std::vector<short> velocity_commands = diff_drive.velocityCommands(); // 送信データの生成
        // std::cout << "speed: " << velocity_commands[0] << ", radius: " << velocity_commands[1] << std::endl;
        sendCommand(Command::SetVelocityControl(velocity_commands[0], velocity_commands[1])); // 送信する関数に投げる

        //experimental; send raw control command and received command velocity
        velocity_commands_debug=velocity_commands;
        velocity_commands_debug.push_back((short)(velocity_commands_received[0]*1000.0)); // mm/s → m/s
        velocity_commands_debug.push_back((short)(velocity_commands_received[1]*1000.0));
        sig_raw_control_command.emit(velocity_commands_debug);
    }

.. code-block:: c++

    void Kobuki::sendCommand(Command command) // 引数：(速度[mm/s], 回転半径[mm])
    {
        if( !is_alive || !is_connected ) {
            if( !is_alive     ) sig_debug.emit("Serial connection opened, but not yet receiving data.");
            if( !is_connected ) sig_debug.emit("Serial connection not open.");
            // std::cout << is_enabled << ", " << is_alive << ", " << is_connected << std::endl;
            return;
        }
        command_mutex.lock();
        kobuki_command.resetBuffer(command_buffer);

        // command.cppでデータの型作成
        if (!command.serialise(command_buffer))
        {
            sig_error.emit("command serialise failed.");
        }
        command_buffer[2] = command_buffer.size() - 3;
        unsigned char checksum = 0;
        for (unsigned int i = 2; i < command_buffer.size(); i++)
            checksum ^= (command_buffer[i]);

        command_buffer.push_back(checksum);

        // ここでシリアル通信でデータを送信している
        serial.write((const char*)&command_buffer[0], command_buffer.size());

        // ここで送信したデータを表示している
        sig_raw_data_command.emit(command_buffer);
        command_mutex.unlock();
    }