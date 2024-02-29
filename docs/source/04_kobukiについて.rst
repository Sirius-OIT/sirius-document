kobukiについて
================================================================

見るべき関数
----------------------------------------------------------------

マイコンにシリアル通信経由でデータを送信する関数
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
kobuki_core/src/driver/kobuki.cpp
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
.. code-block:: c++

    void Kobuki::sendCommand(Command command)
    {
    if( !is_alive || !is_connected ) {
        if( !is_alive     ) sig_debug.emit("Serial connection opened, but not yet receiving data.");
        if( !is_connected ) sig_debug.emit("Serial connection not open.");
        // std::cout << is_enabled << ", " << is_alive << ", " << is_connected << std::endl;
        return;
    }
    command_mutex.lock();
    kobuki_command.resetBuffer(command_buffer);

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
