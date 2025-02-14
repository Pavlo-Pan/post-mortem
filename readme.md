### **Post Mortem по инциденту с веб-сервером**  

#### **Описание инцидента**  
На сервере работал веб-сервис, однако в определенный момент он перестал отвечать на запросы. Попытка перейти на сайт по IP-адресу не дала результата.  
![ошибка](1.jpg)
#### **Причины инцидента**  
1. **Переполнение дискового пространства** – лог-файлы веб-сервера заполнили весь доступный объем диска, из-за чего система не могла выполнять критически важные операции.  
2. **Файл конфигурации приложения находился в неправильном месте**, что привело к невозможности его запуска.  
3. **Некорректная настройка задачи в cron**, которая создавала архивы лога каждую минуту, тем самым еще больше заполняя диск.  

#### **Шаги по восстановлению сервиса**  
1. **Подключение к серверу через SSH:**
      ![ошибка](2.jpg)  
   - Исправление прав доступа к приватному ключу.
      ![ошибка](3.jpg)    
   - Успешное подключение к серверу.  

2. **Диагностика состояния сервиса:**  
   - Проверка статуса веб-сервера `httpd` – он был выключен.
     ![ошибка](4.jpg)   
   - Попытка его запуска.
      ![ошибка](7.jpg) 
      ![ошибка](8.jpg)  

3. **Поиск причины сбоя:**  
   - Проверка загрузки процессора (`top`) – нагрузка в норме. 
      ![ошибка](5.jpg)   
   - Проверка свободного дискового пространства (`df -h`) – места нет.  
      ![ошибка](6.jpg)  
   - Поиск крупных файлов (`find / -type f -size +100M`) – обнаружен огромный лог-файл `/var/log/httpd/access_log`.  
      ![ошибка](15.jpg)  

4. **Очистка дискового пространства:**  
   - Удаление лог-файла `/var/log/httpd/access_log`.  
   - Проверка доступного места – теперь достаточно для работы.  

5. **Исправление расположения конфигурационного файла:**  
   - Перемещение `LocalSettings.php` в `/var/www/html/mediawiki/`.  

6. **Настройка механизма очистки логов:**  
   - Создан и настроен скрипт `/home/ec2-user/backup_logs.sh`, который:  
     - Архивирует логи.  
     - Удаляет старые архивы.  
   - Добавлено задание в cron для ежедневного выполнения.  

#### **Результаты**  
- Веб-сервис успешно восстановлен.
      ![ошибка](16.jpg)    
- Проблема переполнения диска устранена.

- Настроен механизм предотвращения подобных инцидентов в будущем.  

#### **Рекомендации по предотвращению подобных инцидентов**  
1. **Мониторинг дискового пространства:**  
   - Настроить автоматические уведомления при низком свободном месте (`df -h`).  
2. **Ограничение размера лог-файлов:**  
   - Настроить `logrotate` для автоматического управления логами веб-сервера.  
3. **Корректная настройка cron-задач:**  
   - Проверять корректность настроенных задач и избегать слишком частого выполнения архивирования.  
4. **Резервное копирование конфигурационных файлов:**  
   - Создать резервные копии критически важных файлов (`LocalSettings.php`).  

**Вывод:** инцидент произошел из-за нехватки дискового пространства, вызванной некорректным хранением логов. Принятые меры позволили восстановить работоспособность системы и минимизировать вероятность повторения аналогичных проблем.

![ошибка](1.jpg)  
![ошибка](2.jpg)  
![ошибка](3.jpg)  
![ошибка](4.jpg)  
![ошибка](5.jpg)  
![ошибка](6.jpg)  
![ошибка](7.jpg)  
![ошибка](8.jpg)  
![ошибка](9.jpg)  
![ошибка](10.jpg)  
![ошибка](11.jpg)  
![ошибка](12.jpg)  
![ошибка](13.jpg)  
![ошибка](14.jpg)  
![ошибка](15.jpg)  
![ошибка](16.jpg)  
![ошибка](17.jpg)    