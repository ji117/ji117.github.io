## 2D Space Shooter Game

Here is a game that I made using a simple game engine that was provided to me which could only draw and play sounds.
I had to add additional functionality to the game engine in order to produce the game shown below. The game was programmed using C++.

https://user-images.githubusercontent.com/61008475/174500305-38eb4651-d913-44fd-b145-1b92bba48cc6.mp4  

This video shows gameplay of the game.  



https://user-images.githubusercontent.com/61008475/174502081-f314c709-ea53-4c6d-983b-f3f54583f860.mp4  

This video shows the moving star effect in the background the simulates the ships moving fowards. 

![screenshot_20220619223232811](https://user-images.githubusercontent.com/61008475/174501795-80a7aa21-5269-47a5-bc48-07a51d60b43c.jpg)
 
This screenshot shows the game's game over screen. 


The goal of the game is to kill all the enemies present in a level. When all enemies in a level are killed after a short delay the next level is started. 

Listed below are the additions I made to the game engine for the game.  
Added a gameobject super class  
Added classes for each individual game object  
Added an animation system  
Added a system to play sounds  
Added an object manager to handle and keep track of all the game objects  
Added a way for game objects to detect collisions  
Added a level manager to control the levels in the game  

Below are code snippets from the game.  

``` 
GameObject::~GameObject()
{
}

void GameObject::LoadImage(const wchar_t* filename)
{
	image = MyDrawEngine::GetInstance()->LoadPicture(filename);
}

void GameObject::Render()
{
	MyDrawEngine::GetInstance()->DrawAt(position, image, size, angle, transparency);
}

bool GameObject::GetActive()
{
	return active; 
}

void GameObject::Deactivate()
{
	active = false; 
} 
```


