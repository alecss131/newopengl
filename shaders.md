# Шейдеры

В данном разделе будут рассмотрены нововведения в шейдерах, но не в самих шейдерах \(языке glsl\), а в их использовании в программе, то есть их создание и использование. Разумеется тут не все изменения, а те которые я использую и о которых знаю.

Общие данные для большинства примеров

```cpp
	GLuint vertexShader;
	GLuint fragmentShader;
	const char * vertexShaderSource = "";
	const char * fragmentShaderSource = "";
	GLint success;
	GLchar infoLog[512];
	GLuint shaderProgram;
```

На версии 3.3 доступны 3 вида шейдеров: вершинный \(2.0\), геометрический \(3.2\) и фрагментный \(2.0\), но для шейдерной программы достаточно только двух шейдеров \(вершинный и фрагментный\), геометрический может быть а может и не быть, но обязательно должна быть пара вершинный + фрагментный. Геометрический шейдер \(впрочем как и любой другой шейдер\) создается так же как и остальные, так что в примерах будет рассмотрена минимальная рабочая шейдерная программа \(из 2 шейжеров\) а так же проверки на ошибки.

```cpp
	//создание вершинного шейдера
	vertexShader = glCreateShader(GL_VERTEX_SHADER);
	glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
	glCompileShader(vertexShader);
	//проверка ошибок
	glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
	if (!success) {
		glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
		std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
	}
	//создание фрагментного шейдера
	fragmentShader = glCreateShader(GL_FRAGMENT_SHADER);
	glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
	glCompileShader(fragmentShader);
	//проверка ошибок
	glGetShaderiv(fragmentShader, GL_COMPILE_STATUS, &success);
	if (!success) {
		glGetShaderInfoLog(fragmentShader, 512, NULL, infoLog);
		std::cout << "ERROR::SHADER::FRAGMENT::COMPILATION_FAILED\n" << infoLog << std::endl;
	}
	//создание шейдерной программы
	shaderProgram = glCreateProgram();
	glAttachShader(shaderProgram, vertexShader);
	glAttachShader(shaderProgram, fragmentShader);
	glLinkProgram(shaderProgram);
	//проверка ошибок
	glValidateProgram(shaderProgram);
	glGetProgramiv(shaderProgram, GL_LINK_STATUS, &success);
	if(!success) {
		glGetProgramInfoLog(shaderProgram, 512, NULL, infoLog);
		std::cout << "ERROR::SHADER::PROGRAM::VALIDATION_FAILED\n" << infoLog << std::endl;
	}
	//удаление шейдеров, так как они больше не нужны после включения в шейдерную программу
	glDeleteShader(vertexShader);
	glDeleteShader(fragmentShader);
	//использование (приввзка) шейдерной программы
	glUseProgram(shaderProgram);
  //отвязывание шейдерной программы
  glUseProgram(0);
	//пример передачи юниформы
	GLint vertexColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
	glUseProgram(shaderProgram);
	glUniform4f(vertexColorLocation, 0.0f, 0.5f, 0.0f, 1.0f); //отправка vec4 в шейдер
	//очистка
	glDeleteProgram(shaderProgram);
```

Не буду пояснять что и как работает, все очевидно. У геометрического шейдера флаг `GL_GEOMETRY_SHADER`.

## OpenGL 4.0

В 4.0 версии появились два новых типа шейдеров. Шейдер управления тесселяцией \(Tessellation Control Shader, TCS\) и шейдер выполнения \(вычисления\) тесселяции \(Tessellation Evaluation Shader, TES\). Не буду пояснять для чего они и как работают, скажу лишь что они обязательно должны идти в паре \(tcs + tes\). Как уже было сказано выше, они создаются и используются так же как и все шейдеры, но у них флаги `GL_TESS_CONTROL_SHADER` и `GL_TESS_EVALUATION_SHADER`.

## OpenGL 4.1

В версии 4.1 появились такие вещи как SSO \(Separate Shader Object\) и Program Pipeline. SSO позволяют нам изменять этапы шейдера на лету, не связывая их заново. То есть теперь можно создать шейдерную программу только из одного шейдера и прикреплять к конвееру любую нужную комбинацию шейдерных программ (ограниения остаются, минимум вершинный и фрагментный шейдер, а шейдеры тесселяции всегда в паре). Но есть один нюанс, теперь входные и выходные параметры шейдеров должны совпадать. Мы можем объявить входные/выходные параметры в одном и том же порядке, с одинаковыми именами, либо мы делаем их местоположение явно совпадающим с помощью квалификаторов, то есть руками указывать \(например`layout (location = 0) in vec4 name;`\).
В качестве обеспечения более строгого интерфейса нам также нужно объявить встроенные блоки ввода и вывода, которые мы хотим использовать для каждого этапа. Не обязательно объявлять блок целиком, достаточно указать только то что используется.

Встроенные интерфейсы блоков определены как \([из вики](https://www.khronos.org/opengl/wiki/Built-in_Variable_(GLSL))\):

Вершинный:

```glsl
out gl_PerVertex
{
  vec4 gl_Position;
  float gl_PointSize;
  float gl_ClipDistance[];
};
```

Управления тесселяцией:

```glsl
out gl_PerVertex
{
  vec4 gl_Position;
  float gl_PointSize;
  float gl_ClipDistance[];
} gl_out[];
```

Выполнения тесселяции:

```glsl
out gl_PerVertex {
  vec4 gl_Position;
  float gl_PointSize;
  float gl_ClipDistance[];
};
```

Геометрический:

```glsl
out gl_PerVertex
{
  vec4 gl_Position;
  float gl_PointSize;
  float gl_ClipDistance[];
};
```

Сами шейдеры можно создавать как и раньше, только добавив специальный флаг к шейдеру перед сборкой программы из одного шейдера. Пример такой программы состоящей только из вершинного шейдера:

```cpp
  GLuint vertexShaderProgram;
  //создание шейдера
  vertexShader = glCreateShader(GL_VERTEX_SHADER);
	glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
	glCompileShader(vertexShader);
	//проверка ошибок
	glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
	if (!success) {
		glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
		std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
	}
  vertexShaderProgram = glCreateProgram();
  glProgramParameteri(vertexShaderProgram, GL_PROGRAM_SEPARABLE, GL_TRUE); //новый флаг
  glAttachShader(vertexShaderProgram, vertexShader);
	glLinkProgram(vertexShaderProgram);
	//check errors
	glValidateProgram(vertexShaderProgram);
	glGetProgramiv(vertexShaderProgram, GL_LINK_STATUS, &success);
	if(!success) {
		glGetProgramInfoLog(vertexShaderProgram, 512, NULL, infoLog);
		std::cout << "ERROR::SHADER::PROGRAM::VALIDATION_FAILED\n" << infoLog << std::endl;
	}
	//удаление шейдера
	glDeleteShader(vertexShader);
  //пример передачи юниформы
  GLint vertexColorLocation = glGetUniformLocation(vertexShaderProgram, "ourColor");
  glProgramUniform4f(vertexShaderProgram, vertexColorLocation, 0.5f, 0.0f, 1.0f, 1.0f); // новая функция
  //очистка
  glDeleteProgram(vertexShaderProgram);
```

Появились две новые функции: `glProgramParameteri(vertexShaderProgram, GL_PROGRAM_SEPARABLE, GL_TRUE);` где мы задаем шейдерной программе что она будет sso и `glProgramUniform4f(vertexShaderProgram, vertexColorLocation, 0.5f, 0.0f, 1.0f, 1.0f);` новая функция для передачи юниформы, где первый параметром передается sso шейдерная программа которой надо передать юниформу. Замечу что я специально не привел использование шейдерной программы, так как оно изменилось и будет рассмотрено далее.

Вместе с sso появился новый и более короткий способоб их создания, вместо примера выше можно их создавать и так:

```cpp
  //создание программы
	GLuint vertexShaderProgram;
	vertexShaderProgram = glCreateShaderProgramv(GL_VERTEX_SHADER, 1, &vertexShaderSource);
  //проверка ошибок
	glValidateProgram(vertexShaderProgram);
	glGetProgramiv(vertexShaderProgram, GL_LINK_STATUS, &success);
	if (!success) {
		glGetProgramInfoLog(vertexShaderProgram, 512, NULL, infoLog);
		std::cout << "ERROR::SHADER::PROGRAM::VALIDATION_FAILED\n" << infoLog << std::endl;
	}
  //пример передачи юниформы
  GLint vertexColorLocation = glGetUniformLocation(vertexShaderProgram, "ourColor");
	glProgramUniform4f(vertexShaderProgram, vertexColorLocation, 0.5f, 0.0f, 1.0f, 1.0f);
	//очистка
	glDeleteProgram(vertexShaderProgram);
```

Как можно заметить все сократилось (без проверки ошибок) до 1 строчки в функцию `glCreateShaderProgramv(GL_VERTEX_SHADER, 1, &vertexShaderSource);` где первым параметром идет тип шейдерной программы, вторым количество, а последним код шейдеров.

Так же появилась возможность сохранять и загружать готовые скомпилированные бинарные шейдеры и шейдерные программы. Для загрузки используется функции [glShaderBinary](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glShaderBinary.xhtml) и [glProgramBinary](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glProgramBinary.xhtml) а для сохранения [glGetProgramBinary](https://www.khronos.org/registry/OpenGL-Refpages/gl4/html/glGetProgramBinary.xhtml), функции glGetShaderBinary я не нашел (интересно почему?). Не буду подробно рассматривать их использование, скажу лишь что загрузка через `glShaderBinary` заменяет функции `glShaderSource` и `glCompileShader`.

Теперь про использование sso и про program pipeline. Program pipeline представляет из себя конвеер состоящий из шейдерных программ (sso), к которому можно присоединять разные программы без их перекомпилирования.

```cpp
	GLuint pipeline;
	//создание
	glGenProgramPipelines(1, &pipeline);
	//привязывание стадий
	glUseProgramStages(pipeline, GL_VERTEX_SHADER_BIT, vertexShaderProgram);
	//проверка на ошибки
	glValidateProgramPipeline(pipeline);
	glGetProgramPipelineiv(pipeline, GL_VALIDATE_STATUS, &success);
	if (!success) {
		glGetProgramPipelineInfoLog(pipeline, 512, NULL, infoLog);
		std::cout << "ERROR::PIPELINE::VALIDATION_FAILED\n" << infoLog << std::endl;
	}
	//использование (привязка)
	glBindProgramPipeline(pipeline);
	// отвязывание
	glBindProgramPipeline(0);
	//очистка
	glDeleteProgramPipelines(1, &pipeline);
```

Принцип создания похож на создание шейдерной программы, используется так же (вместо sso надо привязвать конвеер содержащий sso, а юниформы передаются sso). После создания надо привязать sso через `glUseProgramStages` используя новые флаги стадий. Флаги для всех типов шейдеров `GL_VERTEX_SHADER_BIT`, `GL_FRAGMENT_SHADER_BIT`, `GL_GEOMETRY_SHADER_BIT`, `GL_TESS_CONTROL_SHADER_BIT`, `GL_TESS_EVALUATION_SHADER_BIT`.

## OpenGL 4.3

В версии 4.3 появилась возможность вручную указывать расположение юниформ в шейдерах. Что избавляет от использования функции `glGetUniformLocation`. Пример использования в шейдере:

```glsl
layout (location = 0) uniform vec4 ourColor;
```

Пример передачи юниформы в программе:

```cpp
  // без sso
  glUseProgram(shaderProgram);
  glUniform4f(0, 0.0f, 0.5f, 0.0f, 1.0f);
  // с sso
  glProgramUniform4f(vertexShaderProgram, 0, 0.5f, 0.0f, 1.0f, 1.0f);
```

## OpenGL 4.5

В версии 4.5 появилась новая функция для создания pipeline `glCreateProgramPipelines` которая по сути просто переименовалась на манер DSA, то есть в имени Gen заменилось на Create.

## OpenGL 4.6

В данной версии появилась поддержка (и возможность использования и загрузки) бинарных шейдеров SPIR-V используемых в Vulkan. Замечу что это единственное нововведение всего 4.6 обновления (последнего).

Их можно использовать всеми способами рассмотренными выше кроме краткого создания sso. Приведу пример одного из способов использования, используя sso.

```cpp
	GLuint vertexShaderProgram;
	std::vector<unsigned char> vertexSpirv; //бинарные данные скомпилированного шейдера
	//создание шейдера
	vertexShader = glCreateShader(GL_VERTEX_SHADER);
	glShaderBinary(1, &vertexShader,  GL_SHADER_BINARY_FORMAT_SPIR_V, vertexSpirv.data(), vertexSpirv.size());
	glSpecializeShader(vertexShader, "main", 0, NULL, NULL);
	//проверка ошибок
	glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);
	if (!success) {
		glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
		std::cout << "ERROR::SHADER::VERTEX::VALIDATION_FAILED\n" << infoLog << std::endl;
	}
	vertexShaderProgram = glCreateProgram();
	glProgramParameteri(vertexShaderProgram, GL_PROGRAM_SEPARABLE, GL_TRUE);
	glAttachShader(vertexShaderProgram, vertexShader);
	glLinkProgram(vertexShaderProgram);
	//check errors
	glValidateProgram(vertexShaderProgram);
	glGetProgramiv(vertexShaderProgram, GL_LINK_STATUS, &success);
	if (!success) {
		glGetProgramInfoLog(vertexShaderProgram, 512, NULL, infoLog);
		std::cout << "ERROR::SHADER::PROGRAM::COMPILATION_FAILED\n" << infoLog << std::endl;
	}
```

Потом созданную программу используем как стадию в конвеере. Замечу что использовать в одном конвеере комбинации из glsl и spir-v нельзя, все шейдеры должны быть одного типа. С помощью `glShaderBinary(1, &vertexShader,  GL_SHADER_BINARY_FORMAT_SPIR_V, vertexSpirv.data(), vertexSpirv.size());` мы загружаем бинарный шейдер, указав тип данных `GL_SHADER_BINARY_FORMAT_SPIR_V`, а с помощью `glSpecializeShader(vertexShader, "main", 0, NULL, NULL);` указываем точку входа в шейдер (главную функцию), последующие параметры на не интересуют.
