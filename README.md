# Steam-X-Ray-in-Unity
 ```cpp
Shader "Custom/X-Ray" {
	Properties{
		_MainTex_Normal("Albedo Normal (RGB)", 2D) = "white" {}
		_MainTex_Sexy("Albedo Sexy (RGB)", 2D) = "white" {}
	    _MaskTex("Mask", 2D) = "white" {} //add
		_EdgeBlurLength("EdgeBlur Length", Float)= 0.008
		_MaskRadius("Mask Radius", Float)= 0.15
		_Pos("Position", Vector) = (0.5,0.5,0,0)
	}
		SubShader{
		Tags{ "RenderType" = "Opaque" "Queue" = "Transparent" } //modified
		Blend SrcAlpha OneMinusSrcAlpha //add

		CGPROGRAM
#pragma surface surf Standard keepalpha //modified
#pragma target 3.0

	sampler2D _MainTex_Normal;
	sampler2D _MainTex_Sexy;
	sampler2D _MaskTex; //add
	float _EdgeBlurLength;
	float _MaskRadius;
	float4 _Pos;

	struct Input {
		float2 uv_MainTex_Normal;
	};

	// 创建圆
	fixed3 createCircle(float2 pos, float radius, float2 uv) 
	{
		//当前像素到中心点的距离
		float dis = distance(pos, uv);
		//  smoothstep 平滑过渡
		float col = smoothstep(radius + _EdgeBlurLength, radius, dis);
		return fixed3(col, col, col);
	}

	void surf(Input IN, inout SurfaceOutputStandard o) {
		fixed4 n = tex2D(_MainTex_Normal, IN.uv_MainTex_Normal);
		fixed4 s = tex2D(_MainTex_Sexy, IN.uv_MainTex_Normal); //add
		fixed3 mask = createCircle(_Pos.xy, _MaskRadius, IN.uv_MainTex_Normal);
		
		fixed3 centre= s.rgb*mask.rgb;
		mask = 1 - mask;
		fixed3 border = n.rgb*mask.rgb;
		o.Albedo = centre.rgb + border.rgb;
		o.Alpha = s.a;
	}
	ENDCG
	}
		FallBack "Diffuse"
}
```

```cpp
//in c#

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class NewBehaviourScript : MonoBehaviour
{ 
    // shader
    public Shader myShader;
    public Material material = null;
    // 遮罩中心位置
    private Vector2 pos = new Vector4(-0.5f, -0.5f);
 

    // Update is called once per frame
    void Update()
    {
        Vector2 mousePos = Input.mousePosition;
        //将mousePos转化为（0，1）区间
        pos = new Vector2(mousePos.x / Screen.width, mousePos.y / Screen.height);

        material.SetVector("_Pos", new Vector4(pos.x, pos.y, 0, 0));
    }
}
```
