
This document contains some examples of disassembled shaders.

For details of the shader ISA see `rnndb/isa.xml`.

Vertex shader examples
========================

Basic vertex shader
--------------------

    uniform mat4 modelviewMatrix;
    uniform mat4 modelviewprojectionMatrix;
    uniform mat3 normalMatrix;
    
    attribute vec4 in_position;
    attribute vec3 in_normal;
    attribute vec4 in_color;
    
    vec4 lightSource = vec4(2.0, 2.0, 20.0, 0.0);
    
    varying vec4 vVaryingColor;
    
    void main()
    {
        gl_Position = modelviewprojectionMatrix * in_position;
        vec3 vEyeNormal = normalMatrix * in_normal;
        vec4 vPosition4 = modelviewMatrix * in_position;
        vec3 vPosition3 = vPosition4.xyz / vPosition4.w;
        vec3 vLightDir = normalize(lightSource.xyz - vPosition3);
        float diff = max(0.0, dot(vEyeNormal, vLightDir));
        vVaryingColor = vec4(diff * in_color.rgb, 1.0);
    }

Becomes:

    MOV t3.xy, void, void, u4.wwww
    MOV t3.z, void, void, u5.wwww
    MUL t4, u0, t0.xxxx, void
    MAD t4, u1, t0.yyyy, t4
    MAD t4, u2, t0.zzzz, t4
    MAD t4, u3, t0.wwww, t4
    MUL t5.xyz, u4.xyzz, t1.xxxx, void
    MAD t5.xyz, u5.xyzz, t1.yyyy, t5.xyzz
    MAD t1.xyz, u6.xyzz, t1.zzzz, t5.xyzz
    MUL t5, u7, t0.xxxx, void
    MAD t5, u8, t0.yyyy, t5
    MAD t5, u9, t0.zzzz, t5
    MAD t0, u10, t0.wwww, t5
    RCP t1.w, void, void, t0.wwww
    MAD t0.xyz, -t0.xyzz, t1.wwww, t3.xyzz
    DP3 t3.xyz, t0.xyzz, t0.xyzz, void
    RSQ t3.xyz, void, void, t3.xxxx
    MUL t0.xyz, t0.xyzz, t3.xyzz, void
    DP3 t0.x, t1.xyzz, t0.xyzz, void
    SELECT.LT t0.x, u6.wwww, t0.xxxx, u6.wwww
    MUL t0.xyz, t0.xxxx, t2.xyzz, void
    MOV t0.w, void, void, u11.xxxx
    ADD t4.z, t4.zzzz, void, t4.wwww
    MUL t4.z, t4.zzzz, u11.yyyy, void

Uniform mapping:

    u0 - u3          modelviewprojectionMatrix
    u4.xyz - u6.xyz  normalMatrix
    u7 - u10         modelviewMatrix
    u11.xy           1.0, 0.5 (constants used internally)
    u4.w - u6.w      2.0, 20.0, 0.0 (lightSource components, and a zero)

Vertex shader with texture coordinates
---------------------------------------

    uniform mat4 modelviewMatrix;
    uniform mat4 modelviewprojectionMatrix;
    uniform mat3 normalMatrix;
   
    attribute vec4 in_position;
    attribute vec3 in_normal;
    attribute vec2 in_coord;
    
    vec4 lightSource = vec4(2.0, 2.0, 20.0, 0.0);
    
    varying vec4 vVaryingColor;
    varying vec2 coord;
    
    void main()
    {
        gl_Position = modelviewprojectionMatrix * in_position;
        vec3 vEyeNormal = normalMatrix * in_normal;
        vec4 vPosition4 = modelviewMatrix * in_position;
        vec3 vPosition3 = vPosition4.xyz / vPosition4.w;
        vec3 vLightDir = normalize(lightSource.xyz - vPosition3);
        float diff = max(0.0, dot(vEyeNormal, vLightDir));
        vVaryingColor = vec4(diff * vec3(1.0, 1.0, 1.0), 1.0);
        coord = in_coord;
    }

Becomes:

    MOV t3.xy, void, void, u4.wwww
    MOV t3.z, void, void, u5.wwww
    MUL t4, u0, t0.xxxx, void
    MAD t4, u1, t0.yyyy, t4
    MAD t4, u2, t0.zzzz, t4
    MAD t4, u3, t0.wwww, t4
    MUL t5.xyz, u4.xyzz, t1.xxxx, void
    MAD t5.xyz, u5.xyzz, t1.yyyy, t5.xyzz
    MAD t1.xyz, u6.xyzz, t1.zzzz, t5.xyzz
    MUL t5, u7, t0.xxxx, void
    MAD t5, u8, t0.yyyy, t5
    MAD t5, u9, t0.zzzz, t5
    MAD t0, u10, t0.wwww, t5
    RCP t1.w, void, void, t0.wwww
    MAD t0.xyz, -t0.xyzz, t1.wwww, t3.xyzz
    DP3 t3.xyz, t0.xyzz, t0.xyzz, void
    RSQ t3.xyz, void, void, t3.xxxx
    MUL t0.xyz, t0.xyzz, t3.xyzz, void
    DP3 t0.x, t1.xyzz, t0.xyzz, void
    SELECT.LT t0.xyz, u6.wwww, t0.xxxx, u6.wwww
    MOV t0.w, void, void, u11.xxxx
    MOV t1.xy, void, void, t2.xyyy
    ADD t4.z, t4.zzzz, void, t4.wwww
    MUL t4.z, t4.zzzz, u11.yyyy, void

Pixel shaders
==============

Empty (passthrough)
--------------------

    precision mediump float;
    
    varying vec4 vVaryingColor;
    
    void main()
    {
        gl_FragColor = vVaryingColor;
    }

Becomes:

    NOP void, void, void, void

Texture sampling
------------------

    precision mediump float;
    
    varying vec4 vVaryingColor;
    varying vec2 coord;
    
    uniform sampler2D in_texture;
    
    void main()
    {
        gl_FragColor = 3.0 * vVaryingColor * texture2D(in_texture, coord);
    }

Becomes:

    MUL t1, u0.xxxx, t1, void
    TEXLD t2, tex0, t2.xyyy, void, void
    MUL t1, t1, t2, void

Misc notes
=======================
- The shader engine packs together the registers of concurrently-executing shaders into the register bank.
  If you write beyond the number of registers in `VIV_STATE_(PS/VS)_TEMP_REGISTER_CONTROL`, you will overwrite
  registers of the next/previous shader instance, resulting in corrupt output.

- Two instructions are always appended to the vertex shader:

    ADD t0.__z_, t0.zzzz, void, t0.wwww
    MUL t0.__z_, t0.zzzz, u0.xxxx, void ; u0.x=0.5

  This adjusts the output position z, based on w. Likely this works around a difference in interpretation between
  the hardware and the OpenGL standard.

  For the gc2000 in the i.mx6 these two instructions are no longer appended (the only difference in the vertex shader for
  smoothed cube between gc600 and gc2000, as generated by the blob driver, is this). I do not know where the cutoff point is.

- t0 at the beginning of the fragment shader has the x/y/z coordinates
  x and y are in screen space, z is depth in 0.0 .. 1.0

  gl_fragCoord: contains the window-relative coordinates of the current fragment

- In PS, RGROUP 1 register i0.x contains the value of gl_FrontFacing. 
  i0.y also contains a non-zero value. i0.zw are zero.

- i1..i127 are simply aliases of i0, at least on my GC600.

- The exact same texture sampling instruction, TEXLD, is used to sample 2D as well as Cubemap textures
  (and, likely, other types of textures as well on hardware that supports these)

- All GC chips (at least the ones I've encountered, which are up to gc2000) have the same basic shader ISA.

