# Буферы
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

В opengl версии 4.5 появилась такая вещь как Direct State Access (DSA) — прямой доступ к состоянию. Средство изменения объектов OpenGL без необходимости привязывать их к контексту. Это позволяет изменять состояние объекта в локальном контексте, не затрагивая глобальное состояние, разделяемое всеми частями приложения. Это также делает API-интерфейс немного более объектно-ориентированным, поскольку функции, которые изменяют состояние объектов, могут быть четко определены. 

Давайте перепишем примеры выше используя DSA. Начнем с безиндексного варианта.

```cpp
    glCreateVertexArrays(1, &VAO);
    glCreateBuffers(1, &VBO);
    glNamedBufferData(VBO, sizeof(vertices), vertices, GL_DYNAMIC_STORAGE_BIT); //mutable buffer 
    //glNamedBufferStorage(VBO, sizeof(vertices), vertices, GL_DYNAMIC_STORAGE_BIT); //immutable buffer
    //glNamedBufferSubData(VBO, 0, sizeof(vertices), vertices); //buffer subdata
    glEnableVertexArrayAttrib(VAO, 0);
    glVertexArrayAttribFormat(VAO, 0, 3, GL_FLOAT, false, 0);
    glVertexArrayVertexBuffer(VAO, 0, VBO, 0, 3 * sizeof(GLfloat));
    glVertexArrayAttribBinding(VAO, 0, 0);
    //glVertexArrayBindingDivisor(VAO, 0, 1); //instancing
```

Теперь разберем код, как можно увидеть поменялись все функции. Перечислю замены: `glCreateBuffers` вместо `glGenBuffers` + `glBindBuffer`; `glNamedBufferData` заменяет `glBufferData`; `glVertexAttribFormat`, `glVertexArrayVertexBuffer` и `glVertexArrayAttribBinding` вместо `glVertexAttribPointer`; `glVertexAttribDivisor` заменяет `glVertexArrayBindingDivisor`; `glNamedBufferSubData` заменяет `glBufferSubData` и добавился новый тип буфера `glNamedBufferStorage` (которыя является в отличии от `glNamedBufferData` immutable, неизменяемым). Как можно заметить у некоторых функций заменился тип буфера на объект буфера или вершинного массива в первом параметре, например `glEnableVertexAttribArray(0);` поменялся на `glEnableVertexArrayAttrib(VAO, 0);` где мы указываем прямо какому VAO включить 0 атрибут, вместо привязанному в первом варианте.

Описание новых функций:

```cpp
void glVertexArrayAttribFormat(GLuint vaobj, GLuint attribindex, GLint size, GLenum type, GLboolean normalized, GLuint relativeoffset)
```

по сути является аналогом `glVertexAttribPointer`

- vaobj имя объекта массива вершин
- attribindex описываемый индекс атрибутов вершин
- size количество значений на вершину, которые хранятся в массиве. Один из: 1 2 3 4 BGRA
- type тип данных, хранящихся в массиве
- normalized, если true, тогда целочисленные данные нормализуются в диапазоне [-1, 1] или [0, 1], если они подписаны или не подписаны, соответственно. Если false, то целочисленные данные напрямую преобразуются в число с плавающей запятой.
- relativeoffset относительное смещение, измеренного в базовых машинных единицах первого элемента относительно начала привязки буфера вершин, из которого этот атрибут выбирается 

```cpp
void glVertexArrayVertexBuffer(GLuint vaobj,GLuint bindingindex, GLuint buffer, GLintptr offset, GLsizei stride)
```

Связывает буфер с точкой привязки вершинного буфера.

- vaobj имя объекта массива вершин
- bindingindex индекс точки привязки вершинного буфера, к которой привязывается буфер 
- buffer имя существующего буфера для привязки к точке привязки вершинного буфера
- offset смещения первого элемента буфера
- stride расстояние между элементами в буфере

```cpp
void glVertexArrayAttribBinding(GLuint vaobj, GLuint attribindex, GLuint bindingindex)
```

Связывает атрибуты вершин и привязку вершинного буфера. То есть связывает attribindex и bindingindex между собой из функций `glVertexArrayAttribFormat` и `glVertexArrayVertexBuffer` соответственно.

- vaobj имя объекта массива вершин
- attribindex индекс атрибута для привязки к привязке буфера вершин
- bindingindex индекс привязки буфера вершин, с которым связывается общий атрибут вершины

```cpp
void glVertexArrayBindingDivisor(GLuint vaobj, GLuint bindingindex, GLuint divisor)
```

Изменяет скорость, с которой общие атрибуты вершин продвигаются во время рендеринга экземпляров. Например если divisor равен: 0 - каждую вершину, 1 - каждый экземпляр.

- vaobj имя объекта массива вершин
- bindingindex индекс общего атрибута вершины
- divisor количество экземпляров, которые будут проходить между обновлениями атрибута в индексе слота

Теперь пример с использованием индексов.

```cpp
    glCreateVertexArrays(1, &VAO);
    glCreateBuffers(1, &VBO);
    glCreateBuffers(1, &VBE);
    glNamedBufferData(VBO, sizeof(vertices), vertices, GL_DYNAMIC_STORAGE_BIT);
    glNamedBufferData(VBE, sizeof(indices), indices, GL_DYNAMIC_STORAGE_BIT);
    glEnableVertexArrayAttrib(VAO, 0);
    glVertexArrayAttribFormat(VAO, 0, 3, GL_FLOAT, false, 0);
    glVertexArrayVertexBuffer(VAO, 0, VBO, 0, 3 * sizeof(GLfloat));
    glVertexArrayAttribBinding(VAO, 0, 0);
    glVertexArrayElementBuffer(VAO, VBE);
```

Все тоже самое, только добавился еще один буфер, создаваемый так же как и VBO, только привязываемый к VAO функцией `glVertexArrayElementBuffer`

```cpp
void glVertexArrayElementBuffer(GLuint vaobj, GLuint buffer)
```

Связывает объект буфера массива элементов с точкой привязки объекта массива вершин.

- vaobj имя объекта массива вершин
- buffer имя объекта буфера. Если buffer равен нулю, любая существующая привязка буфера массива элементов к vaobj удаляется.

Так же в новой версии появилась возможность хранить индексы и вершины в одном буфере.

```cpp
    glCreateVertexArrays(1, &VAO);
    glCreateBuffers(1, &VBO);
    glNamedBufferData(VBO, sizeof(indices) + sizeof(vertices), NULL, GL_DYNAMIC_STORAGE_BIT);
    glNamedBufferSubData(VBO, 0, sizeof(indices), indices);
    glNamedBufferSubData(VBO, sizeof(indices), sizeof(vertices), vertices);
    glEnableVertexArrayAttrib(VAO, 0);
    glVertexArrayAttribFormat(VAO, 0, 3, GL_FLOAT, false, 0);
    glVertexArrayVertexBuffer(VAO, 0, VBO, sizeof(indices), 3 * sizeof(GLfloat));
    glVertexArrayAttribBinding(VAO, 0, 0);
    glVertexArrayElementBuffer(VAO, VBO);
```

Индексы должны располагаться перед данными вершин а в настройке данных вершин в функции glVertexArrayVertexBuffer надо указать 4 параметром смещение данных вершин относительно начала буфера. В данном коде сначала выделяется буфер а потом заполняется сначала данными индексом с начала буфера а потом данными вершин через `glNamedBufferSubData`.

Изменений в отрисовке нету.

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

## Кадровый и рендер буферы

WIP

## Юниформ буфер (ubo)

WIP

## Буфер хранилища шейдеров (ssbo)

WIP
