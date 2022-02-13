## VBO
顶点缓冲对象(Vertex Buffer Object、VBO)：用于存储顶点的各类属性信息（如顶点坐标、顶点法向量、顶点颜色等）。在渲染时，可以直接从VBO中读取顶点的各类属性数据，由于VBO在显存中而不是在内存中，处理效率更高。每个VBO在OpenGL中有唯一标识ID，通过这个ID可以对特定的VBO内的数据进行存取操作。

## EBO
索引缓冲对象(Element Buffer Object，EBO或Index Buffer Object，IBO)：当需要使用重复的顶点时，可以通过EBO节省内存，提高执行效率。EBO中存储的内容就是顶点位置的索引，EBO跟VBO类似，只不过EBO保存的是顶点的索引。

## VAO
顶点数组对象(Vertex Array Object、VAO)：可以像VBO那样被绑定，任何随后的顶点属性设置都会储存在这个VAO中，之后在绘制物体的时候只需要绑定相应的VAO就可以。在不同顶点数据和属性配置之间切换时，只需绑定不同的VAO就可以。

一个VAO会储存以下这些内容：

- glEnableVertexAttribArray和glDisableVertexAttribArray的调用；
- 通过glVertexAttribPointer设置的顶点属性配置；
- 通过glVertexAttribPointer调用与顶点属性关联的VBO；
- 当目标是GL_ELEMENT_ARRAY_BUFFER的时候，VAO会储存glBindBuffer的函数调用； 
    - 这意味着它也会储存解绑调用，所以确保在解绑VAO之前没有解绑EBO，否则它就没有这个EBO配置了。

## 总结

- VBO
    - 存储顶点数据
- EBO
    - 存储顶点索引
- VAO
    - 存储VBO、EBO的引用
    - 存储顶点的结构定义等

> ![vertex_array_objects_ebo.png](./assets/vertex_array_objects_ebo.png)

``` c
// set up vertex data (and buffer(s)) and configure vertex attributes
float vertices[] = {
     0.5f,  0.5f, 0.0f,  // top right
     0.5f, -0.5f, 0.0f,  // bottom right
    -0.5f, -0.5f, 0.0f,  // bottom left
    -0.5f,  0.5f, 0.0f   // top left 
};
unsigned int indices[] = {  // note that we start from 0!
    0, 1, 3,  // first Triangle
    1, 2, 3   // second Triangle
};

unsigned int VBO, VAO, EBO;
glGenVertexArrays(1, &VAO);
glGenBuffers(1, &VBO);
glGenBuffers(1, &EBO);

// bind the Vertex Array Object first, then bind and set vertex buffer(s), and then configure vertex attributes(s).
glBindVertexArray(VAO);

glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);

glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
glEnableVertexAttribArray(0);

// note that this is allowed, the call to glVertexAttribPointer registered VBO as the vertex attribute's bound vertex buffer object so afterwards we can safely unbind
glBindBuffer(GL_ARRAY_BUFFER, 0); 

// remember: do NOT unbind the EBO while a VAO is active as the bound element buffer object IS stored in the VAO; keep the EBO bound.
//glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, 0);

// You can unbind the VAO afterwards so other VAO calls won't accidentally modify this VAO, but this rarely happens. Modifying other
// VAOs requires a call to glBindVertexArray anyways so we generally don't unbind VAOs (nor VBOs) when it's not directly necessary.
glBindVertexArray(0); 

// ... 省略部分代码

// render loop
// -----------
glBindVertexArray(VAO); // seeing as we only have a single VAO there's no need to bind it every time, but we'll do so to keep things a bit more organized
//glDrawArrays(GL_TRIANGLES, 0, 6);
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
// glBindVertexArray(0); // no need to unbind it every time 
 
glSwapBuffers();
```