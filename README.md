# quik-lua-rpc
RPC-сервис для вызова процедур из QLUA -- Lua-библиотеки торгового терминала QUIK (ARQA Technologies).
An RPC-service over the qlua library API for the QUIK trading terminal

Что это?
--------
*COMING SOON*

Как пользоваться?
--------
### Установка программы

Скопировать репозиторий в `%PATH_TO_QUIK%/lua/`, где `%PATH_TO_QUIK%` -- путь до терминала QUIK. Если папки `lua` там нет, нужно её создать.

### Установка зависимостей

### Легко

*COMING SOON*

### Сложно

##### Установка *LuaRocks* (менеджер пакетов для Lua)
1. Где взять
	* Архивы с дистрибутивами: http://luarocks.github.io/luarocks/releases/
	* Инструкцию по установке можно найти здесь: https://github.com/luarocks/luarocks/wiki/Installation-instructions-for-Windows
2. Разархивировать, в командной строке Windows (cmd.exe) перейти в разархивированную папку.
3. Установить: `install.bat /NOREG /L /P %PATH_TO_LUAROCKS%`, где %PATH_TO_LUAROCKS% -- путь, куда нужно установить LuaRocks. Например, `D:/Programs/Lua/LuaRocks`.

	Почитав мануал, опции для установки можете настроить по своему вкусу.
	Например, /L значит "установить также дистрибутив Lua в папку с LuaRocks" -- он нам пригодится далее, т.к. не у всех стоит отдельный дистрибутив Lua.

	На самом деле, нам нужна не вся Lua, а её бинарники (.dll) и заголовочные файлы. Если не хотите ставить ту, что идёт с LuaRocks, то минимальный набор файлов можно взять здесь: http://luabinaries.sourceforge.net/download.html (например, `lua-5.3.4_Win32_bin.zip`). Качать нужно 32-битные версии (Win32), т.к. QUIK использует 32-битную Lua.
	
##### Установка *protobuf* (библиотека для сериализации/десериализации)
1. Скачать Lua-биндинг для protobuf отсюда: https://github.com/Enfernuz/protobuf-lua

	Это форк форка форка :smile:, наверное, единственного Lua-биндинга для protobuf. По  мере работы с ним я внёс некоторые изменения в плагин для генерации Lua-кода, поэтому эта версия будет полезна тем, кто пожелает доработать RPC-сервис по своему усмотрению.
2. Папку `protobuf` поместить в `%PATH_TO_QUIK%/lua/`
3. (Опционально) Скомпилировать файл protobuf/pb.c как DLL под свою машину. 
	
	Кому не хочется возиться, можете попробовать использовать уже готовую pb.dll: в папке dependencies/protobuf/ можно найти сборки 	под версии терминала 7.2.2.3 и 7.14.1.7. Думаю, любая из них подойдёт, т.к. линковка pb.dll осуществлялась с qlua.dll, которая вряд ли сильно менялась от версии к версии.
	
	Для компиляции я пользовался MinGW с командной оболочкой в виде MSYS.
	1. В терминале MSYS переместиться в папку /protobuf/, где находится файл pb.c
	2. Чтобы файл скомпилировался под Windows, нужно убрать/закомментировать строчки 23-33:
		```С
		#if defined(_ALLBSD_SOURCE) || defined(__APPLE__)
		#include <machine/endian.h>
		#else
		#include <endian.h>
		#endif
		```
		Эти строчки можно убрать безболезненно, т.к. процессоры архитектуры x86 и amd64 имеют little endianness, так что препроцессор не вставит функции из endian.h, которые используются далее в файле, в конечный код.
	
	3. Получить объектный файл: `gcc -O3 -I%PATH_TO_LUA%/include -с pb.c`, где `%PATH_TO_LUA%` -- путь до дистрибутива Lua. Если ставили Lua в комплекте с LuaRocks, то это будет путь до LuaRocks. 
	
		Пример: `gcc -O3 -ID:/programs/LuaRocks/include -с pb.c`
	
	4. Получить DLL: `gcc -shared -o pb.dll pb.o -L%libraries_folder% -l%lua_library%`, где `%libraries_folder%` -- папка с .dll-библиотеками Lua, `%lua_library%` -- имя .dll-библиотеки Lua.
	
		Пример:
		* `%libraries_folder%` -- `D:/QUIK`
		* `%lua_library%` -- `qlua`
		* Итого: `gcc -shared -o pb.dll pb.o -LD:/QUIK -lqlua`

		Линковать лучше с прокси-библиотекой Lua (qlua.dll), которая поставляется в коробке с QUIK. Не уверен, что если слинковаться с DLL из, например, Lua for Windows, или с той, что поставляется с LuaRocks, то всё будет работать. Я пробовал линковаться также с lua5.1.dll, которая находится в корне QUIK, но при запуске скрипта получал ошибку, связанную с загрузкой библиотек.
	
4. Файл pb.dll положить в `%PATH_TO_QUIK%/Include/protobuf/` , где `%PATH_TO_QUIK%` -- путь до терминала QUIK (например, `D:/QUIK`). Если папки `Include` нет, необходимо её создать.
	
##### Установка *lzmq* (Lua-биндинг для ZeroMQ -- библиотеки для межпроцессной коммуникации)
1. Скачать бинарники ZeroMQ для Windows
	* Страница с дистрибутивами: http://zeromq.org/distro:microsoft-windows
	* Пример дистрибутива: http://miru.hk/archive/ZeroMQ-4.0.4~miru1.0-x86.exe -- 32-битная версия, т.к. 64-битная не подойдёт.
2. Устанавливаем в `%PATH_TO_ZMQ%` -- путь выбираем произвольно, например, `D:/programs/ZeroMQ`.

	**Важно:** при установке выбрать галку `ZeroMQ headers and libraries`.
3. После установки переходим в `%PATH_TO_ZMQ%/bin` -- там лежат бинарники (.dll) от ZMQ под Windows.
	1. Определяем .dll-файл, соответствующий своей версии Windows (например, для Windows 7 это будет `libzmq-v120-mt-4_0_4.dll`). 
	
		**Важно:** Не перепутайте с бинарниками, содержащими в имени *gd* -- это библиотеки, собранные для работы в debug-режиме.	
	2. Копируем найденный .dll-файл, копию переименовываем в `libzmq.dll`.
4. Переходим в `%PATH_TO_ZMQ%/lib` -- там лежат .lib-файлы от ZMQ под Windows.
	1. Определяем .lib-файл, по имени соответствующий выбранному .dll-файлу.
	2. Копируем найденный .lib-файл, копию переименовываем в `libzmq.lib`.
5. Дальше нам будет нужен компилятор от Microsoft -- MSVC. Он входит в Visual Studio. Можно скачать бесплатную MS Visual Studio Express Edition, накликать там самый минимум при установке (поддержка C/C++). Это, наверное, самый запарный по времени пункт из всех. Инструкцию не прилагаю -- надеюсь, там всё довольно просто.
6. Установка пакета `lzmq` с помощью LuaRocks
	1. Открыть Developer Command Prompt, которая поставляется с Visual Studio (можно найти через Пуск, начав искать "Command").
	2. В Developer Command Prompt перейти в `%PATH_TO_LUAROCKS%` (путь, куда установили LuaRocks)
	3. Выполнить команду: 
	
	`luarocks install lzmq ZMQ_DIR="%PATH_TO_ZMQ%"`, где `%PATH_TO_ZMQ%` -- путь до установленного в п. 2 ZeroMQ.
	
	Пример: `luarocks install lzmq ZMQ_DIR="D:/programs/ZeroMQ 4.0.4"`
	
	LuaRocks начнёт устанавливать библиотеку `lzmq`, попутно собирая её из исходников. Делает он это с помощью компилятора `cl` -- за этим мы и ставили MSVC и заходили в Developer Command Prompt.
7. После установки `lzmq` заходим в `%PATH_TO_LUAROCKS%/systree/lib/lua/5.1` (путь после `%PATH_TO_LUAROCKS%` у вас может отличаться, если при установке LuaRocks вы использовали опцию `/TREE %dir%`) и копируем содержимое папку `lzmq` и файл `lzmq.dll` в `%PATH_TO_QUIK%/Include`.
8. Заходим в `%PATH_TO_LUAROCKS%/systree/share/lua/5.1` и копируем папку `lzmq` в `%PATH_TO_QUIK%/lua`
9. `lzmq.dll`, которую собрал LuaRocks, линкуется с `libzmq.dll`, причём реально в зависимостях оказывается не `libzmq.dll`, а её первоначальная версия, которую мы переименовали в `libzmq.dll` (например, `libzmq-v120-mt-4_0_4.dll`). Для того, чтобы библиотека `libzmq` была доступна в рантайм, необходимо скопировать соответствующий файл (например, `libzmq-v120-mt-4_0_4.dll`) из `%PATH_TO_ZMQ%/bin` в `%PATH_TO_QUIK%`. 

	Есть вариант, где можно использовать libzmq.dll вместо оригинала, но он предполагает:
	* генерацию .def-файла из libzmq.dll
	* генерацию libzmq.lib из .def-файла
	* повторную сборку lzmq LuaRocks'ом. Тогда у lzmq.dll в зависимостях окажется именно libzmq.dll, а не, скажем, libzmq-v120-	mt-4_0_4.dll
	
	Как видите, вариант геморройный, поэтому проще скопировать оригинальную версию libzmq-???.dll в корень QUIK.
	
	Ещё более геморройный вариант: собрать статические версии libzmq из исходников ZMQ, но этот способ мы оставим особо пытливым и разбирающимся в теме.
	
#### Запуск программы
В терминале QUIK в меню Lua-скриптов добавить скрипт `%PATH_TO_SERVICE%/service.lua`, где `%PATH_TO_SERVICE%` -- путь до папки с программой включительно (например, `D:/QUIK/lua/quik-lua-rpc`).

Адрес, по которому доступен RPC-компонент, определяется в функции `OnInit` в первом аргументе метода `QluaService:start`.

Адрес, по которому доступен компонент для рассылки событий, определяется в функции `OnInit` во втором аргументе метода `QluaService:start`.

Вы можете использовать как оба компонента, так и один из них (передавая в качестве адреса для другого компонента `nil`).
На данный момент ZeroMQ под Windows не поддерживает IPC-абстракцию (`ipc://`), поэтому для транспортного уровня остаётся (`tcp://`).

Примеры
--------
*COMING SOON*
