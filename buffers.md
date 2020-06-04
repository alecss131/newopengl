# Буферы

В данной статье рассмотрены нововведения в буферах. Я рассмотрю самые частоиспользуемые буферы (GL_ARRAY_BUFFER, GL_ELEMENT_ARRAY_BUFFER, GL_UNIFORM_BUFFER, GL_SHADER_STORAGE_BUFFER, GL_RENDERBUFFER, GL_FRAMEBUFFER). Остальные типы буферов (например GL_COPY_READ_BUFFER, GL_COPY_WRITE_BUFFER, GL_PIXEL_UNPACK_BUFFER, GL_PIXEL_PACK_BUFFER, GL_QUERY_BUFFER, GL_TEXTURE_BUFFER, GL_TRANSFORM_FEEDBACK_BUFFER, GL_DRAW_INDIRECT_BUFFER, GL_ATOMIC_COUNTER_BUFFER, GL_DISPATCH_INDIRECT_BUFFER) рассматривать не буду, так как не имел с ними опыта. Принцип изменений у них у всех схожий.

## Вершинные буферы

Исходные общие данные для всех примеров

```cpp
    GLfloat vertices[] = {
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f,
     0.0f,  0.5f, 0.0f
    };
    GLint indices[] = {0, 1, 2};
    GLuint VAO;
    GLuint VBO;
    GLuint VBE;
```

Для простоты возьмем для отрисовки треугольник, заданные массивом vertices  и идексами indices (если нужны будут). Поскольку мы хотим отрисовать один треугольник мы должны предоставить 3 вершины, каждая из которых находится в трехмерном пространстве. Мы определим их в нормализованном виде в GLfloat массиве. Не буду подробно расписывать что и как, VAO объект вершинного массива, VBO объект вершинного буфера для вершин, а VBE объект вершинного буфера для индексов, называемый также элементный (индексный) буфер.

Стандартное создание буферов в opengl версии 3.3 будет выглядеть так. Для начала без индексов.

```cpp
    glGenVertexArrays(1, &VAO);
    glBindVertexArray(VAO);
    glGenBuffers(1, &VBO);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    //glBufferSubData(GL_ARRAY_BUFFER, 0, sizeof(vertices), vertices); //buffer subdata
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0);
    glEnableVertexAttribArray(0);
    //glVertexAttribDivisor(0, 1); //instancing
    glBindVertexArray(0);
```

Сначла создаем и привязываем VAO, потом воздаем, привязывем и заполняем VBO, связываем и включаем вершинные атрибуты используя `glVertexAttribPointer` и `glEnableVertexAttribArray` и в конце отвязываем VAO. Все стандартно. С индексами код не сильно изменится, добавится еще один буфер. Так же привел примеры частичного заполнения буфера через `glBufferSubData` и задания атрибутов для инстансинга используя `glVertexAttribDivisor`.

Вариант с индексами:

```cpp
    glGenVertexArrays(1, &VAO);
    glBindVertexArray(VAO);
    glGenBuffers(1, &VBO);
    glGenBuffers(1, &VBE);
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, VBE);
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(GLfloat), (GLvoid*)0);
    glEnableVertexAttribArray(0);
    glBindVertexArray(0);
```
У функции `glVertexAttribPointer` имеется немного параметров, давайте быстро пробежимся по ним:

- Первый аргумент описывает какой аргумент шейдера мы хотим настроить. Мы хотим специфицировать значение аргумента position, позиция которого была указана следующим образом: layout (location = 0).
- Следующий аргумент описывает размер аргумента в шейдере. Поскольку мы использовали vec3 то мы указываем 3.
- Третий аргумент описывает используемый тип данных. Мы указываем GL_FLOAT, поскольку vec в шейдере использует числа с плавающей точкой.
- Четвертый аргумент указывает необходимость нормализовать входные данные. Если мы укажем GL_TRUE, то все данные будут расположены между 0 (-1 для знаковых значений) и 1. Нам нормализация не требуется, поэтому мы оставляем GL_FALSE;
- Пятый аргумент называется шагом и описывает расстояние между наборами данных.
- Последний параметр имеет тип GLvoid* и поэтому требует такое странное приведение типов. Это смещение начала данных в буфере. У нас буфер не имеет смещения и поэтому мы указываем 0.

Отрисовка

```cpp
    //отрисовка без индексов
    glBindVertexArray(VAO);
    glDrawArrays(GL_TRIANGLES, 0, 3);
    //glDrawArraysInstanced(GL_TRIANGLES, 0, 3, 2); //instanced 2 trinangles
    glBindVertexArray(0);
    //отрисовка с индексами
    glBindVertexArray(VAO);
    glDrawElements(GL_TRIANGLES, 3, GL_UNSIGNED_INT, 0);
    //glDrawElementsInstanced(GL_TRIANGLES, 3, GL_UNSIGNED_INT, 0, 2); //instanced 2 triangles
    glBindVertexArray(0);
```

Удаление буферов

```cpp
    glDeleteBuffers(1, &VBE);
    glDeleteBuffers(1, &VBO);
    glDeleteVertexArrays(1, &VAO);
```

### OpenGL 4.3

В версии 4.3 появились новые функции для работы с буферами, которые в последствии обновились в 4.5 версии под DSA. Лично я не вижу особого смысла в использовании данного нововведения, так как по сути ничего нового не добавилось, а количество необходимого кода увеличилось. Рассматриваю их только потому что они легли в основу следующего обновления и потому надо уметь ими пользоваться. Начну с кода. Версия без индексов.

```cpp
    glGenBuffers(1, &VBO);
    glGenVertexArrays(1, &VAO);
    glBindVertexArray(VAO);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    glEnableVertexAttribArray(0);
    glVertexAttribFormat(0, 3, GL_FLOAT, false, 0);
    glBindVertexBuffer(0, VBO, 0, 3 * sizeof(GLfloat));
    glVertexAttribBinding(0, 0);
    //glVertexBindingDivisor(0, 1); //instancing
    glBindVertexArray(0);
```

Версия с индексами.

```cpp
    glGenBuffers(1, &VBO);
    glGenBuffers(1, &VBE);
    glGenVertexArrays(1, &VAO);
    glBindVertexArray(VAO);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, VBE);
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
    glEnableVertexAttribArray(0);
    glVertexAttribFormat(0, 3, GL_FLOAT, false, 0);
    glBindVertexBuffer(0, VBO, 0, 3 * sizeof(GLfloat));
    glVertexAttribBinding(0, 0);
    glBindVertexArray(0);
```

Теперь разберем код, как можно увидеть поменялись функции связанные с атрибутами вершин. Перечислю замены: `glVertexAttribFormat`, `glBindVertexBuffer` и `glVertexAttribBinding` вместо `glVertexAttribPointer`; `glVertexBindingDivisor` вместо `glVertexAttribDivisor`.

Теперь описания новых функций и их параметров.

```cpp
void glVertexAttribFormat(GLuint attribindex, GLint size, GLenum type, GLboolean normalized, GLuint relativeoffset)
```

по сути является аналогом `glVertexAttribPointer`, только создает атрибты не привязанные к буферу

- attribindex описываемый индекс атрибутов вершин
- size количество значений на вершину, которые хранятся в массиве. Один из: 1 2 3 4 BGRA
- type тип данных, хранящихся в массиве
- normalized, если true, тогда целочисленные данные нормализуются в диапазоне [-1, 1] или [0, 1], если они подписаны или не подписаны, соответственно. Если false, то целочисленные данные напрямую преобразуются в число с плавающей запятой.
- relativeoffset относительное смещение, измеренного в базовых машинных единицах первого элемента относительно начала привязки буфера вершин, из которого этот атрибут выбирается 

```cpp
void glBindVertexBuffer(GLuint bindingindex, GLuint buffer, GLintptr offset, GLsizei stride)
```

Связывает буфер с точкой привязки вершинного буфера.

- bindingindex индекс точки привязки вершинного буфера, к которой привязывается буфер 
- buffer имя существующего буфера для привязки к точке привязки вершинного буфера
- offset смещения первого элемента буфера
- stride расстояние между элементами в буфере

```cpp
void glVertexAttribBinding(GLuint attribindex, GLuint bindingindex)
```

Связывает атрибуты вершин и привязку вершинного буфера. То есть связывает attribindex и bindingindex между собой из функций `glVertexArrayAttribFormat` и `glVertexArrayVertexBuffer` соответственно.

- attribindex индекс атрибута для привязки к привязке буфера вершин
- bindingindex индекс привязки буфера вершин, с которым связывается общий атрибут вершины

```cpp
void glVertexBindingDivisor(GLuint bindingindex, GLuint divisor)
```

Изменяет скорость, с которой общие атрибуты вершин продвигаются во время рендеринга экземпляров. Например если divisor равен: 0 - каждую вершину, 1 - каждый экземпляр.

- bindingindex индекс общего атрибута вершины
- divisor количество экземпляров, которые будут проходить между обновлениями атрибута в индексе слота

Изменений в отрисовке и удалении буферов нету.

### OpenGL 4.4 (Неизменяемые буферы)

В версии 4.4 добавили неизменяемые (immutable) буферы. Новая функция `glBufferStorage` подобная уже знакомой `glBufferData`. Использование почти такое же как и у `glBufferData` за одним исключением, используются другие флаги, использование старых (напимер `GL_STATIC_DRAW`) будет приводить к ошибкам. По сути для простого рисования достаточно одного флага `GL_DYNAMIC_STORAGE_BIT` вместо прошлых трех (так как понимаю что больше нету разделения на STREAM, STATIC и DYNAMIC и это выбирается автоматически).

### OpenGL 4.5 (DSA)

В opengl версии 4.5 появилась такая вещь как Direct State Access (DSA) — прямой доступ к состоянию. Средство изменения объектов OpenGL без необходимости привязывать их к контексту. Это позволяет изменять состояние объекта в локальном контексте, не затрагивая глобальное состояние, разделяемое всеми частями приложения. Это также делает API-интерфейс немного более объектно-ориентированным, поскольку функции, которые изменяют состояние объектов, могут быть четко определены. 

Давайте перепишем примеры выше используя DSA. Начнем с безиндексного варианта.

```cpp
    glCreateVertexArrays(1, &VAO);
    glCreateBuffers(1, &VBO);
    //glNamedBufferData(VBO, sizeof(vertices), vertices, GL_STATIC_DRAW); //mutable buffer 
    glNamedBufferStorage(VBO, sizeof(vertices), vertices, GL_DYNAMIC_STORAGE_BIT); //immutable buffer
    //glNamedBufferSubData(VBO, 0, sizeof(vertices), vertices); //buffer subdata
    glEnableVertexArrayAttrib(VAO, 0);
    glVertexArrayAttribFormat(VAO, 0, 3, GL_FLOAT, false, 0);
    glVertexArrayVertexBuffer(VAO, 0, VBO, 0, 3 * sizeof(GLfloat));
    glVertexArrayAttribBinding(VAO, 0, 0);
    //glVertexArrayBindingDivisor(VAO, 0, 1); //instancing
```

Теперь разберем код. Перечислю замены: `glCreateBuffers` вместо `glGenBuffers` + `glBindBuffer`; `glCreateVertexArrays` вместо `glGenVertexArrays` + `glBindVertexArray`; `glNamedBufferData` заменяет `glBufferData`; `glNamedBufferStorage` заменяет `glBufferStorage`; `glNamedBufferSubData` заменяет `glBufferSubData`; `glEnableVertexArrayAttrib` вместо `glEnableVertexAttribArray`; `glVertexArrayVertexBuffer` вместо `glBindVertexBuffer`; `glVertexArrayAttribBinding` заменяет `glVertexAttribBinding` и `glVertexArrayBindingDivisor` вместо `glVertexBindingDivisor`. Использование функций для создания буферов не изменилось, а у других функций появился новый параметр, первый по счету, отвечающий за объект на который он влияет.

Теперь пример с использованием индексов.

```cpp
    glCreateVertexArrays(1, &VAO);
    glCreateBuffers(1, &VBO);
    glCreateBuffers(1, &VBE);
    glNamedBufferStorage(VBO, sizeof(vertices), vertices, GL_DYNAMIC_STORAGE_BIT);
    glNamedBufferStorage(VBE, sizeof(indices), indices, GL_DYNAMIC_STORAGE_BIT);
    glEnableVertexArrayAttrib(VAO, 0);
    glVertexArrayAttribFormat(VAO, 0, 3, GL_FLOAT, false, 0);
    glVertexArrayVertexBuffer(VAO, 0, VBO, 0, 3 * sizeof(GLfloat));
    glVertexArrayAttribBinding(VAO, 0, 0);
    glVertexArrayElementBuffer(VAO, VBE);
```

Все тоже самое, только добавился еще один буфер, создаваемый так же как и VBO, только привязываемый вручную к VAO функцией `glVertexArrayElementBuffer` и не требующий настройки парметров.

```cpp
void glVertexArrayElementBuffer(GLuint vaobj, GLuint buffer)
```

Связывает объект буфера массива элементов с точкой привязки объекта массива вершин.

- vaobj имя объекта массива вершин
- buffer имя объекта буфера индексов. Если buffer равен нулю, любая существующая привязка буфера массива элементов к vaobj удаляется.

Так же в новой версии появилась возможность хранить индексы и вершины в одном буфере.

```cpp
    glCreateVertexArrays(1, &VAO);
    glCreateBuffers(1, &VBO);
    glNamedBufferStorage(VBO, sizeof(indices) + sizeof(vertices), NULL, GL_DYNAMIC_STORAGE_BIT);
    glNamedBufferSubData(VBO, 0, sizeof(indices), indices);
    glNamedBufferSubData(VBO, sizeof(indices), sizeof(vertices), vertices);
    glEnableVertexArrayAttrib(VAO, 0);
    glVertexArrayAttribFormat(VAO, 0, 3, GL_FLOAT, false, 0);
    glVertexArrayVertexBuffer(VAO, 0, VBO, sizeof(indices), 3 * sizeof(GLfloat));
    glVertexArrayAttribBinding(VAO, 0, 0);
    glVertexArrayElementBuffer(VAO, VBO);
```

Индексы должны располагаться перед данными вершин а в настройке данных вершин в функции glVertexArrayVertexBuffer надо указать 4 параметром смещение данных вершин относительно начала буфера. В данном коде сначала выделяется буфер а потом заполняется сначала данными индексом с начала буфера а потом данными вершин через `glNamedBufferSubData`.

Изменений в отрисовке и удалении буферов нету.

## Кадровый и рендер буферы

Начнем с рендер буфера, так как его можно присоединить к кадровому и для красткости опустим создание текстур (примем что они уже созданы), так как по текстурам будет отдельная статья. Про текстуру добавлю что надо будет создать пустую, передав NULL вместо данных.

Начну как обычно с общих данных

```cpp
    GLuint fbo;
    GLuint rbo;
    GLuint texture;
    int width = 800, height = 600;
```

Здесь у нас указатели на рендер буфер, кадровый буфер и текстуру, а так же размеры для буфера в пикселях. Для примера создадим рендер буфер для глубины и трафарета. Рендер буферы быстры на запись но недоступны в общем случае для чтения.

```cpp
    glGenRenderbuffers(1, &rbo);
    glBindRenderbuffer(GL_RENDERBUFFER, rbo);
    glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, width, height);
    glBindRenderbuffer(GL_RENDERBUFFER, 0);
```

Как видно, принцип создания буфера не сильно отличается от других буфером, главное отличие в функции `glRenderbufferStorage` где мы указываем внутренний формат буфера и его размеры в пикселях вместо выделения памяти. Теперь приступим к созданию кадрового буфера, в котором для буфера цвета будем использовать текстуру.

```cpp
    glGenFramebuffers(1, &fbo);
    glBindFramebuffer(GL_FRAMEBUFFER, fbo);
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texture, 0);
    glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo);
    if (glCheckFramebufferStatus(GL_FRAMEBUFFER) != GL_FRAMEBUFFER_COMPLETE)
        std::cout << "ERROR::FRAMEBUFFER:: Framebuffer is not complete!" << std::endl;
    glBindFramebuffer(GL_FRAMEBUFFER, 0);
```

Налицо уже знакомый и десятки раз примененный нами подход к созданию и использованию объектов библиотеки OpenGL: создаем объект кадрового буфера, привязываем как текущий активный буфер кадра, выполняем необходимые операции и отвязываем кадровый буфер. Перед отвязыванием проверяем кадровый буфер на завершенность. Не буду вдаваться в теорию, функцией `glFramebufferTexture2D` мы привязываем текстуру к буферу в качестве цвета а с помощью `glFramebufferRenderbuffer` рендер буфер в качестве буфера для глубины и трафарета.

И пример использования 

```cpp
    glBindFramebuffer(GL_FRAMEBUFFER, fbo);
    //все операции рисования будут рисовать в привязанный буфер
    glBindFramebuffer(GL_FRAMEBUFFER, 0); //0 это буфер экрана
```

Для поддержки мультисэмплинга (msaa) надо создать мультисэмпл текстуру и рендер буфер с мультисэмплом, изменения будут такие

в создании рендер буфера `glRenderbufferStorageMultisample` вместо `glRenderbufferStorage`

```cpp
    glRenderbufferStorageMultisample(GL_RENDERBUFFER, 4, GL_DEPTH24_STENCIL8, width, height); 
```

и другой тип текстуры в создании кадрового буфера, например `GL_TEXTURE_2D_MULTISAMPLE` вместо `GL_TEXTURE_2D`

```cpp
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D_MULTISAMPLE, texture, 0);
```

Удаление буферов

```cpp
    glDeleteTextures(1, &texture);
    glDeleteRenderbuffers(1, &rbo);
    glDeleteFramebuffers(1, &fbo);
```

### OpenGL 4.5 (DSA)

Принцип DSA распространяется и на эти буферы. Начну с примера.

```cpp
    //создание рендер буфера
    glCreateRenderbuffers(1, &rbo);
    glNamedRenderbufferStorage(rbo, GL_DEPTH24_STENCIL8, width, height);
    // создание кадрового буфера
    glCreateFramebuffers(1, &fbo);
    glNamedFramebufferTexture(fbo, GL_COLOR_ATTACHMENT0, texture, 0);
    glNamedFramebufferRenderbuffer(fbo, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo);
    if (glCheckFramebufferStatus(GL_FRAMEBUFFER) != GL_FRAMEBUFFER_COMPLETE)
        std::cout << "ERROR::FRAMEBUFFER:: Framebuffer is not complete!" << std::endl;
```

Замены как обычно: `glCreateRenderbuffers` вместо `glGenRenderbuffers` + `glBindRenderbuffer`, `glCreateFramebuffers` вместо `glGenFramebuffers` + `glBindFramebuffer`; `glNamedFramebufferTexture` вместо `glFramebufferTexture2D`; `glNamedFramebufferRenderbuffer` вместо `glFramebufferRenderbuffer`; `glNamedRenderbufferStorage` вместо  `glRenderbufferStorage`. В привязывании текстуры пропала необходимость указывать тип текстуры.

В случае с мультисэплом меняется всего одна функция (исключая так же создание нужной текстуры), в отличии от простого кадрового буфера, `glNamedRenderbufferStorageMultisample` вместо `glRenderbufferStorageMultisample`. Весь процесс создания будет выглядеть так

```cpp
    //создание рендер буфера
    glCreateRenderbuffers(1, &rbo);
    glNamedRenderbufferStorageMultisample(GL_RENDERBUFFER, 4, GL_DEPTH24_STENCIL8, width, height);
    //создание кадрового буфера
    glCreateFramebuffers(1, &fbo);
    glNamedFramebufferTexture(fbo, GL_COLOR_ATTACHMENT0, texture, 0);
    glNamedFramebufferRenderbuffer(fbo, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo);
    if (glCheckFramebufferStatus(GL_FRAMEBUFFER) != GL_FRAMEBUFFER_COMPLETE)
        std::cout << "ERROR::FRAMEBUFFER:: Framebuffer is not complete!" << std::endl;
```

Использование и удаление буферов не изменилось.

## Юниформ буфер (ubo)

Исходные общие данные для всех примеров.

> Хотя в более новых версиях появилась и новая разметка std430, но для юниформ буферов ее использовать нельзя, только std140. Возможность использовать std430 для юниформ буфера есть только в glsl скомпилированном под Vulkan, при использовании в opengl шейдерная программа не будет компилироваться.

```cpp
    GLuint uboExampleBlock;
    GLuint shaderId = 0;
    const char* name = "Name";
    int b = true;
```

Здесь у нас указатели на юниформ буфер, шедерную программу, имя буфера в шейдере и пример данных для записи (int так как в std140 bool занимает 4 байта)

```cpp
    glGenBuffers(1, &uboExampleBlock);
    glBindBuffer(GL_UNIFORM_BUFFER, uboExampleBlock);
    glBufferData(GL_UNIFORM_BUFFER, 150, NULL, GL_STATIC_DRAW); // выделяем 150 байт памяти
    glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

Здесь мы создаем буфер,создание ничем не отличается от создания вершинного буфера кроме типа `GL_UNIFORM_BUFFER`. Для примера выделим блок на 150 байт, ничего не мешает сразу заполнить его данными указав данные вместо NULL в функции `glBufferData`. Далее мы привязываем юниформ блоки в шейдерах к точками привязки (надо сделать для каждой шейдерной программы). Сначала мы получаем индекс юниформ блока в шейдере с помощью функции `glGetUniformBlockIndex` и потом этот индекс связываем с точкой привязки, например 0, с помощью функции `glUniformBlockBinding`.

```cpp
    GLuint lights_index = glGetUniformBlockIndex(shaderId, name);
    glUniformBlockBinding(shaderId, lights_index, 0);
```

После этого нам надо привязать сам буфер к той же точки привязки (в нашем случае 0), целиком (`glBindBufferBase`) или частично (`glBindBufferRange`) указав начало и размер памяти из буфера (в примере мы привязываем буфер целиком но двумя разными способами, два последних параметра это смещение относительно начала буфера 0 и размер буфера 150)

```cpp
    glBindBufferBase(GL_UNIFORM_BUFFER, 0, uboExampleBlock);
    //glBindBufferRange(GL_UNIFORM_BUFFER, 0, uboExampleBlock, 0, 150);
```

Теперь осталось заполнить буфер данными, тоже ничего нового, все тоже самое как заполнение части вершинного буфера.

```cpp
    glBindBuffer(GL_UNIFORM_BUFFER, uboExampleBlock);
    glBufferSubData(GL_UNIFORM_BUFFER, 144, 4, &b);
    glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

Удаляется буфер так же как и вершинный

```cpp
    glDeleteBuffers(1, &uboExampleBlock);
```

### OpenGL 4.2

В версии 4.2 появилась возможность в шейдерах явно указать точку привязки юниформ буфера, что избавляет нас от использования функций `glGetUniformBlockIndex` и `glUniformBlockBinding`. Пример в шейдере:

```glsl
layout(binding = 0, std140) uniform Name { ... };
```

Весь код из предыдущего примера немного сократится.

```cpp
    glGenBuffers(1, &uboExampleBlock);
    glBindBuffer(GL_UNIFORM_BUFFER, uboExampleBlock);
    glBufferData(GL_UNIFORM_BUFFER, 150, NULL, GL_STATIC_DRAW);
    glBindBuffer(GL_UNIFORM_BUFFER, 0);
    //тут исчез блок из пар функций для каждой шейдерной программы
    glBindBufferBase(GL_UNIFORM_BUFFER, 0, uboExampleBlock);
    //подразумеваю что заполнять будете несколько позже его создания и привязки
    glBindBuffer(GL_UNIFORM_BUFFER, uboExampleBlock);
    glBufferSubData(GL_UNIFORM_BUFFER, 144, 4, &b);
    glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

Изменений в удалении нету.

### OpenGL 4.3 и 4.5 (immutable buffers и DSA)

С версии 4.3 так же как и для вершин можно использовать неизменяемые буферы, а использование DSA еще сокращает код для создания юниформ буферов, замена `glCreateBuffers` вместо `glGenBuffers` + `glBindBuffer`. Для выделения памяти можно использовать `glNamedBufferStorage` и `glNamedBufferData` вместо `glBufferStorage` и `glBufferData` а для заполнения `glNamedBufferSubData` вместо `glBufferSubData` (никто не мешает заполнить буфер при выделении ему памяти указав вместо NULL данные).

```cpp
    glCreateBuffers(1, &uboExampleBlock);
    glNamedBufferStorage(uboExampleBlock, 150, NULL, GL_DYNAMIC_STORAGE_BIT);
    glBindBufferBase(GL_UNIFORM_BUFFER, 0, uboExampleBlock);
    glNamedBufferSubData(uboExampleBlock, 144, 4, &b);
```

Изменений в удалении нету.

## Буфер хранилища шейдеров (ssbo)

### OpenGL 4.3

В версии 4.3 появился новый тип буферов Shader Storage Buffer, ессли дословно переводить, то буфер хранилища шейдеров, то есть теперь шейдеры могут не только читать из буферов (подобно юниформ буферам), а так же и писать в них (разумеется не стоит забывать про асинхронность выполнения шейдеров), для этих целей и создали этот тип буферов. Помимо неизменяемых буферов появилась и новая, более лучшая, разметка std430 для буферов в замен старой std140. Замечу что шейдеры писать могут только в ssbo, а так же новая разметка применима только к ним (в контексте opengl, так как в вулкане можно использовать std430 с ubo в glsl). Отличий в создании от юниформ буферов нету, кроме указания самого типа буфера. Для разнообразия сразу же заполним данными.

```cpp
    GLint data[] = {0, 1, 2};
    GLuint ssbo;
    glGenBuffers(1, &ssbo);
    glBindBuffer(GL_SHADER_STORAGE_BUFFER, ssbo);
    glBufferData(GL_SHADER_STORAGE_BUFFER, sizeof(data), data, GL_STATIC_DRAW); //можно использовать glBufferStorage
    glBindBufferBase(GL_SHADER_STORAGE_BUFFER, 0, ssbo); //Можно использовать glBindBufferRange
    glBindBuffer(GL_SHADER_STORAGE_BUFFER, 0);
```

Пример буфера в шейдере

```glsl
    layout(binding = 0, std430) buffer Name {...};
```

И стандартное удаление.

```cpp
    glDeleteBuffers(1, &ssbo);
```

### OpenGL 4.5 (DSA)

Тут тоже ничего нового, ни в создании, ни в удалении, все так же как и с другими буферами. Для разнообразия пример с неизменяемым буфером.

```cpp
    glCreateBuffers(1, &ssbo);
    glNamedBufferStorage(ssbo, sizeof(data), data, GL_DYNAMIC_STORAGE_BIT);
    glBindBufferBase(GL_SHADER_STORAGE_BUFFER, 0, ssbo);
    glDeleteBuffers(1, &ssbo);
```
