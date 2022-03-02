# 高级GLSL

## GLSL的内建变量

GLSL 定义了一些以 gl_ 为前缀的变量，以便 读取/写入 数据。

[Built-in_Variable (GLSL)](https://www.khronos.org/opengl/wiki/Built-in_Variable_(GLSL))


## 接口块

当从顶点着色器向片段着色器发送数据时，可以在着色器中声明对应的输入/输出变量，还可以通过 接口块(Interface Block) 将变量组合起来。

接口块的声明和 struct 的声明有点相像，不同的是，使用 in/out 关键字来定义它是一个 输入/输出块(Block)。

!!! tip "注意"
    块名(Block Name) 需保持一致（VS_OUT），但实例名(Instance Name)可以是随意的。
    只要两个接口块的名字一样，它们对应的输入和输出将会匹配起来。

``` c
//Vertext.shader
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aTexCoords;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

out VS_OUT
{
    vec2 TexCoords;
} vs_out;

void main()
{
    gl_Position = projection * view * model * vec4(aPos, 1.0);
    vs_out.TexCoords = aTexCoords;
} 

//Fragment.shader
#version 330 core
out vec4 FragColor;

in VS_OUT
{
    vec2 TexCoords;
} fs_in;

uniform sampler2D texture;

void main()
{
    FragColor = texture(texture, fs_in.TexCoords);   
}
```

## Uniform缓冲对象

Uniform缓冲对象是一种给Shader传递参数的重要方式，允许将数据发送到 Shader 的任意阶段，即每个声明了该 Uniform变量 的着色器都能够访问该缓冲对象。

