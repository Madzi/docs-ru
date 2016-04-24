# Работа с MAVEN
POM - Project Object Model

## Настройка
После того как вы установили Maven его можно настроить для оптимальной
производительности.

### Настраиваем размер кучи
По умолчанию, максимальный размер кучи составляет 256 - 512 Мб (-Xms256m -Xmc512m).
Эти ограничения не подходят для сборки больших, сложных проектов Java, и
рекомендуется увеличить размер кучи хотя бы до 1024 Мб. Если вы получили
`java.lang.OutOfMemoryError` в процессе сборки Mavenа, то вам следует увеличить
память. Вы можете воспользоватьсяс глобальной переменной окружения `MAVEN_OPTS`
чтобы установить максимальный размер кучи.

Следующая команда увеличит размер кучи в Linux

```shell
$ export MAVEN_OPTS="-Xmx1024m -XX:MaxPermSize=128m"
```

### Конфигурация
Конфигурация Maven представляет разбита на три различных уровня:
* Глобальная *расположена в `$MAVEN_HOME/conf/settings.xml`*
* Пользовательская *расположена в `$USER_HOME/.m2/settings.xml`*
* Проектная *расположена в `$PROJECT_HOME/pom.xml`*

Файл `settings.xml` для глобальной и пользовательской конфигурации имеет
одинаковую структуру:

```xml
<settings>
  <localRepository />
  <interactiveMode />
  <usePluginRegistry />
  <offline />
  <pluginGroups />
  <servers />
  <mirrors />
  <proxies />
  <profiles />
  <activeProfiles />
</settings>
```

#### Авторизация и аутентификация на прокси
Если в процессе сборки вам необходимо обращаться к репозиторию за фаерволом, то
вам необходим прокси. Настроить прокси можно в файле глобальной конфигурации:
`$MAVEN_HOME/conf/settings.xml`:

```xml
...
  <proxy>
    <id>internal_proxy</id>
    <active>true</active>
    <protocol>http</protocol>
    <username>proxy_user</username>
    <password>proxy_pass</password>
    <host>proxy.host.net</host>
    <port>80</port>
    <nonProxyHosts>local.net|some.host.com</nonProxyHosts>
  </proxy>
...
```

##### Безопасный пароль
Maven хранит конфеденциальные данные, такие как пароль, в файле `settings.xml`.
В целях безопасности не рекомендуется хранить пароль в открытом виде. Maven
предоставляет возможность храрнить зашифрованные данные в `settings.xml`.

1. Необходимо создать мастер-ключ, используя следующую команду: `$ mvn -emp my_master_password`
2. Необходимо создать файл `settings-security.xml` в папке `$USER_HOME/.m2/` и
добавить в него зашифрованный мастер пароль:
```xml
<settingsSecurity>
  <master>{lJ1MrCQ...NcdMr+G}</master>
</settingsSecurity>
```
3. Теперь, когда задан мастер пароль, можно зашифровать пароль от репозитория:
`$mvn -ep my_password`
4. Скопируем значение зашифрованного пароля в `settings.xml`:
```xml
...
  <server>
    <id>cental</id>
    <username>my_username</username>
    <password>{pNBY...Iddn}</password>
  </server>
...
```
5. Возможно вы не захотите хранить зашифрованный мастер пароль в `$USER_HOME/.m2/security-settings.xml`, тогда вы должны прописать в нём следующее:
```xml
<settingsSecurity>
  <relocation>/path/to/my/usb/disk/settings-security.xml</relocation>
</settingsSecurity>
```

Maven использует шифрование AES-128 м PBE SHA-256 (Password-Based Encryption)

### Минимальный POM файл
Минимальный POM файл выглядит следующим образом:

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>my.test</groupId>
  <artifactId>test-name</artifactId>
  <version>0.0.1-SNAPSHOT</version>
</project>
```

### Максимальный POM файл
Максимальный POM файл включает в себя все возможные разедлы:

```xml
<project>
  <parent>...</parent>
  <modelVersion>4.0.0</modelVersion>
  <groupId>...</groupId>
  <artifactId>...</artifactId>
  <version>...</version>
  <packaging>...</packaging>

  <name>...</name>
  <description>...</description>
  <url>...</url>
  <inceptionYear>...</inceptionYear>
  <licenses>...</licenses>
  <organization>...</organization>
  <developers>...</developers>
  <contributors>...</contributors>

  <dependencies>...</dependencies>
  <dependencyManagement>...</dependencyManagement>
  <modules>...</modules>
  <properties>...</properties>

  <build>...</build>
  <reporting>...</reporting>

  <issueManagement>...</issueManagement>
  <ciManagement>...</ciManagement>
  <mailingList>...</mailingList>
  <scm>...</scm>
  <prerequisites>...</prerequisites>

  <repositories>...</repositories>
  <pluginRepositories>...</pluginRepositories>

  <profiles>...</profiles>
</project>
```

### Структура проекта
Можно изменить структуру (расположение папок) проекта, добавив в POM файл
необходимые значения. Ниже в примере использованы значения по-умолчанию:

```xml
<project>
  <modelVersion>4.0.0</modelVersion>
  <groupId>my.test</groupId>
  <artifactId>test-name</artifactId>
  <version>0.0.1-SNAPSHOT</version>

  <build>
    <sourceDirectory>${basedir}/src/main/java</sourceDirectory>
    <testSourceDirectory>${basedir}/src/test/java</testSourceDirectory>
    <outputDirectory>${basedir}/target/classes</outputDirectory>
  </build>
</project>
```

### Разрешение проблем
Если всё работает нормально, то проблем нет. Однако иногда проблемы возникают.
Сборка Maven может падать по различным причинам. Методы приведённые ниже помогут
в большинстве случаев.

#### Включение отладочных записей
По-умолчанию в Maven отладочные записи не выводятся. Для того чтобы увидеть
отладочные записи в процессе сборке используйте ключ  `-X`:

```shell
$ mvn clean install -X
```

#### Дерево зависимостей
Если у вас проблемы с зависимостями в проекте, то первое что необходимо сделать
построить дерево зависимостей. Дерево покажет какая зависимость откуда взялась.
Для построения дерева зависимотей используте комманду:

```shell
$ mvn dependency:tree
```

#### Просмотр переменных окружения и системных свойств
Если  вас в системе стоит несколько JDK, то вы можете получить "магию" при
сборке Maven. Следующая комманда позволит посмотреть переменные окружения и
системные переменные, которые используются при сборке проекта:

```shell
$mvn help:system
```

#### Просмотр окончательного POM файла
Множество настроек в Maven происходит по-умолчанию. То есть вам не нужно
описывать их в POM файле. Однако перед сборкой Maven строит окончательный POM
файл, в который включаются все настройки. Для того чтобы его увидеть необходимо
использовать следующую комманду:

```shell
$ mvn help:effective-pom
```

#### Просмотр зависимостей в classpath
Следующая команда позволит просмотреть все JAR файлы, находящиеся в CLASSPATH:

```shell
$ mvn dependency:build-classpath
```

## Maven Wagon
Maven Wagon - проект, предоставляющий уровень абстрации для загрузки
зависимостей из репозитория. Wagon представляет несколько стандарнтых путей для
коммуникации:
* Файл
* HTTP
* Lightweight HTTP
* FTP
* WebDAV
* SSH
* SCP
* SCM

Настройка Wagon производится в файле глобальной конфигурации Maven -
`$MAVEN_HOME/conf/settings.xml`:

Пусть имеется описанный в одной из конфигураций репозиторий:

```xml
...
  <repositories>
    <repository>
      <snapshots>
        <enabled>false</enabled>
      </snapshots>
      <id>centeral</id>
      <name>Central Repository</name>
      <url>http://repo.maven.apache.org/maven2</url>
    </repository>
  </repositories>
...
```
тогда настройку для него можно задать через `id`:

```xml
...
  <server>
    <id>centeral</id>
    <configuration>
      <httpHeaders>
        <httpHeader>
          <name>CUSTOM_HEADER_NAME</name>
          <value>CUSTOM_HEADER_VALUE</value>
        </httpHeader>
      </httpHeaders>
      <httpConfiguration>
        <get>
          <params>
            <param>
              <name>PARAM_NAME</name>
              <value>PARAM_VALUE</value>
            </param>
          </params>
          <readTimeout>100000</readTimeout>
          <connectionTimeout>100000</connectionTimeout>
        </get>
      </httpConfiguration>
    </configuration>
  </server>
...
```

### Системные свойства
Поведение Maven Wagon может быть изменено через задание системных свойств:
* `maven.wagon.http.pool`: Позволяет разрешить или запретить создавать пул
соединений для HTTP. По-умолчанию значение `true`. Может быть установлено
следующей коммандой: `$ mvn clean install -Dmaven.wagon.http.pool==true`
* `maven.wagon.httpconnectionManager.maxPerRoute`: Устанавливает максимальное
число HTTP соединений для каждого репозитория. Значение по-умолчанию `20`.
* `maven.wagon.httpconnectionManager.maxTotal`: `40`
* `maven.wagon.http.ssl.insecure`: `false`
* `maven.wagon.http.ssl.allowall`: `false`
* `maven.wagon.http.ssl.ignore.validity.dates`: `false`
* `maven.wagon.rto`: `30 min` (sets up in ms)

## Системы контроля версий
Maven поддерживает интеграцию с следующими системами контроля версий: SVN, GIT,
CVS, Bazzar, Jazz, Mercurial, Perforce, StarTeam, CM Synergy. Существует
частичная поддержка: Visual SourceSafe, Team Foundation Server, Rational ClearCase,
AccuRev.

### Maven и SVN
```xml
<project>
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.packt</groupId>
  <artifactId>app</artifactId>
  <version>1.0.0</version>

  <scm>
    <connection>scm:svn:https://svn.wso2.org/repos/wso2/app/src</connection>
    <developerConnection>scm:svn:https://svn.wso2.org/repos/wso2/app/src</developerConnection>
    <url>https://svn.wso2.org/repos/wso2/app</url>
  </scm>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-scm-plugin</artifactId>
        <version>1.9</version>
        <configuration>
          <connectionType>connection</connectionType>
        </configuration>
      </plugin>
    </plugins>
  </build>
  ...
</project>
```

Здесь используется SCM плагин Mavenа для доступа к Subversion. Плагин получает
детали соединения из секции `scm`. Конфигурация SCM описывает два типа соединения
*connection* и *developerConnection*. Значение *connection* должно указывать на
ссылку только для чтения, а *developerConnection* для чтения и записи.
Большинство систем управления версиями позволяют прочитать исходный код через
обычный HTTP, в то время как запись происходит обычно через HTTPS.
Значение *URL* указывает не код, который можно просмотреть в браузере.
