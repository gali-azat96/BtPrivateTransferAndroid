# BtPrivateTransferAndroid
## Документация использования библиотеки для разработчиков.
Для начала, необходимо создать объект класса BtPrivateTransfer.java. Сделать это можно при помощи объекта Builder находящимся в этом же классе.
После инициализации разработчику доступно два метода:
*	startListenBtSocket(ContextWrapper contextWrapper) – вызовом этого метода запускается поток, ожидающий и автоматически принимающий входные запросы. Если удалось соединиться, придет оповещение через метод onConnected(BluetoothSocket socket, BtDevice device, String socketType). В случае ошибки (например, отключение Bluetooth), будет вызван метод onFailure(Exception e);
*	connect(BtDevice device) – попытаться подключиться к устройству device. При успешном запросе произойдет вызов onConnectingSuccess (BtDevice device), в противном случае будет вызван метод onConnectingFailure (BtDevice device, Throwable t).

Чтобы найти доступные устройства можно использовать объект класса BtDeviceDiscovery.java. Для создания этого объекта рекомендуется использовать другой объект – Builder, который нужен для инициализации. В этом классе доступны следующие методы:
*	start() – запуск процесса поиска доступных устройств;
*	stop() – остановка процесса поиска доступных устройств;
*	destroy() – очистка используемых ресурсов.

Для возможности оповещения о событиях, в Builder нужно передать объект класса DeviceDiscoveryCallback.java. Этот объект содержит 2 метода:
*	deviceFounded(BtDevice device) – вызывается, если найдено устройство device.
*	discoveryFinished(Throwable t) – оповещает об окончании поиска, если это произошло вследствие ошибки, то объект созданный при обработке ошибки будет передан параметром.

После того как устройство найдено, т.е. существует объект класса BtDevice, появляется возможность подключения к девайсу посредством метода:
*	connect(BtDevice device) – попытка установки соединения с устройством device.

Для уведомления об успешном или неуспешном соединении используется объект ConnectingCallback.java, в котором необходимо реализовать 2 метода:
*	onConnectingSuccess(BtDevice device) – будет вызвано в случае успешного соединения;
*	onConnectingFailure(BtDevice device, Throwable t) – оповещает об ошибке подключения.

Для успешного соединения необходимо чтобы принимающее устройство находилось в состоянии LISTENING, т.е. должен был вызван метод startListenBtSocket(ContextWrapper contextWrapper). Для получения информации об успешном или неуспешном приеме входящего соединения можно реализовать интерфейс AcceptCallback.java. В этом интерфейсе могут быть вызваны следующие методы:
*	onConnected(BluetoothSocket socket, BtDevice device, String socketType) – в случае успешного соединения;
*	onFailure(Exception e) – в случае ошибки, например отключения Bluetooth.

В результате описанных выше действий будет установлено подключение между двумя устройствами и станет возможным передать данные между ними. 

Для передачи данных у объекта класса BtDeviceDiscovery предусмотрен метод:
*	sendFiles(TransferModel fileMap), который инициирует передачу файлов и необходимых метаданных.

Для уведомления приложения о произошедших события вызывается один из двух методов: 
*	onSendingSuccess(TransferModel model)– в случае успешной передачи;
*	onFailure(Exception e) – в случае ошибки, например отключения Bluetooth или поврежденного файла.

После передачи данных объект BtDeviceDiscovery переходит в состояние CONNECTED и готов для дальнейшего взаимодействия.
Замечание: после завершения работы библиотеки обязательно нужно вызвать метод destroy(), чтобы все ресурсы, использованные во время работы, были освобождены.

### Класс BtDevice.java.
Объекты класса BtDevice содержат информацию о Bluetooth-устройстве, его порядковом номера и текущем взаимодействии между этим устройством и аппаратом, на котором запущено приложение. Данные объекты передаются приложению при поиске доступных Bluetooth-устройств и при изменении состояния подключения к конкретному устройству. Также эти элементы передаются в качестве параметров при вызове методов для подключения или передачи файлов внутри библиотеки.
*	Поле mDevice описывает устройство. Имеет тип BluetoothDevice, который в OC Android описывает экземпляр Bluetooth-устройства. 
*	Поле id указывает на порядковый номер устройств.  
*	Поле isPaired имеет значение true, если установлено соединение с данным устройством, false – в противном случае.

### Класс TransferModel.java.
Этот класс описывает информацию о передаваемом файле между устройствами. 
*	Поле mBytesCount хранит количество байтов, которое занимает файл. 
*	Поле mDescription содержит дополнительную информацию о передаваемом файле, которая может пригодиться принимающему экземпляру приложения. Например, это может быть название файла, id или его тип.
*	Поле mPath указывает место в хранилище, где лежит передаваемый файл. Используется только отправителем, для использования библиотекой.

Это основные классы, служащие для передачи данных между библиотекой и приложением. Теперь рассмотрим класс с которым будет происходить основное взаимодействие.
### Класс BtPrivateTransfer.java.
Основой для взаимодействия с библиотекой является класс BtPrivateTransfer. Этот элемент предоставляет методы для изменения текущего состояния библиотеки, а также связывает библиотеку с приложением, чтобы вовремя уведомлять о произошедших событиях.  Существует 6 возможных состояний.
1.	IDLE – это начальное состояние в котором находится библиотека. Переход в другие состояния возможен вызовом методов:       
    -	startListenBtSocket(ContextWrapper contextWrapper) – переход в состояние LISTENING,
    -	connect(BtDevice device) – переход в состояние CONNECTING.
    
2.	LISTENING – состояние, в котором приложение ожидает входного запроса на подключение. Переход в другие состояния возможен вызовом методов:
    -	connect(BtDevice device) – переход в состояние CONNECTING,
    -	destroy()-переход в состояние IDLE.

Также возможен переход в состояние CONNECTED случае обнаружения подключения:

3.	CONNECTING – состояние, в котором устройство пытается подключиться к другому устройству, которое должно находиться в состоянии LISTENING. Переход в другие состояния возможен вызовом метода:
    -	destroy(), cancelConnecting() - переход в состояние IDLE,
    -	startListenBtSocket(ContextWrapper contextWrapper) – переход в состояние LISTENING.

Объект класса BtPrivateTransfer автоматически перейдет в состояние:

   - CONNECTED – при удачном подключении,
   - FAILURE – при неудавшимся подключении.
    
4.	CONNECTED – состояние, при котором соединение успешно, и библиотека готова к передаче и приеме данных. Также это состояние используется в процессе получения файла. Переход в другие состояние будет выполнен при вызове методов:
    -	destroy(), stopConnectedThread() - переход в состояние IDLE,
    -	sendFiles() – переход в состояние SENDING,
    -	startListenBtSocket(ContextWrapper contextWrapper) – переход в состояние LISTENING.

Объект класса BtPrivateTransfer автоматически перейдет в состояние:
    
   - FAILURE – при разрыве соединения.
    
5.	SENDING – состояние, в котором выполняется передача файлов. Переход в другие состояние будет выполнен при вызове методов:
    -	destroy(), stopConnectedThread() - переход в состояние IDLE,
    -	startListenBtSocket(ContextWrapper contextWrapper) – переход в состояние LISTENING.

Объект класса BtPrivateTransfer автоматически перейдет в состояние:

   - CONNECTED – при окончании передачи,
   - FAILURE – при ошибке.
    
6.	FAILURE – состояние, переход в которое происходит в случае ошибки. Переход в другие состояние будет выполнен при вызове методов:
    -	destroy() - переход в состояние IDLE,
    -	startListenBtSocket(ContextWrapper contextWrapper) – переход в состояние LISTENING.

## Алгоритм работы с библиотекой.
Перед началом использования необходимо создать собственную реализацию класса Service, предоставляемого Android SDK. Это нужно сделать, для того чтобы работа с Bluetooth происходила в фоне и не имела никакой связи UI. Это убережет от утечек памяти и других проблем. Также это даст возможность продолжения работы даже при закрытом приложении. Еще одним плюсом является то, что OC Android дает сервисам высокий приоритет, из-за чего вероятность завершения задач, выполняемых в Service, мала. Заметим, что в Android сервис обязательно нужно зарегистрировать в манифесте приложения. В созданном сервисе будет происходить вся работа с библиотекой. 

Для использования Bluetooth в manifest-файл приложения нужно добавить разрешения:

    <uses-permission android:name="android.permission.BLUETOOTH" />
  	<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
   	<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />.

На устройствах с ОС начиная  Android 6.0 (API level 23) дополнительно нужно запросить Runtime Permission, сделать это можно в activity которое запустится первым. 

