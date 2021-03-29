# Инструкции к бесплатной части курса Kafka

## 3.Installing Kafka and working with a cluster from one broker

### Введение

**Практическая работа**

В этом уроке мы перейдем к практике и получим фундаментальные знания, необходимые для работы с более высокоуровневым функционалом в дальнейшем.

Мы развернём на нем Кафку в самом простом её варианте — с одним брокером и одной нодой зукипера.

Запишем и прочитаем сообщения, посмотрим в конфиги и увидим как данные хранятся на диске.

Мы специально не будем пользоваться сторонними инструментами и Docker, а только ванильной сборкой, которая доступна на официальном сайте.

Итак, приступим.

**Требования к стендам**

Мы вплотную подобрались к практической части, которая позволит вам поработать с Kafka собственноручно и на практике закрепить понимание вами основных концепций и принципов работы этого ПО.

Однако, в отличие от полного курса, мы не можем предоставить вам стенд для выполнения задания. Но требования к этому стенду совсем несложные, поэтому мы предлагаем вам системные требования и небольшую инструкцию, чтобы любой студент на курсе смог пройти практику.

**Требования к системным ресурсам:**

Как минимум 1 ядро CPU, 1 Gb RAM, 1 Gb disk. 

Однако современные ОС, как правило, требуют больше ресурсов просто для своего запуска и нормальной работы, поэтому скорее всего ресурсов у вас уже достаточно.

**Требования к ПО:**

64-битная ОС, Java Runtime Environment (JRE) 8 или 11.

1. Если у вас ОС Linux или MacOS (современных версий - не старше 5 лет), вы можете установить в систему пакет ``jre-headless`` (либо ``openjdk-11-jre-headless``, названия пакетов у разных дистрибутивов могут отличаться) и делать практику в системном терминале. Других пакетов не понадобится, будут использоваться простые команды типа wget, tar, dd, встроенные в bash и т.д.

Проверить, что JRE установлена и нужной версии, можно командой:

``java –version``

Получим примерно такой ответ:

```
openjdk 11.0.10 2021-01-19 LTS
OpenJDK Runtime Environment Zulu11.45+27-CA (build 11.0.10+9-LTS)
OpenJDK 64-Bit Server VM Zulu11.45+27-CA (build 11.0.10+9-LTS, mixed mode)
```

2. Если у вас относительно новый ПК и установлен Windows 10 – вы можете установить себе **WSL** (Windows Subsystem for Linux), таким образом вы получите терминал с почти полнофункциональной ОС Linux.

Вот здесь можно почитать, как это делается: https://docs.microsoft.com/ru-ru/windows/wsl/install-win10


Далее можно перейти к пункту 1.

3. В Windows 8/10 (и во многих других ОС, в том числе и Linux или MacOS), вы можете установить простой гипервизор **Virtualbox** (https://www.virtualbox.org/) и запустить образ с Linux.

Образы можно найти например вот тут: https://www.linuxvmimages.com/


Спикер использовал для записи **CentOS 7**, вот ссылка сразу на подходящий образ: https://sourceforge.net/projects/linuxvmimages/files/VirtualBox/C/7/CentOS_7.8.2003_VBM.zip/download 

Далее – смотрите пункт 1, потребуется доустановить пакет jre-headless и можно приступать к заданию.

4. Если у вас есть опыт работы с облачными провайдерами или другими гипервизорами, вы уже знаете что делать – создайте виртуальную машину с Linux и установите JRE 8 или 11.

К сожалению, техническую поддержку по стендам бесплатной части курса по Apache Kafka мы не оказываем. Однако если вы хотите успешно освоить навыки и работать с брокером сообщений Apache Kafka, скорее всего нужный уровень знаний Linux у вас уже есть.

### Запуск Kafka

Запуск кластера из одной ноды

Скачиваем архив с кафкой:

```
wget https://archive.apache.org/dist/kafka/2.7.0/kafka_2.13-2.7.0.tgz
tar -xzf kafka_2.13-2.7.0.tgz
cd kafka_2.13-2.7.0
```

Первым делом мы запускаем Zookeeper. Как мы уже обсуждали, Кафка пользуется зукипером для хранения метаданных, а также для координации своей работы (выбора лидеров партиций и контроллера).

```
./bin/zookeeper-server-start.sh config/zookeeper.properties
```

Запускаем брокер Кафки

```
./bin/kafka-server-start.sh config/server.properties
```

### Запись и чтение сообщений

Создаем топик с регистрациями
```
./bin/kafka-topics.sh --create --topic registrations --bootstrap-server localhost:9092
```
Посмотрим на его конфигурацию
```
./bin/kafka-topics.sh --describe --topic registrations --bootstrap-server localhost:9092
```
Давайте запишем первое сообщение
```
./bin/kafka-console-producer.sh --topic registrations --bootstrap-server localhost:9092
>Hello world!
>Hello Slurm!
```
Попробуем его прочитать
```
./bin/kafka-console-consumer.sh --topic registrations --bootstrap-server localhost:9092
```
И… ничего не происходит!

В эту ситуацию попадают многие люди, впервые использующие кафку. Все дело в том, что консьюмер Kafka по умолчанию начинает читать данные с конца топика (см. настройку ```auto.offset.reset```). Для того чтобы прочитать данные с начала, мы должны переопределить эту конфигу.
```
./bin/kafka-console-consumer.sh --topic registrations --bootstrap-server localhost:9092 --consumer-property auto.offset.reset=earliest
```
Или можно сказать “```--from-beginning```”. Ура, мы произвели запись и чтение! Запишем еще одно сообщение — увидим, что оно появляется в консюмере. Обратим внимание на лог и увидим, что каждый запуск консольного консюмера создает новую группу.

Давайте вместо этого зададим свою:
```
./bin/kafka-console-consumer.sh --topic registrations --group slurm --bootstrap-server localhost:9092 --consumer-property auto.offset.reset=earliest
```
Видим сообщения, все ок, а теперь давайте перезапустим... Снова ничего! Вспомним из прошлой лекции, что консюмер группы в Кафке могут коммиттить свои оффсеты для топик-партиции, чтобы при перезапуске продолжать чтение с последней запомненной позиции. Именно это поведение мы и видим. Давайте проверим
```
./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group slurm --describe
```
А теперь сбросим позицию обратно на начало
```
./bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group slurm --to-earliest --reset-offsets --execute --topic registrations
```
При новом запуске консюмера снова давайте отключим автоматический коммит оффсетов
```
./bin/kafka-console-consumer.sh --topic registrations --bootstrap-server localhost:9092 --group slurm --consumer-property auto.offset.reset=earliest --consumer-property enable.auto.commit=false
```
Если запустим консюмера снова, то увидим, что сообщения читаются. Этим мы можем подтвердить, что оффсеты не коммитятся (+ оставим запущенным консюмера, чтобы увидеть идентификатор и адрес).

### Topic Retention Часть 1

**Очистка топика**

Итак, мы успешно записали и прочитали сообщения, давайте посмотрим на механизм ретеншена
```
./bin/kafka-topics.sh --describe --topic registrations --bootstrap-server localhost:9092
```
По умолчанию Кафка проверяет, нужно ли удалить данные по ретеншену каждые 5 минут. Давайте сделаем этот интервал меньше — каждую секунду.

Остановим брокер Кафки и открываем файл на редактирование:
```
vi config/server.properties
```
И выставляем там нужное значение:
```
log.retention.check.interval.ms=1000
```
После этого запускаем снова брокер.

Давайте теперь скажем Кафке, что мы хотим удалять сообщения после одной минуты:
```
./bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name registrations --alter --add-config retention.ms=60000
```
Запустив заново консюмера мы увидим, что сообщения действительно удалились. В логе видим “Found deletable segments with base offsets”.

Давайте поподробнее разберем этот момент. Повторим эксперимент, слегка изменив настройки. 

Скажем Кафке удалять данные после 10 секунд:
```
./bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name registrations --alter --add-config retention.ms=10000
```
А теперь запустим продюсера писать сообщения в цикле.

В одном терминале запустим вот такую конструкцию:
```
touch /tmp/data && tail -f -n0 /tmp/data | ./bin/kafka-console-producer.sh --topic registrations --bootstrap-server=localhost:9092 --sync
```
А во втором терминале - вот такую:
```
for i in $(seq 1 3600); do echo "test${i}" >> /tmp/data; sleep 1; done
```
Читая сообщения спустя минуту, мы по-прежнему видим старые сообщения!
```
./bin/kafka-console-consumer.sh --topic registrations --bootstrap-server localhost:9092 --consumer-property auto.offset.reset=earliest
```
Для того чтобы понять, что происходит, мы должны разобраться во внутренней структуре данных партиции.

### Структура Партиции

Полный перечень настроек здесь:
https://kafka.apache.org/documentation/#configuration

### Topic Retention Часть 2

Очистка топика (продолжение)

Заглянув в папку с данными, видим активный сегмент (а также старые сегменты, помеченные как “deleted”).
```
ls -la /tmp/kafka-logs/registrations-0
```
Выставим более частую ротацию нашему топику, раз в 10 секунд:
```
./bin/kafka-configs.sh --bootstrap-server localhost:9092 --entity-type topics --entity-name registrations --alter --add-config segment.ms=10000
```
Теперь мы видим ротацию сегмента в логе.

И, через некоторое время, увидим что данные удалились - благодаря удалению ротированного сегмента.

### Log Compaction

Живой пример такого приложения: Confluent Schema Registry
https://github.com/confluentinc/schema-registry 

### ZooKeeper 

Информация о кластере Kafka в ZooKeeper
Под конец урока давайте заглянем в ZooKeeper, и посмотрим какую информацию он хранит.

Откроем консоль zookeeper:
```
./bin/zookeeper-shell.sh localhost:2181
```
И уже там можем посмотреть, что у нас есть:
```
ls /
get /controller
get /brokers/topics/registrations/partitions/0/state
stat /brokers/ids/0
```

Итак, вы стали на шаг ближе к тому, чтобы стать опытным пользователем Apache Kafka, поздравляем! :)
Однако, если вы пройдете наш продвинутый курс по Apache Kafka, то сможете стать магистром ордена брокеров сообщений!

Авторы на практике научат работать с Apache Kafka — платформой для передачи и обработки событий в реальном времени. Настраивать распределенный отказоустойчивый кластер, отслеживать метрики, равномерно распределять нагрузку.

**Посмотреть программу продвинутого практического курса: https://slurm.club/31urcgo**
