# В данном ДЗ содаю три пользователя:

-day

-night

-friday

Настраиваю модуль  pam_time в файле /etc/security/time.conf

*;*;day;Al0800-2000

*;*;night;!Al0800-2000

*;*;friday;Fr

Редактирую файл /etc/pam.d/sshd

...

account required pam_nologin.so

account required pam_time.so

...

После чего в отдельном терминале проверяю доступ к серверу по ssh для созданных пользователей.

# Другой способ
Редактирую файл /etc/pam.d/sshd

...

account required pam_nologin.so

account required pam_exec.so /usr/local/bin/test_login.sh

...

добавил модуль pam_exec и, в качестве параметра, указал скрипт, при запуске котого будет передана переменная окружения PAM_USER, содержащая имя пользователя.

Если имя пользователя friday, то проверям день недели, если пятница, то возвращаем 0, если нет, то 1 и завершаем скрипт.

Если же указан другой пользователь, то в строке

is_day_hours=$(($(test $hour -ge 8; echo $?)+$(test $hour -lt 20; echo $?)))

происходит проверка принадлжит ли текущее значение времени (переменная hour) диапазону от 8 до 20 часов. Если да, то is_day_hours примет значение 0, если нет 1. 

Дальше проверяем имя пользователя и соотвествие ему. Если пользователь day и часы "дневные", то возвращаем 0, если пользователь night и часы НЕ дневные, то так же  возвращаем ноль. 
В противном случае скрипт вернет 1. Если в PAM_USER указано какое-то другое имя пользователя, то скрипт вернет 0.

На основании кода завершения скрипта модуль pam_exec принимает решение. Если вернулся 0, то все в порядке и пользователь будет авторизован, в обратном случае нет.

# Далее устанавливаю модуль pam_script.
В файле /etc/pam.d/sshd переименовываю pam_exec в pam_script:

...

account required pam_nologin.so

Account required pam_script.so /usr/local/bin/test_login.sh

...

Устанавливаю дополнительный пакет nmap-ncat и привожу файл /etc/pam.d/sshd к виду

...

auth include postlogin

auth required pam_cap.so

...

Затем создал файл /etc/security/capability.conf содержащий одну строку:

cap_net_bind_service day и выполняю команду:

#sudo setcap cap_net_bind_service=ei /usr/bin/ncat

Далее захожу под пользователем day и проверяю

#ncat -l -p 80

Ошибки нет

# Так же я могу предоставить выбранному пользователю разные права.

-пользователь заносится в группу wheel

команда usermod -G wheel day

-создается отдельный файл в /etc/sudoers.d/

-отдельная строка в /etc/sudoers

с добавлением строки:

day ALL=(ALL) ALL или day ALL=(ALL) NOPASSWD: ALL
