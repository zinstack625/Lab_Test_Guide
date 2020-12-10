# Гайдик по запуску тестов лаб Бородина в Windows

## То что вам понадобится:

- Visual Studio (или другой IDE)
- - MSVC (или GCC из MinGW)
- - CMake
- Клиент Git (рекомендую [консольный](https://git-scm.com/download/win), графический также работает)
- Docker
- WSL 2 (если при установке докера не трогали галочки, установится вместе с ним)
(проверить: `wsl` в `cmd` или `powershell`)
- [Ядро WSL](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

Итак, вы поставили Visual Studio, склонировали репозиторий, захотели запустить, но ничего не вышло?
**Отправьте мне логи, я хочу посмотреть что у вас там**
## Запускаем тесты в вашем IDE

1. (если у вас компилятор из visual studio) В корне вашего репозитория лежит чудесный файлик CMakeLists.txt. В нём необходимо обрамить строчки 24-26 (те которые `string(APPEND...`) в условие:  
`if (CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")`  
`те_самые_строчки`  
`elseif(MSVC)`  
`string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")`  
`string(APPEND CMAKE_CXX_FLAGS " /permissive- /WX /Zi")`  
`endif()`
2. Убедитесь, что в пути до вашего склонированного репозитория (пример: `C:\Users\UserName\GitHub\01-lab-00-intro-UmniyStudent\`) нет символов, отличных от латиницы
3. В качесте цели сборки должно быть `tests` или `tests.exe`

## Что же такое checks и как оно работает?

На данный момент тесты в IDE должны начать работать, но нам также немаловажен скрипт **checks**, его возможно запустить только с среде Linux, на это у нас есть *Docker*
1. Сначала нужно убедиться, что в файлах склонированного репозитория сносом строки выступает **LF**, а не **CRLF**. Для этого:

1.1. Внесём изменения в файл `C:\Program Files\Git\etc\gitconfig`: 
- В разделе '[core]' переменной 'autocrlf' задаём значение 'input'
1.2. Заново клонируем репозиторий, отныне все склонированные через *git* файлы будут иметь окончание строки **LF**
2. Затем устанавливаем Docker (не нужно если уже установлен)
3. Если при установке почему-то не установили WSL, устанавливаем его любым угодным гуглу способом
4. Устанавливаем [ядро WSL](https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi) 
5. Для разверстки контейнера докера используем следующую команду (в `powershell`, **в `cmd` не работает**), предварительно перейдя в директорию с лабой:
`docker run -v ${pwd}:/root/lab -w /root/lab -it rusdevops/bootstrap-cpp`  
5.1. Для GitHub Actions существуют скрипты:
- `scripts/tests.sh` -- тесты на работоспособность вашего кода (смотри первую часть гайда, чтобы обнаружить шикарный способ это делать в IDE)
- `scripts/checks.sh` -- базовая проверка стилистики кода, проверки на утечки памяти и несанкционированный доступ к памяти
- `scripts/coverage.sh` -- проверка на мёртвый код
Все три они в комбинации составляют зелёную галочку на коммите. Соответственно для проверки вашей чудесной гениальности нужно каждый из них запустить и не получить ошибку\
Для запуска нужно в развёрнутом в директории с лабой контейнере (докер) написать `scripts/название_скрипта`\
*Примечание*:\
При запуске `scripts/checks.sh` необходимо вычистить всё, что насобиралось:
- \_builds
- cmake-build-debug или подобные (x64, x86, .vs для visual studio)
- желательно файлики, записываемые вашей лабой (ofstream)
