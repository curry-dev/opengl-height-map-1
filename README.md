# opengl-height-map-1

> [!NOTE]
> This repository holds the completed result of my project but the source code.
> I cannot post the source code publicly here to avoid plagiarism, so I put it in another private repository. Although, I can show it upon request.



### An Overview
Height fields may be found in many applications of computer graphics. They are used to represent terrain in video games and simulations, and also often utilized to represent data in three dimensions. This assignment asks you to create a height field based on the data from an image which the user specifies at the command line, and to allow the user to manipulate the height field in three dimensions by rotating, translating, or scaling it. You also have to implement a vertex shader that performs smoothing of the geometry, and re-adjusts the geometry color. After the completion of your program, you will use it to create an animation. You will program the assignment using OpenGL's core profile.

This assignment is intended as a hands-on introduction to OpenGL and programming in three dimensions. It teaches the OpenGL's core profile and shader-based programming. The provided starter gives the functionality to initialize GLUT, read and write a JPEG image, handle mouse and keyboard input, and display one triangle to the screen. You must write code to handle camera transformations, transform the landscape (translate/rotate/scale), and render the heightfield. You must also write a shader to perform geometry smoothing and re-color the terrain accordingly. Please see the OpenGL Programming Guide for information, or OpenGL.org.

### Background Information
A height field is a visual representation of a function which takes as input a two-dimensional point and returns a scalar value ("height") as output.

Rendering a height field over arbitrary coordinates is somewhat tricky--we will simplify the problem by making our function piece-wise. Visually, the domain of our function is a two-dimensional grid of points, and a height value is defined at each point. We can render this data using only a point at each defined value, or use it to approximate a surface by connecting the points with triangles in 3D.

You will be using image data from a grayscale JPEG file to create your height field, such that the two dimensions of the grid correspond to the two dimensions of the image and the height value is a function of the image grayscale level. Since you will be working with grayscale image, the bytes per pixel (i.e., ImageIO::getBytesPerPixel) is always 1 and you don't have to worry about the case where the bytes per pixel is 3 (i.e., RGB images).

### Render Points, Lines and Triangles
Your program needs to render the height field as points (when the key "1" is pressed on the keyboard), lines ("wireframe"; key "2"), or solid triangles (key "3"). The points, lines and solid triangles must be modeled using GL_POINTS, GL_LINES, GL_TRIANGLES, or their "LOOP" or "STRIP" variants.

### Vertex Shader Requirement
You should write a vertex shader that provides two rendering modes.

The first mode -- keys "1", "2", "3"
In the first mode, simply transform the vertex with the modelview and projection matrix. Leave the color unchanged in the shader. This vertex shader is already provided in the starter code. This mode should be used for point rendering (key "1"), line rendering (key "2"), and triangle rendering (key "3"). To improve the visual quality of the terrain, feel free to scale the terrain height with an arbitrary constant of your choice when using these modes (keys "1", "2", "3"). You can do this on the CPU when creating your VBOs.
The second mode -- key "4"
In the second mode (key "4"), do not scale the terrain height on the CPU; the height should equal 1.0 * heightmapImage->getPixel(i, j, 0) / 255.0f . In this mode, you should change the vertex position to the average position of itself and the four neighboring vertices -- do this in the vertex shader. Specifically, replace p_center with (p_center + p_left + p_right + p_down + p_up) / 5 (see image). This will have the effect of smoothening the terrain (the effect is most visible at low image resolutions, e.g., 128 x 128).

![]()

Furthermore, you should change the vertex color and height, also in the vertex shader, according to the formulas:

outputColor <--- pow(y, exponent)
y <--  scale * pow(y, exponent),

where scale and exponent are two constants. These constants are shader uniform variables and must be provided to your shader from the CPU. You can use the provided helper function PipelineProgram::SetUniformVariablef to do this. In your program on the CPU, you should bind the keys as follows:

+    ... multiply the current "scale" by 2x
-    ... divide the current "scale" by 2x
9    ... multiply the current "exponent" by 2x
0    ... divide the current "exponent" by 2x

The initial values for "scale" and "exponent" should both be 1. Then, when user presses those keys, you should upload the new variable value to the GPU. The variables "scale" and "exponent" are only needed when using key "4", i.e., in the second mode -- you don't need them when using keys "1", "2", "3". In mode "4", you only need to support triangle rendering; you do not need to draw points or lines in this mode.

Finally, in the vertex shader, you should then transform the resulting vertex position with the modelview and projection matrix in the vertex shader as usual.

The positions of the four neighboring vertices should be passed into the vertex shader as additional attributes, in the same way as vertex position and color. In order to accommodate this, you should create additional VBOs. For example, one approach is to create 4 VBOs of 3-floats: position of the left vertex, right vertex, up vertex, down vertex. Note that these positions include the height of the vertex as one of its coordinates. If the position of the right vertex is off the image (this will happen on the image boundary), set the "position" of the right vertex to the position of the center vertex. Perform the equivalent operations also for the up/down vertices along the top and bottom edges of the image. The "scale" and "exponent" should be made available in the shader as uniform variables, as explained above.

You should write one vertex shader that satisfies the above requirements. In order to switch between the two modes, you should create a uniform variable "uniform int mode" in the vertex shader, and upload it to the GPU using PipelineProgram::SetUniformVariablei. When the user presses keys "1", "2" or "3", mode should be set to 0, and when the user presses key "4", mode should be set to 1. When mode=0, the vertex shader should execute the first mode described above. When mode=1, the vertex shader should execute the second mode (smoothen the vertex position and scale/exponentiate the terrain). You can achieve this using an "if" statement in the vertex shader.
