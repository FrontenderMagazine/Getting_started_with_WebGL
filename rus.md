# Подготовка к использованию библиотеки WebGL

[WebGL][1] позволяет веб-контенту использовать API на основе [OpenGL ES 2.0][2] для выполнения 3D-визуализации в HTML-элементе [`canvas`][3] в браузерах, которые его поддерживают без помощи плагинов. Программы на WebGL состоят из кода последовательности операций, написанного на JavaScript, и кода для создания специальных эффектов (шейдеров), выполняемого графическим процессором компьютера. Элементы WebGL можно использовать вперемешку с другими элементами HTML и сочетать с другими частями страницы или её фона.

В этой статье предложено введение в основы применения WebGL. Она рассчитана на то, что вы уже имеете представление о математической составляющей создания 3D-графики и не ставит перед собой цель попытаться научить вас работать с OpenGL.

## Подготовка к визуализации в 3D

Для 3D-визуализации с помощью WebGL вам в первую очередь потребуется холст `canvas`. Фрагмент HTML-кода представленный ниже создаёт такой холст и настраивает обработчик события `onload`, которое инициализирует наш WebGL контекст.

    <body onload="start()">
      <canvas id="glcanvas" width="640" height="480">
        Ваш браузер не поддерживает элемент HTML5 <code>&lt;canvas&gt;</code>.
      </canvas>
    </body>

### Подготовка контекста WebGL

После загрузки документа вызывается JavaScript-функция `start()`. Её миссия состоит в определении WebGL контекста и начале визуализации контента. 

    var gl; // Глобальная переменная для WebGL контекста

    function start() {
      var canvas = document.getElementById("glcanvas");

      gl = initWebGL(canvas);      // Инициализация контекста GL
  
      // Следующий код выполняется только если WebGL поддерживается и работает
  
      if (gl) {
        gl.clearColor(0.0, 0.0, 0.0, 1.0);                      // Устанавливаем цветом для очистки холста чёрный непрозрачный
        gl.enable(gl.DEPTH_TEST);                               // Активация тестирования глубины
        gl.depthFunc(gl.LEQUAL);                                // Ближние элементы перекрывают дальние
        gl.clear(gl.COLOR_BUFFER_BIT|gl.DEPTH_BUFFER_BIT);      // Сброс цвета и буфера глубины.
      }
    }

Первым делом мы создаём привязку к холсту, помещая его в переменную с именем `canvas`. Безусловно, если у вас нет необходимости постоянно обращаться к холсту, следует избегать глобальных переменных и лучше сохранить его в локальной переменной или поле объекта.

Кагда у нас уже есть холст, вызываем функцию с именем `initWebGL();`, её мы создаём тут же. Задача этой функции состоит в инициализации WebGL контекста.

Если контекст инициализирован успешно, `gl` служит привязкой к нему. В этом случае мы изменяем цвет для очистки холста на чёрный и очищаем с его помощью контекст. После этого настраиваем контекст путем установки параметров. В этом случае мы активируем тестирование глубины и указываем что ближние объекты должны перекрывать дальние.

Для поверхностного ознакомления с кодом этого достаточно. Более подробно как собственно начать что-то делать мы разберёмся чуть позже.

### Создание WebGL контекста

Функция `initWebGL()` выглядит так:

    function initWebGL(canvas) {
      gl = null;
  
      try {
        // Попытка захвата стандартного контекста. При её неудаче используется резервный контекст.
        gl = canvas.getContext("webgl") || canvas.getContext("experimental-webgl");
      }
      catch(e) {}
  
      // Если контекста GL нет, прекращение работы с WebGL
      if (!gl) {
        alert("Невозможно инициализировать WebGL. Ваш браузер её не поддерживает.");
        gl = null;
      }
  
      return gl;
    }

Чтобы получить WebGL контекст для холста, мы запрашиваем у `canvas` контекст с именем `webgl`. Если такая попытка заканчивается неудачей, мы пробуем имя `experimental-webgl`. Если это также не приводит к удовлетворительному результату, выводим уведомление пользователю о том, что у него видимо не поддерживается WebGL. Вот и всё. На этом этапе `gl` либо имеет значение `null` (что подразумевает отсутствие WebGL контекста) или является ссылкой на WebGL контекст, в котором мы будем производить визуализацию.

> **Примечание:** имя контекста `experimental-webgl` используется временно пока идёт процесс подготовки спецификации; когда она будет завершена, будет использоваться имя `webgl`.

На этом этапе у вас есть код, которого достаточно для успешной инициализации WebGL контекста и у вас должно получиться большое пустое чёрное поле готовое к принятию контента.

[Кликните здесь чтобы увидеть живой пример][4] если вы используете браузер совместимый с WebGL.

### Изменение размера WebGL контекста

Разрешение области просмотра нового WebGL контекста будет подогнано под высоту и ширину его элемента `canvas`, без CSS, в момент получения контекста. Отредактировав стиль элемента `canvas` можно изменить его отображаемый размер, но нельзя изменить разрешение визуализации. Изменение атрибутов ширины и высоты элемента `canvas` после создания контекста также не повлияет на количество пикселей, которое будет отрисовано. Чтобы можно было изменять разрешение визуализации WebGL, например когда пользователь меняет размер окна выйдя из полноэкранного режима просмотра или же если вы хотите сделать возможной встроенную настройку параметров графики, нужно вызвать функцию `viewport()` контекста WebGL для подтверждения изменений.

Чтобы изменить разрешение визуализации контекста WebGL с переменными `gl` и `canvas` как в примере выше:

    gl.viewport(0, 0, canvas.width, canvas.height);

Размер холста, для которого визуализация выполняется с разрешением, не соответствующим прописанному в CSS, будет принудительно изменён соответственно. Изменение размера через CSS полезно когда нужно сэкономить ресурсы путём визуализации в низком разрешении и масштабировании в браузере; также возможно уменьшение масштаба, которое бы представило прекрасный пример эффекта сглаживания пусть даже и с примитивными результатами и высокой стоимостью производительности. Часто лучше всего полагаться на множественную выборку сглаживания (MSAA) и реализацию фильтрации текстур в браузере пользователя если они предусмотрены и соответствуют вашим требованиям.

### Читайте также

* [Введение в WebGL][5] - Написано Лусом Кабальеро (Luz Caballero) на DEV.OPERA. Эта статья рассказывает о том что собой представляет WebGL и как он работает, о концепции потоковой визуализации и некоторых библиотеках WebGL.
* [Введение в современный OpenGL][6] - комплект интересных статей об OpenGL от Джо Гроффа (Joe Groff). Джо представляет лёгкий для чтения экскурс в OpenGL начиная от его истории и заканчивая важной концепцией конвейера трёхмерной графики, а также приводит примеры работы OpenGL. Если вы не знакомы с OpenGL, начать ознакомление с ним стоит с этой статьи.

[1]: http://www.khronos.org/webgl/
[2]: http://www.khronos.org/opengles/
[3]: https://developer.mozilla.org/en-US/docs/HTML/Canvas
[4]: https://developer.mozilla.org/samples/webgl/sample1/index.html
[5]: http://dev.opera.com/articles/view/an-introduction-to-webgl/
[6]: http://duriansoftware.com/joe/An-intro-to-modern-OpenGL.-Table-of-Contents.html