# Текстуры

В данном разделе будут рассмотрены нововведения в текстурах.

Общие данные для большинства примеров.

```cpp
int width, height, nrChannels;
unsigned char* data; //данные текстуры готовые для загрузки в видеокарту
GLuint texture;
```

На версии 3.3 доступны к созданию одиночные (1D, 2D, 3D) текстуры, массивы текстур (1D, 2D) и кубические текстуры. Создавались они следующим образом (давно не пользовался этими способами, так что могут быть ошибки, но надесь нет)

```cpp
//простые
glGenTextures(1, &texture);
glPixelStorei(GL_UNPACK_ALIGNMENT, 1);
glBindTexture(GL_TEXTURE_2D, texture);
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
glGenerateMipmap(GL_TEXTURE_2D); //генерация мипмапов
//настройка параметров сэмплера
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST_MIPMAP_NEAREST);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
glBindTexture(GL_TEXTURE_2D, 0);

//массив 1D текстур
glTexImage2D(GL_TEXTURE_1D_ARRAY, 0, GL_RGB, width, 3, 0, GL_RGB, GL_UNSIGNED_BYTE, NULL); //1d array 3 elems
for (GLuint i = 0; i < 3; i++) {
	glTexSubImage2D(GL_TEXTURE_1D_ARRAY, 0, 0, i, width, 1, GL_RGB, GL_UNSIGNED_BYTE, data);
}

//массив 2D текстур
glTexImage3D(GL_TEXTURE_2D_ARRAY, 0, GL_RGB, width, height, 3, 0, GL_RGB, GL_UNSIGNED_BYTE, NULL); //2d array 3 elems
for (GLuint i = 0; i < 3; i++) {
	glTexSubImage3D(GL_TEXTURE_2D_ARRAY, 0, 0, 0, i, width, height, 1, GL_RGB, GL_UNSIGNED_BYTE, data);
}

//кубическая карта
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_CUBE_MAP, texture);
for (GLuint i = 0; i < 6; i++) {
	glTexImage2D(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
}
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
glTexParameteri(GL_TEXTURE_CUBE_MAP, GL_TEXTURE_WRAP_R, GL_CLAMP_TO_EDGE);
glBindTexture(GL_TEXTURE_CUBE_MAP, 0);
```

Использование и удаление на примере простой текстуры

```cpp
//использование
glActiveTexture(GL_TEXTURE0);
glBindTexture(GL_TEXTURE_2D, texture);
//здесь рисование
glBindTexture(GL_TEXTURE_2D, 0);

//удаление
glDeleteTextures(1, &texture);
```

## OpenGL 4.0 (массив кубических текстур)

Появился новый вид текстур массивы кубических текстур

```cpp
glTexImage3D(GL_TEXTURE_CUBE_MAP_ARRAY, 0, GL_RGB, width, height, 12, 0, GL_RGB, GL_UNSIGNED_BYTE, NULL); //array 2 maps
for (GLuint i = 0; i < 12; i++) {
	glTexSubImage3D(GL_TEXTURE_CUBE_MAP_ARRAY, 0, 0, 0, i, width, height, 1, GL_RGB, GL_UNSIGNED_BYTE, data);
}
```

## OpenGL 4.2 (immutable текстуры)

Появился новый тип хранения текстур, так называемые неизменяемые (immutable) текстуры. Является более безопасным вариантом и в отличии от предыдущего (изменяемые, mutable) не позволяет менять размер текстур после создания. Сначала создается хранилище для текстуры заданного размера, а потом заполняется данными.

Примеры создания текстур, использование и удаление без изменений.

```cpp
//простые текстуры
glTexStorage2D(GL_TEXTURE_2D, 3, GL_RGBA8, width, height); //immutable empty texture
glTexSubImage2D(GL_TEXTURE_2D, 0, 0, 0, width, height, GL_RGBA, GL_UNSIGNED_BYTE, data); //fill texture

//массив 1D текстур
glTexStorage2D(GL_TEXTURE_1D_ARRAY, 3, GL_RGBA8, width, 3); //1d array
for (GLuint i = 0; i < 3; i++) {
	glTexSubImage2D(GL_TEXTURE_1D_ARRAY, 0, 0, i, width, 1, GL_RGB, GL_UNSIGNED_BYTE, data);
}

//массив 2D текстур
glTexStorage3D(GL_TEXTURE_2D_ARRAY, 3, GL_RGBA8, width, height, 3); //2d array
for (GLuint i = 0; i < 3; i++) {
	glTexSubImage3D(GL_TEXTURE_2D_ARRAY, 0, 0, 0, i, width, height, 1, GL_RGB, GL_UNSIGNED_BYTE, data);
}

//кубические карты
glTexStorage2D(GL_TEXTURE_CUBE_MAP, 3, GL_RGBA8, width, height); //cubemap
for (GLuint i = 0; i < 6; i++) {
	glTexSubImage2D(GL_TEXTURE_CUBE_MAP_POSITIVE_X + i, 0, 0, 0, width, height, GL_RGB, GL_UNSIGNED_BYTE, data);
}

//массив кубических карт
glTexStorage3D(GL_TEXTURE_CUBE_MAP_ARRAY, 3, GL_RGBA8, width, height, 12); //cubemap array 2 maps
for (GLuint i = 0; i < 12; i++) {
	glTexSubImage3D(GL_TEXTURE_CUBE_MAP_ARRAY, 0, 0, 0, i, width, height, 1, GL_RGB, GL_UNSIGNED_BYTE, data);
}
 ```
 
 ## OpenGL 4.5 (DSA)
 В opengl версии 4.5 появилась такая вещь как Direct State Access (DSA) — прямой доступ к состоянию. Средство изменения объектов OpenGL без необходимости привязывать их к контексту. Это позволяет изменять состояние объекта в локальном контексте, не затрагивая глобальное состояние, разделяемое всеми частями приложения. Это также делает API-интерфейс немного более объектно-ориентированным, поскольку функции, которые изменяют состояние объектов, могут быть четко определены. Тоже самое применимо и к текстурам.

Новый способ работы с текстурами, использовать можно и по-старому и по новому.

```cpp
//простые текстуры
glCreateTextures(GL_TEXTURE_2D, 1, &texture);
glPixelStorei(GL_UNPACK_ALIGNMENT, 1);
glTextureStorage2D(texture, 3, GL_RGBA8, width, height);
glTextureSubImage2D(texture, 0, 0, 0, width, height, GL_RGBA, GL_UNSIGNED_BYTE, data);
glGenerateTextureMipmap(texture); //создание мипмапов
//настройка сэмплера
glTextureParameteri(texture, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
glTextureParameteri(texture, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
glTextureParameteri(texture, GL_TEXTURE_MIN_FILTER, GL_NEAREST_MIPMAP_NEAREST);
glTextureParameteri(texture, GL_TEXTURE_MAG_FILTER, GL_NEAREST);

//массив 1D текстур
glCreateTextures(GL_TEXTURE_1D_ARRAY, 1, &texture);
glTextureStorage2D(texture, 3, GL_RGBA8, width, 3); //1d array
for (GLuint i = 0; i < 3; i++) {
	glTextureSubImage2D(texture, 0, 0, i, width, 1, GL_RGB, GL_UNSIGNED_BYTE, data);
}

//кубические текстуры
glCreateTextures(GL_TEXTURE_CUBE_MAP, 1, &texture);
glTextureStorage2D(texture, 3, GL_RGBA8, width, height); //cubemap
for (GLuint i = 0; i < 6; i++) {
	glTextureSubImage3D(texture, 0, 0, 0, i, width, height, 1, GL_RGB, GL_UNSIGNED_BYTE, data);
}

//кубические текстуры
glCreateTextures(GL_TEXTURE_2D_ARRAY, 1, &texture);
glTextureStorage3D(texture, 3, GL_RGBA8, width, height, 3); //2d array
for (GLuint i = 0; i < 3; i++) {
	glTextureSubImage3D(texture, 0, 0, 0, i, width, height, 1, GL_RGB, GL_UNSIGNED_BYTE, data);
}

//массив кубических текстур
glCreateTextures(GL_TEXTURE_CUBE_MAP_ARRAY, 1, &texture);
glTextureStorage3D(texture, 3, GL_RGBA8, width, height, 12); //cubemap array 2 maps
for (GLuint i = 0; i < 12; i++) {
	glTextureSubImage3D(texture, 0, 0, 0, i, width, height, 1, GL_RGB, GL_UNSIGNED_BYTE, data);
}
```

Новый способ использования текстур

```cpp
glBindTextureUnit(0, texture); //привязываем текстуру к 0 сэмплеру
//рисуем здесь
glBindTextureUnit(0, 0);
```

В данном способе две старые функции `glActiveTexture` `glBindTexture` были объеденены в одну `glBindTextureUnit`.
