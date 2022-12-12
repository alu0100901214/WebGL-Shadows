# WebGL-Shadows

In this activity, we are going to show a color suitable for the planar shadow developed earlier. To do this we will compile a second program specifically to render this shadow. 

First we are going to create a new fragment shader that simply gets a dark color that we define for the shadow.

```
    private const string fsShadowSource=@"
        precision mediump float;
        void main(){
            gl_FragColor = vec4(0.0, 0.0, 0.0, 0.9);
        }";
```

Then we are going to define the new program that will render the shadow with its correct color and other variables we will need.

```
    private WebGLShader shadowFragmentShader;
    private WebGLProgram shadowProgram;

    private int shadowPositionAttribLocation;
    private int shadowNormalAttribLocation;
    private int shadowColorAttribLocation;
    
    private WebGLUniformLocation shadowProjectionUniformLocation;
    private WebGLUniformLocation shadowModelViewUniformLocation;
    private WebGLUniformLocation shadowNormalTransformUniformLocation;
```

We modify the function "getAttributeLocations" to specify their locations.

```
    this.shadowPositionAttribLocation = await this._context.GetAttribLocationAsync(this.shadowProgram, "aVertexPosition");
    this.shadowNormalAttribLocation = await this._context.GetAttribLocationAsync(this.shadowProgram, "aVertexNormal");
    this.shadowColorAttribLocation = await this._context.GetAttribLocationAsync(this.shadowProgram, "aVertexColor");
    
    this.shadowProjectionUniformLocation =await this._context.GetUniformLocationAsync(this.shadowProgram, "uProjectionMatrix");
    this.shadowModelViewUniformLocation = await this._context.GetUniformLocationAsync(this.shadowProgram, "uModelViewMatrix");
    this.shadowNormalTransformUniformLocation = await this._context.GetUniformLocationAsync(this.shadowProgram, "uNormalTransformMatrix");
```

Inside "OnAfterRenderAsync", we define the shadow fragment shader and the program shader.

```
    this.shadowFragmentShader = await this.GetShader(fsShadowSource, ShaderType.FRAGMENT_SHADER);
    this.shadowProgram = await this.BuildProgram(this.vertexShader, this.shadowFragmentShader);
```

Inside "Draw" we will use the uniforms and attributes of the locators to draw the shadow.

```
    // Shadows
    await this._context.DrawElementsAsync(Primitive.TRIANGLES,mBuffers.NumberOfIndices,DataType.UNSIGNED_SHORT, 0);

    await this._context.UseProgramAsync(this.shadowProgram);
    await this._context.UniformMatrixAsync(this.shadowProjectionUniformLocation,false,this.ProyMat.GetArray());

    await this._context.BindBufferAsync(BufferType.ARRAY_BUFFER, mBuffers.VertexBuffer);
    await this._context.EnableVertexAttribArrayAsync((uint)this.shadowPositionAttribLocation);
    await this._context.VertexAttribPointerAsync((uint)this.shadowPositionAttribLocation,3, DataType.FLOAT, false, 0, 0L);
            
    await this._context.BindBufferAsync(BufferType.ARRAY_BUFFER, mBuffers.NormalBuffer);
    await this._context.EnableVertexAttribArrayAsync((uint)this.shadowNormalAttribLocation);
    await this._context.VertexAttribPointerAsync((uint)this.shadowNormalAttribLocation,3, DataType.FLOAT, false, 0, 0L);

    await this._context.BindBufferAsync(BufferType.ARRAY_BUFFER, mBuffers.ColorBuffer);
    await this._context.EnableVertexAttribArrayAsync((uint)this.shadowColorAttribLocation);
    await this._context.VertexAttribPointerAsync((uint)this.shadowColorAttribLocation,4, DataType.FLOAT, false, 0, 0L);        

    await this._context.UniformMatrixAsync(this.shadowNormalTransformUniformLocation,false,actor.NormalTransform.GetArray());

    foreach(var smv in actor.ModelViewShadow){
        await this._context.UniformMatrixAsync(this.shadowModelViewUniformLocation,false,smv.GetArray());
    }
```

