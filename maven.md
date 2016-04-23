# Работа с MAVEN

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
