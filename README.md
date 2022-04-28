<div class="tm-misprint-area">
    <div class="tm-misprint-area__wrapper">
        <article class="tm-article-presenter__content tm-article-presenter__content_narrow">
            <div class="tm-article-presenter__header">
                <div class="tm-article-snippet tm-article-presenter__snippet">
                    <h2><span>via <a href="https://habr.com/ru/post/213397/">https://habr.com/ru/post/213397/</a></span></h2>
                    <h1 lang="ru" class="tm-article-snippet__title tm-article-snippet__title_h1"><span>Как я перехватывал трафик покер рума или «Пишем свой MitM SSL прокси на C#»</span></h1>
                </div>
            </div>
            <!---->
            <div data-gallery-root="" class="tm-article-body" lang="ru">
                <div></div>
                <div id="post-content-body">
                    <div>
                        <div class="article-formatted-body article-formatted-body article-formatted-body_version-1">
                            <div xmlns="http://www.w3.org/1999/xhtml">Однажды у меня появилась навязчивая идея: посмотреть, а что же там такого покерный клиент отправляет на сервер. Как Вы понимаете, крупные покерные румы используют SSL для передачи данных. Протоколы, основанные на асимметричном шифровании, подвержены только одному известному мне виду атак — MitM (Man in the middle — человек посередине). <br>
                                <br>
                                Помаявшись с тонной софта, предназначенного для реализации MitM на SSL соединение, я пришел к выводу, что руки растут не из того места либо у разработчиков данных инструментов, либо у меня. Но идея была жутко навязчивая, и было принято решение сделать всё вручную. Если интересно, что же из всего этого вышло, прошу под кат.<br>
                                <img src="https://habrastorage.org/r/w1560/getpro/habr/post_images/da9/cde/e8d/da9cdee8d285cc304fdbbe10ac76e66f.jpg" data-src="https://habrastorage.org/getpro/habr/post_images/da9/cde/e8d/da9cdee8d285cc304fdbbe10ac76e66f.jpg" alt=""><br>
                                Данная статья просвещена написанию простого инструмента для реализации MitM-атаки. Те, кто не знакомы с тем, что такое MitM, могут почитать об этом <a href="http://ru.wikipedia.org/wiki/%D0%A7%D0%B5%D0%BB%D0%BE%D0%B2%D0%B5%D0%BA_%D0%BF%D0%BE%D1%81%D0%B5%D1%80%D0%B5%D0%B4%D0%B8%D0%BD%D0%B5">тут</a>.<br>
                                <br>
                                <h3>Цель</h3><br>
                                В качестве подопытного был выбран клиент <a href="http://www.wincake.com/">Cake Poker</a> по причине моего длительного знакомства с ним. Началось все с того, что я просто запустил покерный клиент и полез в старый добрый менеджер ресурсов, в котором нашел с десяток подключений от него.<br>
                                <img src="https://habrastorage.org/r/w1560/getpro/habr/post_images/27c/6cf/e82/27c6cfe82800e7b7f41695035622a259.png" data-src="https://habrastorage.org/getpro/habr/post_images/27c/6cf/e82/27c6cfe82800e7b7f41695035622a259.png" alt=""><br>
                                Поэкспериментировав, я смог выделить соединение, которое держится всегда. Его я и взял на прицел для проведение MitM атаки.<br>
                                <img src="https://habrastorage.org/r/w1560/getpro/habr/post_images/8fa/109/b0a/8fa109b0a7fb511ab5d6de4c686d1d10.png" data-src="https://habrastorage.org/getpro/habr/post_images/8fa/109/b0a/8fa109b0a7fb511ab5d6de4c686d1d10.png" alt=""><br>
                                Источник соединения известен, конечная цель известна — lb6.playdata.co.uk. <br>
                                <br>
                                <h3>Перенаправление трафика</h3><br>
                                Теперь необходимо было вклиниться между клиентом и сервером. Нечего особо хитрого изобретать не стал, ибо незачем — просто добавил в host доменные имена, соотнесённые с 127.0.0.1. Не сложно догадаться, что если есть lb6.playdata.co.uk, то есть и lb1.playdata.co.uk, и lb8.playdata.co.uk. С ними поступил аналогично, занеся в host, т.к. конечные инстансы, как я понял, выбираются по расположению звёзд. При запуске покерного клиента он повисает в ожидании подключения. Замечательно. Это означает что трафик был перенаправлен на нашу машинку. Идём дальше.<br>
                                <img src="https://habrastorage.org/r/w1560/getpro/habr/post_images/e9a/6b0/3df/e9a6b03dfeca220fb12c90c07cc424ae.png" data-src="https://habrastorage.org/getpro/habr/post_images/e9a/6b0/3df/e9a6b03dfeca220fb12c90c07cc424ae.png" alt=""><br>
                                <br>
                                <h3>Прокси</h3><br>
                                Следующей задачей было написание прокси на C#. Да-да, простой прокси, для заготовки будущей программы. Чтобы не заниматься изобретением велосипеда, я по-быстрому нагуглил подходящее для меня решение: <a href="http://loosexaml.wordpress.com/2011/04/27/tcp-proxy-in-c-using-tasks-parallel-library/">TCP Proxy in C# using Task Parallel Library</a>.<br>
                                Немного его отрефакторил (ненавижу, когда много вложенности), захардкодил конечную точку подключения и запустил. Запускаю покерного клиента — всё работает. В диспетчере ресурсов можем видеть, что трафик от покерного клиента идёт к моему прокси, а от него на сервер и обратно. <br>
                                <br>
                                <h3>MitM на SSL</h3><br>
                                Далее нам предстоит реализовать MitM-атаку на SSL. Разделим её на два этапа: первый — соединение клиента с прокси; второй — прокси с сервером.<br>
                                Для реализации первого этапа, когда клиент подключится к прокси, мы не будем отправлять пришедшие данные дальше, а начнём, так называемую, процедуру рукопожатия. На C# это можно сделать с помощью экземпляра класса SslStream, построенного поверх уже созданного NetworkStream. В момент создания передаётся информация о протоколе и прочая специфическая информация. <br>
                                После этого, передаём клиенту свой сертификат. Это делается с помощью метода AuthenticateAsServer класса SslStream, куда мы должны передать путь до файла сертификата. <br>
                                <br>
                                Файл x509 сертификата был сгенерирована с помощью утилиты Makecert, которую можно вызвать из девелоперской консоли Visual Studio. Пришлось немного помучиться с параметрами, но всё получилось. Вот неплохое описание того, как ей можно пользоваться: <a href="http://ishare2learn.wordpress.com/tag/ssl/">SSL communication in C#</a>. В качестве имени укажем *.playdata.co.uk. Это имя покрывает все домены, которые используются покерным клиентом. <br>
                                <pre>
                                    <code class="hljs cs">
<span>makecert -n CN=MyCA -cy authority -a sha1 -sv “MyCA.pvk” -r “MyCA.cer” <span class="hljs-comment">//Создаём сертификат ЦА</span></span>
<span>certmgr -<span class="hljs-keyword">add</span> -all -c “MyCA.cer” -s -r LocalMachine Root <span class="hljs-comment">//Добавляем в довереные коневые центры сертификаты</span></span>
<span>makecert -n CN=*.playdata.co.uk -ic MyCA.cer -iv MyCA.pvk -a sha1 -sky exchange -pe -sr currentuser -ss my SslServer.cer <span class="hljs-comment">//Создаём серверный сертификат</span></span>
                                    </code>
                                </pre>
                                <br>
                                Мы сгенерировали серверный закрытый ключ и ключ УЦ (удостоверительного центра), которым подписан наш ключ. Ключ УЦ помещаем в «Доверенные корневые центры сертификации», и вуаля! Если посмотреть на наш серверный ключик средствами просмотра ключей Windows, то мы увидим, что система считает его валидным, как и покерный клиент, который запущен в нашей системе. <br>
                                <img src="https://habrastorage.org/r/w1560/getpro/habr/post_images/c71/50b/1a3/c7150b1a3f7126a8446a34fc8419eb9a.png" data-src="https://habrastorage.org/getpro/habr/post_images/c71/50b/1a3/c7150b1a3f7126a8446a34fc8419eb9a.png" alt=""><br>
                                <br>
                                Путь до полученого сертификата мы передаём в метод AuthenticateAsServer. Если всё прошло хорошо, то у нас получится SSL соединение от клиента до прокси, в которое клиент будет посылать данные. Теперь необходимо давать клиенту адекватные ответы на его запросы. Для этого нам потребуется реализовать второй этап MitM-атаки, а именно построить SSL соединение от прокси до сервера. Так же строим SslStream поверх NetworkStream до сервера и авторизовываемся с помощью метода AuthenticateAsClient. Данные, приходящие из SSL соединения клиента и сервера отправляем друг другу. <br>
                                <div><b class="spoiler_title">Процес рукопожатия C#</b>
                                        <pre>
                                            <code class="hljs cs">
<span>var certificate = <span class="hljs-built_in">new</span> X509Certificate("SslServer.cer", "123");</span>
<span>var clientStream = <span class="hljs-built_in">new</span> SslStream(client.GetStream(), <span class="hljs-keyword">false</span>);</span>
<span>clientStream.AuthenticateAsServer(certificate, <span class="hljs-keyword">false</span>, <span class="hljs-keyword">System</span>.<span class="hljs-keyword">Security</span>.Authentication.SslProtocols.<span class="hljs-keyword">Default</span>, <span class="hljs-keyword">false</span>);</span>
<span>var <span class="hljs-keyword">server</span> = <span class="hljs-built_in">new</span> TcpClient("200.26.205.63", <span class="hljs-number">4520</span>);</span>
<span>var serverSslStream = <span class="hljs-built_in">new</span> SslStream(<span class="hljs-keyword">server</span>.GetStream(), <span class="hljs-keyword">false</span>, SslValidationCallback, <span class="hljs-keyword">null</span>);</span>
<span>serverSslStream.AuthenticateAsClient("lb3.playdata.co.uk");</span>
                                            </code>
                                        </pre>
                                        <br>
                                </div><br>
                                Через некоторое время работы можно будет заметить в менеджере ресурсов расхождение количества отправленных байт на прокси и от него. Это обусловлено тем, что при шифровании разные ключи дают разный по размеру результат.<br>
                                <img src="https://habrastorage.org/r/w1560/getpro/habr/post_images/2e0/30e/36d/2e030e36d580c262c586320f747acbbe.png" data-src="https://habrastorage.org/getpro/habr/post_images/2e0/30e/36d/2e030e36d580c262c586320f747acbbe.png" alt=""><br>
                                <h3>Заключение</h3><br>
                                Что же дальше? А дальше мы добавим код для сохранения данных в текстовый файл, чтобы их можно было проанализировать то, что передаёт покерный клиент серверу. Собственно, всё, MitM Proxy написан.<br>
                                <br>
                                Осталось добавить в него немного блекджека. Например, для разбора идущего через него трафика, выдирания карт пользователя и отправки нам и т. д… <br>
                                Я написал разборщик трафика на лету, что бы можно было удобно мониторить, что отправляет клиент, и соотносить это с моими действиями.<br>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </article>
    </div>
    <!---->
</div>

<!--suppress CssUnknownTarget -->
<style>
    @import url("https://assets.habr.com/habr-web/css/app.022a37bf.css");
</style>
<script src="https://assets.habr.com/habr-web/js/chunk-vendors.3720226a.js"></script>