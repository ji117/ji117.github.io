## Graphics Project 

Here is a simple graphics scene that I made using a simple codebase that was provided to me for an assignment I did at university. 
This demonstrates me applying what I learnt during the four weeks of the module. This was coded in C++ and rendered using OpenGL. 

{% include youtube.html id="XjpsnMKtozs" %}

Below are code snippets from the project

Loading textures including bump maps and cube map
```
  rock1Texture = SOIL_load_OGL_texture(TEXTUREDIR"Rock_04_DIFF.png", SOIL_LOAD_AUTO, SOIL_CREATE_NEW_ID, SOIL_FLAG_MIPMAPS);
  rock2Texture = SOIL_load_OGL_texture(TEXTUREDIR"Rock_01_DIFF.png", SOIL_LOAD_AUTO, SOIL_CREATE_NEW_ID, SOIL_FLAG_MIPMAPS);
  grassTexture = SOIL_load_OGL_texture(TEXTUREDIR"grass.png", SOIL_LOAD_AUTO, SOIL_CREATE_NEW_ID, SOIL_FLAG_MIPMAPS);
  bumpMap = SOIL_load_OGL_texture(TEXTUREDIR"Rock_04_NRM.png", SOIL_LOAD_AUTO, SOIL_CREATE_NEW_ID, SOIL_FLAG_MIPMAPS);
  cubeMap = SOIL_load_OGL_cubemap(TEXTUREDIR"xpos.png", TEXTUREDIR"xneg.png", TEXTUREDIR"ypos.png", TEXTUREDIR"yneg.png", TEXTUREDIR"zpos.png", TEXTUREDIR"zneg.png", SOIL_LOAD_RGB,     
  SOIL_CREATE_NEW_ID, 0);
```
Fragment shader used for height map

```
uniform sampler2D rock1Tex;
uniform sampler2D rock2Tex; 
uniform sampler2D bumpTex;
uniform sampler2D grassTex;
uniform sampler2D dirtTex;
uniform vec3 cameraPos;
uniform vec4 lightColour;
uniform vec3 lightPos;
uniform float lightRadius;
uniform float worldHeight;

in Vertex {
  vec3 colour;
  vec2 texCoord;
  vec3 normal;
  vec3 tangent;
  vec3 binormal;
  vec3 worldPos; 
 } IN;

 out vec4 fragColour;

 void main(void) {

  float height = IN.worldPos.y / worldHeight;
  float lowRange = 0.3f;
  float lowRange2 = 0.35f;
  float highRange = 0.5f;
  float highRange2 = 0.49f;
  float heightDifference = height; 
  float grassDirtSpec = 0.3;

  vec3 incident = normalize(lightPos - IN.worldPos);
  vec3 viewDir = normalize(cameraPos - IN.worldPos);
  vec3 halfDir = normalize(incident + viewDir);

  mat3 TBN = mat3(normalize(IN.tangent), normalize(IN.binormal), normalize(IN.normal));

  vec4 rock = texture(rock1Tex, IN.texCoord) + texture(rock2Tex, IN.texCoord);
  vec3 bumpNormal = texture(bumpTex, IN.texCoord).rgb;
  bumpNormal = normalize(TBN * normalize(bumpNormal * 2.0 - 1.0));
  vec4 grass = texture(grassTex, IN.texCoord);
  vec4 dirt = texture(dirtTex, IN.texCoord);

  
  float lambert = max(dot(incident, bumpNormal), 0.0f);
  float distance = length(lightPos - IN.worldPos);
  float attenuation = 1.0 - clamp(distance / lightRadius, 0.0, 1.0);
  float specFactor = clamp(dot(halfDir, bumpNormal), 0.0, 1.0);
  specFactor = pow(specFactor, 60.0);

  if(height > lowRange && height < highRange)
  {
     float lambert = max(dot(incident, IN.normal), 0.0f);
     float specFactor = clamp(dot(halfDir, IN.normal), 0.0, 1.0);
     specFactor = pow(specFactor, 60.0);
     vec3 surface = (grass.rgb * lightColour.rgb);
     fragColour.rgb = surface * lambert * attenuation;
     fragColour.rgb += (lightColour.rgb * (specFactor * grassDirtSpec)) * attenuation * 0.33;
     fragColour.rgb += surface * 0.2f;
     fragColour.a = grass.a;
  }
  if(height < lowRange)
  {
     float lambert = max(dot(incident, IN.normal), 0.0f);
     float specFactor = clamp(dot(halfDir, IN.normal), 0.0, 1.0);
     specFactor = pow(specFactor, 60.0);
     vec3 surface = (dirt.rgb * lightColour.rgb);
     fragColour.rgb = surface * lambert * attenuation;
     fragColour.rgb += (lightColour.rgb * (specFactor * grassDirtSpec)) * attenuation * 0.33;
     fragColour.rgb += surface * 0.2f;
     fragColour.a = dirt.a;
  }
  if (height > highRange)
  {
    vec3 surface = (rock.rgb * lightColour.rgb);
    fragColour.rgb = surface * lambert * attenuation;
    fragColour.rgb += (lightColour.rgb * specFactor) * attenuation * 0.33;
    fragColour.rgb += surface * 0.2f;
    fragColour.a = rock.a;
    
  }
  if (height > lowRange && height < lowRange2)
  {
     heightDifference -= lowRange;
     heightDifference /= (lowRange2 - lowRange);
     float lambert = max(dot(incident, IN.normal), 0.0f);
     float specFactor = clamp(dot(halfDir, IN.normal), 0.0, 1.0);
     specFactor = pow(specFactor, 60.0);
     vec3 surface = (grass.rgb * lightColour.rgb);
     fragColour.rgb = surface * lambert * attenuation;
     fragColour.rgb += (lightColour.rgb * (specFactor * grassDirtSpec)) * attenuation * 0.33;
     fragColour.rgb += surface * 0.2f;
     fragColour.a = grass.a;
     grass = fragColour;
     
     surface = (dirt.rgb * lightColour.rgb);
     fragColour.rgb = surface * lambert * attenuation;
     fragColour.rgb += (lightColour.rgb * (specFactor * grassDirtSpec)) * attenuation * 0.33;
     fragColour.rgb += surface * 0.2f;
     fragColour.a = dirt.a;
     dirt = fragColour;
     fragColour = mix(dirt, grass, heightDifference);
  }
  if (height > highRange2 && height < highRange)
  {
     heightDifference -= highRange2;
     heightDifference /= (highRange - highRange2);
     vec3 surface = (rock.rgb * lightColour.rgb);
     fragColour.rgb = surface * lambert * attenuation;
     fragColour.rgb += (lightColour.rgb * specFactor) * attenuation * 0.33;
     fragColour.rgb += surface * 0.2f;
     fragColour.a = rock.a;
     rock = fragColour;
     float lambert = max(dot(incident, IN.normal), 0.0f);
     float specFactor = clamp(dot(halfDir, IN.normal), 0.0, 1.0);
     specFactor = pow(specFactor, 60.0);
     surface = (grass.rgb * lightColour.rgb);
     fragColour.rgb = surface * lambert * attenuation;
     fragColour.rgb += (lightColour.rgb * specFactor) * attenuation * 0.33;
     fragColour.rgb += surface * 0.2f;
     fragColour.a = grass.a;
     grass = fragColour;
     fragColour = mix(grass, rock, heightDifference);
  }
 }
```
Fragment shader used for water
```
uniform sampler2D reflectTex;
uniform sampler2D bumpMap;
uniform samplerCube cubeTex;
uniform vec3 cameraPos;
uniform vec4 lightColour;
uniform vec3 lightPos;
uniform float lightRadius;


in Vertex {
  vec4 colour;
  vec2 texCoord;
  vec3 normal;
  vec3 worldPos;
} IN;

out vec4 fragColour;

void main(void) {
  vec3 incident = normalize(lightPos - IN.worldPos); //
  vec4 diffuse = texture(reflectTex, IN.texCoord);
  vec3 viewDir = normalize(cameraPos - IN.worldPos);
  vec3 halfDir = normalize(incident + viewDir); //
  
  float lambert = max(dot(incident, IN.normal), 0.0f);
  float distance = length(lightPos - IN.worldPos);
  float attenuation = 1.0 - clamp(distance / lightRadius, 0.0, 1.0);
  float specFactor = clamp(dot(halfDir, IN.normal), 0.0, 1.0);
  specFactor = pow(specFactor, 60.0);

  vec3 reflectDir = reflect(-viewDir, normalize(IN.normal));
  vec4 reflectTex = texture(cubeTex, reflectDir);


  vec3 surface = (reflectTex.rgb * lightColour.rgb);
  fragColour.rgb = surface * lambert * attenuation;
  fragColour.rgb += (lightColour.rgb * (specFactor * 2) * attenuation * 0.33);
  fragColour.rgb += surface * 0.2f;
  fragColour.a = reflectTex.a;

  fragColour = fragColour + (diffuse * 0.25f);
}
```
Setting height map size, loading in meshes (Sphere mesh was used to make clouds), and adding a root scene node
```
  heightMapSize = heightMap->GetHeightmapSize() + Vector3(0.0f, -100.0f, 0.0f);

   sphere = Mesh::LoadFromMeshFile("Sphere.msh");
   tree = Mesh::LoadFromMeshFile("Oak_Tree.msh");
	
   root = new SceneNode();
   root->SetTransform(Matrix4::Translation(Vector3(heightMapSize.x/2, 0, heightMapSize.z/2)));
```
Adding a tree to the scene
```
  treeNode = new SceneNode(tree, Vector4(1, 1, 1, 1));
  treeNode->SetTexture(treeTexture);
  treeNode->SetTransform(Matrix4::Translation(Vector3(-150, 50, -200)));
  treeNode->SetModelScale(Vector3(20, 20, 20));
  root->AddChild(treeNode);
```
Code to update the scene
```
  camera->UpdateCamera(dt);
  viewMatrix = camera->BuildViewMatrix();
  frameFrustum.FromMatrix(projMatrix * viewMatrix);

  waterRotate += dt * 2.0f;
  waterCycle += dt * 0.25f;

  frameTime -= dt;
  while (frameTime < 0.0f)
  {
      currentFrame = (currentFrame + 1) % anim->GetFrameCount();
      frameTime += 1.0f / anim->GetFrameRate();
  }
  root->Update(dt);
```

