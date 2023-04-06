# shader 反弹效果

## **思路**

https://zhuanlan.zhihu.com/p/42539305

- 根据对应的坐标和范围内的顶点，根据法线进行偏移。

- 根据时间计算偏移持续时间和反弹次数。

  

这段代码定义了一个名为 `qqScripts` 的类，它继承自 MonoBehaviour 类，并实现了一个接口 `IElastic`。在该类中，先通过静态构造方法初始化了三个属性值 s_pos、s_nor 和 s_time，然后在 Start 方法中获取了该游戏对象上的 MeshRenderer 组件，并将其赋值给 mesh 变量。定义了一个 OnElastic 方法，该方法会在被调用时传入一个 RaycastHit 对象，根据传入的碰撞信息，将反弹坐标、受影响顶点范围的半径、反弹力度和重置时间等信息设置到该游戏对象所附加的材质上。

```C#
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

interface IElastic
{
    void OnElastic(RaycastHit hit);
}
//RequireComponent添加该脚本时，会自动将所依赖的各个组件(比如这里的MeshRenderer)添加至gameobject上，避免人为操作的失误。
[RequireComponent(typeof(MeshRenderer))]
public class qqScripts  :MonoBehaviour, IElastic
{
    private static int s_pos, s_nor, s_time;
    static qqScripts()
    {
        s_pos = Shader.PropertyToID("_Position");
        s_nor = Shader.PropertyToID("_Normal");
        s_time = Shader.PropertyToID("_PointTime");
    }
    private MeshRenderer mesh;
    private void Start()
    {
        mesh = GetComponent<MeshRenderer>();  
        if(mesh)Debug.Log("yyyyy");
    }
//调用该方法
    public void OnElastic(RaycastHit hit)//只能是public
    {
        Debug.Log("oooo");
        //反弹的坐标
        Vector4 v = transform.InverseTransformPoint(hit.point);
        //受影响顶点范围的半径
        v.w = 0.6f;
        mesh.material.SetVector(s_pos, v);
        //法线方向，该值为顶点偏移方向，可自己根据需求传。
        v = transform.InverseTransformDirection(hit.normal.normalized);
        //反弹力度
        v.w = 0.2f;
        mesh.material.SetVector(s_nor, v);
        //重置时间
        mesh.material.SetFloat(s_time, Time.time);
    }
    
}
```



那么以上代码中，transform.**InverseTransformPoint**(hit.point);的意思是：

`transform.InverseTransformPoint(hit.point)`是将`hit.point`从世界坐标系转换为物体本地坐标系中的坐标。在这里，`hit.point`是与场景中一个碰撞器相交时的碰撞点在世界坐标系中的位置，而`transform`是当前脚本所在物体的变换组件，它保存了物体在世界坐标系中的位置、旋转和缩放信息。通过使用`transform.InverseTransformPoint()`方法，可以将世界坐标系中的点转换为物体本地坐标系中的坐标，以便在着色器中使用。

==位置从世界空间->模型空间==



**InverseTransformDirection**

`transform.InverseTransformDirection()`是将一个向量从世界坐标系转换为物体本地坐标系中的向量。在这个代码中，它被用于将`hit.normal`从世界坐标系转换为物体本地坐标系中的法线方向。`hit.normal`是指与场景中一个碰撞器相交时的碰撞点处表面的法线方向在世界坐标系中的方向。通过使用`transform.InverseTransformDirection()`方法，可以将该法线方向从世界坐标系转换为物体本地坐标系中的向量，以便在着色器中使用。

==法线方向从世界空间->模型空间==



**_Position("xyz:position,w:range",vector) = (0,0,0,1)** 

`_Position`是一个四维向量类型的着色器属性，用于保存反弹点的位置和受影响顶点范围的半径信息。它被定义为`(0, 0, 0, 1)`，表示初始状态下，反弹点的位置位于物体本地坐标系的原点，受影响顶点范围的半径为1个单位长度。

该向量的==前三个分量(`xyz`)表示反弹点在物体本地坐标系中的坐标，第四个分量(`w`)表示受影响顶点范围的半径==。这里使用了一种简单的方式来将位置和范围信息==编码到一个四维向量==中，以便在传递给着色器时使用。这种编码方式可以有效地减少需要传递的信息量，并且可以让代码更易于理解和使用。



**vert**

当这个Unlit/qqshader被应用到一个物体的材质上时，它会对该物体进行渲染。以下是该着色器代码中`vert`函数中各行的具体含义：

```glsl
v2f vert(appdata v)
{
    v2f o;
    float t = _Time.y - _PointTime; // 获取当前时间与反弹点时间的差值
    if (t > 0 && t < _Duration) {   // 如果在反弹持续时间内
        float r = 1 - saturate(length(v.vertex.xyz - _Position.xyz) / _Position.w); // 计算顶点距离反弹点的距离和受影响范围半径的比例
        float l = 1 - t / _Duration; // 计算时间和反弹持续时间的比例
        v.vertex.xyz += r * l * _Normal.xyz * _Normal.w * cos(t * _Frequency); // 计算新的顶点位置偏移量并添加到原始顶点位置上
    }
    o.vertex = UnityObjectToClipPos(v.vertex); // 将顶点坐标从物体空间转换为屏幕空间
    o.uv = TRANSFORM_TEX(v.uv, _MainTex); // 转换纹理坐标为映射到材质上的坐标系 
    UNITY_TRANSFER_FOG(o, o.vertex); // 将雾效传递给片元着色器
    return o; // 返回修改后的顶点信息
}
```

具体来说：

1. `v2f vert(appdata v)` 定义了一个顶点着色器的输入和输出结构，其中`appdata`是原始顶点数据，而`v2f`是修改后的顶点数据。
2. `v2f o;` 定义了一个修改后的顶点数据类型的变量`o`，用于保存新的顶点信息。
3. `float t = _Time.y - _PointTime;` 获取当前时间与反弹点时间的差值。
4. `if (t > 0 && t < _Duration) { ... }` 如果在反弹点持续时间内，执行以下代码块。
1. `float r = 1 - saturate(length(v.vertex.xyz - _Position.xyz) / _Position.w);` 计算顶点距离反弹点的距离和受影响范围半径的比例。
2. `float l = 1 - t / _Duration;` 计算时间和反弹持续时间的比例。
3. `v.vertex.xyz += r * l * _Normal.xyz * _Normal.w * cos(t * _Frequency);` 计算新的顶点位置偏移量并添加到原始顶点位置上。
4. `o.vertex = UnityObjectToClipPos(v.vertex);` 将顶点坐标从物体空间转换为屏幕空间。
5. `o.uv = TRANSFORM_TEX(v.uv, _MainTex);` 转换纹理坐标为映射到材质上的坐标系。
6. `UNITY_TRANSFER_FOG(o, o.vertex);` 将雾效传递给片元着色器。
7. `return o;` 返回修改后的顶点信息。



saturate(length(v.vertex.xyz - _Position.xyz) / _Position.w)

当前点和反弹中心点的距离/反弹范围半径

并不随着时间变化，而是距离越远，看起来收反弹影响越小

```C
具体来说，该函数计算了当前顶点到碰撞点的距离，并将其除以碰撞点所在平面到视点的距离（即_Position.w），然后使用saturate函数将结果限制在[0, 1]范围内。这个值可以理解为“?弹力度”，用于计算顶点在受到碰撞影响时的偏移量。根据代码逻辑，当时间t处于_PointTime和_PointTime+_Duration之间时（即发生碰撞后一段时间内），碰撞点周围半径为_Position.w的范围内的顶点会产生波动效果，波动的幅度会随时间推移而逐渐减小（由l变量控制），并且波动的方向和强度都由_Normal向量决定。
```



`v.vertex.xyz += r * l * _Normal.xyz * _Normal.w * cos(t * _Frequency);` 计算新的顶点位置偏移量并添加到原始顶点位置上。

作用于顶点着色器中的顶点位置变换。该顶点着色器使用了以下属性：

- _Position: 一个包含xyz坐标和w值范围的向量，表示特效的位置和影响范围。
- _Normal: 一个包含xyz坐标和w值强度的向量，用于计算顶点位移的方向和强度。
- _PointTime: 特效开始时间。
- _Duration: 特效持续时间。
- _Frequency: 用于计算cosine函数的频率。 //越大 震动频率越快

具体地说，这个顶点着色器首先计算出当前时间与特效开始时间之间的差值，并判断其是否在特效的持续时间范围内。如果在特效持续时间内，则根据距离特效位置的距离计算出一个饱和度系数r和时间系数l。然后，将当前顶点的位置沿着法线方向（_Normal.xyz）加上一定的位移，以产生特殊的效果。这个位移的大小由r、l、_Normal.w和cos(t * _Frequency)共同决定。



当顶点距离特效位置越近时，饱和度系数r越大，时间系数l越小；当顶点距离特效位置越远时，饱和度系数r越小，时间系数l越大。这样可以让顶点在特效的作用范围内产生更强的位移效果，而在范围外则逐渐减弱。

法线向量的强度_Normal.w用于控制位移的方向，使得顶点在沿着法线方向位移的同时，也会有一定的变化幅度。

cos(t * _Frequency)是一个周期性变化的因素，其中t表示当前时间，_Frequency表示变化的频率。它通过控制cosine函数的输入来控制位移的大小和变化速度，从而产生动态变化的效果。

综合以上因素，就可以通过改变饱和度系数r、时间系数l、法线向量的强度_Normal.w和cos(t * _Frequency)来调整顶点位移的大小、方向和变化速度，从而产生不同的特殊效果。



通过消融实验，看各个参数的意思

```glsl
float r = 1;
float l = 1 ;
//法线方向上的位移
v.vertex.xyz += r * l * _Normal.xyz * _Normal.w * cos(t * _Frequency);
```

那么这个物体只会平移

随着时间左右动 不会停下 按照cos来动

_Frequency越大 震动频率越快 快速左右动



```glsl
float l = 1 - t / _Duration;
```

时间越长，l越小，震动幅度越小





##### 应用 点击屏幕操作相应位置反弹

```C#
void Update () 
{

        Camera_Ray();
    }
    private void Camera_Ray()
    {
        if (Input.GetMouseButtonUp(0))
        {
            Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
            RaycastHit hit;
            bool isCollider = Physics.Raycast(ray, out hit);
            if (isCollider)
            {
                Debug.DrawLine(ray.origin, hit.point, Color.red);
                IElastic1 elastic = (IElastic1)script;
                elastic.OnElastic(hit);
            }
        }
    }
```

