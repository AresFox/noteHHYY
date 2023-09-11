# Unity Build-in管线shader移植到URP管线下

重要：

**[Unity Build-in管线shader移植到URP管线下_黄琅的博客-CSDN博客](https://blog.csdn.net/zakerhero/article/details/118933660)**

[内置管线Shader升级到URP详细手册 - 简书 (jianshu.com)](https://www.jianshu.com/p/3fef69e2efb6)

https://blog.csdn.net/gedongxi/article/details/123915734





三、一些CGInclude常用的接口(重要)
非常重要的一步来了，如果按照上面的做完后，还发现有CGInclude的接口不对的地方。

可能就要把之前Builid-in的shader完全改写成URP管线下的，那就必须把全部CGInclude的接口转写成URP下的。

具体如下：

1、把CGPROGRAM改成HLSLPROGRAM，
把ENDCG改成ENDHLSL。

CGINCLUDE改成HLSLINCLUDE。

2、把#include "UnityCG.cginc"
改成

#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"
3、o.pos = UnityObjectToClipPos( v.vertex );
改成

VertexPositionInputs vertexInput = GetVertexPositionInputs(v.vertex.xyz);
o.pos = vertexInput.positionCS;
————————————————
版权声明：本文为CSDN博主「黄琅」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/zakerhero/article/details/118933660





##### 实例

这次使用的是“Unity Shader基于视差映射的云海效果”，从build-in转为urp

###### 转前 - Unity Build-in管线

```GLSL
Shader "Custom/Cloud Parallax"
{
	Properties{
		_Color("Color",Color) = (1,1,1,1)
		_MainTex("MainTex",2D) = "white"{}
		_Alpha("Alpha", Range(0,1)) = 0.5
		_Height("Displacement Amount",range(0,1)) = 0.15  //控制云的凹凸程度
		_HeightAmount("Turbulence Amount",range(0,2)) = 1
		_HeightTileSpeed("Turbulence Tile&Speed",Vector) = (1.0,1.0,0.05,0.0)
		_LightIntensity("Ambient Intensity", Range(0,3)) = 1.0
		[Toggle] _UseFixedLight("Use Fixed Light", Int) = 1
		_FixedLightDir("Fixed Light Direction", Vector) = (0.981, 0.122, -0.148, 0.0)

	}

		CGINCLUDE
			ENDCG

			SubShader
		{
			LOD 300
			Tags
			{
				"IgnoreProjector" = "True"
				"Queue" = "Transparent-50"
				"RenderType" = "Transparent"
			}

			Pass
			{
				Name "FORWARD"
				Tags
				{
					"LightMode" = "ForwardBase"
				}
				Blend SrcAlpha OneMinusSrcAlpha
			//ZWrite Off
			Cull Off

			CGPROGRAM
			#pragma vertex vert
			#pragma fragment frag

			#define UNITY_PASS_FORWARDBASE
			#include "UnityCG.cginc"
			#include "AutoLight.cginc"
			#include "Lighting.cginc"


			#pragma multi_compile_fwdbase
			#pragma target 3.0

			sampler2D _MainTex;
			float4 _MainTex_ST;
			half _Height;
			float4 _HeightTileSpeed;
			half _HeightAmount;
			half4 _Color;
			half _Alpha;
			half _LightIntensity;

			half4 _LightingColor;
			half4 _FixedLightDir;
			half _UseFixedLight;

			//uv是高度和颜色贴图的uv，uv2是做扰动用的uv
			struct v2f
			{
				float4 pos : SV_POSITION;
				float2 uv : TEXCOORD0;
				float3 normalDir : TEXCOORD1;
				float3 viewDir : TEXCOORD2;
				float4 posWorld : TEXCOORD3;
				float2 uv2 : TEXCOORD4;
				float4 color : TEXCOORD5;
				UNITY_FOG_COORDS(7)
			};

			v2f vert(appdata_full v)
			{
				v2f o;
				o.pos = UnityObjectToClipPos(v.vertex);
				o.uv = TRANSFORM_TEX(v.texcoord,_MainTex) + frac(_Time.y * _HeightTileSpeed.zw);
				o.uv2 = v.texcoord * _HeightTileSpeed.xy;  //uv2是做扰动用的uv,
				o.posWorld = mul(unity_ObjectToWorld, v.vertex);
				o.normalDir = UnityObjectToWorldNormal(v.normal);
				TANGENT_SPACE_ROTATION; //求TBN矩阵->模型空间转到切线空间
				o.viewDir = mul(rotation, ObjSpaceViewDir(v.vertex));
				o.color = v.color;
				UNITY_TRANSFER_FOG(o,o.pos);
				return o;
			}

			float4 frag(v2f i) : COLOR
			{
				//躁波图的rgb是颜色，a是高度
				float3 viewRay = normalize(i.viewDir * -1);
				viewRay.z = abs(viewRay.z) + 0.2; //trick:为了防止摄像机向量V和法向量N夹角过大。本身精度足够是不需要这个值的，但是为了减少迭代次数必须加上这个值。过大时viewDir.z接近于0,因此加上0.2
				viewRay.xy *= _Height;

				//shadeP->T0的uv值
				float3 shadeP = float3(i.uv,0); //shadeP记录的是当前uv的位置,也就是随着时间变换的uv
				float3 shadeP2 = float3(i.uv2,0); //用于记录迭代的状态,shadeP的xy是采样uv，z是当前深度; shadeP2的xy是扰动uv,z没有用到。
				float4 T = tex2D(_MainTex, shadeP2.xy);
				float h2 = T.a * _HeightAmount; //h2就是扰动值,即记录在纹理当中的深度

				//视差遮蔽映射(Parallax Occlusion Mapping, POM)
				float linearStep = 16; //分成16层

				float3 lioffset = viewRay / (viewRay.z * linearStep);
				float d = 1.0 - tex2Dlod(_MainTex, float4(shadeP2.xy,0,0)).a * h2; //应该是做了一步反转操作,d算是currentDepthMapValue,_MainTex大部分的地方都是白色的,而LearnOpenGL官网上这张帖图给的是偏黑色的
				float3 prev_d = d;
				float3 prev_shadeP = shadeP;
				//在进入while循环之前,shadeP = float3(i.uv,0); //shadeP记录的是当前uv的位置,也就是随着时间变换的uv
				//d是待参考的深度扰动值
				while (d > shadeP.z) //shadeP可以理解为当前迭代要走的uv值
				{
					prev_shadeP = shadeP;
					shadeP += lioffset; //对shadeP进行偏移,此时shadeP的z会固定增长某个值
					prev_d = d;
					d = 1.0 - tex2Dlod(_MainTex, float4(shadeP.xy,0,0)).a * h2;
				}
				float d1 = d - shadeP.z;  //currentDepthMapValue - currentLayerDepth;
				float d2 = prev_d - prev_shadeP.z;
				float w = d1 / (d1 - d2);
				shadeP = lerp(shadeP, prev_shadeP, w);

				half4 c = tex2D(_MainTex,shadeP.xy) * T * _Color;
				half Alpha = lerp(c.a, 1.0, _Alpha) * i.color.r;

				float3 normal = normalize(i.normalDir);
				half3 lightDir1 = normalize(_FixedLightDir.xyz);
				half3 lightDir2 = UnityWorldSpaceLightDir(i.posWorld);
				half3 lightDir = lerp(lightDir2, lightDir1, _UseFixedLight);
				float NdotL = max(0,dot(normal,lightDir));
				half3 lightColor = _LightColor0.rgb;
				fixed3 finalColor = c.rgb * (NdotL * lightColor + 1.0);
				return half4(finalColor.rgb, Alpha);
			}
		ENDCG
		}
		}
			}

				FallBack "Diffuse"
}

```



###### 转后 - URP管线

```GLSL
Shader "Unlit/CloudParallaxNewShader"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
        _Color("Color",Color) = (1,1,1,1)
        _Alpha("Alpha", Range(0,1)) = 0.5
        _Height("Displacement Amount(_Height)",range(0,1)) = 0.15  //控制云的凹凸程度
        _HeightAmount("Turbulence Amount(_HeightAmount)",range(0,2)) = 1
        _HeightTileSpeed("Turbulence Tile&Speed(_HeightTileSpeed)",Vector) = (1.0,1.0,0.05,0.0)
        _LightIntensity("Ambient Intensity", Range(0,3)) = 1.0
        [Toggle] _UseFixedLight("Use Fixed Light", Int) = 1
        _FixedLightDir("Fixed Light Direction", Vector) = (0.981, 0.122, -0.148, 0.0)
    }
    SubShader
    {
        Tags { "IgnoreProjector" = "True"
                "Queue" = "Transparent-50"
                "RenderType" = "Transparent" }
        LOD 100

        Pass
        {

            //Name "FORWARD"
            Tags
            {
                //"LightMode" = "ForwardBase"
                "LightMode"="UniversalForward"
            }
            /*Blend SrcAlpha OneMinusSrcAlpha*/
            //ZWrite Off
            Cull Off

            HLSLPROGRAM
            #pragma vertex vert
            #pragma fragment frag


            //#include "UnityCG.cginc"
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Lighting.hlsl"

            sampler2D _MainTex;
            float4 _MainTex_ST;

            half _Height;
            float4 _HeightTileSpeed;
            half _HeightAmount;
            half4 _Color;
            half _Alpha;
            half _LightIntensity;

            half4 _LightingColor;
            half4 _FixedLightDir;
            half _UseFixedLight;

            struct appdata
            {
                float4 vertex : POSITION;
                float2 uv : TEXCOORD0;//第一层UV

                float4 tangent : TANGENT;//正切
                float3 normal : NORMAL;//法线
  
                float4 texcoord : TEXCOORD6; //第二层UV
                float4 color : COLOR; //颜色
            };

            struct v2f
            {
                float2 uv : TEXCOORD0;

                //float4 vertex : SV_POSITION;

                //new
                float4 pos : SV_POSITION;
                float3 normalDir : TEXCOORD1;
                float3 viewDir : TEXCOORD2;
                float4 posWorld : TEXCOORD3;
                float2 uv2 : TEXCOORD4;
                float4 color : TEXCOORD5;

                float4 shadowCoord: TEXCOORD7;
            };

            
            v2f vert (appdata v)
            {
                v2f o;
                //o.pos = UnityObjectToClipPos(v.vertex);
                VertexPositionInputs vertexInput = GetVertexPositionInputs(v.vertex.xyz);
                o.pos = vertexInput.positionCS;

                o.uv = TRANSFORM_TEX(v.uv, _MainTex) + frac(_Time.y * _HeightTileSpeed.zw);
                o.uv2 = v.texcoord * _HeightTileSpeed.xy;  //uv2是做扰动用的uv,
                o.posWorld = mul(unity_ObjectToWorld, v.vertex);

                //o.normalDir = UnityObjectToWorldNormal(v.normal);
                VertexNormalInputs vertexNormalInput = GetVertexNormalInputs(v.normal);
                o.normalDir = vertexNormalInput.normalWS;

                //模型空间转到切线空间
                // *Built-in
                //TANGENT_SPACE_ROTATION; //求TBN矩阵->模型空间转到切线空间
                //o.viewDir = mul(rotation, ObjSpaceViewDir(v.vertex));
                //* Built-in end


                float3 binormal = cross(v.normal, v.tangent.xyz) * v.tangent.w;

                // tangent、binormal、normal为模型坐标系下的表示
                // 转置摆放后（按行摆放，按列摆放的话为 切线空间->模型空间）即为 模型空间->切线空间 的矩阵
                float3x3 rotation = float3x3(v.tangent.xyz, binormal, v.normal);
                
                // URP 中没有 ObjSpaceViewDir
                half3 objectSpaceViewDir = half3(TransformWorldToObject(_WorldSpaceCameraPos.xyz) - v.vertex.xyz);
                o.viewDir = mul(rotation, objectSpaceViewDir).xyz;

                o.color = v.color;

                //lightabout
                o.shadowCoord = GetShadowCoord(vertexInput);
                return o;
            }

            float4 frag (v2f i) : SV_Target
            {
                //躁波图的rgb是颜色，a是高度
                float3 viewRay = normalize(i.viewDir * -1);
                viewRay.z = abs(viewRay.z) + 0.2; //trick:为了防止摄像机向量V和法向量N夹角过大。本身精度足够是不需要这个值的，但是为了减少迭代次数必须加上这个值。过大时viewDir.z接近于0,因此加上0.2
                viewRay.xy *= _Height;

                //shadeP->T0的uv值
                float3 shadeP = float3(i.uv, 0); //shadeP记录的是当前uv的位置,也就是随着时间变换的uv
                float3 shadeP2 = float3(i.uv2, 0); //用于记录迭代的状态,shadeP的xy是采样uv，z是当前深度; shadeP2的xy是扰动uv,z没有用到。
                float4 T = tex2D(_MainTex, shadeP2.xy);
                float h2 = T.a * _HeightAmount; //h2就是扰动值,即记录在纹理当中的深度

                //视差遮蔽映射(Parallax Occlusion Mapping, POM)
                float linearStep = 16; //分成16层

                float3 lioffset = viewRay / (viewRay.z * linearStep);
                float d = 1.0 - tex2Dlod(_MainTex, float4(shadeP2.xy, 0, 0)).a * h2; //应该是做了一步反转操作,d算是currentDepthMapValue,_MainTex大部分的地方都是白色的,而LearnOpenGL官网上这张帖图给的是偏黑色的
                float3 prev_d = d;
                float3 prev_shadeP = shadeP;

                //在进入while循环之前,shadeP = float3(i.uv,0); //shadeP记录的是当前uv的位置,也就是随着时间变换的uv
                //d是待参考的深度扰动值
                while (d > shadeP.z) //shadeP可以理解为当前迭代要走的uv值
                {
                    prev_shadeP = shadeP;
                    shadeP += lioffset; //对shadeP进行偏移,此时shadeP的z会固定增长某个值
                    prev_d = d;
                    d = 1.0 - tex2Dlod(_MainTex, float4(shadeP.xy, 0, 0)).a * h2;
                }
                float d1 = d - shadeP.z;  //currentDepthMapValue - currentLayerDepth;
                float d2 = prev_d - prev_shadeP.z;
                float w = d1 / (d1 - d2);
                shadeP = lerp(shadeP, prev_shadeP, w);

                half4 c = tex2D(_MainTex, shadeP.xy) * T * _Color;
                half Alpha = lerp(c.a, 1.0, _Alpha) * i.color.r;

                float3 normal = normalize(i.normalDir);
                half3 lightDir1 = normalize(_FixedLightDir.xyz);

                Light mainLight = mainLight = GetMainLight(i.shadowCoord);
                float3 lightDirection = mainLight.direction;
                half3 lightColor = mainLight.color.rgb;

                float NdotL = max(0, dot(normal, lightDirection));
                float3 finalColor = c.rgb * (NdotL * lightColor + 1.0);
                return half4(finalColor.rgb, Alpha); 

                /*half3 lightDir2 = UnityWorldSpaceLightDir(i.posWorld);
                half3 lightDir = lerp(lightDir2, lightDir1, _UseFixedLight);
                float NdotL = max(0, dot(normal, lightDir));
                half3 lightColor = _LightColor0.rgb;**
                fixed3 finalColor = c.rgb * (NdotL * lightColor + 1.0);
                return half4(finalColor.rgb, Alpha);*/
                
                
                // sample the texture
                float4 col = tex2D(_MainTex, i.uv);


                return col;
            }
            ENDHLSL
        }
    }
}

```

