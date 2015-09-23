# Mutyda Android Library
**Эта библиотека предназнача для получения авторизации на то или иное действие в вашем приложении посредством сервиса https://mutyda.com**

Подготовка к использованию данной библиотеки
----------
Прежде чем использовать данную библиотеку вам необходимо зайти в личный кабинет партнера на сайте  **[mutyda.com](https://mutyda.com/pcabinet.aspx)** и создать новый сценарий указав в поле “Тип сценария” значение “Авторизация в Android приложении”. Когда вы создадите новый сценарий запомните значение "ID сценария", оно вам понадобится в дальнейшем

Добавление библиотеки в ваш Eclipse проект
-----------
Для добавления библиотеки, просто скопируйте файл  mutyda.jar в папку lib  вашего проекта

![build.gradle](https://mutyda.com/images/for-git/a1.png "build.gradle")

Добавление библиотеки в проект на Android Studio
-----------
Для добавления библиотеки вам необхоимо в ваш проект добавить одну зависимость на файл mutyda.jar. Для добавления этой зависимости вы можете например скопировать файл mutyda.jar в папку .\app\libs вашего проекта, а затем зайдя в окно Project Structure, выбрав модуль app, на закладке Dependencies добавить File dependency выбрав из откывшегося диалогового окна файл \libs\mutyda.jar

![build.gradle](https://mutyda.com/images/for-git/a2.png "build.gradle")

Использование библиотеки
-----------
После подключения библиотеки mutyda.jar к вашему проекту, вы должны в манифесте вашего приложения AndroidManifest.xml обязательно указать ссылку на Activity, которая находится в данной библиотеке.

```xml
< activity
            android:name ="com.meshsoft.mutyda.MutydaHandler"
            android:launchMode ="singleTask" >
            < intent-filter>
                < action android:name ="android.intent.action.VIEW" />
                < category android:name ="android.intent.category.DEFAULT" />
                < category android:name ="android.intent.category.BROWSABLE" />
                < data android:host ="auth" android:scheme= "mutyda" />
            </ intent-filter>
</ activity>
```

В качестве значения атрибутов android:host и android:scheme вы можете указать ваши, если они вам нужны либо оставить значения как в приведенном выше примере.

Так же в манифесте вашего приложения в разделе application вы обязательно должны указать следущие мета данные

```xml
< meta-data
            android:name ="com.meshsoft.mutyda.Scenario_ID"
            android:value ="id_вашего_сценария" >
</ meta-data>
```

В качестве значени атрибута android:value  вы должны указать ID вашего сценария, который автоматически сгенерировался в момент его создания в **[личном кабинете](https://mutyda.com/pcabinet.aspx)** партнера на сайте mutyda.com

Авторизация с помощью Mutyda сервиса
-----------

Для осуществления авторизации посредством Mutyda,  для начала вы должны реализовать интерфейс MutydaHelper.AuthCallback в котором две функции success и error Сделать это можно например так:

```java
MutydaHelper.AuthCallback callback= new MutydaHelper.AuthCallback() {
            
             @Override
             public void success(JSONObject liderData) {
            }

             @Override
             public void error(JSONObject decineUserData) {

            }
};
```

В том участке кога, где вам необходимо выполнить Mutyda авторизацию для того или иного вашего сценария, вы должны вызвать статический метод MutydaHelper.MutydaAuth, который объявлен со следующими образом:

```java
public static Boolean MutydaAuth(Context context, String sessionGuid,
					String callbackUrl, AuthCallback callback)
```

**context** -  контекст вашего приложения либо ваше Activity, которое вызывает данный метод

**sessionGuid** - это любая уникальная текстовая последовательность, которую вы должны сгенерировать и которая подобно сесии в Web веб приложениях идентифицирует сеанс. Вы можете использовать например UUID.randomUUID().toString() для получения случайного идентификатора
ВНИМАНИЕ:

**callbackUrl** - данный параметр должен соответсвовать тем данным которые вы указали в манифесте вашего приложения, при описании активити ***com.meshsoft.mutyda.MutydaHandler***
Если смотреть на приведенный выше пример описания этой активити, то видно, что определен элемент ***< data android:host ="auth" android:scheme= "mutyda" />***
В данном случае значение callbackUrl должно быть mutyda://auth

**callback** - это ссылка на экземпляр класса, который реализует интерфейс MutydaHelper.AuthCallback либо вы можете указать интерфейсную переменную типа MutydaHelper.AuthCallback

После вызова этого главного метода ***MutydaHelper.MutydaAuth*** пользователь будет перенаправлен в браузер на безопасную страницу авторизации сервиса mutyda.com. В случае успешного осуществления авторизации окно браузера будет закрыто и будет вызван реализованный вами метод success интерфейса MutydaHelper.AuthCallback Если авторизация бует запрещена одним из членом mutyda группы, бует вызван  реализованный вами метод error интерфейса MutydaHelper.AuthCallback 

В обоих реализованных вами методах success и error интерфейса MutydaHelper.AuthCallback передается единственный параметр типа JSONObject в котором находятся все необходимые вам данные, а именно: в случае успешной авторизации это будут данные лидера группы (фамилия, имя, тел. и т.д.), в случае не успешной авторизации передается имя того члена группы, кто отказал в авторизации. 

Имена полей этого JSONObject который передаентся в методы success и error могут добавляться, на данный момент это следующие имена полей

JSON свойство | Назначение |
--- | --- |
first_name | Имя лидера группы |
last_name | Фамилиия лидера группы |
second_name | Отчество лидера группы |
user_id|ID лидера группы в системе Mutyda  |
phone|Телефон лидера группы  |
token|авторизационный токен (вам не понадобиться его использовать где либо)  |
email|Отчество лидера группы  |
status|Статус авторизации (true все в порядке, fase запрет авторизации)  |
android_guid|внутренний идентификатор запросa в системе Mutyda|
user_decline|используется только в методе error показываем имя пользователя из группы, кто запретил авторизацию  |

Пример интеграции Android приложения и сервиса Mutyda вы можете скачать с GitHub **[https://github.com/mutyda/mutyda-auth-example](https://github.com/mutyda/mutyda-auth-example)** 


## License

The MIT license (Refer to the [LICENSE.md][license] file)

 [license]: https://github.com/mutyda/android-library/blob/master/LICENSE.md
