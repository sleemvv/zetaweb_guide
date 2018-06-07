# Настройка редиректов \(IIS\)

Zeta Web в качестве веб-сервера использует  [Internet Information Services \(IIS\)](https://www.iis.net/).

Настройка редиректов \(а также переопределений\) происходит через Диспетчер служб IIS \(Internet Information Services Manager\) с установленными расширениями:

1. [URL Rewrite](https://www.iis.net/downloads/microsoft/url-rewrite). Данное расширение обычно устанавливается вместе с веб-сервером IIS
2. [IIS Manager for Remote Administration](https://www.iis.net/downloads/microsoft/iis-manager). Модуль необходим для удаленного администрирования IIS в случае отсутствия прямого доступа к диспетчеру служб IIS на хостинге.

{% hint style="info" %}
Zeta Web поддерживает автоматические редиректы и не требует настройки модуля URL Rewrite в следующих случаях:

* изменение url страницы
* изменение url папки каталога
* изменение url номенклатуры и характеристики номенклатуры
* изменение url информационного блока
* измненеие url файла / картинки

При изменении url средствами CMS.
{% endhint %}

### Пример настройки 301 редиректа \(видео\)

Пример настройки при переходе на Zeta Web с другого решения.

{% embed data="{\"url\":\"https://youtu.be/Zamy8sijU0g\",\"type\":\"video\",\"title\":\"Zeta Web. Настройка редиректов средствами IIS\",\"description\":\"В данном видео рассмотрен пример настройки 301 редиректа\",\"icon\":{\"type\":\"icon\",\"url\":\"https://www.youtube.com/yts/img/favicon\_144-vfliLAfaB.png\",\"width\":144,\"height\":144,\"aspectRatio\":1},\"thumbnail\":{\"type\":\"thumbnail\",\"url\":\"https://i.ytimg.com/vi/Zamy8sijU0g/maxresdefault.jpg\",\"width\":1280,\"height\":720,\"aspectRatio\":0.5625},\"embed\":{\"type\":\"player\",\"url\":\"https://www.youtube.com/embed/Zamy8sijU0g?rel=0&showinfo=0\",\"html\":\"<div style=\\\"left: 0; width: 100%; height: 0; position: relative; padding-bottom: 56.2493%;\\\"><iframe src=\\\"https://www.youtube.com/embed/Zamy8sijU0g?rel=0&amp;showinfo=0\\\" style=\\\"border: 0; top: 0; left: 0; width: 100%; height: 100%; position: absolute;\\\" allowfullscreen scrolling=\\\"no\\\"></iframe></div>\",\"aspectRatio\":1.7778}}" %}

### Полезные ссылки

* [Установка IIS на Windows Server](https://firstvds.ru/technology/install-iis)
* [Использование модуля URL Rewrite](https://docs.microsoft.com/ru-ru/iis/extensions/url-rewrite-module/using-the-url-rewrite-module) \(документация с официального сайта Microsoft на английском языке\)
* [Создание правил ](https://docs.microsoft.com/ru-ru/iis/extensions/url-rewrite-module/creating-rewrite-rules-for-the-url-rewrite-module)\(документация с официального сайта Microsoft на английском языке\)
* [Импорт правил из Apache](https://docs.microsoft.com/en-us/iis/extensions/url-rewrite-module/importing-apache-modrewrite-rules) \(документация с официального сайта Microsoft на английском языке\)
* [Статья про настройку правил и использование регулярных выражений](https://habr.com/post/91797/)

{% tabs %}
{% tab title="Пример файла \*.htaccess" %}
```text
RewriteEngine On
# 301 --- http://www.test.com/test/ => http://www.test.com/spiders/
RewriteRule ^test/$ /spiders/? [L,R=301]

# 301 --- http://www.test.com/faq.html?faq=13&layout=bob => http://www.test2.com/faqs.html
RewriteCond %{HTTP_HOST} ^www\.test\.com$
RewriteCond %{QUERY_STRING} (^|&)faq\=13($|&)
RewriteCond %{QUERY_STRING} (^|&)layout\=bob($|&)
RewriteRule ^faq\.html$ http://www.test2.com/faqs.html? [L,R=301]

# 301 --- http://www.test3.com/faq.html?faq=13&layout=bob => bbq.html
RewriteCond %{QUERY_STRING} (^|&)faq\=13($|&)
RewriteCond %{QUERY_STRING} (^|&)layout\=bob($|&)
RewriteRule ^faq\.html$ /bbq.html? [L,R=301]

# 301 --- text/faq.html?faq=20 => helpdesk/kb.php
RewriteCond %{QUERY_STRING} (^|&)faq\=20($|&)
RewriteRule ^text/faq\.html$ /helpdesk/kb.php? [L,R=301]

```
{% endtab %}

{% tab title="Результат импорта правил в IIS" %}
```markup
<rewrite>
  <rules>
    <!--# 301 - - - http://www.test.com/test/ => http://www.test.com/spiders/-->
    <rule name="Импортированное правило 1" stopProcessing="true">
      <match url="^test/$" ignoreCase="false" />
      <action type="Redirect" redirectType="Permanent" url="/spiders/?" appendQueryString="false" />
    </rule>
    <rule name="Импортированное правило 2" stopProcessing="true">
      <match url="^faq\.html$" ignoreCase="false" />
      <conditions>
        <!--# 301 - - - http://www.test.com/faq.html?faq=13&layout=bob => http://www.test2.com/faqs.html-->
        <add input="{HTTP_HOST}" pattern="^www\.test\.com$" ignoreCase="false" />
        <add input="{QUERY_STRING}" pattern="(^|&amp;)faq\=13($|&amp;)" ignoreCase="false" />
        <add input="{QUERY_STRING}" pattern="(^|&amp;)layout\=bob($|&amp;)" ignoreCase="false" />
      </conditions>
      <action type="Redirect" redirectType="Permanent" url="http://www.test2.com/faqs.html?" appendQueryString="false" />
    </rule>
    <rule name="Импортированное правило 3" stopProcessing="true">
      <match url="^faq\.html$" ignoreCase="false" />
      <conditions>
        <!--# 301 - - - http://www.test3.com/faq.html?faq=13&layout=bob => bbq.html-->
        <add input="{QUERY_STRING}" pattern="(^|&amp;)faq\=13($|&amp;)" ignoreCase="false" />
        <add input="{QUERY_STRING}" pattern="(^|&amp;)layout\=bob($|&amp;)" ignoreCase="false" />
      </conditions>
      <action type="Redirect" redirectType="Permanent" url="/bbq.html?" appendQueryString="false" />
    </rule>
    <rule name="Импортированное правило 4" stopProcessing="true">
      <match url="^text/faq\.html$" ignoreCase="false" />
      <conditions>
        <!--# 301 - - - text/faq.html?faq=20 => helpdesk/kb.php-->
        <add input="{QUERY_STRING}" pattern="(^|&amp;)faq\=20($|&amp;)" ignoreCase="false" />
      </conditions>
      <action type="Redirect" redirectType="Permanent" url="/helpdesk/kb.php?" appendQueryString="false" />
    </rule>
  </rules>
</rewrite>
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Поддерживается импорт правил из файла .htaccess, указанных только в формате регулярных выражений. Правило вида "Redirect 301 /some-page/ /some-new-page/" не будет сконвертировано.
{% endhint %}

