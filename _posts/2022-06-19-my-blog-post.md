## 2D Space Shooter Game

Here is a game that I made using a simple game engine that was provided to me which could only draw and play sounds.
I had to add additional functionality to the game engine in order to produce the game shown below. The game was programmed using C++.

http://user-images.githubusercontent.com/61008475/174500305-38eb4651-d913-44fd-b145-1b92bba48cc6.mp4  

This video shows gameplay of the game.  



http://user-images.githubusercontent.com/61008475/174502081-f314c709-ea53-4c6d-983b-f3f54583f860.mp4  

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

Gameobject super class

``` 
GameObject::GameObject()
{
	image = -1;
	angle = 0;
	size = 1.0f; 
	position.set(0, 0);
	transparency = 0;
	active = false; 
}

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
Update function for player's ship  

```
void SpaceShip::Update(double framerate)
{
	startTimer = startTimer - framerate; //this is the player's respawn timer
	if (startTimer <= 0)
	{
		if (transparency >= 0) //this code makes the player fade in when they respawn
		{
			transparency = transparency - 0.05f;
		}

		MyInputs* pInputs = MyInputs::GetInstance();

		if (pInputs->KeyPressed(DIK_D))
		{
			Vector2D acceleration(speed, 0);
			velocity = velocity + acceleration * framerate;
		}

		if (pInputs->KeyPressed(DIK_A))
		{
			Vector2D acceleration(-speed, 0);
			velocity = velocity + acceleration * framerate;
		}
		velocity = velocity - velocity * 1.8 * framerate;
		position = position + velocity * framerate; //needs to be outside the if statement in order for the object to carry on moving after the key has been let go

		shootDelay = shootDelay - framerate;

		if (pInputs->KeyPressed(DIK_SPACE) && shootDelay <= 0) //code that allows the player to shoot 
		{
			shootDelay = 0.15f;

			Bullet* pBullet = new Bullet();
			pBullet->Intialise(position, angle);
			Game::instance.GetObjectManager().AddObject(pBullet);
			Game::instance.GetSoundFX().PlayShot();
		}

		//code that prevents the player's ship from going to far to the left or right
		if (position.XValue > 1300)
		{
			position.XValue = 1300;
		}

		if (position.XValue < -1300)
		{
			position.XValue = -1300;
		}

		collisionShape.PlaceAt(position, 40); // this places the player's ship hitbox on the player

		if (health <= 0)
		{
			PlayerDead();
			health = 100;

			if (playerLives <= 0)
			{
				Deactivate(); //destroys player object when lives run out
			}
		}
	}
}
```
Update function for jet flame animation behind the player's ship  
```
void Jet::Update(double framerate)
{
	if (pShipJ->IsAlive())
	{
		if (transparency >= 0)
		{
			transparency = transparency - 0.05f;
		}

		if (jetFire >= 1 && jetFire < 2)
		{
			LoadImage(L"fire1.bmp");
		}

		if (jetFire >= 2 && jetFire < 3)
		{
			LoadImage(L"fire2.bmp");
		}

		if (jetFire >= 3 && jetFire < 4)
		{
			LoadImage(L"fire3.bmp");
		}

		if (jetFire >= 4 && jetFire < 5)
		{
			LoadImage(L"fire4.bmp");
		}

		if (jetFire >= 5 && jetFire < 6)
		{
			LoadImage(L"fire5.bmp");
		}

		if (jetFire >= 6 && jetFire < 7)
		{
			LoadImage(L"fire6.bmp");
		}


		if (jetFire >= 7)
		{
			jetFire = 1; // reset animation
		}


		jetFire = jetFire + animationSpeed * framerate;
		if (pShipJ)
		{
			position = pShipJ->GetPosition() + flamePosition;
		}
	}
```
Code used to check for collisions  
```
void ObjectManager::CheckCollisions()
{
	std::list<GameObject*>::iterator it1; 
	std::list<GameObject*>::iterator it2;

	for (it1 = pObjectList.begin(); it1 != pObjectList.end(); it1++)
	{
		for (it2 = next(it1); it2 != pObjectList.end(); it2++)
		{
			if ((*it1) && (*it2) && (*it1)->GetActive() && (*it2)->GetActive() && (*it1)->GetShape().Intersects((*it2)->GetShape()))
			{
				(*it1)->ProcessCollision(**it2);
				(*it2)->ProcessCollision(**it1); 
			}
		}
	}
}
```
Code of function to load sounds
```
void SoundFX::LoadSounds()
{
    //get the pointer to the sound engine
    MySoundEngine* pSoundEngine = MySoundEngine::GetInstance();

    //load the explosions sounds
    for (int i = 0; i < NUMEXPLOSIONSOUNDS; i++)
    {
        //if sounds are currently loaded, no need to load them again
        if (m_Explosions[i] == 0)
        {
            switch (i % 5)		//only 5 wave sounds
            {
            case 0:
                m_Explosions[i] = pSoundEngine->LoadWav(L"explosion1.wav");
                break;
            case 1:
                m_Explosions[i] = pSoundEngine->LoadWav(L"explosion2.wav");
                break;
            case 2:
                m_Explosions[i] = pSoundEngine->LoadWav(L"explosion3.wav");
                break;
            case 3:
                m_Explosions[i] = pSoundEngine->LoadWav(L"explosion4.wav");
                break;
            case 4:
                m_Explosions[i] = pSoundEngine->LoadWav(L"explosion5.wav");
                break;
            }
        }
    }
    //load the thrust sound, as long as not already loaded
    if (m_Thrust == 0)
    {
        m_Thrust = pSoundEngine->LoadWav(L"thrustloop2.wav");
    }
    for (int i = 0; i < numOfShots; i++)
    {
        m_Shot[i] = pSoundEngine->LoadWav(L"shot2.wav");
        m_EnemyShot[i] = pSoundEngine->LoadWav(L"shoot.wav");
    }
}
```
Code of function to play a explosion sound  
```
void SoundFX::PlayExplosion()
{
    //play the sound
    MySoundEngine::GetInstance()->SetFrequency(m_Explosions[m_NextExplosion], 16000);
    MySoundEngine::GetInstance()->SetVolume(m_Explosions[m_NextExplosion], 0);
    MySoundEngine::GetInstance()->Play(m_Explosions[m_NextExplosion]);
    m_NextExplosion += rand() % 3;			//stops it sounding too repetitive
    if (m_NextExplosion >= NUMEXPLOSIONSOUNDS)
        m_NextExplosion = 0;
}
```
