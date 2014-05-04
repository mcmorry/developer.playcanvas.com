---
title: Custom Shaders
template: tutorial-page.tmpl.html
position: 3
---

<iframe src="http://apps.playcanvas.com/playcanvas/tutorials/custom_shaders?overlay=false" ></iframe>

*This tutorial uses a custom shader on a material to create a dissolve effect in GLSL*

When you import your 3D models into PlayCanvas by default they will use a Phong Material. This is a versatile material type that can cover a lot of your rendering needs. It's especially useful as it maps well to the materials that you create in 3D creation programs.

However, you will often want to perform special effects or special cases for your materials. To do this you will need to write a custom shader.

**GLSL Shader Example**

WebGL uses GLSL (GL Shader Language) to write shaders that can be run across all browsers. In PlayCanvas this code is assigned to a [Shader Definition][shader_def] and then used in the constructor to make a new `pc.fw.Shader`.

~~~javascript~~~
var shaderDefinition = {
    attributes: {
        aPosition: pc.gfx.SEMANTIC_POSITION,
        aUv0: pc.gfx.SEMANTIC_TEXCOORD0
    },
    vshader: [
        "attribute vec3 aPosition;",
        "attribute vec2 aUv0;",
        "",
        "uniform mat4 matrix_model;",
        "uniform mat4 matrix_viewProjection;",
        "",
        "varying vec2 vUv0;",
        "",
        "void main(void)",
        "{",
        "    vUv0 = aUv0;",
        "    gl_Position = matrix_viewProjection * matrix_model * vec4(aPosition, 1.0);",
        "}"
    ].join("\n"),
    fshader: [
        "precision " + gd.precision + " float;",
        "",
        "varying vec2 vUv0;",
        "",
        "uniform sampler2D uDiffuseMap;",
        "uniform sampler2D uHeightMap;",
        "uniform float uTime;",
        "",
        "void main(void)",
        "{",
        "    float height = texture2D(uHeightMap, vUv0).r;",
        "    vec4 color = texture2D(uDiffuseMap, vUv0);",
        "    if (height < uTime) {",
        "      discard;",
        "    }",
        "    if (height < (uTime+0.04)) {",
        "      color = vec4(0, 0.2, 1, 1.0);",
        "    }",
        "    gl_FragColor = color;",
        "}"
    ].join("\n")
};
~~~

The shader definition contains three sections. In `attributes` you must specify the variable names and the value of attributes that will be defined for each Vertex that your vertex shader is executed for. These values are later declared in your vertex shader as and `attribute`.

The Vertex Shader code is supplied as a string in the `vshader` property and the Fragment Shader is supplied as a string in the 'fshader' property.

Above is the shader definition used to make the dissoving effcet. Notice that in order to make the GLSL code readable in the script file, we use this trick of creating a list of strings and calling `.join("\n")` to create a string.

Aside from attributes there are two other special types of variable in the GLSL shaders: `varying` and `uniform`

**GLSL `varying` variables**

A variable that is declared **varying** will be set in the vertex shader, but used in the fragment shader. It's a way of passing data on from the first program to the second.

**GLSL `uniform` variables**

A variable declared **`uniform`** will be declared in both vertex and fragment shaders. The value of this variable must be passed into the shader from the main application. For example, the position of a light in your scene.

**Creating Materials**

~~~javascript~~~
    // Create the shader from the definition
    this.shader = new pc.gfx.Shader(gd, shaderDefinition);

    // Create a new material and set the shader
    this.material = new pc.scene.Material();
    this.material.setShader(this.shader);

    // Set the initial parameters
    this.material.setParameter('uTime', 0);
    this.material.setParameter('uDiffuseMap', this.diffuseTexture);

    // Replace the material on the model with our new material
    model.meshInstances[0].material = this.material;
~~~

Once we've got the shader definition we create a new Shader and a new Material and pass the shader onto the material using `setShader()`. The uniforms are then set using the `setParameter()` method. Finally we are replacing the original material on the model with the new material we've created. Notice, that each mesh in a model has it's own material. So if your model has more than one mesh, you may need to set the material onto more than once.

*You can (and should) use the same material on more than one mesh.*

**Using a texture in a new Material**

~~~javascript~~~
    // Get the "clouds" height map from the assets and set the material to use it
    var asset = context.assets.find("clouds");
    if (asset) {
        context.assets.load(asset).then(function (results) {
            this.heightMap = results[0];
            this.material.setParameter('uHeightMap', this.heightMap);

        }.bind(this));
    }
~~~

The effect demonstrated in this tutorial is achieved using a height map texture. We access the texture from the asset registry using the code above. When it's loaded we can set the uniform `uHeightMap` to be the `pc.gfx.Texture` object.

**Updating uniforms**

~~~javascript~~~
// Called every frame, dt is time in seconds since last update
update: function (dt) {
    this.time += dt;

    // Bounce value of t 0->1->0
    var t = (this.time % 2);
    if (t > 1) {
        t = 1 - (t - 1);
    }

    // Update the time value in the material
    this.material.setParameter('uTime', t);
}
~~~

To achieve the disappearing effect we use the height map value as a threshold, and we increase the threshold over time. In the update method above we bounce the value of `t` between 0 and 1 and we set this as the `uTime` uniform.

In our shader if the value of the heightmap on a pixel is less than the value time value we don't draw the pixel. In addition at values that are close to the threshold, we draw the pixel in blue to display a nice 'edge' to the effect.

**Complete listing**

~~~javascript~~~
pc.script.create('custom_shader', function (context) {
    // Creates a new Custom_shader instance
    var Custom_shader = function (entity) {
        this.entity = entity;

        this.time = 0;
        this.heightMap = null;
        this.shader = null;
    };


    Custom_shader.prototype = {
        // Called once after all resources are loaded and before the first update
        initialize: function () {
            var model = this.entity.model.model;
            var gd = context.graphicsDevice;

            // Save the diffuse map from the original material before we replace it.
            this.diffuseTexture = model.meshInstances[0].material.diffuseMap;

            // A shader definition used to create a new shader.
            var shaderDefinition = {
                attributes: {
                    aPosition: pc.gfx.SEMANTIC_POSITION,
                    aUv0: pc.gfx.SEMANTIC_TEXCOORD0
                },
                vshader: [
                    "attribute vec3 aPosition;",
                    "attribute vec2 aUv0;",
                    "",
                    "uniform mat4 matrix_model;",
                    "uniform mat4 matrix_viewProjection;",
                    "",
                    "varying vec2 vUv0;",
                    "",
                    "void main(void)",
                    "{",
                    "    vUv0 = aUv0;",
                    "    gl_Position = matrix_viewProjection * matrix_model * vec4(aPosition, 1.0);",
                    "}"
                ].join("\n"),
                fshader: [
                    "precision " + gd.precision + " float;",
                    "",
                    "varying vec2 vUv0;",
                    "",
                    "uniform sampler2D uDiffuseMap;",
                    "uniform sampler2D uHeightMap;",
                    "uniform float uTime;",
                    "",
                    "void main(void)",
                    "{",
                    "    float height = texture2D(uHeightMap, vUv0).r;",
                    "    vec4 color = texture2D(uDiffuseMap, vUv0);",
                    "    if (height < uTime) {",
                    "      discard;",
                    "    }",
                    "    if (height < (uTime+0.04)) {",
                    "      color = vec4(0, 0.2, 1, 1.0);",
                    "    }",
                    "    gl_FragColor = color;",
                    "}"
                ].join("\n")
            };

            // Create the shader from the definition
            this.shader = new pc.gfx.Shader(gd, shaderDefinition);

            // Create a new material and set the shader
            this.material = new pc.scene.Material();
            this.material.setShader(this.shader);

            // Set the initial parameters
            this.material.setParameter('uTime', 0);
            this.material.setParameter('uDiffuseMap', this.diffuseTexture);

            // Replace the material on the model with our new material
            model.meshInstances[0].material = this.material;


            // Get the "clouds" height map from the assets and set the material to use it
            var asset = context.assets.find("clouds");
            if (asset) {
                context.assets.load(asset).then(function (results) {
                    this.heightMap = results[0];
                    this.material.setParameter('uHeightMap', this.heightMap);

                }.bind(this));
            }
        },

        // Called every frame, dt is time in seconds since last update
        update: function (dt) {
            this.time += dt;

            // Bounce value of t 0->1->0
            var t = (this.time % 2);
            if (t > 1) {
                t = 1 - (t - 1);
            }

            // Update the time value in the material
            this.material.setParameter('uTime', t);
        }
    };

    return Custom_shader;
});
~~~

Here is the complete script. Attach this to an Entity with a model component to see it in action. *Note, it only works on a model with a single mesh.* It's left as an exercise to the reader to implement a shader which performs this dissolve effect on a model with many meshes and materials.


[shader_def]: http://developer.playcanvas.com/engine/api/stable/symbols/pc.gfx.Shader.html